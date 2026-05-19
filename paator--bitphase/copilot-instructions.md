## general

> Bitphase is a chiptune tracker for creating music on retro sound chips. Built with Svelte 5, TypeScript, Vite, and Tailwind 4.


Bitphase is a chiptune tracker for creating music on retro sound chips. Built with Svelte 5, TypeScript, Vite, and Tailwind 4.

## Architecture

### Directory Structure

- `src/lib/chips/` - Chip implementations (currently AY-8910). Each chip has: `schema.ts` (channel/field definitions), `adapter.ts` (data manipulation), `renderer.ts` (pattern display), `types.ts`
- `src/lib/models/` - Domain models: `project.ts`, `song.ts`, `project-fields.ts`
- `src/lib/services/` - Business logic: `audio/`, `file/` (import/export), `pattern/` (editing, navigation, clipboard), `project/`, `modal/`
- `src/lib/stores/` - Reactive state using `.svelte.ts` files with `$state` rune
- `src/lib/ui-rendering/` - Canvas-based renderers for pattern editor and order list
- `src/lib/components/` - UI components organized by feature (Menu, Song, Tables, Instruments, Modal, etc.)
- `src/lib/config/` - App configuration (menu definitions, settings)
- `tests/` - Test files mirroring src structure

### Key Patterns

- **Chip abstraction**: Never hardcode chip-specific features. Use `src/lib/chips/base/schema.ts` for generic definitions. Chips implement adapters and renderers extending base classes.
- **State management**: Use Svelte 5 runes (`$state`, `$derived`, `$effect`) in `.svelte.ts` files. Do not use writable stores.
- **Pattern editing**: Field-based editing system in `src/lib/services/pattern/editing/` with strategies per field type.

### External Dependencies

- `external/ayumi/` - AY-8910 emulator C code, compiled to WASM via Emscripten
- WASM output goes to `public/ayumi.wasm`

### Path Alias

`@` maps to `./src` (configured in vite and vitest)

- Svelte 5 syntax only (runes, `onclick` not `on:click`)
- No comments in code - write self-documenting code
- Tailwind 4 for styling
- Follow KISS, DRY, SOLID principles. Keep very good OOP practices.
- Tests go in `tests/` directory mirroring `src/` structure

Currently bitphase supports only AY-8910/YM2149F chip, but architecture needs to support other chips in the future. Therefore, it is important to keep the code modular and easy to extend. Using AY specific code in generic parts is not allowed, because when we get to the point of adding another chip, we will have to rewrite a lot of code.

Please never allow lint errors. If you see any, fix them. We had lots of production bugs because of lint errors.

---
> Source: [paator/bitphase](https://github.com/paator/bitphase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
