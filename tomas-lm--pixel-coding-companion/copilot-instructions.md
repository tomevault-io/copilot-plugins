## pixel-coding-companion

> Primary maintainer workflows differ by OS: Lorenzo develops on Linux; teammates may use macOS.

# Agent notes

## Linux vs macOS UI in this Electron app

Primary maintainer workflows differ by OS: Lorenzo develops on Linux; teammates may use macOS.

Electron/Chromium inherits platform scrollbars and form-control chrome. Without explicit styling, Linux often shows **wide, light-themed scrolltracks** inside otherwise dark panels, which looks broken next to deliberately themed regions (workspace rail lists, terminals).

When adding **scrollable areas** (`overflow: auto` / `scroll`) on dark backgrounds:

1. Set **`color-scheme: dark`** on the dark container where appropriate so GTK/system-influenced controls lean dark.
2. Match app scrollbars using the same approach as **`workspace-rail-scroll`** (and companion panel): **`scrollbar-width: thin`**, **`scrollbar-color`**, and **`::-webkit-scrollbar`** thumb/track rules so Chromium-based builds look consistent.

If a new scroll region looks fine on macOS but wrong on Linux, check these first before changing layout logic.

---
> Source: [tomas-lm/pixel-coding-companion](https://github.com/tomas-lm/pixel-coding-companion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
