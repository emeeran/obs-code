---
language: sql
type: query
tags:
  - snippet
  - sql
  - database
status: example
---

# SQL - Select Recent Records

## Purpose
Retrieve the most recent records from a table.

## Code

```sql
SELECT id, created_at, status
FROM your_table
ORDER BY created_at DESC
LIMIT 20;
```

## Usage
Replace `your_table` and the column names with those from your schema.

## Compatibility
- `LIMIT` is supported by SQLite, PostgreSQL, and MySQL.
- SQL Server uses `TOP` or `OFFSET ... FETCH`, depending on the query.

## Notes
- Add a deterministic tie-breaker, such as `ORDER BY created_at DESC, id DESC`, if timestamps may repeat.
- This is a template; validate it against your database schema.
