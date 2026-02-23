# Full Python Coding Standards

Sylla-specific standards for the Python data-engine codebase. All rules below are settled and enforced in code review.

---

## Type Annotations

### `T | None` — Always

Use the modern union syntax `T | None` instead of `Optional[T]`. This is cleaner, more readable, and the standard style in Python 3.10+.

```python
# ✅ Modern union syntax
def find_book(isbn: str) -> HarvestedBook | None:
    ...

def process(items: list[str] | None = None) -> None:
    ...

# ❌ Legacy Optional
from typing import Optional
def find_book(isbn: str) -> Optional[HarvestedBook]:
    ...
```

### Built-in Generics

Use built-in generic types (`list`, `dict`, `tuple`, `set`) instead of their `typing` module equivalents. The `typing` versions are deprecated since Python 3.9.

```python
# ✅ Built-in generics
def get_titles(books: list[Book]) -> dict[str, str]:
    ...

items: tuple[str, int, float] = ("a", 1, 2.0)
unique: set[str] = {"a", "b"}

# ❌ typing module generics
from typing import List, Dict, Tuple
def get_titles(books: List[Book]) -> Dict[str, str]:
    ...
```

### Explicit Return Types on Public Functions

Required on all public (exported) functions, class methods, and anything that forms part of a module's API. Internal helpers may rely on inference.

```python
# ✅ Public — explicit return type
def harvest_books(query: str, max_results: int = 100) -> list[HarvestedBook]:
    ...

class BookProcessor:
    def process(self, book: RawBook) -> ProcessedBook:
        ...

# ✅ Internal helper — inference acceptable
def _normalize_isbn(raw: str):
    return raw.replace("-", "").strip()
```

### `@property` for Computed Attributes

Use `@property` for computed attributes that are simple, derived, parameter-free, side-effect-free, and cheap. Reserve `get_x()` methods for expensive operations, side effects, or when parameters are needed.

```python
# ✅ @property — simple derived attribute
class HarvestedBook(BaseModel):
    identifiers: list[Identifier] = []

    @property
    def isbn(self) -> str | None:
        for ident in self.identifiers:
            if ident.type == "isbn":
                return ident.value
        return None

    @property
    def doi(self) -> str | None:
        for ident in self.identifiers:
            if ident.type == "doi":
                return ident.value
        return None

# Usage: book.isbn, book.doi — natural alongside Pydantic fields

# ❌ Java-style getter methods
class HarvestedBook(BaseModel):
    def get_isbn(self) -> str | None: ...
    def get_doi(self) -> str | None: ...
```

### Datetime

Always use timezone-aware datetime. `datetime.utcnow()` is deprecated in Python 3.12.

```python
# ✅ Timezone-aware
from datetime import datetime, timezone

now = datetime.now(timezone.utc)
timestamp = datetime.now(timezone.utc).isoformat()

# ❌ Deprecated
now = datetime.utcnow()
```

### TypeVar and Generics

Use `TypeVar` for generic functions. Prefer the new syntax when targeting Python 3.12+.

```python
# ✅ TypeVar for generic functions
from typing import TypeVar

T = TypeVar("T")

def first_or_none(items: list[T]) -> T | None:
    return items[0] if items else None
```

---

## Imports & Module Organization

### Import Location

All imports go at the top of the module. Function-level imports are only acceptable to break genuine circular dependencies — and should include a comment explaining why.

```python
# ✅ Top-level imports
import asyncio
from datetime import datetime, timezone

import httpx
import structlog
from pydantic import BaseModel, ConfigDict, Field

from worker.harvesting.models import HarvestedBook

# ❌ Import inside function (no circular dependency)
def process_book(isbn: str) -> None:
    from worker.harvesting.models import HarvestedBook  # ❌
    ...

# ✅ Exception — circular dependency with comment
def get_processor() -> "BookProcessor":
    # Circular: processor imports this module for models
    from worker.processing.processor import BookProcessor
    return BookProcessor()
```

