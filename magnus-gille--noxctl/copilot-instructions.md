## noxctl

> CLI and MCP server for Fortnox accounting (invoices, customers, bookkeeping, VAT).

# CLAUDE.md — noxctl

## What is this?

CLI and MCP server for Fortnox accounting (invoices, customers, bookkeeping, VAT).

## CLI-first in Claude Code

**Prefer the CLI over MCP tools when running in Claude Code.** The MCP server exists for environments without a shell (Claude Desktop, Web, Mobile).

```bash
# Reading data
noxctl invoices list
noxctl invoices list --output json
noxctl customers get 25

# Invoice PDFs (read-only — uses /preview, does not mark the invoice as sent)
noxctl invoices pdf 28                       # -> invoice-28.pdf
noxctl invoices pdf 28 --file /tmp/f.pdf
noxctl invoices pdf 28 --mark-sent --yes     # ALSO flags it as sent (mutation)

# Writing data (prompts for confirmation on TTY; use --yes to skip)
echo '{"InvoiceRows": [...]}' | noxctl invoices update 28 --input - --yes
noxctl invoices send 28              # prompts: Continue? [y/N]
noxctl invoices send 28 --yes        # skip prompt (non-interactive/scripting)

# Dry run first
noxctl invoices create --customer 25 --input data.json --dry-run
```

## Backlog

See `TODO.md` for the prioritized backlog and instructions for adding new resources.

## Project structure

- `src/operations/` — Fortnox API calls (shared by CLI and MCP)
- `src/tools/` — MCP tool registrations (Zod schemas)
- `src/cli.ts` — Commander CLI definitions
- `src/fortnox-client.ts` — HTTP client with rate limiting and retry
- `src/views.ts` — Column definitions for table output
- `src/formatter.ts` — Table/JSON output formatting
- `api-spec/openapi-fingerprint.json` — opaque per-endpoint/per-schema hashes of the Fortnox spec, for drift detection. The full spec is **not** committed (it's Fortnox's call structure — Developer Agreement cl. 6.1/6.3); it's fetched on demand into the git-ignored `api-spec/openapi.json` cache.
- `scripts/check-api-changes.sh` + `scripts/api-fingerprint.py` — drift checker (`npm run check:api`; used by CI)

## Dev commands

```bash
npm run build       # TypeScript compile
npm test            # Vitest (759 unit tests)
npm run test:live   # Live API tests (needs credentials)
npm run lint        # ESLint
npm run format      # Prettier
```

## Conventions

- All MCP tool descriptions are in Swedish
- CLI commands mirror MCP tools 1:1
- Mutations prompt for confirmation on TTY; require `--yes` when piped (CLI) or `confirm: true` (MCP)
- Both support `--dry-run` / `dryRun` to preview without executing
- For substantive code changes, default to red/green TDD: write the failing test first, confirm it fails, then implement until it passes. Skip for refactors with no behavior change, config tweaks, and trivial fixes.

---
> Source: [Magnus-Gille/noxctl](https://github.com/Magnus-Gille/noxctl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
