## clawinboxrag

> ClawInboxRAG community skill for querying a local Gmail RAG system through natural-language `mail ...` commands. Use when the user asks mailbox retrieval/search/sync via `mail ...` with safe filters and concise output. Do not use it for sending mail or modifying mailbox state.


# ClawInboxRAG (Community Skill)

> Parse and execute `mail ...` commands against a local Gmail RAG installation with safe defaults, bounded output, and a read-only posture.

## Overview

This skill provides a portable command language for mailbox retrieval workflows backed by a local Gmail RAG stack.

Supported parser actions:

- Search commands: `mail <query> ...`
- Operational commands: `mail help`, `mail status`, `mail sync`
- Parser-recognized passthrough commands: `mail labels`, `mail recents`
- Filters: `after`, `before`, `between ... and ...`, `label`, `tag`
- Output hint: `resume`

Design constraints:

- Keep Gmail access read-only
- Return short excerpts, not full raw message bodies
- Avoid hardcoded local paths or user-specific assumptions

## Prerequisites

- A working local Gmail RAG checkout referenced by `GMAIL_RAG_REPO`
- A Python runner such as `uv`
- A valid Gmail OAuth token with read-only scope
- Local data under `GMAIL_RAG_BASE` for search/status operations
- A FAISS index if semantic or hybrid search is expected

## OAuth and local setup

The skill expects an existing local Gmail OAuth token with read-only scope.

Setup points to document clearly:

1. Enable the Gmail API in the Google Cloud project used for the local backend.
2. Configure OAuth consent and create client credentials for the local workflow.
3. Authorize only read-only Gmail scope.
4. Store the token at the path expected by the backend, usually via `GMAIL_TOKEN_PATH`.
5. Verify with a read-only command such as `mail status` or a small search.

If a valid local token already exists, the agent should use it rather than re-running setup.

## Configuration

Environment variables used by this skill and wrapper:

- `GMAIL_RAG_REPO` - absolute path to the backend checkout
- `GMAIL_RAG_UV_BIN` - runner command, default `uv`
- `GMAIL_RAG_BASE` - local data directory, default `~/.openclaw/gmail-rag`
- `GMAIL_TOKEN_PATH` - Gmail OAuth token path, default `~/.openclaw/gmail/token.json`
- `MAIL_DEFAULT_MODE` - `hybrid`, `keyword`, or `semantic`
- `MAIL_DEFAULT_LIMIT` - default top-N result count, default `5`
- `MAIL_MAX_LIMIT` - hard max top-N result count, default `25`

## Instructions

### Step 1: Parse commands safely

- Trigger only when the input starts with `mail` (case-insensitive).
- Parse actions and options with `scripts/parse_mail.py`.
- Return help guidance when the command is empty or malformed.

Accepted date formats:

- `YYYY`
- `MM/YYYY`
- `YYYY-MM`
- `YYYY-MM-DD`

Recognized aliases:

- Modes: `keyword`, `fts`, `semantic`, `semantique`, `sémantique`, `hybrid`, `mix`, `fusion`
- Limit: `max`, `top`, `limit`, `limite`
- Label filter: `label`, `tag`
- Summary hint: `resume`, `résume`, `résumé`, `summary`

### Step 2: Map parser actions to wrapper/backend commands

Use `scripts/run_cli.sh` as the command wrapper.

Action map:

- `help` -> print usage examples
- `status` -> wrapper status command
- `search` -> `search <query> --limit N [--label <label>] [--after <iso>] [--before <iso>]` plus:
  - no extra flag for keyword mode
  - `--semantic` for semantic mode
  - `--hybrid` for hybrid mode
- `sync` -> run in order:
  1. `ingest-primary --limit 50`
  2. `embed --limit 200`
  3. `refresh-labels`
- `labels` and `recents` -> only document them as passthrough operations when the configured backend actually exposes those subcommands

### Step 3: Handle degraded modes and errors

- If semantic or hybrid search is requested but the FAISS index is missing, explain the next step (`mail sync` or `embed`) and optionally fall back to keyword mode.
- If `GMAIL_RAG_REPO`, runner, token, or local data are missing, return setup-oriented diagnostics.
- For CLI failures, explain what failed and which local prerequisite to check.

### Step 4: Format responses for chat channels

For search:

- Return concise numbered or clearly separated results
- Include `date | from | subject`
- Include a stable Gmail permalink when available
- Keep excerpts short
- If `resume` is requested, add a brief per-result summary

For operations:

- Keep output human-readable and compact
- Avoid raw machine dumps unless the user asks for them

## Examples

### Example 1: Default search

User: `mail conference`

Expected behavior:

- Parse as search with default mode and default limit
- Execute the wrapper-backed search
- Return concise results with references when available

### Example 2: Date and label filters

User: `mail conference March label business/external between 12/2025 and 02/2026 max 10 resume`

Expected behavior:

- Convert `between` into ISO `after` and exclusive `before` bounds
- Apply the label prefix filter
- Return up to 10 concise results

### Example 3: Maintenance

User: `mail sync`

Expected behavior:

- Run the conservative maintenance sequence
- Return a compact summary of each step

## Troubleshooting

- `GMAIL_RAG_REPO` missing or invalid: verify the backend checkout path.
- Runner missing: install `uv` or point `GMAIL_RAG_UV_BIN` to the correct executable.
- Token/auth failure: verify the local token exists and uses Gmail read-only scope.
- Semantic/hybrid failure: build or refresh embeddings/index before retrying.

## Security Considerations

- Require Gmail read-only scope for all mailbox operations.
- Never print tokens, secrets, or full sensitive message bodies.
- Clamp result counts and validate date input before execution.
- Pass user input as separate arguments; do not interpolate raw shell strings.

## References

- Setup guide: `references/setup.md`
- Command behavior: `references/commands.md`
- Security guidance: `references/security.md`
- Troubleshooting: `references/troubleshooting.md`

---
> Source: [dmoraine/ClawInboxRAG](https://github.com/dmoraine/ClawInboxRAG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
