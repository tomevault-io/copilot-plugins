## tasksync

> - TypeScript strict mode is enabled — do not use `any` or suppress type errors with `as` casts


# TypeScript Conventions

## Strict Mode
- TypeScript strict mode is enabled — do not use `any` or suppress type errors with `as` casts
- Use `satisfies` over `as` for message types (e.g., `} satisfies ToWebviewMessage`)
- `satisfies` validates shape at compile time; `as` silently bypasses checks

## Style
- Indentation: tabs (enforced by Biome)
- Quotes: double quotes (enforced by Biome)
- Imports: auto-organized by Biome — do not manually reorder

## Async I/O
- Always prefer async file operations over synchronous equivalents
- Never use synchronous blocking calls on the VS Code extension host

## Logging
- Use `debugLog()` from `webviewUtils.ts` for debug output (gated behind `tasksync.debugLogging`)
- Use `console.error` only for genuine error/failure paths
- Never use `console.log` or `console.warn` in production code

## SSOT / DRY
- Constants live in `src/constants/` — do not duplicate config keys
- Shared types live in `webviewTypes.ts` — one canonical definition per type
- Shared helpers live in `webviewUtils.ts` — extract repeated logic (3+ call sites) into helpers
- Handler modules (`*Handlers.ts`) receive a `P` interface — do not import from `webviewProvider.ts` directly

## KISS / YAGNI
- Prefer the simplest solution that works
- Do not add abstraction layers without clear justification
- Do not add features or code paths "just in case"
- Small duplication is acceptable if the alternative is complex abstraction for only 2 call sites

---
> Source: [4regab/TaskSync](https://github.com/4regab/TaskSync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
