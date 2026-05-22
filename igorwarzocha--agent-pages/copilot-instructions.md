## agent-pages

> - `src/style.css` is only for the Agent Pages app shell: global tokens, layout, sidebar, landing page, and generic reusable shell classes.

# Agent Pages repository rules

- `src/style.css` is only for the Agent Pages app shell: global tokens, layout, sidebar, landing page, and generic reusable shell classes.
- Artifact-specific CSS must live inside the artifact TSX file itself, normally with a local `<style>{`...`}</style>` tag or inline styles.
- Do not modify app code while adding artifacts unless the user explicitly asks to change Agent Pages itself.
- Artifacts must remain single self-contained TSX files under `src/pages`.
- Avoid horizontal page scrolling in artifacts unless absolutely necessary; if wide content is required, scope `overflow-x: auto` to that element.
- For customising or extending Agent Pages itself, start with `docs/change-map.md`. Do not use `docs/` for normal artifact creation unless the artifact needs a new dependency; in that case, ask the user before customising the repo.

---
> Source: [IgorWarzocha/agent-pages](https://github.com/IgorWarzocha/agent-pages) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
