---
name: ticket-creator
description: Creates well-structured Linear tickets from feature requests, bug reports, or technical tasks. Use when asked to create a ticket, write a Linear issue, plan work, or break down a task into subtasks.
allowed-tools: Read, Bash, Edit
---

# Ticket Creator

## Instructions

### Title conventions

- Imperative mood, max 80 characters
- Prefix with area in brackets: `[API]`, `[Frontend]`, `[Auth]`, `[Infra]`, `[Curriculum]`
- Be specific: `[API] Add rate limiting to search endpoint` not `Fix search`

### Description format

Every ticket follows this structure:

```markdown
## Context
Why this work matters. Link to the trigger — a support ticket, a product decision, a tech debt observation.

## Requirements
- Numbered list of what needs to happen
- Each requirement is independently verifiable

## Acceptance Criteria
- [ ] Checkboxes that define "done"
- [ ] Written from the user/system perspective
- [ ] Include edge cases worth calling out

## Technical Notes (optional)
Implementation hints, relevant files, related PRs. Don't over-specify — trust the engineer.
```

### Team assignment

- **Sylla Tech** — backend, infrastructure, APIs, database, DevOps
- **Sylla Product** — frontend, UI/UX, user-facing features, design implementation
- **SyllaCX** — customer support tooling, onboarding, CX workflows

### Label selection

Apply exactly one primary label:

| Label | When to use |
|-------|------------|
| **Feature** | New capability that doesn't exist yet |
| **Bug** | Something is broken or behaving incorrectly |
| **Improvement** | Enhancing existing functionality |
| **Task** | Operational work, migrations, config changes |
| **Spike** | Time-boxed research or investigation |
| **Tech debt** | Refactoring, cleanup, performance work |
| **Support Ticket** | Triggered by a customer support request |

### Priority assignment

| Priority | Criteria |
|----------|----------|
| **Urgent** (P1) | Production is broken, data loss risk, security issue |
| **High** (P2) | Blocks other work, affects multiple users, committed deadline |
| **Normal** (P3) | Default. Important but not time-sensitive |
| **Low** (P4) | Nice-to-have, exploratory, long-term cleanup |

When in doubt, use Normal. Urgency inflation helps nobody.

### Estimation

Use Linear's point scale as a rough proxy for complexity:

| Points | Meaning |
|--------|---------|
| 1 | Trivial — config change, copy update, obvious fix |
| 2 | Small — well-scoped, single-file, < half day |
| 3 | Medium — touches multiple files, needs some investigation |
| 5 | Large — multi-day, cross-cutting, needs design decisions |
| 8 | Epic-sized — break this down into subtasks |

If you estimate 8, stop and create subtasks instead.

### Subtasks vs standalone tickets

**Create subtasks** when:
- A single feature has 3+ distinct implementation steps
- Work can be parallelized across engineers
- You want incremental reviewability

**Keep it standalone** when:
- The work is atomic and coherent
- Breaking it down adds overhead without clarity

Subtask titles must be independently understandable — someone reading the subtask outside the parent should know what to do.

### Linking

- Reference related tickets: "Related to TECH-123"
- Link PRs in the technical notes when known
- If a ticket blocks another, set the blocking relationship explicitly

## Formatting rules

- Title: imperative mood, max 80 chars, area prefix in brackets
- Description: use the Context → Requirements → Acceptance Criteria → Technical Notes template
- Subtask titles: independently understandable, no "Part 1 / Part 2"
- Use markdown formatting in descriptions — bullet lists, checkboxes, code blocks
- Don't write novels. A good ticket is scannable in 30 seconds.

## Examples

### Feature request

**Title:** `[Curriculum] Add bulk import for learning objectives`

**Description:**
```markdown
## Context
Teachers are manually entering learning objectives one at a time. Schools with 200+ objectives need a faster path. Requested by multiple onboarding customers.

## Requirements
1. Accept CSV upload with columns: objective title, subject, grade level
2. Validate all rows before importing (fail entire batch on errors)
3. Show preview table before confirming import
4. Create audit log entry for bulk imports

## Acceptance Criteria
- [ ] CSV with valid data imports all objectives correctly
- [ ] Invalid CSV shows clear error messages per row
- [ ] Preview step shows exactly what will be created
- [ ] Import of 500 objectives completes in < 10s

## Technical Notes
Existing single-create endpoint: `POST /api/objectives`. Consider a new batch endpoint rather than looping. See `ObjectiveService.create()` for validation logic to reuse.
```

**Label:** Feature | **Priority:** Normal | **Estimate:** 5 | **Team:** Sylla Tech

### Bug report

**Title:** `[Auth] Session expires during active use after IdP token refresh`

**Description:**
```markdown
## Context
Support ticket from Marnix College. Teachers report being logged out mid-session, especially around the 1-hour mark. Likely related to the IdP token refresh flow not extending our session TTL.

## Requirements
1. Diagnose whether session expiry ignores IdP token refresh events
2. Extend session TTL when a valid token refresh occurs
3. Add logging around session extension decisions

## Acceptance Criteria
- [ ] Active users are not logged out when their IdP token refreshes
- [ ] Session TTL extends by the configured duration on valid refresh
- [ ] Auth debug logs include refresh-triggered extension events

## Technical Notes
Session middleware: `src/middleware/session.ts`. IdP callback: `src/auth/idp-callback.ts`. Current TTL is 3600s, configured in `SESSION_TTL_SECONDS`.
```

**Label:** Bug | **Priority:** High | **Estimate:** 3 | **Team:** Sylla Tech
