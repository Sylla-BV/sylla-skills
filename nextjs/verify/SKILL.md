---
name: verify
description: >-
  This skill should be used when the user says "verify", "check my work",
  "am I done", "run checks", or "validate". Also use when implementation is
  complete, a plan has been executed, a mid-to-large feature is finished, or
  the user is about to open a PR or run superpowers:finishing-a-development-branch.
version: 1.0.0
allowed_tools:
  - Bash
  - Task
  - AskUserQuestion
  - Bash(agent-browser:*)
  - Bash(bunx agent-browser:*)
  - mcp__linear__get_issue
---

# Verify

Multi-stage quality gate. Run after finishing implementation, before `superpowers:finishing-a-development-branch`.

**Fail-fast:** Stop at the first failing stage. Do not continue.

---

## Stage 1 — Get Changed Files

```bash
git diff HEAD --name-only
```

Store this list. Every conditional stage below checks against it.

---

## Stage 2 — Lint

Run **only on changed files** (never without args — it will timeout):

```bash
bun lint <changed-files-space-separated>
```

**STOP — do not continue to Stage 3.**

---

## Stage 3 — Typecheck

```bash
bun typecheck
```

**STOP — do not continue to Stage 4.**

---

## Stage 4 — Coding Standards Audit (conditional)

**Trigger:** Any changed file matches `**/*.ts` or `**/*.tsx`

Dispatch a `general-purpose` subagent via the Task tool with this prompt:

```
Read ./coding-standards/SKILL.md (the full skill, including the Code Smell Checklist).

Run through every item in the Code Smell Checklist against the following changed files:

Changed files: <list of .ts/.tsx files>

For each violation found, report:
- File path and approximate line number
- Which checklist item it violates
- The offending code snippet

Report violations only. If everything is clean, report "No violations found."
```

**STOP — do not continue to Stage 5.**

---

## Stage 5 — UI Verification (conditional)

**Trigger:** Any changed file matches `src/app/**/*.tsx`, `src/components/**`, or `src/styles/**`

### 5a — Determine Acceptance Criteria

1. Check the current conversation — user may have described expected behavior, a Linear ticket, or a spec
2. If a Linear issue is active or referenced → fetch it with `mcp__linear__get_issue`
3. If still unclear → use `AskUserQuestion` with the most logical options for this context, always including an "Other (describe)" option

### 5b — Start Dev Server (if not already running)

```bash
if curl -s http://localhost:3000 > /dev/null 2>&1; then
  echo "Dev server already running — skipping start"
  DEV_STARTED_BY_VERIFY=false
else
  bun dev &
  DEV_PID=$!
  DEV_STARTED_BY_VERIFY=true
  until curl -s http://localhost:3000 > /dev/null 2>&1; do sleep 2; done
  echo "Dev server ready"
fi
```

### 5c — Dispatch Agent-Browser Task

Spawn a `general-purpose` subagent via the Task tool. In the prompt:
- Tell it to read `./agent-browser/SKILL.md` for the full command reference and session management patterns
- Include the acceptance criteria from 5a
- Include the relevant route(s) to navigate to
- Ask it to use `--session-name verify` (auto-saves/restores Clerk session across runs)
- Ask it to use `screenshot --annotate` at key states (gives visual layout + numbered refs)
- Ask it to report pass/fail per criterion and close the session when done

**Example prompt (adapt route and criteria to this feature):**
```
Read ./agent-browser/SKILL.md for browser automation command reference, then verify the following at http://localhost:3000.

Use --session-name verify for ALL commands. This auto-saves Clerk session state after
first login so subsequent runs skip authentication.

Route: /[relevant path]

Acceptance criteria:
- <criterion 1>
- <criterion 2>

Workflow:
1. agent-browser --session-name verify open http://localhost:3000/[route]
2. agent-browser --session-name verify wait --load networkidle
3. Check current URL — if redirected to Clerk login:
   a. agent-browser --session-name verify snapshot -i
   b. Fill email with $AGENT_BROWSER_EMAIL, password with $AGENT_BROWSER_PASSWORD
   c. agent-browser --session-name verify wait --url "**/localhost:3000**"
   (On subsequent runs, Clerk session is restored automatically — skip this step)
4. For each criterion: navigate/interact as needed, take annotated screenshot, verify
5. agent-browser --session-name verify screenshot --annotate  (at each key state)
6. Report pass/fail per criterion with screenshot evidence
7. agent-browser --session-name verify close
```

**Credentials:** `$AGENT_BROWSER_EMAIL` / `$AGENT_BROWSER_PASSWORD` from `env.example` → `.env.local`. To reset an expired session: `agent-browser state clear verify`.

### 5d — Stop Dev Server (only if we started it)

```bash
if [ "$DEV_STARTED_BY_VERIFY" = "true" ]; then
  kill $DEV_PID 2>/dev/null
fi
```

---

## Stage Summary

| Changed files contain | Stage triggered |
|---|---|
| `**/*.ts` or `**/*.tsx` | Stage 4: Coding standards audit |
| `src/app/**/*.tsx`, `src/components/**`, `src/styles/**` | Stage 5: UI verification |
| Neither | Stages 1–3 only |

---

## Final Report

When all stages pass, output:

```
✅ Lint              — N files
✅ Typecheck
✅ Coding standards — N files, no violations  (if triggered)
✅ UI verified       — N/N criteria passed    (if triggered)

→ Ready for: superpowers:finishing-a-development-branch
```
