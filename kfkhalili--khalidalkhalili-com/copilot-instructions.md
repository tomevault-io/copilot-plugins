## do-not-edit-plugin-managed

> Never change Digital Garden plugin–managed content and generated files


# Do Not Edit Plugin-Managed Paths

**Never create, edit, or delete files in these paths.** They are owned by the [Obsidian Digital Garden](https://github.com/oleeskild/obsidian-digital-garden) plugin or generated at build time. See [dg-docs.ole.dev](https://dg-docs.ole.dev).

## 1. Note content — `src/site/notes/`

- All note files (`.md`, `.canvas`) and the notes index (`notes.json`, `notes.11tydata.js`) are published or maintained by the plugin from your Obsidian vault.
- **Do not** create, edit, or delete anything under `src/site/notes/`.
- To change note content: edit in Obsidian and publish again. To change how notes are rendered: change layouts, includes, or helpers outside `src/site/notes/`.

## 2. Generated theme CSS — `src/site/styles/_theme.*.css`

- These files are **generated** by `get-theme.js` from the `THEME` URL (e.g. in `.env`) and are overwritten on each build.
- **Do not** edit `_theme.*.css`. Customize appearance via:
  - [CSS Customization](https://dg-docs.ole.dev/advanced/css-customization/): `src/site/styles/custom-style.scss` or `src/site/styles/user/*.scss`
  - CSS variables listed in the main README

## 3. Plugin config (edit with care)

- **`.env`** — Used for `THEME` and other build/deploy settings. Avoid changing keys the plugin or `get-theme.js` rely on; add new keys only when needed for your setup.

---

When a task would require editing a plugin-managed path, suggest editing in Obsidian, changing template/layout code elsewhere, or overriding styles in `custom-style.scss` / `src/site/styles/user/` instead.

---
> Source: [kfkhalili/khalidalkhalili.com](https://github.com/kfkhalili/khalidalkhalili.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