### Import Order

isort enforces three groups separated by blank lines:

1. Standard library (`import os`, `from datetime import ...`)
2. Third-party (`import httpx`, `from pydantic import ...`)
3. Local/project (`from worker.harvesting import ...`)

```python
# ✅ Correct order
import asyncio
import json
from datetime import datetime, timezone

import httpx
import structlog
from dramatiq import actor
from pydantic import BaseModel, Field

from worker.config import settings
from worker.harvesting.models import HarvestedBook
```

---

## Functions & Classes

### Docstrings — Google Style

Public functions and classes use Google-style docstrings. Internal helpers do not need docstrings if the function name and types are self-explanatory.

```python
# ✅ Google-style docstring on public API
def harvest_books(
    query: str,
    max_results: int = 100,
    timeout: float = 30.0,
) -> list[HarvestedBook]:
    """Harvest books from external APIs matching the query.

    Searches WorldCat and Open Library in parallel, deduplicates
    results by ISBN, and returns the merged list.

    Args:
        query: Search query string (title, author, or ISBN).
        max_results: Maximum number of results to return.
        timeout: HTTP request timeout in seconds.

    Returns:
        List of harvested books, deduplicated by ISBN.

    Raises:
        HarvestingError: If all external APIs fail.
    """
    ...
```

### Function Length

Keep functions under ~50 lines. If a function is longer, split it into focused helpers. A function that fetches, transforms, and stores data should be three functions orchestrated by a fourth.

```python
# ✅ Orchestrator calling focused helpers
def process_batch(books: list[RawBook]) -> BatchResult:
    validated = _validate_books(books)
    enriched = _enrich_with_metadata(validated)
    return _store_results(enriched)

# ❌ Monolithic function doing everything
def process_batch(books: list[RawBook]) -> BatchResult:
    # 150 lines of validation, enrichment, and storage...
    ...
```

### Early Returns

Prefer early returns over deeply nested conditionals. Maximum 3 levels of nesting.

```python
# ✅ Early returns
def process_book(book: RawBook) -> ProcessedBook | None:
    if not book.isbn:
        logger.warning("No ISBN", book_id=book.id)
        return None

    if book.is_duplicate:
        return None

    metadata = fetch_metadata(book.isbn)
    if not metadata:
        logger.warning("No metadata found", isbn=book.isbn)
        return None

    return ProcessedBook.from_raw(book, metadata)

# ❌ Deeply nested
def process_book(book: RawBook) -> ProcessedBook | None:
    if book.isbn:
        if not book.is_duplicate:
            metadata = fetch_metadata(book.isbn)
            if metadata:
                return ProcessedBook.from_raw(book, metadata)
            else:
                logger.warning("No metadata found")
                return None
        else:
            return None
    else:
        logger.warning("No ISBN")
        return None
```

### ABC vs Protocol

Use `ABC` with `@abstractmethod` for shared contracts where subclasses share behavior (like `BaseHarvester`). Use `Protocol` for structural typing where you care about interface shape, not inheritance.

```python
# ✅ ABC — shared behavior, enforced contract
from abc import ABC, abstractmethod

class BaseHarvester(ABC):
    @abstractmethod
    async def harvest(self, query: str) -> list[HarvestedBook]:
        ...

    def deduplicate(self, books: list[HarvestedBook]) -> list[HarvestedBook]:
        """Shared deduplication logic for all harvesters."""
        seen: set[str] = set()
        result: list[HarvestedBook] = []
        for book in books:
            if book.isbn and book.isbn not in seen:
                seen.add(book.isbn)
                result.append(book)
        return result

# ✅ Protocol — structural typing
from typing import Protocol

class Embeddable(Protocol):
    def to_embedding_text(self) -> str: ...
```

---

## Error Handling

### Specific Exceptions

Always catch specific exception types. Bare `except` and `except: pass` are banned. Broad `except Exception` is only acceptable at top-level boundaries (actor entry points, API handlers) with proper logging.

