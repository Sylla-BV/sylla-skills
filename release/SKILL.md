---
name: release
description: >-
  This skill should be used when the user says "/release", "release procedure",
  "fix bug in prod", "PR to production", "create release PR", "open PR for bug fix",
  "submit my feature", "push to release", or wants to open pull requests following
  Sylla's monthly release-branch model (feat/* → 1 PR to master; fix/* → 2 PRs
  with cherry-pick to the current release branch and master).
version: 1.0.0
allowed_tools:
  - Bash(git branch:*)
  - Bash(git log:*)
  - Bash(git checkout:*)
  - Bash(git cherry-pick:*)
  - Bash(git push:*)
  - Bash(git merge-base:*)
  - Bash(git rev-parse:*)
  - Bash(gh pr create:*)
  - Bash(gh pr view:*)
  - Bash(gh pr edit:*)
  - Bash(gh auth:*)
---

# Release

Route pull requests through Sylla's monthly release-branch model. Detects the current branch, validates its base, and creates the correct PRs.

## Prerequisites

```bash
gh auth status 2>/dev/null | grep -q "Logged in" || echo "GH_NOT_AUTHENTICATED"
BRANCH=$(git rev-parse --abbrev-ref HEAD)
echo "On branch: $BRANCH"
```

Stop and report if:
- `gh` CLI is not authenticated
- Branch is not `feat/*` or `fix/*` — explain the convention below and stop

**Branch convention:**
- `feat/<name>` — new feature, branched from `master`
- `fix/<name>` — bug fix, branched from the current `release/YYYY-MM` branch

## Step 1 — Gather Context

```bash
BRANCH=$(git rev-parse --abbrev-ref HEAD)
RELEASE_BRANCH=$(git branch -r --list 'origin/release/*' | sort | tail -1 | tr -d ' ')
echo "Branch:          $BRANCH"
echo "Release branch:  $RELEASE_BRANCH"
```

If `RELEASE_BRANCH` is empty: stop and tell the user no `origin/release/*` branch was found — they may need to run `git fetch origin` first.

| Branch pattern | Flow |
|---|---|
| `feat/*` | Feature flow → 1 PR to `master` |
| `fix/*` | Bug flow → 2 PRs: release branch + master |
| anything else | Explain convention and stop |

---

## Feature Flow (`feat/*`)

### Validate base — must diverge from `master`

The release branch tip must NOT be in this branch's history (if it is, the branch was started from release, not master):

```bash
git merge-base --is-ancestor "$RELEASE_BRANCH" HEAD \
  && echo "WARN: branch contains release commits — must start from master" \
  || echo "OK: based on master"
```

If `WARN`: tell the user the feature branch must start from `master` (not from `$RELEASE_BRANCH`) and stop.

### Create PR → `master`

Push the branch, then read `./references/pr-templates.md` for the feature PR body template.

```bash
git push -u origin "$BRANCH"
gh pr create \
  --base master \
  --title "<title>" \
  --body "<body from template>"
```

Report the PR URL and finish.

---

## Bug Flow (`fix/*`)

### Validate base — must diverge from current release branch

The release branch tip MUST be in this branch's history (if it isn't, the branch was started from master, not release):

```bash
git merge-base --is-ancestor "$RELEASE_BRANCH" HEAD \
  && echo "OK: based on release" \
  || echo "WARN: branch does not contain release commits — must start from $RELEASE_SHORT"
```

If `WARN`: tell the user the fix branch must start from the current release branch (`$RELEASE_BRANCH`) — not from `master` — and stop.

### PR #1 → Release Branch (production fix)

Push the branch, then read `./references/pr-templates.md` for the fix PR body template.

```bash
git push -u origin "$BRANCH"
RELEASE_SHORT="${RELEASE_BRANCH#origin/}"
gh pr create \
  --base "$RELEASE_SHORT" \
  --title "<title>" \
  --body "<body from template — leave related PR placeholder>"
PR1_URL=$(gh pr view --json url -q .url)
PR1_NUMBER=$(gh pr view --json number -q .number)
```

### Cherry-pick onto Master

Collect commits unique to this branch (not in the release branch), oldest first:

```bash
COMMITS=$(git log --reverse --format="%H" "${RELEASE_BRANCH}..HEAD")
echo "Commits to port: $COMMITS"
```

If `COMMITS` is empty: stop — the branch has no unique commits to port.

Create a new branch from `master` and cherry-pick (`-B` resets the branch if it already exists, safe for re-runs):

```bash
CHERRY_BRANCH="${BRANCH}-master"
git checkout -B "$CHERRY_BRANCH" origin/master
git cherry-pick $COMMITS
git push -u origin "$CHERRY_BRANCH"
```

If cherry-pick produces conflicts: resolve them, run `git cherry-pick --continue`, then push.

### PR #2 → Master (indefinite fix)

```bash
gh pr create \
  --base master \
  --head "$CHERRY_BRANCH" \
  --title "<same title as PR #1>" \
  --body "<body from template — include PR1_URL as related>"
PR2_URL=$(gh pr view "$CHERRY_BRANCH" --json url -q .url)
```

### Cross-link PRs

Update PR #1 body to reference PR #2:

```bash
CURRENT_BODY=$(gh pr view "$PR1_NUMBER" --json body -q .body)
gh pr edit "$PR1_NUMBER" --body "${CURRENT_BODY}

---
Related: $PR2_URL (master port)"
```

### Output

```
PR #1 → release: <PR1_URL>
PR #2 → master:  <PR2_URL>
```

---

## Quick Reference

| Branch | Base | PRs |
|---|---|---|
| `feat/*` | `master` | 1 PR → `master` |
| `fix/*` | `release/YYYY-MM` | PR #1 → release, PR #2 → master (cherry-pick) |

Release branch is always detected dynamically — never hardcoded.
