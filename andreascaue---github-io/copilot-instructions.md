## github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal portfolio and code examples website hosted on GitHub Pages. Static HTML/CSS/JS — no build tools, no package manager, no compilation step.

## Local Development

Open `index.html` directly in a browser, or use VS Code Live Server on port **5501** (configured in `.vscode/settings.json`).

Deployment is automatic via GitHub Pages from the `main` branch.

## Architecture

### Structure

- `index.html` — Main landing page (uses Tailwind CSS via CDN)
- `modal.html` — Contact modal
- `part_*/` — Technology-specific sections (JavaScript, AngularJS, jQuery, etc.)
  - Each section has a `main-{technology}.html` entry point
  - Sections may have nested `/js` and `/style` subdirectories
- `js/` — Shared scripts: `main.js`, `menu.js`, `modal.js`
- `style/` — Shared styles: `main.css`, `normalize.css`, `modal.css`
- `img/` — Images

### JavaScript

Vanilla JS component pattern — class-based with closure for private state. No framework.

- `main.js` — Initializes Menu, Carousel; toggles `no-js`/`js` body class
- `menu.js` — Responsive nav menu (breakpoint: 1024px), smooth height transitions
- `modal.js` — Click-outside-to-close modal via `.hero__btn` trigger

### CSS

Progressive enhancement: the `no-js`/`js` body class controls JS-dependent styles. Container max-width is 980px. External fonts from Google Fonts (Open Sans, Sintony).

### External Dependencies (CDN only)

- Tailwind CSS — landing page
- Font Awesome 5.2.0 — icons
- Google Fonts — typography
- AngularJS — `part_angularJS/` examples

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andreascaue)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/andreascaue)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
