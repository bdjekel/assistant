# Python Patterns

## Type Hints

Always use type hints for function signatures:

```python
from typing import Optional, List, Dict, Any

def process_data(
    items: List[Dict[str, Any]],
    config: Optional[Dict[str, str]] = None
) -> List[str]:
    ...
```

## Testing

Use pytest with fixtures:

```python
import pytest

@pytest.fixture
def sample_data():
    return [{"id": 1, "name": "test"}]

def test_process_data(sample_data):
    result = process_data(sample_data)
    assert len(result) > 0
```

## Virtual Environments

```bash
python -m venv .venv
source .venv/bin/activate  # macOS/Linux
pip install -r requirements.txt
```

## Project Structure

```
project/
├── src/
│   └── package/
│       ├── __init__.py
│       └── module.py
├── tests/
│   └── test_module.py
├── pyproject.toml
└── requirements.txt
```

## Error Handling

```python
class DataProcessingError(Exception):
    """Raised when data processing fails."""
    pass

def process(data):
    try:
        # processing logic
    except ValueError as e:
        raise DataProcessingError(f"Invalid data: {e}") from e
```

## Logging

```python
import logging

logger = logging.getLogger(__name__)

def process(data):
    logger.info("Starting processing", extra={"count": len(data)})
    # ...
    logger.debug("Processing complete")
```
