## notion-cli

> Notion API CLI for AI agents — search, page, db, block, comment, user operations with JSON output


# notion-cli

Notion API CLI that outputs JSON. Built for AI agents.

## Install

```bash
pip install git+https://github.com/tellang/notion-cli.git
export NOTION_TOKEN=ntn_...
```

## Commands

### Search
```bash
notion search "keyword" --type db --limit 5 --fields "id,title"
```

### Page
```bash
notion page get <id> --props "Name,Score" --fields "id,Name,Score"
notion page create --title "Title" --parent-db <id> --body "text" --dry-run
notion page update <id> --props-json '{"Status": {"select": {"name": "Done"}}}' --dry-run
```

### Database
```bash
notion db schema <id>
notion db query <id> --filter '{"property":"Status","select":{"equals":"Done"}}' --sort "Date:descending" --limit 10 --fields "id,Name,Date"
```

### Block
```bash
notion block list <page-id> --limit 20 --fields "id,type,text"
notion block get <block-id>
notion block append <page-id> --body "new paragraph" --dry-run
notion block delete <block-id> --dry-run
```

### Comment
```bash
notion comment list <page-id> --limit 10
notion comment create --page-id <id> --body "comment text" --dry-run
```

### User
```bash
notion user list
notion user me
```

### Schema (self-introspection)
```bash
notion schema              # list all commands
notion schema "page create" # show parameters for specific command
```

## Agent Rules

1. **Always --dry-run first** for write operations (create, update, append, delete)
2. **Always --fields** on list/query to save tokens (e.g. `--fields "id,Name,Score"`)
3. **Use `notion schema`** to discover parameters instead of guessing
4. IDs are validated — path traversal and control characters are rejected
5. Errors are JSON: `{"error": {"code": "...", "message": "..."}}`
6. Use data_source ID from `notion search` for `db query` and `db schema`
7. `--props` filters Notion properties, `--fields` filters output JSON keys

---
> Source: [tellang/notion-cli](https://github.com/tellang/notion-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
