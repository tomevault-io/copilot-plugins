## securehealth-docs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Development Commands

### Development
- `npm start` - Start the development server with hot reload
- `npm run build` - Build the static site for production
- `npm run serve` - Serve the built static site locally
- `npm run clear` - Clear the Docusaurus cache

### Deployment
- `npm run deploy` - Deploy to GitHub Pages

### Internationalization
- `npm run write-translations` - Extract translation messages
- `npm run write-heading-ids` - Generate heading IDs for docs

## High-Level Architecture

This is a Docusaurus v3 workshop template designed to help create lab documentation sites. Key architectural components:

### Core Structure
- **Docusaurus Configuration** (`docusaurus.config.js`): Central configuration defining site metadata, theme settings, plugins, and custom fields including UTM parameters
- **Content Organization**: MDX-based documentation in `docs/` folder with auto-generated sidebar from folder structure
- **Internationalization**: Full i18n support with Spanish (`es`) translations in `i18n/es/`
- **Custom Components**: Reusable React components in `src/components/` for enhanced documentation features

### Key Customizations
- **UTM Tracking**: Built-in UTM parameter system for tracking workshop engagement (configured in `docusaurus.config.js`)
- **Custom Link Component** (`src/components/Link.js`): Automatically appends UTM parameters to external links
- **Screenshot Component** (`src/components/Screenshot.js`): Wrapper for images with browser window styling
- **BrowserWindow Component**: Visual component for displaying screenshots in a browser-like frame

### Theming & Plugins
- **Search**: Local search powered by `docusaurus-lunr-search` with multi-language support
- **Mermaid Diagrams**: Built-in support for rendering Mermaid diagrams
- **Code Highlighting**: Prism-based syntax highlighting with support for multiple languages
- **Google Analytics**: Integrated with gtag for tracking

### Deployment
- Configured for GitHub Pages deployment with automatic builds via GitHub Actions
- Base URL and organization name are configurable in `docusaurus.config.js`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mrlynn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
