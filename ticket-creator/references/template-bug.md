# Bug Ticket Template

Use this template for any bug, error, or unexpected behavior — from Sentry, user reports, or internal QA.

---

## Sentry MCP Integration

If the user provides a Sentry issue ID or URL:

1. Use `mcp__sentry__get_sentry_issue` to fetch the issue details
2. Extract: error message, stack trace, file location, breadcrumbs, environment, affected user count
3. Auto-populate the template fields from Sentry data
4. Ask the user to confirm and supplement the auto-filled information

If the user describes a bug without a Sentry link:

1. Optionally use `mcp__sentry__search_sentry_issues` to find matching Sentry events
2. If found, offer to link the Sentry issue to the ticket
3. If not found, proceed with manual information gathering

---

## Template Structure

```markdown
## Problem

[1-3 sentences: What is broken and what is the user/system impact?]

## Error

```
[Exact error message and stack trace if available]
```

## Affected File(s)

- `path/to/file.ts:lineNumber` — [brief description]

## Reproduction Steps

1. [Step-by-step to trigger the bug]
2. [Be specific about inputs, user state, environment]

**Environment:** [production | staging | local]
**Frequency:** [always | intermittent | rare]
**Sentry link:** [URL if applicable]

---

<!-- CONDITIONAL: Include when Sentry data is available -->
## Sentry Context

- **Affected users:** [count]
- **First seen:** [date]
- **Frequency:** [occurrences]
- **Breadcrumbs:** [key user actions leading to the error]
<!-- END CONDITIONAL: Sentry -->

---

## Root Cause Analysis

<!-- Include if known. If not, describe the hypothesis. -->

**Known root cause:**
[Explain why this happens. Reference specific code if known.]

**OR**

**Hypothesis:**
[Best guess at what's causing this.]

---

## Current Behavior

[What happens now (the broken state)]

## Expected Behavior

[What should happen instead (the fixed state)]

---

## Technical Context

**Repo:** [nextjs | data-engine]

**Existing pattern to follow:** [If the fix follows an established pattern, reference it]

**Systems involved:** [e.g., AWS Lambda, SQS, NeonDB, Clerk, OpenFGA]

**Side effects to check:**
- [Does fixing this affect anything else?]

**Git history:**
- [Recent commits that may have introduced the regression]

---

## Fix Approach

<!-- Optional but recommended: suggest the fix direction without prescribing code -->

[Brief description of how to fix — direction, not implementation.]

---

## Acceptance Criteria

- [ ] The error no longer occurs under the reproduction steps
- [ ] [Specific condition that proves it's fixed]
- [ ] No regression for [related functionality]
- [ ] `bun typecheck` passes (nextjs) / `mypy` passes (data-engine)

---

## Verification

[Lightweight — nudge towards a regression test if practical.]

- [ ] [Regression test or manual smoke check]
- [ ] Original reproduction steps no longer trigger the bug

---

## Implementation Toolkit

skills: [discovered skills listed here during pattern discovery]

---

## Out of Scope

- [What we are NOT fixing in this ticket]
```

---

## Field Priority (for gathering information)

1. **Critical**: Error message, Affected file, Expected vs actual behavior
2. **High**: Reproduction steps, Root cause (if known), Fix approach
3. **Medium**: Stack trace, Sentry link, Side effects, Git history
4. **Nice-to-have**: Out of scope
