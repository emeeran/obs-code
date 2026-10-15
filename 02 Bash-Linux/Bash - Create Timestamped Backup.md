---
language: bash
type: script
tags:
  - snippet
  - bash
  - linux
  - backup
status: tested
---

# Bash - Create Timestamped Backup

## Purpose
Create a timestamped compressed archive of a directory.

## Code

```bash
#!/usr/bin/env bash
set -euo pipefail

source_dir="${1:?Usage: backup.sh SOURCE_DIR [DEST_DIR]}"
dest_dir="${2:-$HOME/Backups}"

if [[ ! -d "$source_dir" ]]; then
  printf 'Source directory does not exist: %s\\n' "$source_dir" >&2
  exit 1
fi

mkdir -p "$dest_dir"
timestamp="$(date +%Y%m%d_%H%M%S)"
name="$(basename -- "$source_dir")"

tar -czf "$dest_dir/${name}_${timestamp}.tar.gz" \
  -C "$(dirname -- "$source_dir")" "$name"

printf 'Backup created: %s\\n' "$dest_dir/${name}_${timestamp}.tar.gz"
```

## Usage

```bash
chmod +x backup.sh
./backup.sh "$HOME/Documents/Project" "$HOME/Backups"
```

## Dependencies
- Bash
- GNU/Linux utilities and `tar`

## Notes
- This creates a new archive; it does not delete old backups.
- Confirm that the destination has enough free space.
