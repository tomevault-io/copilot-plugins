## waibibabu

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Nuxt 3 web application providing two text encoding schemes using Chinese characters:
- **歪比吧卜** (waibibabu): Simple base-3 encoding mapping characters to `["歪","比","吧"]` with `卜` as delimiter
- **咕咕嘎嘎** (gugugaga): Advanced base-4 encoding with compression, mapping 4-bit pairs to Chinese words/emojis

## Tech Stack

- **Framework**: Nuxt 3 with Vue 3 Composition API
- **UI Library**: Vuetify 3 with Material Design icons
- **Language**: TypeScript with Pug templates
- **Package Manager**: pnpm
- **Utilities**: VueUse (clipboard functionality via `useClipboard`)

## Development Commands

```bash
pnpm install          # Install dependencies
pnpm run dev          # Start dev server on port 3200
pnpm run build        # Build for production (SSR)
pnpm run generate     # Generate static site
pnpm run typecheck    # Run TypeScript type checking
pnpm run prepare      # Prepare Nuxt (generates .nuxt directory)
```

## Architecture

### File-Based Routing
- `/` → `pages/index.vue` (waibibabu encoder/decoder)
- `/gugugaga` → `pages/gugugaga.vue` (gugugaga encoder/decoder)

### Encoding System (utils/)

**waibibabu.ts** - Note: function names are inverted (decodeWabibabu actually encodes)
- `decodeWabibabu(input)`: Converts plaintext → base-3 → Chinese characters
- `encodeWabibabu(input)`: Converts Chinese characters → base-3 digits → plaintext
- Uses `charCodeAt(0).toString(3)` for base-3 conversion
- Each character terminated with `卜`

**gugugaga.ts**
- `encodeWithPairsCompressed(input)`: Plaintext → base-4 → compressed word pairs
- `decodeWithPairsCompressed(encoded)`: Word pairs → base-4 digits → plaintext
- Compression: `⚡` replaces 8 zeros, `🐧` replaces 4 zeros
- Uses 12-digit base-4 representation padded with zeros
- Token matching uses longest-first strategy to prevent ambiguous parsing

### Key Configuration

**nuxt.config.ts**
- Prerenders `/` and `/gugugaga` routes for SEO
- Nitro preset: `node-server` (can be changed for static hosting)
- Vuetify auto-import enabled
- Language set to `zh-CN`

**netlify.toml**
- Static site deployment configuration
- Node.js 22 required
- Client-side routing redirects via `_redirects` rule

### Component Patterns

Pages use Vue 3 Composition API with:
- `ref()` for reactive state (input/output text)
- `computed()` for real-time encoding/decoding
- `useClipboard()` from VueUse for copy functionality
- Pug templates with Vuetify components
- Scoped CSS for styling

## Important Notes

1. **Function Naming**: `utils/wabibabu.ts` has inverted function names - `decodeWabibabu` encodes, `encodeWabibabu` decodes
2. **Package Resolution**: Uses `resolutions` in package.json to pin specific versions of `unplugin`, `vite`, and `vite-plugin-inspect`
3. **Vuetify Integration**: Custom module setup in nuxt.config.ts hooks into Vite config
4. **TypeScript**: `noImplicitAny` is disabled for flexibility

---
> Source: [shadowdreamer/waibibabu](https://github.com/shadowdreamer/waibibabu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
