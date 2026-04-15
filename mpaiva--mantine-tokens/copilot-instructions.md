## mantine-tokens

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a design token system using Style Dictionary v4.3.3 that generates CSS variables, TypeScript exports, and documentation. The system uses DTCG (Design Token Community Group) format and supports dynamic prefix configuration.

## Commands

### Build Commands
```bash
npm run build           # Build Mantine tokens
npm run build:verbose   # Build with detailed logging
npm run build:custom    # Build custom tokens
npm run build:all      # Build both Mantine and custom tokens  
npm run build:brands    # Build brand-specific tokens
npm run build:figma     # Build Figma-compatible output
npm run dev            # Watch mode for Mantine tokens
npm run dev:custom     # Watch mode for custom tokens
npm run dev:all        # Watch mode for both token sets
npm run dev:brands     # Watch mode for brand tokens
npm run validate       # Validate token structure
npm run check:collisions # Check for token path collisions
npm run clean          # Clean build directory
```

### Development Commands
```bash
npm install            # Install dependencies
npm test               # Run Jest tests
npm run test:watch     # Run tests in watch mode
npm run test:coverage  # Generate coverage report
npm run lint           # Run ESLint
npm run lint:fix       # Fix linting issues
npm run format         # Format with Prettier
npm run format:check   # Check formatting
```


## Architecture

### Token Structure
- **Source tokens**: `tokens/` directory in DTCG format
- **Primitives**: `tokens/primitives/*-dtcg.json` (colors, spacing, typography, etc.)
- **Semantic**: `tokens/semantic/*-dtcg.json` (theme-specific tokens)
- **Components**: `tokens/components/*-dtcg.json` (button, card, input, etc.)
- **Custom tokens**: `tokens/custom/*.json` (additional brand-specific tokens)
- **Prefix config**: 
  - `tokens/_prefix.json` defines the Mantine prefix (currently "mantine")
  - `tokens/_custom-prefix.json` defines the custom token prefix (default "clearco")
- **Token references**: Use `{token.path}` syntax in DTCG format

### Build System
- **Entry points**: 
  - `build-standard.js` - Builds Mantine tokens
  - `build-custom.js` - Builds custom tokens with custom prefix
  - `build-all.js` - Builds both token sets
- **Dynamic prefix**: Reads from respective prefix files and applies to outputs
- **Custom transforms**: 
  - `name/prefix-css`: Adds dynamic prefix to CSS variables
  - `name/custom-prefix-css`: Adds custom prefix to CSS variables
  - Custom formats for DTCG compliance
- **Output formats**: 
  - Mantine tokens:
    - CSS variables: `build/css/` (variables.css, theme-light.css, theme-dark.css)
    - SCSS variables: `build/scss/`
    - JavaScript: `build/js/`
    - TypeScript: `build/ts/` (with typed exports)
    - Documentation: `build/docs/`
  - Custom tokens:
    - CSS/SCSS/JS/TS files in `build/custom/` directory
  - Brand tokens:
    - CSS/JS/TS files in `build/brands/{brand}/` directory
  - All JSON tokens consolidated in `build/json/`:
    - `mantine.primitive.tokens.json` - Primitive tokens only (colors, spacing, typography, etc.)
    - `mantine.component.tokens.json` - Component tokens only (button, card, input, etc.)
    - `mantine.semantic.light.tokens.json` - Light theme semantic tokens
    - `mantine.semantic.dark.tokens.json` - Dark theme semantic tokens
    - `mantine.custom.tokens.json` - Custom tokens (spacing, typography, components)
    - `{brand}.light.tokens.json` - Brand-specific light theme tokens
    - `{brand}.dark.tokens.json` - Brand-specific dark theme tokens

### Token Categories
- **Mantine Tokens**:
  - **Primitives**:
    - Colors (full palette with shades 50-900)
    - Typography (font families, sizes, weights, line heights, letter spacing)
    - Spacing (0, xs, sm, md, lg, xl, 2xl, 3xl)
    - Shadows (xs, sm, md, lg, xl)
    - Border radius (none, xs, sm, md, lg, xl, full)
  - **Semantic**:
    - Light theme (background, surface, border, text, state colors)
    - Dark theme (inverted color mappings)
  - **Components**:
    - Button, Card, Input, Modal, Badge, Alert
- **Custom Tokens** (examples):
  - Brand colors (primary, secondary, accent)
  - Custom spacing (xxs, 4xl, 5xl, section, page)
  - Custom typography (display, brand fonts)

## Tools & Scripts

### Available Scripts
- `npm run build` - Build Mantine tokens
- `npm run build:verbose` - Build with detailed logging
- `npm run build:custom` - Build custom tokens
- `npm run build:custom:verbose` - Build custom tokens with logging
- `npm run build:all` - Build both token sets
- `npm run build:all:verbose` - Build all with logging
- `npm run dev` / `npm run watch` - Watch mode for Mantine tokens
- `npm run dev:custom` - Watch mode for custom tokens
- `npm run dev:all` - Watch mode for both token sets
- `npm run validate` - Validate token structure
- `npm run clean` - Clean build directory
- `npm run test` - Run Jest tests
- `npm run lint` - Run ESLint
- `npm run format` - Format code with Prettier

### MCP Integration
This project supports MCP (Model Context Protocol) servers for enhanced Claude integration:
- **Filesystem Server**: Token file management and batch updates
- **Git Server**: Version control integration
- **GitHub Server**: PR and issue automation

See `docs/MCP_SETUP.md` for detailed configuration instructions.

## Important Notes

- Uses ES modules (`"type": "module"` in package.json)
- Jest configured for ES modules testing
- TransformStream polyfill added for Node.js compatibility
- All CSS variables use dynamic prefix from respective prefix files
- DTCG format compliance with $value, $type, $description
- Watch mode available for development workflow
- Supports multiple token sets with different prefixes
- Custom tokens are built separately and output to `build/custom/`
- Post-build hooks handle prefix changes automatically for Mantine tokens

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mpaiva)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mpaiva)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
