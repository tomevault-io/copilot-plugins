## src-core

> Core source files - main entry point, types, and constants


- TypeScript strict mode enabled - use explicit types, avoid `any`
- All public APIs must have JSDoc comments with `@public` tag
- Internal types/constants use `@internal` tag in JSDoc
- Use ES modules (`import`/`export`) exclusively
- Class-based API: `AmbientGlow` is the main exported class
- Main modules (`index.ts`, `types.ts`, `constants.ts`) must include `@module` and `@packageDocumentation` tags
- Other modules use only `@module` tag
- Constants should be exported from `constants.ts` as `as const` objects
- Type definitions belong in `types.ts` - use interfaces for public types, `@internal` for private
- Public API methods must have usage examples in JSDoc `@example` blocks
- Export types alongside the class for external consumption
- Use descriptive error messages with "AmbientGlow:" prefix (errors) or console.warn for warnings
- Follow the pattern: constructor → setup → event handlers → cleanup in `destroy()`
- @file src/index.ts

---
> Source: [mike-at-redspace/video-ambient-glow](https://github.com/mike-at-redspace/video-ambient-glow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
