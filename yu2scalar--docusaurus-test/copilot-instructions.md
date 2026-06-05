## docusaurus-test

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Docusaurus-based documentation site for ScalarDB and ScalarDL FAQ, configured for GitHub Pages deployment. The site is built as a static website generator with search functionality and organized documentation categories.

## Common Commands

### Development
```bash
yarn install          # Install dependencies
yarn start            # Start local development server with live reload
yarn build            # Build static site for production
yarn serve            # Serve built site locally
```

### Deployment
```bash
yarn deploy                                    # Deploy to GitHub Pages (requires SSH)
USE_SSH=true yarn deploy                      # Deploy using SSH
GIT_USER=<username> yarn deploy              # Deploy without SSH
```

### Utility
```bash
yarn clear                    # Clear Docusaurus cache
yarn swizzle                 # Customize Docusaurus components
yarn write-translations      # Extract translatable strings
yarn write-heading-ids       # Generate heading IDs
```

## Architecture

### Site Configuration
- **docusaurus.config.js**: Main configuration file defining site metadata, theme, plugins, and navbar
- **sidebars.js**: Sidebar navigation structure (currently using auto-generated from file structure)
- **package.json**: Node.js dependencies and scripts

### Content Structure
- **docs/**: Main documentation directory serving as site root (`routeBasePath: '/'`)
  - **ScalarDB/**: ScalarDB-related documentation organized by category
    - **Deploy/**, **Develop/**, **General/**, **Manage/**, **Troubleshoot/**
  - **ScalarDL/**: ScalarDL-related documentation with same category structure
- **src/**: React components and custom CSS
- **static/**: Static assets including images and icons

### Key Features
- Lunr search plugin with English/Japanese language support
- Auto-generated sidebars from directory structure  
- GitHub Pages deployment configured for yu2scalar/docusaurus-test
- Custom ScalarDB branding and logo

### File Naming Convention
Documentation files use descriptive names with numeric IDs (e.g., `scalardb_10399920368143.md`, `scalardl_10612598636175.md`).

---
> Source: [yu2scalar/docusaurus-test](https://github.com/yu2scalar/docusaurus-test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
