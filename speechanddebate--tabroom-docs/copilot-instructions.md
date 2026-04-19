## tabroom-docs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the documentation website for Tabroom.com, built with Docusaurus 3.8.1. The site provides comprehensive help documentation for tournament directors, coaches, and participants using the Tabroom platform for speech and debate tournaments.

## Development Commands

### Package Management
- Uses `yarn` for package management
- Install dependencies: `yarn`

### Development Server
- Start development server: `yarn start`
- Opens browser with live reload enabled

### Build and Deployment
- Build for production: `yarn build`
- Type checking: `yarn typecheck` or `tsc`
- Clear Docusaurus cache: `yarn clear`
- Serve built site locally: `yarn serve`

### Deployment Options
- SSH deployment: `USE_SSH=true yarn deploy`
- HTTPS deployment: `GIT_USER=<username> yarn deploy`
- With legacy OpenSSL: `NODE_OPTION=--openssl-legacy-provider GIT_USER=<username> USE_SSH=false npx yarn deploy`

## Architecture and Structure

### Core Configuration
- **docusaurus.config.ts**: Main Docusaurus configuration
  - Site URL: https://docs.tabroom.com
  - Organization: speechanddebate
  - Future v4 compatibility enabled
  - Custom navbar with link back to tabroom.com

- **sidebars.ts**: Auto-generated sidebar from docs folder structure
- **package.json**: Uses Docusaurus 3.8.1 with React 19 and TypeScript

### Content Organization
- **docs/**: All documentation content in Markdown format
  - Auto-generated sidebar from folder structure
  - Route base path set to `/` (docs serve as homepage)
  - Sidebar collapsed is disabled for better navigation

### Key Features
- TypeScript support with `@docusaurus/tsconfig`
- Custom CSS in `src/css/custom.css`
- Static assets in `static/` including images, fonts, and screenshots
- Extensive screenshot collection in `static/screenshots/` for documentation

### Content Structure
- **index.md**: Main help center landing page with navigation to all features
- **Numbered category structure**: Docs organized in numbered folders (1-overview, 2-quick-start, etc.) with automatic sidebar generation
- **Category organization**:
  - `1-overview/`: FAQ and videos
  - `2-quick-start/`: Tournament setup guides and checklists
  - `3-account/`: User account management (students, coaches, judges)
  - `4-administration/`: School admin, registration, circuit management
  - `5-settings/`: Tournament configuration (judging, events, schedule, money, etc.)
  - `6-entries/`: Entry management, judges, reports, data
  - `7-paneling/`: Round paneling, room assignment, judge management
  - `8-schematics/`: Tournament schematics and pairings
  - `9-results/`: Result display, reporting, web publishing
  - `10-public/`: Public-facing features (calendars, results, paradigms)
  - `11-online/`: Online tournament features (NSDA Campus, async events)
- **_deprecated/**: Contains older documentation files that may need cleanup
- Each category has a `_category_.json` file for Docusaurus sidebar configuration

### Deployment Target
- Hosted at docs.tabroom.com
- GitHub Pages deployment configured
- Repository: speechanddebate/tabroom-docs

## Development Notes

- Uses modern Docusaurus features including future v4 compatibility
- Extensive use of screenshots for visual documentation
- Content focuses on speech and debate tournament management
- No custom React components currently in use - relies on standard Docusaurus features
- Documentation follows numbered category structure with prefix-based ordering
- Screenshots stored in `static/screenshots/` and referenced as `/screenshots/filename.png`
- Internal links use kebab-case format matching file structure

## Content Migration Guidelines

When migrating documentation from other formats:
- Use kebab-case filenames (e.g., `school-administration.md`)
- Add Docusaurus frontmatter with title metadata
- Update image paths to use `/screenshots/` prefix
- Convert internal links to match new file structure
- Remove MediaWiki-specific syntax (figure/figcaption, category links, etc.)
- Place files in appropriate numbered category folders

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/speechanddebate) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
