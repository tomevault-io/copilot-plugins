## stickers

> - `npm run dev` - Start development server

# AGENTS.md

## Build Commands
- `npm run dev` - Start development server
- `npm run build` - Build for production (runs TypeScript check + Vite build)
- `npm run tauri build` - Build Tauri application
- `cargo build` - Build Rust backend
- `npx tsc --noEmit` - Run TypeScript type checking

## Test Commands
No test framework configured. Add Jest/Vitest for unit tests.

## Code Style Guidelines

### TypeScript/React
- Use functional components with hooks
- Strict TypeScript: no unused locals/parameters, strict mode enabled
- Async/await for Tauri commands: `await invoke("command", { params })`
- Import order: React, third-party libraries, local imports
- Naming: camelCase for variables/functions, PascalCase for components
- Error handling: Use try/catch for async operations

### Rust
- Standard Tauri command pattern: `#[tauri::command]`
- Use serde for serialization with `#[derive(Serialize, Deserialize)]`
- Error handling: Use `Result<T, E>` and `expect()` for critical errors

### General
- Single-note focused design
- Clean, distraction-free interface
- Notion-inspired typography and styling
- Always-on-top window behavior
- No toolbar, keyboard shortcuts only
- No title input, content only
- No comments unless necessary
- Follow existing patterns in codebase
- Use absolute imports for clarity

---
> Source: [zackzhou-work/stickers](https://github.com/zackzhou-work/stickers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
