## kokoro-engine

> Kokoro Engine is a Tauri v2 app with a React/TypeScript frontend and Rust backend. Frontend code lives in `src/`: `main.tsx` and `App.tsx` are entry points, `src/components/ui` holds primitives, `src/features` holds feature modules, `src/lib` holds bridge/services/tests, `src/ui` holds product UI/locales, and `src/windows` holds extra Tauri windows. Backend code lives in `src-tauri/src`: IPC is in `commands`, with domain modules such as `llm`, `tts`, `stt`, `vision`, `mods`, `mcp`, and `ai`. Assets are in `public`, `pictures`, and `src/assets`; sample mods are in `mods`; docs are in `docs`.

# Repository Guidelines

## Project Structure & Module Organization
Kokoro Engine is a Tauri v2 app with a React/TypeScript frontend and Rust backend. Frontend code lives in `src/`: `main.tsx` and `App.tsx` are entry points, `src/components/ui` holds primitives, `src/features` holds feature modules, `src/lib` holds bridge/services/tests, `src/ui` holds product UI/locales, and `src/windows` holds extra Tauri windows. Backend code lives in `src-tauri/src`: IPC is in `commands`, with domain modules such as `llm`, `tts`, `stt`, `vision`, `mods`, `mcp`, and `ai`. Assets are in `public`, `pictures`, and `src/assets`; sample mods are in `mods`; docs are in `docs`.

## Build, Test, and Development Commands
- `npm install`: install JavaScript dependencies.
- `npm run tauri dev`: run the full desktop app with Vite.
- `npm run dev`: run the Vite frontend only.
- `npm run build`: typecheck with `tsc` and build frontend assets.
- `npm run tauri build`: build a distributable Tauri app.
- `npm test`: run Vitest unit tests.
- `cargo test --manifest-path src-tauri/Cargo.toml`: run Rust tests.
- `cargo clippy --manifest-path src-tauri/Cargo.toml --lib -- -D warnings`: check Rust warnings before review.

## Coding Style & Naming Conventions
Use strict TypeScript, React function components, and `@/*` imports when they improve clarity. Match existing two-space TypeScript indentation and Rust `rustfmt` defaults. Name React components and TSX files with `PascalCase`, hooks as `useSomething`, utility files in local style such as `audio-player.ts`, and Rust modules/files with `snake_case`. Keep user-facing strings in `src/ui/locales/*.json`.

## Testing Guidelines
Vitest tests live beside frontend code as `*.test.ts` or `*.test.tsx`. Rust tests are inline `#[cfg(test)]` modules or files such as `src-tauri/src/vision/tests/*.rs`. Add focused tests for changed behavior, especially IPC bridges, providers, memory, chat, audio, and vision. Run the targeted suite for your area plus broader checks when risk is shared, for example `npm test` and `cargo test --manifest-path src-tauri/Cargo.toml llm`.

## Commit & Pull Request Guidelines
Recent history uses short imperative subjects, sometimes with prefixes such as `docs:` or `chore(app):`. Follow that style: `Fix stale vision context` or `docs: update setup notes`. PRs should describe behavioral impact, list test commands run, link related issues, and include screenshots or GIFs for visible UI changes. Call out config, model, or network requirements.

## Security & Configuration Tips
Do not commit secrets, local databases, model files, generated `dist`, or `target*` directories. Keep provider tokens in local config or environment variables. Review changes to permissions, file access, command execution, and remote provider calls carefully.

---
> Source: [chyinan/Kokoro-Engine](https://github.com/chyinan/Kokoro-Engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