```python
# ✅ Specific exception handling
try:
    response = await client.get(url, timeout=30.0)
    response.raise_for_status()
except httpx.TimeoutException:
    logger.error("Request timed out", url=url)
    raise HarvestingError(f"Timeout fetching {url}")
except httpx.HTTPStatusError as exc:
    logger.error("HTTP error", url=url, status=exc.response.status_code)
    raise HarvestingError(f"HTTP {exc.response.status_code} for {url}")

# ✅ Broad catch only at top-level boundary
@actor
def process_book_actor(book_id: str) -> None:
    try:
        process_book(book_id)
    except Exception:
        logger.exception("Unhandled error in actor", book_id=book_id)
        raise  # Let Dramatiq handle retry

# ❌ Bare except
try:
    process()
except:
    pass

# ❌ Silent swallowing
try:
    response = await client.get(url)
except Exception:
    pass  # ❌ Error silently lost
```

### Structured Logging with structlog

Use `structlog` with bound loggers. Include contextual identifiers in every log message.

```python
# ✅ structlog with context
import structlog

logger = structlog.get_logger()

async def harvest_book(isbn: str) -> HarvestedBook | None:
    log = logger.bind(isbn=isbn, source="worldcat")
    log.info("Starting harvest")

    try:
        result = await _fetch_from_worldcat(isbn)
        log.info("Harvest complete", title=result.title)
        return result
    except HarvestingError as exc:
        log.error("Harvest failed", error=str(exc))
        return None
```

### Custom Exception Hierarchy

Define domain-specific exceptions that inherit from a project base, not bare `Exception`.

```python
# ✅ Domain exception hierarchy
class DataEngineError(Exception):
    """Base exception for all data-engine errors."""

class HarvestingError(DataEngineError):
    """Error during book harvesting from external APIs."""

class ProcessingError(DataEngineError):
    """Error during book data processing."""

class EmbeddingError(DataEngineError):
    """Error during vector embedding generation."""
```

### Log and Re-raise

When catching exceptions at intermediate layers, log with context and re-raise. Do not swallow errors.

```python
# ✅ Log context, then re-raise
async def enrich_book(book: HarvestedBook) -> EnrichedBook:
    try:
        embeddings = await generate_embeddings(book.description)
    except EmbeddingError:
        logger.error("Embedding failed", isbn=book.isbn, title=book.title)
        raise
    return EnrichedBook(**book.model_dump(), embeddings=embeddings)
```

---

## Pydantic Models

### ConfigDict

All Pydantic models must use `ConfigDict(extra="forbid")` to reject unexpected fields. This catches typos and schema drift at runtime.

```python
# ✅ Strict model
from pydantic import BaseModel, ConfigDict, Field

class HarvestedBook(BaseModel):
    model_config = ConfigDict(extra="forbid")

    title: str = Field(description="Book title as reported by the source")
    isbn: str | None = Field(default=None, description="ISBN-13 identifier")
    authors: list[str] = Field(default_factory=list, description="Author names")

# ❌ Missing ConfigDict — extra fields silently ignored
class HarvestedBook(BaseModel):
    title: str
    isbn: str | None = None
```

### Field Metadata

Use `Field(description="...")` on every field. This serves as inline documentation and powers auto-generated API docs.

### Computed Properties

Use `@property` for derived attributes, not methods. See the Type Annotations section for the full rationale.

### Enum Patterns

Use `StrEnum` for string enums in Pydantic models. This ensures proper serialization and comparison.

```python
# ✅ StrEnum
from enum import StrEnum

class BookStatus(StrEnum):
    PENDING = "pending"
    PROCESSING = "processing"
    COMPLETE = "complete"
    FAILED = "failed"

class BookRecord(BaseModel):
    model_config = ConfigDict(extra="forbid")
    status: BookStatus = Field(default=BookStatus.PENDING, description="Processing status")
```

---

## Async Patterns

