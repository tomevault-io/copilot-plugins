## mdvault

> Hybrid BM25 + vector search over local Markdown vaults. SQLite-backed (FTS5 + sqlite-vec).

# mdvault

Hybrid BM25 + vector search over local Markdown vaults. SQLite-backed (FTS5 + sqlite-vec).

## Architecture

```
mdvault/
  cli.py        — Typer CLI (10 commands, all support --json)
  db.py         — SQLite schema, connection, serialize_f32
  indexer.py    — Markdown chunking, file indexing, link extraction
  retriever.py  — BM25 search, vector search, RRF fusion, re-ranking
  memory.py     — Store/delete memories (chunked + embedded like files)
  mcp_server.py — MCP server (FastMCP) for Claude Code integration
  vault.py      — Vault facade class (used by MCP server)
```

## Key concepts

- **Vault**: a directory of .md files, indexed under `{vault_name}/{relative_path}`
- **Chunks**: files split by `##` headings, embedded with model2vec (`minishlab/potion-base-8M`)
- **Hybrid search**: BM25 (FTS5) + cosine vector similarity, fused via RRF (k=15), then multi-signal re-ranking
- **Memories**: stored like files under `memory:///{id}`, searchable alongside vault content
- **DB location**: `--db` flag > `VAULT_DB` env > `platformdirs.user_data_dir("mdvault")/vault.db`

## CLI commands

```
mdvault index <path> [--full] [--no-gitignore] [--json]
mdvault search <query> [--top-k N] [--vault NAME] [--source files|memories] [--paths-only] [--no-truncate] [--expand] [--json]
mdvault stats [--json]
mdvault list [--vault NAME] [--pattern GLOB] [--json]
mdvault read <file_path> [--json]
mdvault related <file_path> [--top-k N] [--json]
mdvault remember <content> [--namespace NS] [--source SRC] [--json]
mdvault forget [--id ID] [--namespace NS] [--json]
mdvault memories [--namespace NS] [--json]
mdvault serve [--db PATH]
```

## Dev

- Python >=3.11, managed with `uv`
- Tests: `uv run pytest -q --tb=short`
- Lint: ruff (pre-commit hook)
- Global install: `uv tool install --force --editable .` (exposes `mdvault` command)
- Run without install: `uv run mdvault ...`

## Testing patterns

- CLI tests use `typer.testing.CliRunner` with `runner.invoke(app, [...])`
- Fixtures in `tests/fixtures/` (small markdown vault)
- Each test creates its own tmp DB via `tmp_path`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sderosiaux) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
