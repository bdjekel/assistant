# Coding Patterns

## Code Review Standards

### What to Look For

1. **Correctness** — Does it do what it's supposed to?
2. **Readability** — Can someone else understand it in 6 months?
3. **Performance** — Any obvious inefficiencies?
4. **Security** — Input validation, secrets handling?
5. **Testing** — Are edge cases covered?

### Review Checklist

- [ ] Function/method names describe what they do
- [ ] No magic numbers — use named constants
- [ ] Error handling is explicit
- [ ] No commented-out code
- [ ] Tests cover happy path and edge cases
- [ ] Documentation for public APIs

### Feedback Style

- Be specific: "This loop could be O(n²) with large datasets" not "This is slow"
- Suggest alternatives: "Consider using a dict for O(1) lookup"
- Ask questions: "What happens if `user_id` is None?"

## Testing Patterns

### Test Structure (AAA)
```python
def test_process_order():
    # Arrange
    order = Order(id=1, amount=100)
    
    # Act
    result = process_order(order)
    
    # Assert
    assert result.status == "completed"
```

### What to Test

- **Happy path** — Normal expected behavior
- **Edge cases** — Empty inputs, nulls, boundaries
- **Error cases** — Invalid inputs, failures
- **Integration points** — External service interactions (mocked)

### Test Naming
```python
def test_<function>_<scenario>_<expected>():
    ...

# Examples
def test_calculate_total_with_discount_returns_reduced_price():
def test_validate_email_with_invalid_format_raises_error():
```

## Documentation Templates

### Function Docstring
```python
def process_data(items: List[Dict], config: Config) -> Result:
    """Process a list of items according to configuration.
    
    Args:
        items: List of item dictionaries with 'id' and 'value' keys.
        config: Processing configuration.
    
    Returns:
        Result object containing processed items and metadata.
    
    Raises:
        ValidationError: If items contain invalid data.
    """
```

### Module Docstring
```python
"""Data processing utilities for ETL pipelines.

This module provides functions for:
- Data validation
- Transformation
- Output formatting

Example:
    >>> from processing import validate, transform
    >>> data = validate(raw_data)
    >>> result = transform(data)
"""
```

## Error Handling Conventions

### Be Specific
```python
# Bad
except Exception:
    pass

# Good
except ValueError as e:
    logger.warning(f"Invalid value: {e}")
    raise
```

### Fail Fast
```python
def process(data):
    if not data:
        raise ValueError("Data cannot be empty")
    if not isinstance(data, list):
        raise TypeError(f"Expected list, got {type(data)}")
    # ... proceed with valid data
```

### Wrap External Errors
```python
try:
    response = api_client.fetch(url)
except requests.RequestException as e:
    raise DataFetchError(f"Failed to fetch from {url}") from e
```
