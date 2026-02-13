# Ticket Quality Guidelines

## What Makes a Good Ticket

A good ticket is NOT one that the implementing agent completes in one shot without thinking. A good ticket is one where:

1. **The implementing agent can create a solid plan** — based on the ticket context, codebase exploration, and conversation with the engineer
2. **After the plan is made, there are no open-ended decisions left** — the coding agent doesn't need to invent patterns, guess at data shapes, or make architectural choices that weren't discussed
3. **The scope is narrow enough** that the agent stays on the main path without drifting

The ticket provides the WHAT, WHY, and enough HOW (patterns, references, contracts) that the remaining implementation decisions are mechanical, not architectural.

## Pattern-First Outcomes

| Situation | Outcome |
|-----------|---------|
| Pattern exists + is referenced | Agent plans well, stays on track |
| Pattern exists + NOT referenced | May invent new pattern |
| No pattern exists + good docs/examples | Needs iteration but doable |
| No pattern exists + no docs/examples | Will free-roam, unexpected code |

Always ask for an existing pattern to follow. This is the highest-impact field.

## Writing Guidelines

### What to Include

- Concrete file paths with line numbers where relevant
- TypeScript/Python type definitions for contracts
- Example API responses (actual JSON, not just schemas)
- Specific, testable acceptance criteria
- References to existing patterns and components
- Dependencies (blocked by / blocks)

### What to Exclude

- Implementation steps (let the agent decide HOW to code it)
- Exhaustive test plans (keep verification lightweight)
- Story point estimates
- Label suggestions
- Detailed project management metadata

### How Much Detail

- **Small tickets** (config changes, translations, simple fixes): Summary + pattern + acceptance criteria only
- **Medium tickets** (standard features): Full template with relevant conditional sections
- **Large tickets** (complex features): Full template + consider splitting into smaller tickets

## Verification Guidelines

Every ticket includes a lightweight `## Verification` section. The purpose is NOT to create an exhaustive test plan — it's to nudge towards verifiable work and gradually build test coverage.

### Rules

- Keep it short — 2-4 items max
- Prefer automated tests for pure functions, server actions, data transformations
- Don't force it — UI layout or config changes can be manual smoke checks
- Think regression prevention — the most valuable test catches future breakage

### Verification Examples

**Backend (testable):**
```markdown
## Verification
- [ ] Unit test: `searchWithVectors()` returns correctly shaped results
- [ ] `bun typecheck` passes
```

**UI (manual):**
```markdown
## Verification
- [ ] Page renders correctly with sidebar open/closed
- [ ] `bun typecheck` passes
```

**Bug fix (regression test):**
```markdown
## Verification
- [ ] Regression test: calling with `event_id=None` no longer throws validation error
- [ ] Original reproduction steps no longer trigger the bug
```

**Trivial (smoke check only):**
```markdown
## Verification
- [ ] `bun typecheck` passes
```

## Quality Checklist

Before creating a ticket, verify:

- [ ] **Pattern referenced** — an existing file or implementation is named as the base pattern
- [ ] **Scope is clear** — requirements are atomic and verifiable
- [ ] **No ambiguous decisions** — the implementing agent won't need to make architectural choices
- [ ] **Acceptance criteria are testable** — each criterion can be verified in 30 seconds
- [ ] **File paths are specific** — not "somewhere in the codebase" but exact paths
- [ ] **Quality test passes** — "Could the implementing agent create a complete plan without making any open-ended architectural decisions?"
