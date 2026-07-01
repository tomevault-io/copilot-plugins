## artplayer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ArtPlayer.js is a modern, full-featured HTML5 video player. It's organized as a monorepo using Lerna with independent versioning.

- **Homepage**: https://artplayer.org
- **Online Editor**: https://artplayer.org (for testing examples)
- **API Docs**: https://artplayer.org/document

## Repository Structure

```
packages/
  artplayer/              # Core player package
  artplayer-plugin-*/     # 17+ plugins (danmuku, ads, chromecast, etc.)
  artplayer-proxy-*/       # Canvas, Mediabunny proxies
  artplayer-tool-*/       # iframe, thumbnail tools
  artplayer-vitepress/    # Documentation site

docs/                     # Documentation and demo site
  assets/                  # Example assets and TypeScript demos
  compiled/               # Built packages output
  document/               # Generated API documentation
  uncompiled/             # Development build output
  test/                   # Browser test files

scripts/                  # Build tooling
  build.js                # Production build
  dev.js                  # Development server
  build-i18n.js           # I18n bundle builder
  build-docs.js           # Documentation generator
  build-llm.js            # LLMs.txt generator
  plugin/create.js         # New plugin scaffolding
```

## Common Commands

### Development
```bash
# Start dev server on port 8082 with hot reload
npm run dev

# Build for production (interactive prompt to select package)
npm run build

# Build all packages
npm run build all

# Build everything (packages, i18n, types, docs) + lint
npm run build:all
```

### Code Quality
```bash
# Lint with auto-fix (covers packages, scripts, tests, docs assets)
npm run lint
```

### Plugin Development
```bash
# Create new plugin from template
npm run create:plugin <kebab-case-name>

# Example:
npm run create:plugin my-feature
```

### I18n
```bash
# Build i18n bundles for all languages
npm run build:i18n
```

### TypeScript
```bash
# Generate TypeScript declaration files
npm run build:ts
```

## Architecture

### Core Player (`packages/artplayer`)

The main `Artplayer` class extends `Emitter` and initializes sub-components:

```javascript
// Key components instantiated in constructor:
this.template = new Template(this)    // DOM structure
this.events = new Events(this)        // Event handling
this.player = new Player(this)        // Video playback
this.controls = new Control(this)     // Control bar
this.layers = new Layer(this)         // Custom layers
this.contextmenu = new Contextmenu(this)
this.subtitle = new Subtitle(this)
this.setting = new Setting(this)
this.plugins = new Plugins(this)      // Plugin manager
// ... and more
```

Source organization in `src/`:
- `config/` - Default configuration values
- `control/` - UI controls (play, progress, volume, etc.)
- `events/` - Event handling (click, hover, resize, etc.)
- `i18n/` - Localization files
- `icons/` - SVG icons
- `player/` - Video playback logic (hls, flv, dash, etc.)
- `plugins/` - Built-in plugins
- `setting/` - Settings panel components
- `style/` - LESS stylesheets
- `template.js` - HTML template generation
- `utils/` - Utility functions

### Plugin Architecture

Plugins are functions that receive the player instance and options:

```javascript
// Template from scripts/plugin/template/src/index.js
export default function artplayerPluginTemplate(option = {}) {
  return (art) => {
    // Plugin initialization
    // Access: art.player, art.controls, art.template, etc.

    return {
      name: 'artplayerPluginTemplate',
      // Expose methods to art.plugins['artplayerPluginTemplate']
    }
  }
}
```

Plugin naming convention:
- Package: `artplayer-plugin-<name>`
- Global variable: `artplayerPlugin<Name>` (camelCase)
- Exported function uses same name as global

### Styling

Uses LESS with inline imports:
```javascript
import style from './style/index.less?inline'
```

Styles are injected at runtime via `utils.setStyleText()`.

### Build System

Uses Vite via custom Node.js scripts:

**Development** (`scripts/dev.js`):
- Builds IIFE bundle to `docs/uncompiled/<package>/`
- Serves `docs/` directory on port 8082
- Watches source files for changes

**Production** (`scripts/build.js`):
- Builds 3 formats: UMD (`.js`), UMD legacy (`.legacy.js`), ESM (`.mjs`)
- Outputs to `packages/<name>/dist/`
- Copies to `docs/compiled/`

### I18n System

Language files in `packages/artplayer/src/i18n/`:
- Base language: `zh-cn.js`
- Additional languages: `en.js`, `fr.js`, etc.
- Built as separate UMD/ESM bundles via `build-i18n.js`
- Global naming: `artplayerI18n<Language>` (e.g., `artplayerI18nEn`)

### Creating New Plugins

The `npm run create:plugin <name>` command:
1. Copies `scripts/plugin/template/` to `packages/artplayer-plugin-<name>/`
2. Replaces placeholders in filenames and content
3. Creates example file at `docs/assets/example/<name>.js`

Plugin template structure:
```
src/
  index.js       # Main plugin code
  style.less     # Plugin styles
package.json
README.md
types/
  <name>.d.ts    # TypeScript declarations (optional)
```

### Utility Functions

Core utilities in `packages/artplayer/src/utils/`:
- `utils.js` - Common helpers (isMobile, isSafari, query, etc.)
- `emitter.js` - Event emitter base class
- `scheme.js` - Option validation schema

## Development Notes

- **Package manager**: Uses Yarn with Lerna
- **Node version**: >= 20.0.0
- **ESLint**: Uses @antfu/eslint-config
- **Browser support**: Last 1 Chrome version (ES2020), legacy builds target ES2015
- **UMD builds**: Always expose global even in AMD/RequireJS environments

---
> Source: [zhw2590582/ArtPlayer](https://github.com/zhw2590582/ArtPlayer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
