## metingjs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MetingJS is a JavaScript library that creates a custom HTML element `<meting-js>` for embedding music players using the APlayer HTML5 music player. It integrates with various music platforms (Netease, Tencent QQ Music, Xiami, etc.) through the Meting API to fetch music metadata and create beautiful music players in web pages.

## Architecture

### Core Architecture
- **Custom Web Component**: Built as a Web Components custom element (`MetingJSElement` extending `HTMLElement`)
- **Single Source File**: All functionality is contained in `src/Meting.js`
- **External Dependencies**: Relies on APlayer for the actual music player UI and functionality
- **API Integration**: Connects to Meting API endpoints to fetch music metadata from various platforms

### Key Components
- **MetingJSElement Class**: The main custom element that handles:
  - Attribute parsing and configuration
  - URL pattern matching for auto-detection of music links
  - API communication with Meting servers
  - APlayer instance creation and management
  - Lifecycle management (connect/disconnect callbacks)

### Data Flow
1. HTML `<meting-js>` element with attributes (server, type, id, etc.)
2. Element initialization parses attributes into config and metadata
3. For external music: API call to Meting service to fetch track data
4. For self-hosted: Direct use of provided URL, name, artist, cover
5. APlayer instance created with the processed audio data
6. Music player rendered in the DOM

## Development Commands

### Build System
```bash
# Build the minified distribution file
npm run build

# Clean and rebuild (used by build)
del dist && mkdir dist && rollup -c
```

### Code Quality
```bash
# Lint JavaScript code
npm run lint

# Run ESLint on src directory
eslint src
```

### Testing
```bash
# Run tests (currently just builds the project)
npm test
```

### Release Process
```bash
# Build before publishing to npm
npm run prepublishOnly
```

## Build Configuration

### Rollup Configuration (`rollup.config.js`)
- **Input**: `src/Meting.js`
- **Output**: Single minified IIFE file `dist/Meting.min.js`
- **Plugins**: Version replacement and Terser minification
- **Format**: Immediately Invoked Function Expression (IIFE) for browser compatibility

### ESLint Rules (`.eslintrc.js`)
- **Environment**: Browser, CommonJS, ES6
- **Style**: 2-space indentation, single quotes, no semicolons, Unix line breaks
- **Configuration**: `eslint:recommended` extended

## File Structure
```
/
├── src/
│   └── Meting.js          # Main source file with MetingJSElement class
├── dist/                  # Built distribution files (generated)
├── package.json           # NPM configuration and build scripts
├── rollup.config.js      # Rollup build configuration
├── .eslintrc.js          # ESLint code style configuration
└── README.md             # Project documentation and usage examples
```

## Code Style Guidelines
- Use 2-space indentation
- Single quotes for strings
- No semicolons
- Unix line breaks
- ES6+ features with Rollup/Terser minification for browser distribution
- Console logging allowed (ESLint rule: `"no-console": "off"`)

## Integration Notes
- Requires APlayer library to be loaded before MetingJS
- Uses `window.customElements.define()` to register the custom element
- Supports custom API endpoints via `window.meting_api` global variable
- Implements Web Components lifecycle callbacks for proper cleanup

---
> Source: [metowolf/MetingJS](https://github.com/metowolf/MetingJS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
