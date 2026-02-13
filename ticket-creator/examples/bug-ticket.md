# Example: Bug Ticket (Root Cause Known)

> Based on real ticket S-1222 (warm Lambda broker caching bug), enhanced with reproduction steps and fix approach.

---

## Problem

When AWS Lambda reuses a warm Python process across invocations targeting different environments, the cached Dramatiq SQS broker retains the previous environment's SQS namespace. This causes messages to be routed to the wrong queues (e.g., prod messages sent to staging queues).

## Error

```
No explicit error — messages silently routed to wrong environment's SQS queues.
Detected via: staging queue receiving prod-volume messages during cross-environment actor invocations.
```

## Affected File(s)

- `lambda/actor_invoke/actor_invoke.py` — Lambda handler that invokes Dramatiq actors

## Reproduction Steps

1. Deploy `actor_invoke` Lambda with warm-start enabled (default)
2. Invoke the Lambda targeting `staging` environment → broker initializes with `sylla-data-engine-staging` namespace
3. Immediately invoke the same Lambda targeting `production` environment
4. Observe that the Dramatiq broker still uses the `staging` namespace from step 2
5. Messages from the `production` invocation arrive in the `staging` SQS queue

**Environment:** production + staging (cross-environment issue)
**Frequency:** intermittent — depends on Lambda warm-start reuse

---

## Root Cause Analysis

**Known root cause:**

The Dramatiq `SQSBroker` is created at module scope in `orchestrator.src.broker` with a namespace tied to the environment. Once imported, the broker and all `@dramatiq.actor` stubs are cached in `sys.modules`. On warm Lambda starts, if a subsequent invocation targets a different environment, the cached broker routes messages to the wrong SQS namespace.

---

## Current Behavior

Warm Lambda reuses the cached broker from the previous invocation. If environments differ, messages go to the wrong SQS queue silently.

## Expected Behavior

Each invocation routes messages to the correct environment's SQS queue. When the environment changes between invocations, the broker re-initializes with the correct namespace.

---

## Technical Context

**Repo:** data-engine

**Existing pattern to follow:** Follow the existing `lambda_handler` flow in `actor_invoke.py` — add the cache-bust logic before the environment config is set.

**Systems involved:** AWS Lambda, Amazon SQS, Dramatiq actor framework

**Side effects to check:**
- Dramatiq itself is NOT purged — `dramatiq.set_broker()` replaces the global reference
- Re-importing actor modules re-runs `@dramatiq.actor` which binds stubs to the new broker
- Same-environment warm starts should NOT trigger the purge

**Git history:**
- Environment config flow last changed in PR #149 (Lambda multi-env support)

---

## Fix Approach

Add a `_reset_broker_modules_if_needed()` function that:
1. Tracks the previous invocation's target environment
2. Compares current target with previous on each invocation
3. If different: purges `orchestrator.src.broker`, `orchestrator.src.actors.*`, and `orchestrator.src.triggers.*` from `sys.modules`
4. Emits a structured log when environment change triggers module reset
5. Environment config must be set AFTER the module purge

---

## Acceptance Criteria

- [ ] Warm Lambda invocations targeting a different environment correctly re-initialize the broker
- [ ] Messages routed to the correct environment's SQS namespace
- [ ] No regression for same-environment warm starts (modules not unnecessarily purged)
- [ ] Structured log emitted on environment change
- [ ] `mypy` passes

---

## Verification

- [ ] Unit test: `_reset_broker_modules_if_needed()` purges modules when env changes, skips when env is the same
- [ ] Reproduction steps no longer trigger the bug

---

## Out of Scope

- Refactoring broker initialization to avoid module-scope caching entirely
- Automated tests for warm Lambda behavior in CI
- Changes to other Lambda handlers
