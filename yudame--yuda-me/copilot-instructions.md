## yuda-me

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Single-page business website for Yudame, deployed on GitHub Pages at `yuda.me`. Corporate homepage showcasing Yudame as an innovation lab.

## Development Commands

### Build Process
```bash
npm install          # Install dependencies
npm run build        # Build production files (CSS + copy HTML/assets)
npm run dev          # Start dev server with hot reload
npm run clean        # Clean dist directory
```

### Local Development
```bash
npm run dev          # Tailwind watch + local server on port 8080
```

**Automatic deployment**: Push to `main` branch triggers GitHub Actions deployment

## Architecture

### Technology Stack
- **Static HTML** - Single-page site, no framework
- **Tailwind CSS** - Utility-first CSS with local build process
- **GitHub Pages** - Static site hosting
- **GitHub Actions** - CI/CD for automatic deployment on push to main

### Build Pipeline
1. **Source files** in `src/` (HTML) and `assets/` (images)
2. **Build process** compiles Tailwind CSS and copies files to `dist/`
3. **GitHub Actions** builds and deploys `dist/` to GitHub Pages
4. **Live site** served at `yuda.me` via GitHub Pages

### Project Structure
```
yuda.me/
├── src/                    # Source files (edit these)
│   ├── index.html         # Homepage (only page)
│   └── styles.css         # Tailwind input file
├── dist/                   # Built files (auto-generated, gitignored)
├── assets/                 # Static images
│   ├── logos/             # Company logos and favicons
│   └── profiles/          # Team member photos
└── .github/workflows/     # GitHub Actions deployment
```

### Key Architectural Decisions
- **No JavaScript framework** - All interactivity via inline `<script>` tags in HTML
- **Tailwind utility classes** - All styling via Tailwind, custom styles in HTML `<style>` tags
- **GitHub Pages deployment** - Automatic via GitHub Actions on push to main
- **Static images** - Served directly from `assets/` directory (not base64 encoded)

## Important Context

This is the marketing landing page only. The main application lives at `app.yuda.me`. Primary CTA directs to `https://app.yuda.me/getting-started/0/`.

### Development Guidelines
- Keep all JavaScript inline in HTML `<script>` tags
- Use Tailwind utility classes for all styling
- Test locally with `npm run dev` before pushing
- Maintain brand colors via Tailwind config: `yudame-yellow` (#FFC107), `yudame-red` (#DC2626), `yudame-dark` (#1F2937)
- Use Raleway font family (configured as `font-raleway`)

## Deployment Process

**GitHub Pages Configuration**:
- Deployment triggered automatically on push to `main` branch
- GitHub Actions workflow builds and deploys `dist/` directory
- Custom domain `yuda.me` configured via GitHub Pages settings
- Build artifacts are not committed - `dist/` is gitignored

## Business Context

For business context, project notes, and assets see the work vault: `~/src/work-vault/Yudame/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yudame)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yudame)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
