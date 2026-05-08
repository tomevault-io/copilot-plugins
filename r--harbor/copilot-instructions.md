## browser-priority

> Browser priority and testing requirements for Harbor


# Browser Priority

**Firefox is the PRIMARY browser for Harbor development and testing.**

## Key Requirements

1. **All browser testing must target Firefox first** - Chromium/Chrome is secondary
2. **Extensions are built for Firefox by default** - `npm run build` produces Firefox builds
3. **Demo pages should be tested in Firefox** with both extensions loaded
4. **Safari is the tertiary target** for the macOS App Store distribution

## Extension Architecture

Harbor consists of TWO browser extensions that work together:

1. **Harbor Extension** (`extension/`) - Core infrastructure providing:
   - LLM provider management
   - MCP server hosting
   - Native bridge communication
   - OAuth flows

2. **Web Agents API Extension** (`web-agents-api/`) - Page-facing API providing:
   - `window.ai` - AI/LLM access for web pages
   - `window.agent` - Agent tools and browser control

Both extensions must be installed for full functionality.

## Build Commands

```bash
# Firefox (default)
npm run build

# Chrome (secondary)
npm run build:chrome

# Safari (tertiary)
npm run build:safari
```

## Testing in Firefox

Use `web-ext` to run Firefox with extensions:

```bash
cd extension && npx web-ext run --source-dir dist-firefox
```

For E2E tests with both extensions, see `tests/e2e/README.md`.

---
> Source: [r/Harbor](https://github.com/r/Harbor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
