## flip-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A UltiMaker Cura 5.x plugin that adds a one-click UI to copy user-overridden settings from one extruder to another on multi-extruder printers. The plugin lives in its own directory (`flip-app/`) which is installed directly into Cura's `plugins/` folder.

## Installation for development

Symlink or copy this directory into Cura's plugin folder, then restart Cura:
```
# Linux/macOS
ln -s /path/to/flip-app ~/.local/share/cura/5.x/plugins/ExtruderSettingsCopy
# Windows
mklink /D "%APPDATA%\cura\5.x\plugins\ExtruderSettingsCopy" C:\path\to\flip-app
```

The plugin appears under **Extensions → Extruder Tools → Copy Extruder Settings…** after restart.

## Architecture

```
__init__.py                        ← Cura entry point; exports ExtruderSettingsCopyPlugin
ExtruderSettingsCopyPlugin.py      ← Extension subclass; registers menu item, lazily loads QML dialog
SettingsCopyService.py             ← QObject backend exposed to QML via createQmlComponent context
qml/CopyExtruderSettingsDialog.qml ← Three-page wizard UI (select → preview → results)
plugin.json                        ← Cura plugin metadata (name, api version, etc.)
```

### Data flow

1. `ExtruderSettingsCopyPlugin._openDialog()` calls `Application.createQmlComponent()`, injecting `SettingsCopyService` as `settingsCopyService` into QML context.
2. QML calls Python slots synchronously for `getExtruderNames()`, `getCategories()`, and `buildPreview()`.
3. QML calls `copySettings()`, which runs the copy and emits `copyFinished(copied, skipped, failed, issues)` back to QML via a `Connections` block.

### Settings model key points

- Source overrides come from `extruder_stack.userChanges` (the top container in the stack). Only keys present there are considered — inherited defaults are intentionally ignored.
- `stack.getProperty(key, "settable_per_extruder")` gates whether a key is extruder-scoped before copying.
- `EXCLUDED_SETTING_KEYS` in `SettingsCopyService.py` hard-blocks hardware geometry and G-code fields regardless of user selection.
- Category filtering walks the definition parent chain (`_category_of()`) to resolve which top-level category a setting belongs to.

### QML dialog pages

| Page | Description |
|------|-------------|
| 0    | Source/target ComboBox + optional category filter checkboxes |
| 1    | Preview: counts, affected categories, nozzle/material warnings |
| 2    | Results: copied / skipped / failed counts + scrollable issue list |

## Cura API surface used

- `CuraApplication.getInstance().getGlobalContainerStack()` → `global_stack.extruderList`
- `extruder_stack.userChanges` → `getAllKeys()`, `getProperty(key, "value")`, `setProperty(key, "value", value)`
- `extruder_stack.getProperty(key, "settable_per_extruder")`
- `extruder_stack.definition.findDefinitions(key=key)`
- `Application.getInstance().createQmlComponent(path, context_props)`

## Compatibility

- **Target**: UltiMaker Cura 5.x (plugin API 8.0, PyQt6, Qt 6)
- The plugin does **not** work on Cura 4.x (PyQt5) without porting the `PyQt6` imports.
- Tested against printers that expose `extruderList` on the global stack (standard dual-extruder profiles). Single-extruder machines are handled gracefully — the UI disables the action.

---
> Source: [Land5hark/flip-app](https://github.com/Land5hark/flip-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-05 -->
