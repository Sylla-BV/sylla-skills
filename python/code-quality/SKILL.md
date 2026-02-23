---
name: code-quality-python
description: >-
  This skill should be used when the user asks to "review Python code",
  "check code quality", "run a code review", "review this PR",
  "check for code smells", "review against SOLID", "check DRY violations",
  "review Python files", "quality check", "is this clean code",
  "review my changes", or wants a structured code quality review
  of Python files against SOLID, DRY, KISS, YAGNI, clean code,
  error handling, security, and performance principles.
---

# Python Code Quality Review

Review modified Python files against coding standards and quality principles. Report only concrete violations that need fixing — not style nits or theoretical concerns.

Before starting, read the **Code Smell Checklist** in `python/coding-standards/SKILL.md` — those rules are authoritative and take precedence.

## Workflow

### Step 1: Identify Target Files

Determine which `.py` files to review:

- If the user specifies files, use those.
- Otherwise, check `git diff --name-only HEAD` and `git diff --name-only --cached` for recently modified `.py` files.
- Include test files — they follow the same principles.

If no modified files are found, ask the user which files or directory to scan.

### Step 2: Run the Code Smell Checklist

Read each target file and check every item in the `python/coding-standards/SKILL.md` Code Smell Checklist. These are hard rules — any match is a MUST FIX.

### Step 3: Review Against Quality Principles

For each file, check for violations of these principles. Focus on the **changed code** — do not audit the entire file unless asked.

**SOLID Principles:**
- **SRP** — Does the class/function do more than one thing? Flag a function that fetches AND transforms AND stores.
- **OCP** — Are there `if/elif` chains that grow when adding types? Flag hardcoded dispatch that should use a registry.
- **LSP** — Does a subclass break the parent's contract? Flag overrides that raise where the parent does not.
- **ISP** — Is a class forced to implement unused methods? Flag `pass` or `NotImplementedError` stubs.
- **DIP** — Does high-level logic instantiate low-level dependencies directly? Flag `client = httpx.Client()` in business logic.

**DRY** — Duplicated code blocks (3+ lines) across modified files or within the same file.

**KISS** — Unnecessary complexity: metaclasses, decorators, or abstractions for a single use case. Deeply nested logic (3+ levels).

**YAGNI** — Code for hypothetical future requirements: unused parameters, feature flags for non-existent features, ABCs with only one subclass. Exception: established project patterns like BaseHarvester.

**Clean Code** — Meaningless names, functions >50 lines, deep nesting, stale comments, comments explaining *what* instead of *why*.

**Error Handling** — Broad `except Exception`, silent `except: pass`, missing input validation at boundaries, uninformative error messages.

**Security** — Unsanitized input in SQL/shell/file paths, hardcoded secrets, `pickle.loads`/`eval`/`exec`, HTTP requests without timeout.

**Performance** — N+1 patterns (DB/API calls inside loops), large collections loaded into memory, sync blocking calls in async.

### Step 4: Present Findings

Only report **concrete, actionable violations**. Skip anything subjective or borderline.

Group by severity:

```
## Code Quality Review

### MUST FIX (correctness/security risk)
- `book_api_client.py:45` — **Security**: JWT secret hardcoded as default parameter. Move to config.
- `llm_classifier.py:112` — **Error Handling**: bare `except: pass` swallows all errors including KeyboardInterrupt.

### SHOULD FIX (maintainability)
- `embedding_providers.py:30-55` — **DRY**: retry logic duplicated across `embed_batch()` and `embed_single()`. Extract to a shared `_retry_request()` method.
- `worldcat_scraper_actor.py:89` — **SRP**: `scrape_and_parse()` does HTTP fetching, HTML parsing, AND data transformation. Split into focused functions.

### CONSIDER (minor improvements)
- `book_api_client.py:78` — **KISS**: `_build_headers()` has 3 levels of nested conditionals. Simplify with early returns.
```

### Step 5: Suggest Fixes

For each MUST FIX and SHOULD FIX item, describe the refactoring in one sentence. Do not rewrite the code.

## Rules

- **Only flag real violations** — if the code is simple, correct, and readable, say so and move on
- **Do not flag established project patterns** — if the codebase uses a pattern consistently (e.g., BaseHarvester hierarchy), do not flag it as over-engineering
- **Be proportional** — a 5-line utility function does not need SRP analysis
- **Focus on the diff** — review what changed, not the entire file history
- **No false positives** — if you are unsure whether something is a violation, skip it
- **Respect YAGNI in your own recommendations** — do not suggest adding abstractions or infrastructure the current code does not need
