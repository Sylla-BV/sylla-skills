---
name: update-docstrings-python
description: >-
  This skill should be used when the user asks to "update docstrings",
  "audit docstrings", "fix docstrings", "add docstrings", "check docstrings",
  "review documentation", "Google-style docstrings", "add field descriptions",
  "Pydantic field descriptions", "missing docstrings", "stale docstrings",
  "document this module", "add Args/Returns/Raises sections",
  or wants to ensure Python docstrings match current implementations.
---

# Update Python Docstrings

Audit and update Python docstrings across modified files to keep documentation in sync with implementations.

## Workflow

### Step 1: Identify Target Files

Determine which `.py` files to audit:

- If the user specifies files or a directory, use those.
- Otherwise, check `git diff --name-only HEAD` and `git diff --name-only --cached` for recently modified `.py` files.
- Exclude test files (`test_*.py`, `*_test.py`) and `__init__.py` unless they contain substantial logic.

If no modified files are found, ask the user which service or directory to scan.

### Step 2: Read Reference Style

Before making changes, read the existing docstring style in the target service. Match its local conventions. If no established convention exists, use the Google-style convention in the Docstring Convention section below.

### Step 3: Audit Each File

For each target file, check every public class, method, and function for:

1. **Missing docstrings** — Public methods/functions with no docstring at all.
2. **Stale docstrings** — `Args`, `Returns`, or `Raises` sections that do not match the current signature:
   - Parameters listed in docstring but not in signature (or vice versa).
   - Return type documented but function returns `None` (or vice versa).
   - Exceptions documented but not raised (or vice versa).
3. **Missing Pydantic field descriptions** — `Field()` calls without `description=` on public models.

**Skip these (do not add docstrings to):**
- Private methods (`_method`) unless they contain complex logic.
- Dunder methods (`__repr__`, `__str__`, `__eq__`).
- Simple property getters that return a field.
- Trivial one-line functions where the name is self-explanatory.
- Test functions.

### Step 4: Present Findings

Group findings by file and severity:

```
## Docstring Audit Report

### worker/services/book_api_client.py
- MISSING: `is_production_environment()` — no docstring
- MISSING: `create_jwt_token()` — no docstring
- STALE: `store_chapters()` — Args section missing `chapter_data` param

### worker/actors/worldcat_scraper_actor.py
- MISSING: `extract_isbn_from_row()` — no docstring
- FIELD: `HarvestConfig.timeout` — Field() missing description
```

### Step 5: Update Docstrings

Apply fixes using the Edit tool. Only modify docstrings — never change implementation code.

## Docstring Convention (Google Style)

**Functions and methods:**

```python
def method_name(self, arg: str, timeout: int = 30) -> bool:
    """Brief one-line summary.

    Longer description if the logic is not obvious.

    Args:
        arg: Description of the argument.
        timeout: Description with default noted.

    Returns:
        Description of return value.

    Raises:
        SpecificError: When this condition occurs.
    """
```

**Classes:**

```python
class MyClass:
    """Brief one-line summary.

    Longer description of the class purpose and behavior.

    Attributes:
        attr_name: Description of the attribute.
    """
```

**Pydantic models:**

```python
class MyModel(BaseModel):
    """Brief one-line summary."""

    name: str = Field(description="The display name.")
    count: int = Field(default=0, description="Number of items.")
```

## Rules

- **Only touch docstrings** — never modify implementation code, imports, or logic
- **Match the local style** — if the service uses a different convention, follow the service's existing style
- **Be concise** — do not pad docstrings with obvious information
- **Document the why, not just the what** — for complex methods, explain the reasoning
- **Include all Args** — every parameter (except `self`/`cls`) must appear in the Args section
- **Include Returns** — unless the function returns `None` with no meaningful side-effect
- **Include Raises** — only for exceptions explicitly raised in the method body
- **Use `Field(description=...)`** for Pydantic models instead of inline comments
