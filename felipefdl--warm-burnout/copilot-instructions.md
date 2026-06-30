## warm-burnout

> See the root [`AGENTS.md`](../AGENTS.md) for the canonical palette, design principles, and brand rules. Do not duplicate palette tables here.

# Windows Terminal -- Agent Instructions

## Platform Reference

See the root [`AGENTS.md`](../AGENTS.md) for the canonical palette, design principles, and brand rules. Do not duplicate palette tables here.

## Windows Terminal Theme Format

- Windows Terminal uses JSON color scheme files.
- **Standalone files** (`warm-burnout-dark.json`, `warm-burnout-light.json`): single scheme objects for pasting into `settings.json`'s `schemes` array.
- **Fragment file** (`warm-burnout.json`): wraps both schemes in a `{"schemes": [...]}` object for drop-in install via the [JSON fragment extensions](https://learn.microsoft.com/en-us/windows/terminal/json-fragment-extensions) directory.
- Each scheme is a flat JSON object with `name`, color keys (`background`, `foreground`, `cursorColor`, `selectionBackground`), and 16 ANSI color keys.

## Color Representation

- Colors use uppercase hex strings with `#` prefix (e.g., `#1A1510`).
- All hex values are 6-digit RGB.

## Color Key Mapping

| Windows Terminal Key | Canonical Source |
|---------------------|-----------------|
| `black` through `white` | Ghostty `palette = 0` through `palette = 7` |
| `brightBlack` through `brightWhite` | Ghostty `palette = 8` through `palette = 15` |
| `background` | Editor background (`#1A1510` dark, `#F5EDE0` light) |
| `foreground` | Editor foreground (`#BFBDB6` dark, `#3A3630` light) |
| `cursorColor` | Canonical cursor (`#F5C56E` dark, `#8A6600` light) |
| `selectionBackground` | Selection background (`#33393A` dark, `#E5E8E2` light) |

Note: Windows Terminal uses `purple` / `brightPurple` for ANSI 5/13 (not `magenta`).

## File Naming

- Dark theme: `warm-burnout-dark.json`
- Light theme: `warm-burnout-light.json`

---
> Source: [felipefdl/warm-burnout](https://github.com/felipefdl/warm-burnout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
