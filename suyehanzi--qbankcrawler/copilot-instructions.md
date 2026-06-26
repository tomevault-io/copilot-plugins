## qbankcrawler

> This workspace is a configurable Playwright export tool for authenticated question-bank style workflows. Keep it public-safe: code and templates may be shared, but local browser state, network logs, exported content, and credentials must stay private.

# AGENTS.md

## Purpose

This workspace is a configurable Playwright export tool for authenticated question-bank style workflows. Keep it public-safe: code and templates may be shared, but local browser state, network logs, exported content, and credentials must stay private.

## Key Scripts

- `npm run open`
  - Launches a visible Playwright browser with persistent `.pw-profile` state.
  - Captures selected requests and responses into `artifacts/network.ndjson`.
- `npm run export:auto`
  - Reads the latest session log and infers the current bank size.
  - Runs a full export from question `1` to the detected total count.
- `npm run fetch:first5:word`
  - Exports a manually specified range using `FETCH_START` and `FETCH_COUNT`.

## Configuration

Use `.env` for local target settings. Do not commit `.env`.

Public template:

- `.env.example`

Expected settings:

- `TARGET_URL`
- `API_ORIGIN`
- `PRACTICE_PATH`
- `START_PLAN_ENDPOINT`
- `ROUTE_PARAMS_ENDPOINT`
- `QUESTION_TYPE_ENDPOINT`
- `QUESTION_DETAIL_ENDPOINT`

## Workflow Expectations

1. Start the browser with `npm run open`.
2. Let the user navigate to an authorized target bank.
3. Make sure the page is in the intended practice or review mode before formal export.
4. Run `npm run export:auto` for the whole bank, or `npm run fetch:first5:word` for a custom range.
5. Move finalized outputs from `artifacts/` into `archives/` only when the user asks for local archiving.

## File Handling Rules

- Treat `artifacts/` as disposable local output.
- Do not clear `artifacts/` casually, because it may contain the latest `network.ndjson` needed for export inference.
- Treat `.pw-profile/` as persistent local browser state.
- Keep local archive folders under `archives/YYYY-MM-DD/`.
- Never commit `network.ndjson`, exported JSON/DOCX files, cookies, tokens, browser profiles, or credentials.

## Export Behavior

- One fetch can produce both:
  - `with-answers`
  - `no-answers`
- Word exports should exist for both variants.
- If the target Word file is open, the exporter falls back to a timestamped filename.

## Documentation Maintenance

- Update `README.md` when commands, workflow, or output locations change.
- Update this file when workspace conventions or safety rules change.

---
> Source: [suyehanzi/QBankCrawler](https://github.com/suyehanzi/QBankCrawler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
