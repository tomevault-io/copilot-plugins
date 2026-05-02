## jdocmunch-mcp

> **Version:** 1.9.0 | **Tests:** `pytest tests/ -q` (400 total)

# jdocmunch-mcp

**Version:** 1.9.0 | **Tests:** `pytest tests/ -q` (400 total)

## Purpose
Documentation section indexing for the jMunch suite. Companion to jcodemunch-mcp (which owns code symbols). Do NOT add code/docstring parsing here.

## Supported Formats
`.md/.mdx`, `.rst`, `.adoc`, `.ipynb`, `.html`, `.txt`, `.yaml/.yml` (OpenAPI only), `.json/.jsonc`, `.xml/.svg/.xhtml`, `.tscn/.tres` (Godot scenes/resources)

## Key Modules
- `storage/doc_store.py` — DocIndex, DocStore, detect_changes, incremental_save
- `parser/` — one file per format (markdown, rst, asciidoc, notebook, html, text, openapi, json, xml)
- `tools/` — index_local, index_repo, index_file, get_toc, get_toc_tree, search_sections, get_section, get_sections, list_repos, delete_index, get_broken_links, get_doc_coverage, get_backlinks, get_stale_pages, get_wiki_stats
- `cli/hooks.py` — PreToolUse (Read interceptor) + PostToolUse (auto-reindex) + PreCompact (session snapshot) hook handlers for Claude Code
- `cli/init.py` — `jdocmunch-mcp init` full onboarding: client detection, config patching, CLAUDE.md policy, Cursor/Windsurf rules, hooks, index; `claude-md` subcommand
- `embeddings/` — provider.py (Gemini + OpenAI), cosine_similarity, embed_sections, embed_query

## CLI Subcommands
| Subcommand | Purpose |
|------------|---------|
| `serve` (default) | Run the MCP server (stdio) |
| `init` | One-command onboarding: detect clients, write config, install policy, hooks, index |
| `claude-md` | Print or install the Doc Exploration Policy (`--install global\|project`) |
| `index-local --path <dir>` | Index a local folder (CLI, no MCP session needed) |
| `index-file <path>` | Re-index a single file within an existing index |
| `hook-pretooluse` | PreToolUse hook: intercept Read on large doc files (reads stdin) |
| `hook-posttooluse` | PostToolUse hook: auto-reindex doc files after Edit/Write (reads stdin) |
| `hook-precompact` | PreCompact hook: session snapshot before context compaction (reads stdin) |

## Architecture
- INDEX_VERSION=1; version mismatch triggers full re-index
- O(1) section lookup via `DocIndex.__post_init__` id dict
- `pyyaml>=6.0` required (hard dep)
- Hybrid search (v1.9.0): `search_sections` fuses BM25 + semantic cosine when embeddings exist. `use_embeddings` defaults to `"auto"` (embed when provider configured). `search_sections` params: `semantic` (None/auto, True, False), `semantic_only`, `semantic_weight` (0.0–1.0, default 0.5). `_meta.search_mode` reports `hybrid`/`semantic_only`/`lexical`.
- Embedding providers: GOOGLE_API_KEY (Gemini, text-embedding-004) or OPENAI_API_KEY (text-embedding-3-small); override with JDOCMUNCH_EMBEDDING_PROVIDER env var
- Summarizer providers: ANTHROPIC_API_KEY, GOOGLE_API_KEY, OPENAI_API_KEY, MINIMAX_API_KEY, ZHIPUAI_API_KEY; override with JDOCMUNCH_SUMMARIZER_PROVIDER env var (values: anthropic, gemini, openai, minimax, glm, none)

---
> Source: [jgravelle/jdocmunch-mcp](https://github.com/jgravelle/jdocmunch-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
