# Test Patterns

## Fixture Patterns

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

## Test Class Structure

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

## Test File Mapping

| Source file | Expected test file |
|---|---|
| `{service}/worker/services/foo.py` | `{service}/tests/test_foo.py` |
| `{service}/worker/actors/bar_actor.py` | `{service}/tests/test_bar_actor.py` |
| `{service}/worker/harvesting/harvesters/baz.py` | `{service}/tests/test_baz_harvester.py` |

## Test Naming

- `test_{method}_{scenario}` — e.g., `test_store_chapters_success`
- Group tests for one class in a `TestClassName` class.
- Group tests for module-level functions in a `TestModuleName` class.

## Mocking Rules

- **Mock external HTTP** at the client level (`httpx.Client`, `httpx.AsyncClient`) — not at the method level.
- **Mock external services** — S3, SQS, databases, APIs, AI providers.
- **Do not mock internal project code** — test real interactions between project modules.
- **Use `@pytest.fixture`** — never inline setup; fixtures go at module level or in `conftest.py`.
- **Prefer `patch()` context managers** over `@patch` decorators for clarity.

## Harvester Test Pattern

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
