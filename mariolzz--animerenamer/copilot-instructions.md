## animerenamer

> Python CLI script (`rename.py`) + tkinter GUI (`gui.py`) that recursively renames anime episode files for Jellyfin/media server compatibility. No runtime dependencies beyond the standard library.

# AGENTS.md

## Project

Python CLI script (`rename.py`) + tkinter GUI (`gui.py`) that recursively renames anime episode files for Jellyfin/media server compatibility. No runtime dependencies beyond the standard library.

## Running

```bash
python rename.py                  # dry-run (no changes)
python rename.py --force          # actually rename files
python rename.py --dir /path      # target a different directory
python rename.py --undo           # revert renames using manifest
python rename.py --version        # print version and exit
python gui.py                     # launch tkinter GUI
```

No install or build step. The script operates on a directory (CWD by default, or `--dir`).

## Flags

- `--force` — execute renames (default is dry-run).
- `--dir PATH` — directory to scan (default: cwd).
- `--undo` — revert renames from `rename_manifest.json`.
- `--manifest PATH` — custom manifest path (default: `<dir>/rename_manifest.json`).
- `--version` — print version and exit.

## Manifest / Undo

On `--force`, the script writes `rename_manifest.json` in the target directory. It records each successful rename as `{"old": "original.ext", "new": "cleaned.ext"}`. Running `--undo` reads this manifest and restores original names. The manifest is deleted after a successful undo.

If conflicts or errors occur during rename, the manifest still reflects only the successful renames, and the script exits with status 1.

## Testing

```bash
uv run pytest test_rename.py -v    # run all 51 tests
```

Uses `uv` for dependency management. `pyproject.toml` defines `pytest` as a dev dependency. Always use `uv run pytest`, not bare `pytest`.

## Conventions

- `ALLOWED_EXTENSIONS` in `rename.py:11` controls which files are touched (video + subtitle formats). Don't add non-media extensions.
- `clean_name()` at line 19 contains all renaming logic. Regex patterns are intentionally conservative to avoid false positives on romanized Japanese filenames (e.g. "2-banme", "3-gatsu"). Uses `re.VERBOSE` for readability.
- The script has two passes: first it collects all planned renames, then it executes them. This enables the count preview and conflict summary.
- GUI (`gui.py`) imports from `rename.py` — keep shared logic in `rename.py`.
- Output tags: `[OK]`, `[SKIP]`, `[ERROR]`, `[DRY-RUN]`.

---
> Source: [mariolzz/AnimeRenamer](https://github.com/mariolzz/AnimeRenamer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
