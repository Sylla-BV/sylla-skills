---
name: code-upkeep
description: >-
  This skill should be used when the user asks to "update docstrings",
  "audit docstrings", "fix docstrings", "add docstrings", "check docstrings",
  "Google-style docstrings", "add field descriptions", "update tests",
  "audit test coverage", "generate tests", "add tests", "fix tests",
  "write pytest tests", "check test coverage", "missing tests",
  "stale tests", "test this module", "add test cases",
  "bring code up to date", "code upkeep", "maintain code",
  or wants to audit and update Python docstrings or tests.
---

# Code Upkeep — Docstrings & Tests

Audit and update Python docstrings and test coverage across modified files. This skill generates code — it modifies docstrings and creates/updates test files.

## Workflow

### Step 1: Identify Target Files

Determine which `.py` source files to audit:

- If the user specifies files or a directory, use those.
- Otherwise, check `git diff --name-only HEAD` and `git diff --name-only --cached` for recently modified `.py` files.
- Exclude test files from docstring audit. Include source files for test audit.

If no modified files are found, ask the user which service or directory to scan.

### Step 2: Determine Scope

Ask the user (or infer from their request) which stages to run:

| Stage | Trigger phrases |
|-------|----------------|
| Docstrings | "update docstrings", "add docstrings", "fix docs" |
| Tests | "update tests", "add tests", "check coverage" |
| Both | "code upkeep", "bring up to date", or no specific mention |

### Step 3: Docstring Audit (if in scope)

Read the docstring convention in `python/code-upkeep/references/docstring-convention.md`.

For each target file, check every public class, method, and function for:

1. **Missing docstrings** — Public methods/functions with no docstring at all.
2. **Stale docstrings** — `Args`, `Returns`, or `Raises` sections that do not match the current signature.
3. **Missing Pydantic field descriptions** — `Field()` calls without `description=` on public models.

**Skip:** Private methods (unless complex), dunders, simple property getters, trivial one-liners, test functions.

### Step 4: Test Coverage Audit (if in scope)

Read the test patterns in `python/code-upkeep/references/test-patterns.md`.

For each source file, map to the corresponding test file and check for:

1. **Missing test file** — No test file exists at all.
2. **Untested public methods** — Source methods with no corresponding test.
3. **Stale test assertions** — Tests referencing old signatures or removed parameters.
4. **Outdated mock fixtures** — Mocks that do not match current config or service signatures.

### Step 5: Present Findings

```
## Code Upkeep Report

### Docstring Audit
#### worker/services/book_api_client.py
- MISSING: `create_jwt_token()` — no docstring
- STALE: `store_chapters()` — Args section missing `chapter_data` param
- FIELD: `HarvestConfig.timeout` — Field() missing description

### Test Coverage Audit
#### worker/services/book_api_client.py
- NO TEST FILE: tests/test_book_api_client.py does not exist
- Public methods needing tests:
  - `BookAPIClient.store_chapters()` — API call with retry
```

### Step 6: Apply Fixes

- **Docstrings:** Apply fixes using the Edit tool. Only modify docstrings — never change implementation code.
- **Tests:** Write tests following the patterns in `references/test-patterns.md`. After generating, run:

```bash
cd {service-directory} && poetry run pytest {test_file} -v
```

Fix any failures before finishing.

## Rules

- **Match the local style** — if the service uses a different convention, follow it
- **Only touch docstrings** — never modify implementation code when updating docs
- **Run tests after generating** — always execute and fix failures
- **Do not over-mock** — test pure logic directly without mocks
- **Test behavior, not implementation** — assert on outcomes, not call counts
- **One assertion focus per test** — each test verifies one logical behavior
- **Use realistic test data** — sample data should resemble real payloads
- **Check for `conftest.py`** — use shared fixtures instead of duplicating
