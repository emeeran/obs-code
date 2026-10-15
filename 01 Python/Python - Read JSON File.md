---
language: python
type: function
tags:
  - snippet
  - python
  - json
  - files
status: tested
---

# Python - Read JSON File

## Purpose
Read a UTF-8 JSON file and return the decoded Python value.

## Code

```python
import json
from pathlib import Path
from typing import Any

def read_json(path: str | Path) -> Any:
    \"\"\"Read JSON data from a UTF-8 file.\"\"\"
    with Path(path).open("r", encoding="utf-8") as file:
        return json.load(file)
```

## Usage

```python
config = read_json("config.json")
print(config)
```

## Dependencies
- Python 3.10+
- Standard library only

## Notes
- Raises `FileNotFoundError` if the file does not exist.
- Raises `json.JSONDecodeError` for malformed JSON.
- Returns whatever JSON value is at the top level (object, list, string, number, boolean, or null).

## Source
- https://docs.python.org/3/library/json.html
