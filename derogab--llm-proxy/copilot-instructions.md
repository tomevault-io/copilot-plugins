## llm-proxy

> This file provides guidance to agents when working with code in this repository.

# Agent Guidelines

This file provides guidance to agents when working with code in this repository.

## Build Commands

```bash
npm install          # Install dependencies
npm run build        # Build all outputs (CJS, ESM, and types)
npm run build:cjs    # Build CommonJS output only
npm run build:esm    # Build ESM output only
npm run build:types  # Build type declarations only
```

## Test Commands

```bash
npm test              # Run all tests (unit + integration)
npm run test:unit     # Run unit tests only
npm run test:llama    # Run Llama.cpp integration tests only
npm run test:watch    # Run tests in watch mode
npm run test:coverage # Run tests with coverage report
```

**Important**: Always run `npm test` after making changes to verify nothing is broken. Tests are located in the `test/` folder.

Tests are written using Vitest and cover:
- Provider selection logic (OpenAI, Cloudflare, Ollama priority)
- Error handling for all providers
- Message format conversion for Llama.cpp
- API request formatting

## Architecture

This is a TypeScript npm package (`@derogab/llm-proxy`) that provides a unified interface for multiple LLM providers. The entire implementation is in a single file: `src/index.ts`.

### Provider Selection

The `generate()` function automatically selects a provider based on environment variables in this priority order:
1. **OpenAI** - if `OPENAI_API_KEY` is set
2. **Cloudflare AI** - if `CLOUDFLARE_ACCOUNT_ID`, `CLOUDFLARE_AUTH_KEY`, and `CLOUDFLARE_MODEL` are all set
3. **Ollama** - if `OLLAMA_URI` is set
4. **Llama.cpp** - if `LLAMA_CPP_MODEL_PATH` is set

### Build Output

The package builds to three output formats:
- `dist/cjs/` - CommonJS (for `require()`)
- `dist/esm/` - ES Modules (for `import`)
- `dist/types/` - TypeScript declarations

## Conventional Commits

This project follows the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) specification.

### Format

```
<type>: <description>
```

### Types

| Type | Description |
|------|-------------|
| `feat` | New feature or functionality |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `test` | Adding or modifying tests |
| `chore` | Maintenance tasks (dependencies, configs) |
| `build` | Changes to build system or dependencies |
| `perf` | Performance improvement |
| `ci` | CI/CD configuration changes |

### Examples

```bash
feat: add support for new LLM provider
fix: resolve timeout issue with Ollama requests
docs: update README with configuration examples
refactor: simplify provider selection logic
test: add unit tests for error handling
chore: update dependencies
```

### Breaking Changes

Add `!` after the type for breaking changes:

```bash
feat!: change API response format
```

## Proxy Collection

This package is part of a family of plug-and-play proxy packages:

- [`@derogab/llm-proxy`](https://github.com/derogab/llm-proxy) - LLM provider proxy (this package)
- [`@derogab/stt-proxy`](https://github.com/derogab/stt-proxy) - Speech-to-Text provider proxy

**Important**: Before implementing any changes, you MUST check the other projects in this family to see if similar changes have already been made. If so, follow the same structure, patterns, and specifications used there. This ensures consistency across all proxy packages in the long term.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/derogab)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/derogab)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
