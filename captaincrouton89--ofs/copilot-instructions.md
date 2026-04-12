## ofs

> Unified keybinding extractor. Pulls bindings from Karabiner, tmux, neovim, Zed, BTT, Homerow, and Raycast into a single searchable output.

# ofs — Optimized for Speed

Unified keybinding extractor. Pulls bindings from Karabiner, tmux, neovim, Zed, BTT, Homerow, and Raycast into a single searchable output.

## Run

```bash
./ofs build                        # Extract all → bindings.tsv
./ofs build --format json|md|tsv   # Output format
./ofs build --only tmux,neovim     # Subset of sources
./ofs build -o custom.tsv          # Custom output path
./ofs list-sources                 # Show configured sources + paths
./ofs search <pattern>             # Regex grep through last build
```

## Project layout

- `ofs` — CLI entry point (executable Python script)
- `ofs.defaults.json` — Default source paths (committed)
- `ofs.config.json` — Local overrides (gitignored), shallow-merged per source key
- `lib/keymap.py` — `Keybinding` dataclass (tool, key, action, context, enabled, source_file, raw_key)
- `lib/modifiers.py` — Modifier normalization, macOS keycode table, per-tool combo parsers
- `lib/output.py` — TSV/JSON/Markdown formatters
- `extractors/<tool>.py` — Each exports `extract(config: dict) -> list[Keybinding]`

## Conventions

- Python 3 stdlib only. Zero dependencies.
- All modifiers normalize to: `ctrl`, `opt`, `shift`, `cmd`, `fn`, `hyper`. Canonical order: ctrl → opt → shift → cmd → fn → hyper. If all four of ctrl+opt+shift+cmd present, collapse to `hyper`.
- `normalize_key_combo(raw, style, **kwargs)` is the single entry point for key normalization. Style determines parsing: "karabiner" takes `modifiers=list`, "btt" takes `bitmask=int` with raw as int keycode, others take raw as string.
- Extractors handle missing files gracefully (warn to stderr, return empty list).
- Adding a new extractor: create `extractors/foo.py` with `extract(config) -> list[Keybinding]`, add entry to `EXTRACTORS` dict in `ofs`, add default config to `ofs.defaults.json`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CaptainCrouton89)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/CaptainCrouton89)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
