---
language: python
type: function
tags:
  - snippet
  - python
  - files
status: tested
---

# Python - Safe File Write

## Purpose
Write text to a file using UTF-8, creating parent directories when needed.

## Code

```python
from pathlib import Path

def write_text(path: str | Path, content: str) -> None:
    \"\"\"Write text to a file, creating parent directories if needed.\"\"\"
    target = Path(path)
    target.parent.mkdir(parents=True, exist_ok=True)
    target.write_text(content, encoding="utf-8")
```

## Usage

```python
write_text("output/reports/summary.txt", "Report complete.\\n")
```

## Dependencies
- Python 3.10+
- Standard library only

## Notes
- Overwrites an existing file. Use a temporary file and atomic replacement if you need stronger protection against interrupted writes.
- Raises normal filesystem exceptions such as `PermissionError`.

## Source
- https://docs.python.org/3/library/pathlib.html
