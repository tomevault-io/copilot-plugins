## html-slides-editor

> Use this repository when a user wants to manually edit an AI-generated HTML slide deck in the browser preview.

# HTML Slides Editor Agent Guide

Use this repository when a user wants to manually edit an AI-generated HTML slide deck in the browser preview.

Built for Codex, but usable by any AI coding agent or a plain terminal. Codex can load the bundled skill (see [Codex Skill](#codex-skill) below); every other agent should drive the `html-slides-editor` npm CLI directly, as shown next.

## Primary Command

After the npm package is published, prefer:

```bash
npx html-slides-editor enable --autosave --serve path/to/index.html
```

For a local checkout before npm publishing, use:

```bash
node bin/html-slides-editor.js enable --autosave --serve path/to/index.html
```

Use the printed `http://127.0.0.1:<port>/` URL for editing. Do not use `file://` when edits need to persist.

## Common Tasks

Enable editing:

```bash
npx html-slides-editor enable --autosave --serve path/to/index.html
```

Disable editing:

```bash
npx html-slides-editor disable path/to/index.html
```

Check status:

```bash
npx html-slides-editor status path/to/index.html
```

Local checkout equivalents replace `npx html-slides-editor` with `node bin/html-slides-editor.js`.

## Guardrails

- This is not an HTML checker.
- Do not build or require a Chrome extension.
- Do not rewrite slide content unless the user asks.
- Preserve the user's HTML as an ordinary local file.
- Persistent editing requires the local save server from `--serve` or `serve`.
- If editing is active, normal slide interactions are intentionally paused until the user pauses the editor.

## Codex Skill

Codex can also use the bundled skill:

```text
.codex/skills/html-slides-editor/
```

The skill source of truth is `.codex/skills/html-slides-editor/references/spec.md`.

---
> Source: [imvanessali/HTML-Slides-Editor](https://github.com/imvanessali/HTML-Slides-Editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
