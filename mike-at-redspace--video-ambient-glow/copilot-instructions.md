## src-lib

> Internal library modules - canvas, frame processing, event handling


- All exports must be marked `@internal` in JSDoc - these are not part of public API
- Functions should be pure and focused on single responsibilities
- Canvas operations go in `canvas.ts` - creation, styling, context handling
- Frame processing logic in `frameProcessor.ts` - color extraction, blending
- Event handling in `eventHandlers.ts` - factory functions for event listeners
- Use named exports, not default exports
- All functions need JSDoc with parameter descriptions and return types
- Canvas contexts must be checked with type guards before use
- Error handling: throw descriptive errors with context about which canvas/operation failed
- Use constants from `../constants.ts` instead of magic numbers
- Functions should accept options/types as parameters rather than accessing globals
- All lib modules use `@module` tag (not `@packageDocumentation`)
- Re-export via `src/lib/index.ts` barrel export
- Prefer functional approach - avoid classes in lib modules
- @file src/lib/canvas.ts
- @file src/lib/frameProcessor.ts
- @file src/lib/eventHandlers.ts

---
> Source: [mike-at-redspace/video-ambient-glow](https://github.com/mike-at-redspace/video-ambient-glow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
