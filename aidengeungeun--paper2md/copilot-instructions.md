## paper2md

> `paper2md` is an agent-first local arXiv-to-Markdown CLI: one arXiv paper in, one JSON receipt out, one Markdown artifact on disk. Paper content never goes to stdout.

# AGENTS.md - paper2md

## Overview

`paper2md` is an agent-first local arXiv-to-Markdown CLI: one arXiv paper in, one JSON receipt out, one Markdown artifact on disk. Paper content never goes to stdout.

This tool is intentionally narrow. It accepts modern arXiv IDs and arXiv `/abs/` or `/pdf/` URLs, fetches arXiv resources directly, and runs a local Rust engine. It must not call Hugging Face, `markxiv.org`, or any hosted paper reader as a backend.

## Commands

```bash
npm run build
npm run typecheck
npm test
paper2md read 1706.03762
paper2md read https://arxiv.org/pdf/1706.03762.pdf
printf '{"input":"1706.03762"}' | paper2md read -
```

## Dependencies

- Build: Rust/Cargo and Node.js `>=20.19.0`.
- Runtime: `pandoc` and `pdftotext` on `PATH` for fallback paths. HTML-first conversion still currently checks both at startup so dependency failures are explicit.
- `tar` is not a runtime dependency in this implementation because archive extraction is done in Rust.

## Architecture

```text
cli.ts -> lib/app.ts
  1. Parse `paper2md read` argv, JSON argv, or JSON stdin via `-`.
  2. Normalize modern arXiv IDs and arXiv `/abs/` or `/pdf/` URLs.
  3. Create the output directory and same-directory temp Markdown path.
  4. Run `dist/paper2md-engine --id <id> --output <temp>`.
  5. Validate temp Markdown is non-empty and atomically rename it to `<id>.md`.
  6. Print exactly one JSON receipt to stdout; write at most a short meter/error line to stderr.
```

`runCli` accepts an injectable `engineRunner`, `now`, and `randomSuffix` so tests cover the command contract without live arXiv or real conversion tools.

The Rust engine fetches arXiv metadata, tries arXiv HTML first for cleaner citations/figure links, falls back to source archive conversion through `pandoc`, then falls back to PDF text extraction through `pdftotext`. It prints only a small JSON status object for the wrapper to map.

The distributable skill lives at `skills/paper2md/SKILL.md` so GitHub-based skill installers name it `paper2md` by default; the same file is reused across Codex, OpenCode, Cursor, and Claude Code. The Claude Code plugin and self-hosted marketplace manifests live at `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`; the plugin auto-discovers `skills/paper2md/SKILL.md` from the repo root (no duplication). `test/plugin.test.ts` guards manifest shape and keeps `plugin.json` `version` in sync with `package.json`. The README hero asset lives at `assets/paper2md-banner.jpg`. Keep `scripts/install.sh`, package `files`, and README install paths in sync with those locations.

Quality target: useful paper reading, not perfect scholarly typesetting. Preserve citation links, section structure, equations, tables, and figure URLs as well as the available arXiv representation allows; never claim image/graph understanding.

## Output contract

Success receipts include:

```json
{
  "success": true,
  "tool": "paper2md",
  "status": "ok",
  "input": "1706.03762",
  "id": "1706.03762",
  "paperUrl": "https://arxiv.org/abs/1706.03762",
  "outputDir": "/tmp/paper2md-1706.03762-Ab12Cd",
  "artifacts": { "markdown": "/tmp/paper2md-1706.03762-Ab12Cd/1706.03762.md" },
  "warnings": [],
  "elapsedMs": 4210
}
```

Failure receipts are also JSON on stdout and include stable `error.code` values: `INVALID_INPUT`, `UNSUPPORTED_INPUT`, `ARXIV_NOT_FOUND`, `ARXIV_UNAVAILABLE`, `MISSING_DEPENDENCY`, `CONVERSION_FAILED`, `EMPTY_OUTPUT`, `OUTPUT_EXISTS`, `WRITE_FAILED`, `ENGINE_FAILED`, and `INTERNAL_ERROR`.

## Scope boundaries

Do not add VLMs, OCR beyond `pdftotext`, general PDF parsing, arbitrary URL fetching, DOI/search flows, metadata-only commands, MCP, server management, content JSON artifacts, image sidecars, Hugging Face backend calls, or a hosted markxiv wrapper.

## Attribution

The conversion strategy follows the MIT-licensed `tonydavis629/markxiv` project as a reference for local arXiv conversion. This package reimplements the narrow one-shot CLI path and does not vendor markxiv's axum routes, MCP crate, server, cache sweeper, request logging, or public web service behavior.

## Code style

- TypeScript strict mode, ESM, Node16 module resolution.
- 4-space indentation, semicolons, named exports only.
- Keep errors branchable through stable codes and avoid stack traces in normal CLI failures.

---
> Source: [AidenGeunGeun/paper2md](https://github.com/AidenGeunGeun/paper2md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
