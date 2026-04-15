## py-file-organizerv2-0

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the App

```bash
python file_organizer_v2.py
```

Requires Python 3.6+ and Tkinter. On Linux, install Tkinter if missing:
```bash
sudo apt-get install python3-tk
```

No external dependencies — stdlib only.

## Architecture

The entire application is a single file: `file_organizer_v2.py`.

**`FileOrganizerApp`** is the sole class, instantiated with a `tk.Tk` root. Initialization order matters:
1. `load_folders_config()` — reads `folders_config.json` (sibling to the script) and sets `self.folders` and `self.theme`
2. `apply_style()` — configures `ttk.Style` with the "clam" theme
3. `setup_ui()` — builds all widgets
4. `apply_theme()` — applies light/dark colors to existing widgets

**Configuration persistence** (`folders_config.json`): a JSON dict where keys are category names mapped to extension lists, plus a reserved `"_theme"` key for the light/dark preference. Saved automatically on every category add/update/delete and theme toggle.

**File organization flow** (`organize_files`): runs on a daemon thread (started by `start_organization`). Uses `_collect_files_to_organize()` to gather files — either top-level only (default) or recursively via `rglob()` when the "Organize subfolders recursively" checkbox is enabled. Recursion skips paths whose first segment is a category name or `others`, preventing re-processing already-organized files. Matches extensions to categories and moves files into subfolders. Unrecognized extensions go to `others/`. Uses `_unique_destination()` to avoid overwriting files with the same name. The "Preview" checkbox (`backup_first_var`) enables dry-run mode — logs moves without executing them. Relative paths are logged for clarity when processing nested files.

**UI threading**: all UI updates from the worker thread go through `self.root.after(0, ...)` to stay on the main thread.

**Category management**: the listbox reflects `self.folders` (sorted). Add/Update/Delete/Restore all call `save_folders_config()` immediately after modifying `self.folders`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jimjamscott22)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jimjamscott22)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
