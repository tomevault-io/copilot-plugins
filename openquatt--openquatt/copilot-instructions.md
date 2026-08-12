## openquatt

> - `configs/`: ESPHome target entrypoints for hardware/topology/connection combinations.

# OpenQuatt Codex Instructions

## Repo Map

- `configs/`: ESPHome target entrypoints for hardware/topology/connection combinations.
- `openquatt/`: shared ESPHome packages, profiles, topology, connection and webserver YAML.
- `components/`: custom ESPHome Python/C++ components.
- `openquatt/web/`: embedded web UI source and generated bundles.
- `docs/`: user docs, dashboard/install pages and generated site inputs.
- `scripts/`: local validation, release, docs and build tooling.

## Editing Rules

- Do not make functional code changes unless the user requested them for the task.
- Do not perform broad refactors, architecture changes, or cleanup passes unless explicitly asked.
- Do not make formatting-only changes.
- Preserve existing style, naming, YAML structure, and generated-file conventions.
- Keep edits inside the smallest relevant module, package, component, or document.
- Treat `secrets.yaml` and local reference material as sensitive/local; do not quote values.

## Avoid Reading

- `.git/`, `.esphome/`, `.pio/`, `node_modules/`, `.venv*/`, `.cache/`, `.tmp/`, `tmp/`, `output/`, `__pycache__/`.
- Generated web bundles: `openquatt/web/js/openquatt-app.js` and `openquatt/web/css/openquatt-app.css`.
- Large logs, recordings, debug dumps, firmware binaries, release exports, and local references unless directly requested.

## Tests And Builds

- Run the smallest check that matches the change.
- For firmware/YAML changes, prefer `python3 scripts/dev.py validate --config-only --config <target.yaml>`.
- Run ESPHome compile or full `scripts/dev.py validate` only when explicitly requested or clearly necessary.
- For web source changes, run `npm run build:web`.
- For docs/tooling changes, prefer the specific script touched by the task.
- Summarize build/test output by command, pass/fail, and the few relevant error lines.

## GitHub And PRs

- Do not prefix PR titles with `[codex]`; use a concise human-readable title instead.
- Use `.github/pull_request_template.md` for PR bodies and fill every relevant section.
- In PRs, issues and review comments, write canonical project commands without personal shell wrappers.

---
> Source: [OpenQuatt/OpenQuatt](https://github.com/OpenQuatt/OpenQuatt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