### `asyncio.gather` for Parallel Work

Use `asyncio.gather` when you have independent async operations that can run in parallel.

```python
# ✅ Parallel fetches
async def harvest_all(isbn: str) -> list[HarvestedBook]:
    results = await asyncio.gather(
        worldcat_harvester.harvest(isbn),
        openlibrary_harvester.harvest(isbn),
        return_exceptions=True,
    )
    books: list[HarvestedBook] = []
    for result in results:
        if isinstance(result, Exception):
            logger.error("Harvester failed", error=str(result))
            continue
        books.extend(result)
    return books

# ❌ Sequential when parallel is possible
worldcat = await worldcat_harvester.harvest(isbn)
openlibrary = await openlibrary_harvester.harvest(isbn)
```

### httpx with Timeouts

Always use `httpx.AsyncClient` with explicit timeouts. Never make HTTP requests without timeout.

```python
# ✅ Explicit timeout and connection pooling
async with httpx.AsyncClient(timeout=httpx.Timeout(30.0)) as client:
    response = await client.get(url)

# ❌ No timeout — can hang indefinitely
async with httpx.AsyncClient() as client:
    response = await client.get(url)
```

### No Sync in Async

Never call synchronous blocking functions inside async code. If unavoidable, use `asyncio.to_thread()`.

```python
# ✅ Offload sync work to thread pool
import asyncio

async def parse_pdf(content: bytes) -> str:
    return await asyncio.to_thread(_parse_pdf_sync, content)

def _parse_pdf_sync(content: bytes) -> str:
    # CPU-bound PDF parsing
    ...

# ❌ Blocking call in async context
async def parse_pdf(content: bytes) -> str:
    return _parse_pdf_sync(content)  # Blocks the event loop
```

### Client Lifecycle

Use context managers for HTTP clients and database connections to ensure proper cleanup.

```python
# ✅ Context manager for client lifecycle
async def fetch_metadata(isbns: list[str]) -> list[Metadata]:
    async with httpx.AsyncClient(timeout=httpx.Timeout(30.0)) as client:
        tasks = [_fetch_one(client, isbn) for isbn in isbns]
        return await asyncio.gather(*tasks)

# ❌ Client created but never closed
client = httpx.AsyncClient()
response = await client.get(url)
# client.aclose() never called
```

---

## Actor / Harvester Architecture

### BaseHarvester Pattern

All harvesters extend `BaseHarvester` (ABC). The base class provides shared logic (deduplication, rate limiting, logging). Subclasses implement the `harvest()` method.

```python
class BaseHarvester(ABC):
    def __init__(self, source_name: str) -> None:
        self.source_name = source_name
        self.logger = structlog.get_logger().bind(source=source_name)

    @abstractmethod
    async def harvest(self, query: str) -> list[HarvestedBook]:
        ...

    def deduplicate(self, books: list[HarvestedBook]) -> list[HarvestedBook]:
        ...

class WorldCatHarvester(BaseHarvester):
    def __init__(self) -> None:
        super().__init__("worldcat")

    async def harvest(self, query: str) -> list[HarvestedBook]:
        ...
```

### Harvester Registry

Use a registry dict to dispatch harvesters by name. Avoid growing `if/elif` chains.

```python
# ✅ Registry pattern
HARVESTER_REGISTRY: dict[str, type[BaseHarvester]] = {
    "worldcat": WorldCatHarvester,
    "openlibrary": OpenLibraryHarvester,
    "google_books": GoogleBooksHarvester,
}

def get_harvester(source: str) -> BaseHarvester:
    cls = HARVESTER_REGISTRY.get(source)
    if not cls:
        raise ValueError(f"Unknown source: {source}")
    return cls()
```

### Dramatiq Actors

Actors are thin entry points that delegate to business logic functions. Keep actor functions short.

