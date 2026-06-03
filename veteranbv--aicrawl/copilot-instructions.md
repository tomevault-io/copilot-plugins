## aicrawl

> Project guidance for Codex and other coding agents working in `aicrawl`.

# AGENTS.md

Project guidance for Codex and other coding agents working in `aicrawl`.

## Purpose

`aicrawl` is a local-first OpenClaw-style archive for personal AI conversation history. The v0.1 product is a high-integrity importer and search/export CLI for official Claude and ChatGPT data exports.

Optimize for durable, private, verifiable preservation of user data. Do not optimize for convenience at the cost of correctness, privacy, maintainability, or provider terms.

## Public-repo hygiene

This repository is intended to be public.

Do not commit private planning artifacts, Codex prompts, raw exports, local source data, archive databases, generated snapshots, logs, caches, credentials, or machine-local configuration. In particular, keep these out of git:

- `docs/IMPLEMENTATION_PLAN.md`
- `docs/CODEX_GOAL_PROMPT.md`
- Claude or ChatGPT export ZIPs
- extracted export files such as `conversations.json` or `chat.html`
- Claude Code or other local transcript JSONL files
- SQLite databases and WAL/SHM files
- local `.aicrawl/`, `.crawlbar/`, `.crabbox/`, export, import, raw-data, backup, and snapshot directories

If a private file is already tracked, `.gitignore` is not enough. Remove it from the index with `git rm --cached <path>` and verify with `git status` before committing.

Synthetic fixtures are allowed only when they are small, clearly fake or redacted, and intentionally named as fixtures, for example `testdata/redacted/*.fixture.json`. Never commit real user conversation text, account identifiers, email addresses, attachment contents, API tokens, session tokens, or provider export files.

## Source of truth

Use the public repo files as the durable source of truth: code, tests, `README.md`, committed public docs, and this `AGENTS.md`.

Private local planning files may exist while bootstrapping the project. They are context, not public project artifacts. Do not require them for builds or tests, and do not commit them.

When docs and code disagree, inspect the current code and update the smallest necessary surface. Do not silently invent missing architecture. If a documented requirement is wrong or stale, explain the evidence and make the smallest correction needed.

## OpenClaw / crawlkit boundary

Use `github.com/openclaw/crawlkit` for reusable local archive mechanics where it naturally fits: config paths, SQLite helpers, snapshots, mirrors, sync state, output helpers, progress/status/control payloads, terminal browsing, and safe local cache reads.

Keep provider-specific behavior in `aicrawl`, not in `crawlkit`: Claude and ChatGPT schemas, export parsing, source detection, privacy filtering, attachments, account/workspace quirks, user-facing import behavior, and app-specific database schema.

If a change to `crawlkit` seems necessary, keep the API additive and small. Only move logic upstream when it is clearly reusable across multiple crawl apps. Otherwise keep the downstream app change narrow.

## Non-negotiable product constraints

- Use official/local sources only for v0.1: Claude export ZIP/JSON, ChatGPT export ZIP/JSON, and later documented local transcript files.
- Do not implement session-token scraping, private product API calls, browser-console snippets, headless browser export automation, or hidden network sync.
- Preserve raw source payloads alongside parsed normalized columns so parser bugs can be repaired later.
- Re-imports must be idempotent. Duplicate conversations/messages from overlapping exports are bugs.
- Treat ChatGPT `mapping` as a graph. Store every node and explicit edges; never collapse branches/regenerations into a lossy linear transcript.
- Parse Claude exports defensively. Tolerate unknown fields and optional missing metadata, but fail clearly on missing required stable IDs unless a documented fallback is implemented.
- Keep provider-specific parsing downstream under `internal/ingest/*`.
- Protect private data by default: private file permissions, redacted logs, safe ZIP handling, read-only SQL, synthetic/redacted fixtures only, and no imported content sent to cloud services by default.

## Engineering rules

Build clean, robust, maintainable, production-ready code. Keep changes small, coherent, and scoped to the task.

Do:

- Prefer the simplest solution that fully meets the need.
- Use the Go standard library and `crawlkit` before adding dependencies.
- Preserve existing conventions before introducing new ones.
- Add abstractions only after the concrete need is proven.
- Name constants for limits and defaults; explain why values exist.
- Write tests for correctness, edge cases, idempotency, malformed input, and security-sensitive behavior.
- Use temp dirs and temp SQLite files in tests. Never touch live user stores.
- Review your diff like a senior engineer before calling the task done.

Do not:

- Add fake implementations, placeholders, stubs, TODO-driven behavior, hidden technical debt, or unexplained hardcoded values.
- Add dependencies, layers, config files, generated code, background services, or architectural patterns unless clearly justified.
- Broaden scope into embeddings, TUI, watch daemons, enterprise compliance APIs, cloud sync, or mirror/publish flows unless the task explicitly asks for that phase.
- Log message text, attachment contents, account emails, tokens, long raw IDs, or source export filenames unless explicitly required and safely redacted.
- Claim validation you did not run.

## Expected repository shape

Use this layout unless existing code establishes a better local convention:

```text
cmd/aicrawl/main.go
internal/
  app/                 # command wiring and dependency setup
  ingest/
    claudeexport/      # official Claude export ZIP/JSON parser
    chatgptexport/     # official ChatGPT export ZIP/JSON parser
    claudecode/        # later: local JSONL transcript parser
  archive/             # DB writes, import transactions, search queries
  schema/              # migrations and schema helpers
  textnorm/            # UTF-8, whitespace, zero-width cleanup, FTS quoting
  security/            # ZIP safety, permissions, redaction
  control/             # aicrawl-specific control payload mapping
testdata/
  redacted/            # synthetic or redacted fixtures only
README.md
LICENSE
AGENTS.md
.gitignore
```

