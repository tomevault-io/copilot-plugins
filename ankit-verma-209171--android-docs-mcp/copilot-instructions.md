## android-docs-mcp

> - This repository contains `android-docs-mcp`, a Bun/Node-based MCP server that fetches and parses Android documentation from `developer.android.com`.

# AGENTS.md

## Project Overview
- This repository contains `android-docs-mcp`, a Bun/Node-based MCP server that fetches and parses Android documentation from `developer.android.com`.
- Runtime entrypoint: `src/index.ts`.
- Tool implementations live in `src/tools/`.
- Shared utilities live in `src/lib/`.

## Working Agreements
- Keep changes focused and minimal; avoid unrelated refactors.
- Preserve the existing ESM TypeScript style and naming patterns already used in `src/`.
- Prefer small composable helpers in `src/lib/` over duplicating logic across tools.
- Do not add new dependencies unless clearly necessary.
- Update `README.md` when changing tool behavior, configuration, setup, or public usage.

## Source Layout
- `src/index.ts`: MCP server registration and bootstrap.
- `src/tools/`: public MCP tools. Keep each file centered on one tool.
- `src/lib/cache.ts`: cache behavior.
- `src/lib/fetcher.ts`: HTTP fetching, retries, and cache integration.
- `src/lib/parser.ts`: HTML-to-Markdown conversion.
- `src/lib/url-resolver.ts`: Android/API URL resolution.
- `.vscode/mcp.json`: local MCP client config.

## Build And Validation
- Install dependencies with `bun install`.
- Run local development with `bun run dev`.
- Build distributable output with `bun run build`.
- When behavior changes, validate at minimum with `bun run build` before finalizing.

## Implementation Notes
- Keep tool names, parameter names, and response shapes stable unless the task explicitly requires a breaking change.
- Prefer deterministic parsing and normalization logic; avoid brittle string matching when a structured DOM approach is available.
- Reuse existing fetch/cache/parser utilities before introducing new plumbing.
- Keep network-facing behavior respectful of upstream docs structure and existing retry/caching patterns.

## Documentation Expectations
- If you add a new tool, document it in `README.md` with parameters and usage.
- If you change environment variables, update the configuration table in `README.md`.
- If you change package scripts or startup flow, keep `README.md` and `.vscode/mcp.json` aligned.

## Output Hygiene
- Do not commit generated output from `dist/`, package tarballs, coverage, or dependency directories.
- Keep `.gitignore` updated if new generated artifacts are introduced.

---
> Source: [ankit-verma-209171/android-docs-mcp](https://github.com/ankit-verma-209171/android-docs-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
