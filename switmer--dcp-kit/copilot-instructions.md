## dcp-kit

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Testing
- `npm test` - Run Jest tests with experimental VM modules
- `npm run test:coverage` - Run tests with coverage reporting (80% threshold required)

### Building
- `npm run build` - Compile TypeScript to JavaScript
- `npm run prepublishOnly` - Run tests and build before publishing

### Storybook
- `npm run storybook` - Start Storybook development server on port 6006
- `npm run build-storybook` - Build Storybook for production
- `npm run build:storybook` - Build Storybook with docs and extract metadata

### CLI Commands
- `npm run serve` - Run the local development server
- `npm run extract` - Extract component metadata

## Project Architecture

### Core Components
- **Agent System** (`agent/`): Autonomous component analysis tool that extracts React component APIs and design tokens
- **CLI Interface** (`bin/`, `commands/`): Command-line tools for building, validating, and scaffolding DCP registries
- **Core Parser** (`core/parser.js`): TypeScript/React component parser using react-docgen-typescript with Babel AST analysis
- **Token Extraction**: Multi-source design token detection (Tailwind, CSS modules, CSS-in-JS, CSS variables)

### Key Workflows

#### Component Processing Pipeline
1. **Discovery**: Glob patterns find React components (`**/*.tsx`, `**/*.jsx`)
2. **Parsing**: TypeScript parser extracts props, types, and documentation
3. **Token Analysis**: Scans for design tokens across multiple sources
4. **Schema Generation**: Outputs DCP-compliant JSON schemas
5. **Refactor Proposals**: Analyzes and suggests API improvements

#### Token Sources
- **Tailwind CSS**: Extracts utility classes and maps to design token categories
- **CSS Modules**: Parses co-located `.css` files for classes and variables
- **CSS-in-JS**: Detects styled-components, emotion, and similar libraries
- **CSS Variables**: Extracts `--custom-property` definitions

### Configuration
- `dcp.config.json` - Main configuration file for registry settings
- `tsconfig.json` - TypeScript configuration with path mapping support
- Component discovery uses recursive tsconfig.json resolution

### ESM Module System
- Pure ESM project (`"type": "module"` in package.json)
- Jest configured with experimental VM modules for ESM support
- Uses `.js` extensions for imports even with TypeScript source

### Output Structure
```
dcp-registry/
├── components/           # DCP component schemas
│   └── *.dcp.json
└── proposals/           # Refactor suggestions
    └── *.md
```

## Important Notes

### Parser Configuration
- Uses `react-docgen-typescript` with custom prop filtering
- Includes inherited props from external libraries (react-aria-components)
- Automatically resolves TypeScript path aliases from tsconfig.json

### Token Mapping Strategy
- Categorizes tokens by usage: `background`, `text`, `spacing`, `typography`
- Handles CSS state variants (hover, focus, disabled)
- Maps CSS animations, media queries, and pseudo-classes

### Error Handling
- Graceful failures for unparseable components
- Verbose logging available via `--loglevel debug`
- Caches parser instances per tsconfig for performance

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/switmer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/switmer)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
