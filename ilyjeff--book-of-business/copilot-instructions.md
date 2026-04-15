## book-of-business

> Avoid OS menu bar and Electron app menus; use in-app UI only


# No system menus for app features

- Do **not** add or grow **Electron `Menu`** templates (File, Edit, View, Window, Help) for navigation, settings, data actions, or other product features. Those belong **inside the renderer** (sidebar, toolbar, modals, custom dropdowns, sheets).
- Keep shell menus **minimal**: see `electron/main/index.ts` (`installApplicationMenu`). On Windows and Linux the application menu bar is off; on macOS only a tiny app menu remains (OS expectation), not a full desktop menu tree.
- Do **not** tell users to use the top menu bar for core workflows. If something needs a shortcut, prefer in-app affordances or document optional accelerators without relying on visible menu clutter.
- **Fine to use**: text inputs, checkboxes, date fields. For **email/phone label presets**, use the in-app **`ChannelLabelMenu`** pattern (like `DepartmentMenu`), not a native **`<select>`**.

When adding new capabilities, default to the same patterns as the rest of the shell: **LOCAL LEDGER** sidebar, detail panes, and Settings in-app.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ILYJEFF)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ILYJEFF)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
