---
language: bash
type: command
tags:
  - snippet
  - bash
  - linux
  - files
status: tested
---

# Bash - Find Large Files

## Purpose
List the 20 largest regular files under a directory.

## Code

```bash
find "$HOME" -type f -printf '%s\\t%p\\n' 2>/dev/null |
  sort -nr |
  head -n 20 |
  numfmt --field=1 --to=iec
```

## Usage
Run in a terminal. Replace `"$HOME"` with a directory you want to inspect.

## Dependencies
- GNU/Linux
- GNU `find`, `sort`, `head`, and `numfmt`

## Notes
- Permission-denied messages are suppressed.
- Large trees may take time to scan.
- This command only lists files; it does not delete anything.