```python
# ✅ Thin actor delegating to business logic
@dramatiq.actor(queue_name="harvesting", max_retries=3)
def harvest_book_actor(isbn: str, sources: list[str] | None = None) -> None:
    result = harvest_book(isbn, sources)
    store_result(result)

# ❌ Business logic inside actor
@dramatiq.actor
def harvest_book_actor(isbn: str) -> None:
    # 100 lines of harvesting, parsing, storing...
    ...
```

---

## Configuration

### pydantic-settings

Use `pydantic-settings` for all configuration. Load from environment variables with explicit types and defaults.

```python
from pydantic import Field
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    model_config = ConfigDict(env_prefix="DATA_ENGINE_")

    database_url: str = Field(description="PostgreSQL connection string")
    redis_url: str = Field(default="redis://localhost:6379", description="Redis URL")
    harvesting_timeout: float = Field(default=30.0, description="HTTP timeout in seconds")
    max_retries: int = Field(default=3, description="Max retry attempts for actors")
    debug: bool = Field(default=False, description="Enable debug logging")

settings = Settings()
```

### No Hardcoded Secrets

Never hardcode secrets, API keys, or credentials. Always load from environment.

```python
# ✅ From environment via settings
api_key = settings.worldcat_api_key

# ❌ Hardcoded
API_KEY = "sk-abc123..."

# ❌ Default parameter with real value
def get_client(api_key: str = "sk-abc123...") -> Client:
    ...
```

---

## Testing

### Pytest Conventions

Tests mirror the source structure under a `tests/` directory.

```
src/
  worker/
    harvesting/
      worldcat.py
tests/
  worker/
    harvesting/
      test_worldcat.py
```

### Test Naming

Test functions use `test_<what>_<condition>_<expected>` naming:

```python
# ✅ Descriptive test names
def test_harvest_valid_isbn_returns_book() -> None: ...
def test_harvest_invalid_isbn_returns_none() -> None: ...
def test_harvest_timeout_raises_harvesting_error() -> None: ...
async def test_gather_partial_failure_returns_successes() -> None: ...
```

### Fixtures

Use `pytest` fixtures for shared setup. Prefer factory fixtures for flexible test data.

```python
# ✅ Factory fixture
@pytest.fixture
def make_book() -> Callable[..., HarvestedBook]:
    def _make(**overrides: Any) -> HarvestedBook:
        defaults = {
            "title": "Test Book",
            "isbn": "9781234567890",
            "authors": ["Test Author"],
        }
        return HarvestedBook(**(defaults | overrides))
    return _make

def test_dedup_removes_duplicate_isbn(make_book: Callable[..., HarvestedBook]) -> None:
    books = [make_book(isbn="123"), make_book(isbn="123"), make_book(isbn="456")]
    result = deduplicate(books)
    assert len(result) == 2
```

### Mocking

Mock at the boundary, not the internals. Prefer `httpx` response mocking over patching internal functions.

```python
# ✅ Mock the HTTP boundary
@pytest.fixture
def mock_worldcat(respx_mock: respx.MockRouter) -> None:
    respx_mock.get("https://api.worldcat.org/search").mock(
        return_value=httpx.Response(200, json=SAMPLE_RESPONSE)
    )

# ❌ Patching internal methods
@patch("worker.harvesting.worldcat.WorldCatHarvester._parse_response")
def test_harvest(mock_parse: MagicMock) -> None: ...
```

---

## Naming Conventions

| Thing | Convention | Example |
|-------|-----------|---------|
| Files & folders | snake_case | `book_processor.py`, `harvesting/` |
| Variables | snake_case | `total_count`, `is_valid` |
| Functions | snake_case, verb-first | `harvest_books`, `validate_isbn` |
| Classes | PascalCase | `BookProcessor`, `WorldCatHarvester` |
| Constants | SCREAMING_SNAKE_CASE | `MAX_RETRIES`, `DEFAULT_TIMEOUT` |
| Private | Single underscore prefix | `_parse_response`, `_client` |
| Type variables | PascalCase | `T`, `ResponseT`, `BookT` |
| Boolean variables | `is_`, `has_`, `can_` prefix | `is_valid`, `has_isbn`, `can_retry` |
| Enum members | SCREAMING_SNAKE_CASE | `BookStatus.PENDING` |
| Pydantic fields | snake_case | `created_at`, `source_name` |

