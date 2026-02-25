---
name: coding-standards
description: >-
  This skill should be used when the user asks about "Python coding standards",
  "Python code style", "type hints", "type annotations", "Optional vs union",
  "T | None", "mypy", "black", "isort", "flake8", "ruff",
  "Pydantic models", "ConfigDict", "async patterns", "asyncio",
  "structlog", "error handling in Python", "Python naming conventions",
  "snake_case", "reviewing a Python PR", "does this pass Python code review",
  "BaseHarvester", "Dramatiq actors", "pydantic-settings",
  "pytest conventions", or wants to know whether code follows Sylla's
  Python data-engine repository standards.
---

# Python Coding Standards

Sylla-specific standards for the Python data-engine codebase. All rules are settled and enforced in code review. Full rationale and code examples for every rule are in **`references/standards.md`**.

## How to Apply This Skill

- For quick lookups (rule name, convention), use the Quick Reference tables below.
- For rationale, code examples, or edge cases, consult `references/standards.md`.
- When reviewing code, run through the Code Smell Checklist at the bottom of this file.

---

All work must satisfy readability, KISS, DRY, YAGNI, and security — specifically: no `eval`/`exec`/`pickle`, `T | None` over `Optional[T]`, explicit return types on public functions, and `@property` for computed attributes.

---

## Quick Reference

### Type Annotations

| Rule | Standard |
|------|----------|
| Optional types | `T \| None` — never `Optional[T]` |
| Return types on public functions | Required |
| Return types on internal helpers | Inferred is acceptable |
| Computed attributes | `@property` — never `get_x()` methods |
| Generics | `list[str]`, `dict[str, int]` — never `List`, `Dict` from `typing` |
| Datetime | `datetime.now(timezone.utc)` — never `datetime.utcnow()` |

### Imports & Modules

| Rule | Standard |
|------|----------|
| Import location | Top of module always |
| Function-level imports | Only to break circular dependencies |
| Import order | stdlib > third-party > local (isort enforced) |
| Unused imports | Remove — never comment out |

### Functions & Classes

| Rule | Standard |
|------|----------|
| Docstrings | Google-style on public APIs |
| Max function length | ~50 lines — split if longer |
| Nesting | Max 3 levels — use early returns |
| Abstract base classes | `ABC` + `@abstractmethod` for shared contracts |
| Protocols | Use for structural typing when ABC is too rigid |

### Error Handling

| Rule | Standard |
|------|----------|
| Bare `except` | Banned |
| Broad `except Exception` | Only at top-level boundaries with logging |
| Silent `except: pass` | Banned |
| Logging library | `structlog` with bound loggers |
| Error messages | Include context (IDs, operation name) |
| Custom exceptions | Inherit from domain-specific base, not bare `Exception` |

### Async Patterns

| Rule | Standard |
|------|----------|
| Parallel async calls | `asyncio.gather(*tasks)` |
| HTTP client | `httpx.AsyncClient` with explicit timeouts |
| Sync calls in async | Banned — use `asyncio.to_thread()` if unavoidable |
| Client lifecycle | Context managers (`async with`) for connection pools |

### Pydantic Models

| Rule | Standard |
|------|----------|
| Extra fields | `model_config = ConfigDict(extra="forbid")` |
| Field metadata | `Field(description="...")` on every field |
| Computed attributes | `@property` — never `get_x()` |
| Enums in models | Use `StrEnum` for string enums |

### Naming

| Thing | Convention |
|-------|-----------|
| Files & folders | snake_case |
| Variables & functions | snake_case |
| Classes | PascalCase |
| Constants | SCREAMING_SNAKE_CASE |
| Private attributes | Single leading underscore `_name` |
| Type variables | PascalCase (`T`, `ResponseT`) |
| Boolean variables | `is_`, `has_`, `can_` prefix |

### Tooling

| Tool | Config |
|------|--------|
| Black | line-length 88, target py311 |
| isort | profile "black", line_length 88 |
| MyPy | strict — `disallow_untyped_defs = true`, `warn_return_any = true` |
| Pytest | `pythonpath = ["."]`, `testpaths = ["tests"]` |
| Flake8 | Ignore E203, W503 (Black-incompatible rules) |

---

## Code Smell Checklist

Before opening a PR, check:

- [ ] `Optional[T]` — change to `T | None`
- [ ] `datetime.utcnow()` — change to `datetime.now(timezone.utc)`
- [ ] `get_x()` method for a simple computed attribute — change to `@property`
- [ ] Import inside a function body (no circular dependency reason) — move to top of module
- [ ] Function >~50 lines — split it
- [ ] >3 levels of nesting — use early returns
- [ ] Bare `except` or `except: pass` — catch specific exceptions, log the error
- [ ] `except Exception` in non-boundary code — narrow the exception type
- [ ] Magic number/string — extract to `SCREAMING_SNAKE_CASE` constant
- [ ] `typing.List`, `typing.Dict`, `typing.Tuple` — use built-in `list`, `dict`, `tuple`
- [ ] Missing return type on public function — add explicit annotation
- [ ] `eval()`, `exec()`, `pickle.loads()` — find a safe alternative
- [ ] HTTP request without timeout — add explicit timeout
- [ ] `datetime.utcnow()` — use `datetime.now(timezone.utc)`
- [ ] Sync blocking call inside `async` function — use `asyncio.to_thread()`
- [ ] N+1 query pattern (DB/API call inside loop) — batch or use `asyncio.gather`
- [ ] Pydantic model missing `ConfigDict(extra="forbid")` — add it
- [ ] `get_x()` method that is a simple property — convert to `@property`
