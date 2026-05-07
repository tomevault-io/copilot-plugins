## opencode-plugin-mempalace

> **Generated:** 2026-04-07

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-04-07
**Project:** opencode-plugin-mempalace

## OVERVIEW
An OpenCode plugin that integrates MemPalace's "lifetime memory" (L0-L3 memory stack, AAAK compression, auto context saving) into the OpenCode terminal assistant. This plugin will use the `execa` library to wrap the local `mempalace` Python CLI to maintain 100% alignment with official logic.

## STRUCTURE
```
.
├── src/
│   ├── index.ts          # Plugin entry point, registers OpenCode hooks
│   ├── mempalace-cli.ts  # Wrapper for Python MemPalace CLI calls
│   ├── state.ts          # Session state management (e.g. message counters)
│   └── utils.ts          # Utility functions for inferring wings from paths
├── package.json          # Node project metadata
└── README.md             # Installation and usage instructions
```

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| Plugin Hooks | `src/index.ts` | Intercepts `experimental.session.compacting`, `chat.message`, `experimental.chat.system.transform` |
| CLI Invocation | `src/mempalace-cli.ts` | All `execa` calls to `mempalace` go here |
| State Mgmt | `src/state.ts` | Keep track of message count per session to trigger saves |
| CLI Utils | `src/utils.ts` | Determine the current `wing` based on workspace directory |

## CONVENTIONS
- Use `@opencode-ai/plugin` to integrate into OpenCode lifecycle.
- Fail gracefully: If `mempalace` CLI is missing, the plugin should warn via logs and do nothing rather than crash the session.
- Token Limits: Truncate injected memories to avoid overloading the LLM context.
- Concurrent Writes: Use debounce or simple lock mechanism to prevent simultaneous saves during rapid chat messages.

## COMMANDS
```bash
npm run build   # Build TypeScript to dist/
npm run test    # Run TDD test suite
npm run lint    # ESLint and Prettier
```

## NOTES
- This is an independent NPM package designed to be loaded via `plugin_origins` in `.opencode/config.json`.
- Uses TDD mode for development. Test first, implement second.

---
> Source: [option-K/opencode-plugin-mempalace](https://github.com/option-K/opencode-plugin-mempalace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