---

## Tooling Configuration

### Canonical pyproject.toml

```toml
[tool.black]
line-length = 88
target-version = ['py311']

[tool.isort]
profile = "black"
line_length = 88

[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true

[[tool.mypy.overrides]]
module = "tests.*"
disallow_untyped_defs = false

[tool.pytest.ini_options]
pythonpath = ["."]
testpaths = ["tests"]
```

### Flake8

Ignore rules that conflict with Black:

```ini
# .flake8
[flake8]
max-line-length = 88
extend-ignore = E203, W503
```

---

## Security

### No Dynamic Execution

`eval()`, `exec()`, and `pickle.loads()` are banned. Use `json.loads()` for data deserialization. Use AST-based parsing if you need to interpret code strings.

```python
# ❌ Banned
result = eval(user_input)
data = pickle.loads(raw_bytes)
exec(code_string)

# ✅ Safe alternatives
data = json.loads(raw_string)
config = Settings.model_validate_json(raw_string)
```

### Input Validation at Boundaries

Validate all external input (API requests, queue messages, file content) at system boundaries using Pydantic models.

```python
# ✅ Validate at boundary
@app.post("/harvest")
async def harvest_endpoint(request: HarvestRequest) -> HarvestResponse:
    # request is already validated by Pydantic
    result = await harvest_books(request.query, request.max_results)
    return HarvestResponse(books=result)
```

### Timeouts on External Calls

Every HTTP request, database query, and external service call must have an explicit timeout.

```python
# ✅ Explicit timeouts everywhere
async with httpx.AsyncClient(timeout=httpx.Timeout(30.0)) as client:
    response = await client.get(url)

# For database
async with asyncio.timeout(10.0):
    result = await db.execute(query)
```

---

## Performance

### N+1 Queries

Never make database or API calls inside loops. Batch or parallelize.

```python
# ✅ Batch API call
async def enrich_books(books: list[Book]) -> list[EnrichedBook]:
    isbns = [b.isbn for b in books if b.isbn]
    metadata_map = await fetch_metadata_batch(isbns)
    return [_merge(book, metadata_map.get(book.isbn)) for book in books]

# ❌ N+1 — API call per book
async def enrich_books(books: list[Book]) -> list[EnrichedBook]:
    results = []
    for book in books:
        metadata = await fetch_metadata(book.isbn)  # ❌ N calls
        results.append(_merge(book, metadata))
    return results
```

### Streaming for Large Data

Use generators and streaming for large datasets instead of loading everything into memory.

```python
# ✅ Generator for large result sets
def iter_books(file_path: Path) -> Iterator[RawBook]:
    with open(file_path) as f:
        for line in f:
            yield RawBook.model_validate_json(line)

# ❌ Loading entire file into memory
def get_books(file_path: Path) -> list[RawBook]:
    with open(file_path) as f:
        return [RawBook.model_validate_json(line) for line in f.readlines()]
```

### Connection Pooling

Reuse HTTP clients and database connections. Do not create a new client per request.

```python
# ✅ Shared client with connection pool
class WorldCatHarvester(BaseHarvester):
    def __init__(self) -> None:
        super().__init__("worldcat")
        self._client = httpx.AsyncClient(
            timeout=httpx.Timeout(30.0),
            limits=httpx.Limits(max_connections=20),
        )

# ❌ New client per request
async def fetch(url: str) -> httpx.Response:
    async with httpx.AsyncClient() as client:  # ❌ Created and destroyed each call
        return await client.get(url)
```

---

## Code Smell Checklist

For the Code Smell Checklist used during PR reviews, see the **Code Smell Checklist** section in `SKILL.md`.
