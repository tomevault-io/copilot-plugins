## figma-to-rn-toolkit

> Project context


# Figma to React Native Toolkit

## Project Overview

A comprehensive TypeScript toolkit (v1.4.1) for automatically converting Figma designs to React Native components. Provides both CLI tools and programmatic APIs, with MCP (Model Context Protocol) real-time access support for design-to-code workflows.

## Tech Stack

- **Language**: TypeScript 5.x (strict mode)
- **Runtime**: Node.js >= 16.0.0, NPM >= 8.0.0
- **Target**: ES2020, CommonJS modules
- **Peer Dependencies**: React >= 17.0.0, React Native >= 0.65.0
- **Core Dependencies**: axios (HTTP), commander (CLI), dotenv (env config), eventsource (SSE)
- **Dev Tools**: Jest + ts-jest (testing), ESLint + Prettier (linting/formatting), rimraf (clean), ts-node (dev execution)

## Development Commands

| Command | Description |
|---------|-------------|
| `npm run build` | Compile TypeScript to `dist/` |
| `npm run dev` | Run dev mode via ts-node |
| `npm run cli` | Run main CLI tool |
| `npm run mcp` | Run MCP CLI tool |
| `npm test` | Run Jest tests |
| `npm run test:watch` | Jest in watch mode |
| `npm run lint` | ESLint check |
| `npm run lint:fix` | ESLint auto-fix |
| `npm run format` | Prettier format |
| `npm run demo` | Run basic demo |
| `npm run demo:mcp` | Run MCP demo |

## Node/Runtime Version

- Node.js >= 16.0.0
- NPM >= 8.0.0
- No `.nvmrc` present

## Directory Structure

```
src/
├── api/                        # Figma API layer
│   ├── FigmaAPI.ts             # Core Figma API client
│   ├── FigmaAPIClient.ts       # Low-level API request handling
│   └── FigmaNodeFetcher.ts     # Node-level data retrieval & URL parsing
├── generator/
│   └── ReactNativeGenerator.ts # Converts parsed nodes to RN component code
├── mcp/                        # MCP (Model Context Protocol) integration
│   ├── FigmaMCPClient.ts       # MCP client (SSE-based)
│   └── FigmaMCPIntegration.ts  # Bridges MCP with Figma API
├── parser/
│   ├── ComponentParser.ts      # Figma component structure analysis
│   └── StyleParser.ts          # Figma styles -> RN StyleSheet conversion
├── types/
│   └── index.ts                # Core type definitions (FigmaNode, errors, options)
├── utils/
│   ├── figmaUtils.ts           # Figma data helper utilities
│   ├── figmaToRNStyles.ts      # Figma-to-RN style conversion functions
│   └── stringUtils.ts          # String manipulation helpers
└── index.ts                    # Library entry point (public exports)

tests/                          # Jest test files
├── styleParser.test.ts
└── figmaUtils.test.ts

cli.ts                          # CLI entry point (`figma-to-rn` bin)
mcp-cli.ts                      # MCP CLI entry point (`figma-mcp` bin)
setup-rn-project.ts             # Setup script for target RN projects (`figma-rn-setup` bin)
```

## Architecture Notes

### Entry Points
- `src/index.ts` - Library entry, exports `FigmaNodeFetcher` utilities and style conversion functions
- `cli.ts` - CLI binary (`figma-to-rn`): generate, preview, batch, validate commands
- `mcp-cli.ts` - MCP CLI binary (`figma-mcp`): connect, status, generate-selection, interactive

### Data Flow
1. **Input**: Figma URL -> `FigmaNodeFetcher.parseFigmaUrl()` extracts file key & node ID
2. **Fetch**: `FigmaAPI` / `FigmaAPIClient` retrieves node data from Figma REST API
3. **Parse**: `ComponentParser` analyzes structure; `StyleParser` converts styles
4. **Generate**: `ReactNativeGenerator` produces TypeScript RN component code + StyleSheet
5. **Output**: Written to user-specified output directory

### MCP Integration
- `FigmaMCPClient` connects via SSE (Server-Sent Events) to Figma MCP server
- `FigmaMCPIntegration` bridges MCP context with the generation pipeline
- Enables real-time access to current Figma selection

### Error Handling
- Custom error classes: `FigmaAPIError`, `ParseError`, `GenerationError` (defined in `src/types/index.ts`)
- API calls include retry logic and rate limit handling

### Path Aliases
- `@/*` maps to `src/*` (configured in tsconfig.json & jest.config.js)

## Cursor Figma Plugin Integration

This project works with the Cursor Figma plugin (`/add-plugin figma`) in two complementary ways:

1. **Design System Rules**: `.cursor/rules/figma-design-system.mdc` teaches the Figma plugin's `implement-design` skill how to generate proper React Native code (StyleSheet patterns, component mapping, flex layout conventions).

2. **Setup CLI**: `figma-rn-setup <target-path>` copies RN design rules and `.cursorignore` to any target RN project.

3. **Programmatic API**: Other RN projects can install this as a dev dependency and use `FigmaToReactNative` class for deterministic Figma-to-RN conversion without AI.

### Usage in Other RN Projects

**Approach A (AI-powered via Cursor Figma plugin):**
- Run `npx figma-rn-setup .` in the target RN project
- In Cursor Chat: "implement design <figma-url>"

**Approach B (Deterministic via CLI):**
- `npx figma-to-rn generate -u <figma-url> -o ./src/components`

**Approach C (Programmatic API):**
- `import { FigmaToReactNative } from 'figma-to-rn-toolkit'`

## Build/Deploy Notes

- `npm run build` compiles to `dist/` with declaration files and source maps
- `prebuild` cleans `dist/` via rimraf
- `prepare` hook auto-builds on `npm install`
- Published to NPM as `figma-to-rn-toolkit`
- Distributed files: `dist/`, `src/`, `.cursor/rules/figma-design-system.mdc`, `examples/`, README, LICENSE

## Code Style and Conventions

### ESLint
- Parser: `@typescript-eslint/parser`
- Extends: `eslint:recommended`, `@typescript-eslint/recommended`, `prettier`
- Key rules: `no-explicit-any` (warn), `no-unused-vars` (error, `_` prefix ignored), `prefer-const`, `no-var`, `no-console` (warn)
- Ignores: `dist/`, `node_modules/`, `*.js`

### Prettier
- Single quotes, semicolons, trailing commas (ES5)
- 80 char print width, 2-space indent, LF line endings
- Arrow parens: avoid when possible

### Testing
- Jest with ts-jest preset, node environment
- Test roots: `src/`, `tests/`
- Pattern: `**/*.test.ts`, `**/*.spec.ts`
- Coverage collected from `src/**/*.ts` (excludes `.d.ts` and `.test.ts`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/StarksJohn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
