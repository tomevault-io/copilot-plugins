## fysik1

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Interactive web-based study tools for a Swedish structural engineering/building mechanics exam course (Fysik 1). Four calculators help students compute beam diagrams, snow loads, timber beam dimensioning, and wall weights. All UI text is in Swedish.

## Tech Stack & Architecture

- **Pure static site**: HTML5 + vanilla JavaScript + inline CSS. No frameworks, no bundler, no npm.
- **Each calculator is a self-contained HTML file** with embedded `<script>` and `<style>` blocks — no shared JS/CSS files.
- **Visualizations**: HTML5 Canvas API for diagrams (beam V/M diagrams, roof shapes, cross-sections, 3D wall views).
- **Theme system**: Three themes (Seriös, Prinsessa, Katter) with dynamic CSS injection and localStorage persistence. Theme code is duplicated across pages.
- **Layout pattern**: Each calculator uses a two-column layout — controls/inputs on the left, visualizations on the right.

## Pages

| File | Purpose |
|------|---------|
| `index.html` | Landing page with navigation cards to each tool |
| `beam-calculator.html` | Shear force (V) and moment (M) diagram calculator with preset load cases |
| `snow-load-calculator.html` | Snow load per SS-EN 1991-1-3 Swedish standard |
| `timber-beam-calculator.html` | 7-step timber beam dimensioning (kmod, γm, kh, fd, moment/shear/normal checks) |
| `wall-weight-calculator.html` | Wall construction weight calculator with 2D/3D visualization |

## Development

**No build step required.** Open any HTML file directly in a browser to develop and test.

**Deployment:** GitHub Actions (`.github/workflows/deploy.yml`) deploys to GitHub Pages on push to `main`. No build step in the pipeline — it uploads the static files directly.

## Key Patterns

- Input controls use paired number input + range slider kept in sync via event listeners.
- Preset systems load predefined scenarios by setting input values and triggering recalculation.
- Canvas drawing functions are defined inline in each page's `<script>` block.
- Swedish engineering standards are implemented directly (e.g., SS-EN 1991-1-3 shape factors, Eurocode timber material classes C14–C40 and GL24c–GL32h).
- Load combination factors follow Eurocode: 1.2G + 1.5Q.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jellyfrog)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Jellyfrog)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
