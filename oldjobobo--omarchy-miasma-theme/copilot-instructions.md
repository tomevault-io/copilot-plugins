## omarchy-miasma-theme

> - Root files are per-app theme configs (e.g., `hyprland.conf`, `waybar.css`, `kitty.conf`, `alacritty.toml`).

# Repository Guidelines

## Project Structure & Module Organization
- Root files are per-app theme configs (e.g., `hyprland.conf`, `waybar.css`, `kitty.conf`, `alacritty.toml`).
- `backgrounds/` contains the bundled wallpapers used by the theme.
- `waybar-theme/` holds a JSONC + CSS pair for Waybar variants.
- `preview.png` is the visual snapshot used in the README.

## Build, Test, and Development Commands
This repo is configuration-only; there is no build system or automated test runner.
- Install the theme with Omarchy: `omarchy-theme-install https://github.com/OldJobobo/omarchy-miasma-theme`.
- For local iteration, edit files in place and reload the target app (e.g., restart Waybar or reload Hyprland).

## Coding Style & Naming Conventions
- Follow the existing formatting in each file type; do not reformat unrelated sections.
- Indentation: Hyprland configs use 4 spaces inside blocks; Lua uses 2 spaces.
- Colors use hex or rgba formats as already present (e.g., `#c2c2b0`, `rgba(0, 0, 0, 0.6)`).
- Keep filenames descriptive and lowercase, especially for wallpapers in `backgrounds/`.

## Testing Guidelines
- No automated tests are defined.
- Verify changes visually in the target app and update `preview.png` when the UI changes.
- For Neovim, confirm the fallback theme still loads when `xero/miasma.nvim` is absent.

## Commit & Pull Request Guidelines
- Commit history is short and direct (e.g., “update readme”, “color corrections”); keep messages concise and action-focused.
- In PRs, list the files/apps affected and include screenshots when UI output changes.
- If you add or rename assets, update the README’s “What’s included” or wallpapers table accordingly.

---
> Source: [OldJobobo/omarchy-miasma-theme](https://github.com/OldJobobo/omarchy-miasma-theme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
