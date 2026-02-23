---
name: update-tests-python
description: >-
  This skill should be used when the user asks to "update tests",
  "audit test coverage", "generate tests", "add tests", "fix tests",
  "write pytest tests", "check test coverage", "missing tests",
  "stale tests", "test this module", "add test cases",
  "pytest conventions", "mock patterns", "fixture patterns",
  or wants to audit and generate Python tests following project conventions.
---

# Update Python Tests

Audit and update Python tests across modified source files to maintain coverage and correctness.

## Workflow

### Step 1: Identify Target Source Files

Determine which `.py` source files to audit tests for:

- If the user specifies files or a directory, use those.
- Otherwise, check `git diff --name-only HEAD` and `git diff --name-only --cached` for recently modified `.py` files.
- Exclude test files themselves — focus on source files that need test coverage.

If no modified files are found, ask the user which service or directory to scan.

### Step 2: Read Reference Test Patterns

Before writing tests, read existing test patterns in the target service. If the service has tests, follow its conventions. Look for `conftest.py` with shared fixtures.

### Step 3: Map Source Files to Test Files

For each source file, determine the corresponding test file:

| Source file | Expected test file |
|---|---|
| `{service}/worker/services/foo.py` | `{service}/tests/test_foo.py` |
| `{service}/worker/actors/bar_actor.py` | `{service}/tests/test_bar_actor.py` |
| `{service}/worker/harvesting/harvesters/baz.py` | `{service}/tests/test_baz_harvester.py` |

Check if the test file exists. If not, it needs to be created.

### Step 4: Audit Test Coverage

For each source file, catalog all public methods and functions. Then check the corresponding test file for:

1. **Missing test file** — No test file exists at all.
2. **Untested public methods** — Source methods with no corresponding test.
3. **Stale test assertions** — Tests referencing old signatures, removed parameters, or changed return types.
4. **Outdated mock fixtures** — Mocks that do not match current config fields or service signatures.

### Step 5: Present Coverage Gap Report

```
## Test Coverage Audit Report

### worker/services/book_api_client.py
- NO TEST FILE: tests/test_book_api_client.py does not exist
- Public methods needing tests:
  - `BookAPIClient.__init__()` — JWT setup, config loading
  - `BookAPIClient.store_chapters()` — API call with retry
  - `BookAPIClient.is_production_environment()` — Environment check

### worker/services/llm_classifier.py
- PARTIAL COVERAGE: tests/test_llm_classifier.py exists
- Untested methods:
  - `LLMChapterClassifier._create_together_chain()` — Chain creation
```

### Step 6: Generate or Update Tests

Write tests following the patterns below. After generating, run them:

```bash
cd {service-directory} && poetry run pytest {test_file} -v
```

Fix any failures before finishing.

## Test Patterns

### Fixture Patterns

```python
@pytest.fixture
def mock_config():
    """Mock the config module."""
    with patch("worker.services.book_api_client.config") as mock:
        mock.api_host_url = "https://api.test.com"
        mock.api_signing_secret = "test-secret"
        mock.environment = "test"
        yield mock

@pytest.fixture
def sample_api_response() -> dict:
    """Create a sample API response for testing."""
    return {"id": "123", "title": "Test Book", "status": "success"}

@pytest.fixture
def client(mock_config) -> BookAPIClient:
    """Create a BookAPIClient instance with mocked config."""
    return BookAPIClient()
```

### Test Class Structure

```python
class TestBookAPIClient:
    """Tests for BookAPIClient."""

    def test_init_sets_base_url(self, client, mock_config):
        """Test client initializes with correct base URL."""
        assert client.base_url == "https://api.test.com"

    def test_store_chapters_success(self, client, sample_api_response):
        """Test store_chapters with valid input."""
        with patch.object(client, "_make_request", return_value=sample_api_response):
            result = client.store_chapters(book_id="123", chapters=[...])
            assert result == expected

    def test_store_chapters_retries_on_failure(self, client):
        """Test store_chapters retries on transient errors."""
        ...
```

### Test Naming

- `test_{method}_{scenario}` — e.g., `test_store_chapters_success`
- Group tests for one class in a `TestClassName` class.
- Group tests for module-level functions in a `TestModuleName` class.

### Mocking Rules

- **Mock external HTTP** at the client level (`httpx.Client`, `httpx.AsyncClient`) — not at the method level.
- **Mock external services** — S3, SQS, databases, APIs, AI providers.
- **Do not mock internal project code** — test real interactions between project modules.
- **Use `@pytest.fixture`** — never inline setup; fixtures go at module level or in `conftest.py`.
- **Prefer `patch()` context managers** over `@patch` decorators for clarity.

### Harvester Test Pattern

For harvester classes, use a concrete test implementation:

```python
class ConcreteHarvester(BaseHarvester):
    """Concrete implementation for testing abstract base."""

    source_name = "test"
    source_url = "https://test.example.com"

    async def fetch_catalog(self) -> list[dict]:
        return [{"id": "1", "title": "Test"}]

    async def process_entry(self, entry: dict) -> ProcessedBook:
        return ProcessedBook(...)


class TestConcreteHarvester:
    @pytest.fixture
    def harvester(self, mock_config):
        return ConcreteHarvester()

    def test_fetch_catalog_returns_entries(self, harvester):
        ...
```

## Rules

- **Run tests after generating** — always execute `poetry run pytest {test_file} -v` and fix failures
- **Do not over-mock** — if a method is pure logic with no I/O, test it directly without mocks
- **Test behavior, not implementation** — assert on outcomes, not on call counts
- **One assertion focus per test** — each test verifies one logical behavior
- **Include error cases** — test exception handling, invalid input, and edge cases
- **Use realistic test data** — sample data should resemble real payloads
- **Keep test files focused** — one test file per source module
- **Check for `conftest.py`** — use shared fixtures instead of duplicating
