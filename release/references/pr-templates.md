# PR Body Templates

## Feature PR (feat/* → master)

```markdown
## Summary

<!-- What does this feature do? Link to Linear ticket if applicable. -->

## Changes

<!-- Bullet list of key changes -->

## Test plan

- [ ] Manually verified on local dev
- [ ] No regressions in adjacent flows
```

---

## Bug Fix PR #1 (fix/* → release/YYYY-MM)

```markdown
## Summary

Fixes <!-- describe the bug --> in production.

## Root cause

<!-- What caused the bug? -->

## Fix

<!-- What was changed and why? -->

## Test plan

- [ ] Reproduced the bug before the fix
- [ ] Verified the fix resolves it on staging

---
*Also see: [master port](TO_BE_ADDED)*
```

---

## Bug Fix PR #2 (fix/*-master → master)

```markdown
## Summary

Ports the production fix for <!-- describe the bug --> to master.

This is a cherry-pick of the same fix applied in the release branch.

## Test plan

- [ ] No conflicts after cherry-pick
- [ ] Verified no regressions

---
*Also see: [production fix (release branch)](PR1_URL)*
```