Provider adapters should parse source formats. Archive/storage code should own canonical writes, transactions, idempotency, migrations, and query behavior.

## Database and import requirements

- Use SQLite with WAL mode and `PRAGMA user_version` migrations.
- Fail fast when a local DB schema is newer than the binary supports.
- Use deterministic provider-prefixed IDs for conversations and messages; do not expose autoincrement IDs as stable CLI identifiers.
- Keep an import ledger keyed by source kind and file hash.
- Store raw conversation/message/node payloads, preferably compressed where appropriate.
- Normalize only the searchable projection; never mutate raw payloads.
- Use FTS5 with safe user query handling. Parameterize and quote terms so reserved words like `AND`, `OR`, `NOT`, `NEAR`, and `*` are searchable terms, not accidental operators.
- Store ChatGPT edges in a dedicated edge representation, not only a single `parent_id` column.
- Treat null messages, malformed timestamps, multi-root graphs, missing roots, unknown roles, unknown content types, and attachments/files arrays as expected edge cases.

## Security and privacy requirements

- Create config, cache, and data directories with private permissions.
- Reject unsafe ZIP entries: absolute paths, `..` traversal, symlinks unless explicitly supported, excessive entry counts, and excessive uncompressed size.
- Keep limits named and documented.
- Open SQL command paths read-only, or reject anything except safe read-only statements.
- Never commit real Claude or ChatGPT exports. Fixtures must be synthetic or redacted.
- Do not auto-delete source ZIPs after import; remind users that export files contain private data.
- Do not make network calls for import, search, SQL, or markdown export.

## CLI expectations for v0.1

Required commands:

```text
aicrawl version
aicrawl init
aicrawl doctor [--json]
aicrawl metadata [--json]
aicrawl status [--json]
aicrawl import <zip-or-json> [--provider claude|chatgpt|auto]
aicrawl conversations [--provider claude|chatgpt] [--limit 50]
aicrawl messages --conversation <id> [--path current|all]
aicrawl search <query> [--provider claude|chatgpt|all] [--limit 25]
aicrawl sql <readonly-sql>
aicrawl export markdown --out <dir> [--provider claude|chatgpt|all]
aicrawl crawlbar manifest --out ~/.crawlbar/apps/aicrawl.json
```

`metadata --json`, `status --json`, and `doctor --json` are control surfaces for automation and CrawlBar compatibility. They must be valid JSON, read-only, and safe to run from an empty temp home.

Defer these until after v0.1 unless explicitly requested: `sync`, `watch`, `tui`, `snapshot`, `embed`, `backup`, `mirror`, `publish`, `subscribe`, and enterprise compliance ingestion.

## Validation

Use temp directories for tests and smoke runs. Never mutate live user archives or depend on private local files.

Run before handoff when Go code exists:

```bash
GOWORK=off go mod tidy
git diff --exit-code -- go.mod go.sum
GOWORK=off go vet ./...
GOWORK=off go test -count=1 ./...
```

For temp-home smoke testing:

```bash
TMP_HOME="$(mktemp -d)"
export HOME="$TMP_HOME"
export XDG_CONFIG_HOME="$TMP_HOME/.config"
export XDG_CACHE_HOME="$TMP_HOME/.cache"
export XDG_DATA_HOME="$TMP_HOME/.local/share"

go run ./cmd/aicrawl --help
go run ./cmd/aicrawl version
go run ./cmd/aicrawl init
go run ./cmd/aicrawl doctor --json
go run ./cmd/aicrawl metadata --json
go run ./cmd/aicrawl status --json
```

After redacted fixtures exist, also smoke:

```bash
go run ./cmd/aicrawl import ./testdata/redacted/claude-export.fixture.zip --provider claude
go run ./cmd/aicrawl import ./testdata/redacted/chatgpt-export.fixture.zip --provider chatgpt
go run ./cmd/aicrawl search "known fixture phrase"
go run ./cmd/aicrawl sql "select count(*) from messages"
go run ./cmd/aicrawl export markdown --out "$TMP_HOME/exported-md"
```

Report any command that could not be run and why. Do not imply success from code inspection alone.

## Definition of done

A task is done only when:

- The implementation matches the requested scope and project docs.
- Relevant tests and smoke checks pass, or failures/blockers are reported precisely.
- Edge cases and regressions have been considered.
- Private data is not leaked in logs, fixtures, errors, generated files, or docs.
- The diff does not add needless abstractions, dependencies, config, or scope.
- User-facing docs are updated only for behavior that actually exists.
- Any uncertainty that affects correctness, architecture, data safety, security, user experience, or maintainability is called out clearly.

## Blocked-work protocol

If blocked by missing export examples, crawlkit API drift, CrawlBar manifest uncertainty, schema ambiguity, or provider format changes:

1. Stop before making speculative changes that could damage correctness.
2. State the exact blocker.
3. Describe what was tried.
4. Include the evidence observed.
5. Name the smallest user input, upstream clarification, or fixture needed to proceed.

For low-risk uncertainty, make the most reasonable assumption, state it in the handoff, and keep the implementation easy to revise.

---
> Source: [veteranbv/aicrawl](https://github.com/veteranbv/aicrawl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
