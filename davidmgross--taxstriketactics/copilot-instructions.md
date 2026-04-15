## taxstriketactics

> This file provides guidance to Claude Code (claude.ai/code) when it works with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when it works with code in this repository.

## Project Overview

This is "Tax Strike Tactics" by David M. Gross - a book published as a documentation website using Astro with the Starlight theme. The book covers historical tactics of tax resistance and non-violent protest.

## Development Commands

| Command | Description |
|---------|-------------|
| `npm install` | Install dependencies |
| `npm run dev` | Start local development server at localhost:4321 |
| `npm run build` | Build production site to ./dist/ |
| `npm run preview` | Preview production build locally |
| `npm run astro ...` | Run Astro CLI commands |

## Architecture

### Content Structure
- **Content Location**: All book content is in `src/content/docs/` as MDX files
- **Organization**: Content is organized into thematic directories that map to book chapters
- **Navigation**: Sidebar navigation is auto-generated from directory structure via `astro.config.mjs`
- **Base Path**: Site is deployed with base path `/TPL/TaxStrikeTactics`

### Key Components
- **ExtendedAside**: Custom component extending Starlight's default Aside with a 'wikipedia' variant for Wikipedia citations
- **WikipediaBadge**: Component for Wikipedia-related UI elements
- **Custom Fonts**: Uses Aleo serif font for body text and Edu SA Beginner for workshop elements

### Styling
- **Custom CSS**: `src/styles/custom.css` contains typography overrides and component styling
- **Font Loading**: Multiple font variants loaded via fontsource packages
- **Theme Extensions**: Custom aside variants and detail/summary element styling

### Content Features
- **Definition Lists**: Uses `remark-definition-list` plugin for enhanced markdown support
- **Images**: Static assets in `src/assets/` with historical photos and illustrations
- **Documents**: PDF worksheets available in `public/documents/`

### Starlight Configuration
- Auto-generated sidebar from directory structure
- Custom component overrides for Aside elements
- Social links and metadata configuration
- Collapsed sidebar sections by default

## Development Resources

### MCP Tools
- **context7 MCP** provides LLM-optimized documentation for frameworks like Astro and Starlight. Use this instead of WebSearch/WebFetch for accurate, up-to-date framework information.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DavidMGross)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DavidMGross)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
