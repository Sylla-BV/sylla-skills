# Docstring Convention (Google Style)

## Functions and Methods

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

## Classes

```python
class MyClass:
    """Brief one-line summary.

    Longer description of the class purpose and behavior.

    Attributes:
        attr_name: Description of the attribute.
    """
```

## Pydantic Models

```python
class MyModel(BaseModel):
    """Brief one-line summary."""

    name: str = Field(description="The display name.")
    count: int = Field(default=0, description="Number of items.")
```

## Rules

- **Be concise** — do not pad docstrings with obvious information
- **Document the why, not just the what** — for complex methods, explain the reasoning
- **Include all Args** — every parameter (except `self`/`cls`) must appear in the Args section
- **Include Returns** — unless the function returns `None` with no meaningful side-effect
- **Include Raises** — only for exceptions explicitly raised in the method body
- **Use `Field(description=...)`** for Pydantic models instead of inline comments
