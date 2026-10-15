# Code Library — Obsidian Vault

A ready-to-use Obsidian vault for collecting, testing, organizing, and reusing code snippets.

## Open the vault

1. Download and extract `Code_Library_Obsidian_Vault.zip`.
2. Open Obsidian.
3. Choose **Open folder as vault**.
4. Select the extracted `Code_Library_Obsidian_Vault` folder.

Your notes are plain Markdown files and can also be edited with other editors.

## Included structure

- `00 Inbox/` — capture snippets before organizing them.
- `01 Python/` — Python functions and patterns.
- `02 Bash-Linux/` — shell commands and Linux notes.
- `03 JavaScript/` — JavaScript examples.
- `04 SQL/` — SQL query patterns.
- `05 HTML-CSS/` — web snippets.
- `06 Tools-APIs/` — Git, HTTP, CLI and API notes.
- `07 Templates/` — reusable Obsidian note templates.
- `08 Archive/` — retired or superseded snippets.
- `Code Library Dashboard.md` — home page.

## Configure templates

The vault includes the `07 Templates/Snippet Template.md` file.

1. In Obsidian, open **Settings → Core plugins** and enable **Templates**.
2. Open **Settings → Templates**.
3. Set **Template folder location** to `07 Templates`.
4. Use the Command Palette (`Ctrl+P`) and choose **Templates: Insert template**.

## Snippet workflow

1. Capture new code in `00 Inbox`.
2. Add a descriptive title, purpose, usage example, dependencies, and source URL.
3. Test the code in an appropriate environment before marking it `tested`.
4. Move it to the appropriate language or tool folder.
5. Apply consistent tags and link related notes.

## Naming convention

Use descriptive names such as:
- `Python - Read JSON File.md`
- `Bash - Find Large Files.md`
- `Git - Undo Last Commit.md`

## Search examples

Find Python snippets tagged for file handling:

    tag:#python tag:#files

Find snippets that still need testing:

    tag:#untested

Find API-related notes:

    tag:#api

## Backup

The vault is just a folder of files. Back it up to an external drive or another independent location. Never commit passwords, API keys, tokens, or private data to Git.
