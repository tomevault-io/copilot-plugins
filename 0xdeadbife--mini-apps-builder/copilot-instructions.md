## mini-apps-builder

> This repository is a workspace for producing single-file vanilla HTML mini apps.

# Codex Instructions

This repository is a workspace for producing single-file vanilla HTML mini apps.

## Default Behavior

- Read `meta-system-prompt.md` before generating a new app.
- Use `templates/single-file-app.html` as the implementation baseline when useful.
- Use `standards/` for product, data, UI and output rules.
- Put generated user-specific apps in `apps/`.
- Put imported CSV/JSON, reports and local datasets in `data/`.
- Do not commit files from `apps/` or `data/` unless the user explicitly asks.

## Output Contract

Generated apps should:

- Be one complete `.html` file.
- Use HTML, CSS and JavaScript vanilla.
- Open directly in a browser without a server.
- Persist locally when useful.
- Include import/export JSON when useful.
- Prefer client-side solutions and CDN libraries only when they clearly help.
- Be responsive and visually polished.

## Design Defaults

Use `Workbench Calm` unless the user asks for another pack.

Available packs:

- `Workbench Calm`
- `Studio Glass`
- `Ledger Sharp`
- `Notebook Warm`
- `Tokyo Mono`
- `Graphite Hacker`

## Safety

- Treat anything in `data/` and `apps/` as local/private by default.
- Do not expose secrets, private datasets or bug bounty findings in tracked files.
- For network testing, prefer non-invasive requests unless the user clearly authorizes a deeper test and scope is appropriate.

---
> Source: [0xdeadbife/mini-apps-builder](https://github.com/0xdeadbife/mini-apps-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
