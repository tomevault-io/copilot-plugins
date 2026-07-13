## promptwright

> - ESM only: always use `.js` extension in imports

# Source of truth: CLAUDE.md
# See CLAUDE.md and packages/*/CLAUDE.md for detailed coding instructions.
# This file exists for Cursor IDE compatibility.

# Promptwright — QA Assistant powered by GitHub Copilot SDK
# TypeScript, pnpm monorepo, Electron + React, Playwright MCP, CDP

## Key Rules
- ESM only: always use `.js` extension in imports
- Module resolution: NodeNext, no path aliases
- Node.js 22+ required
- Never log full MCP configs or Copilot SDK payloads (contain secrets)
- Desktop app must run in Electron, not browser
- Renderer accesses backend only through `window.jarvis`
- After UI changes: `pnpm build && pnpm test:e2e:smoke`

---
> Source: [sahajamit/promptwright](https://github.com/sahajamit/promptwright) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-13 -->
