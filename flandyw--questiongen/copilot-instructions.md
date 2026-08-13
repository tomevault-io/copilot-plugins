## questiongen

> - **Frameworks**: Tauri 2 (Rust) + React 19 (Vite, TS, Tailwind 4).

# QuestionGen Agent Guide

## High-Signal Context

- **Frameworks**: Tauri 2 (Rust) + React 19 (Vite, TS, Tailwind 4).
- **Domain**: VCE (Victorian Certificate of Education) exam question generation.
- **AI**: Login with ChatGPT through a bundled local Tauri sidecar.
- **State**: Zustand (slices pattern) in `src/store/`. `src/store.ts` is the
  aggregate entry point.
- **Math Architecture**: MathJax 4 for rendering.
  - **Shielding**: Frontend uses `shieldMathForMarkdown`
    (`src/lib/math-normalization.ts`) to replace `$..$` with tokens before
    markdown parsing.
  - **Protection**: Backend Rust (`src-tauri/src/parsing.rs`) uses
    `protect_latex_in_raw_json` to prevent JSON escapes (e.g., `\f` in `\frac`)
    from mangling LaTeX before `serde_json` parsing.
  - **Cleaning**: `clean_field` in Rust normalizes delimiters and repairs LLM
    LaTeX errors (e.g., `\fty` -> `\infty`).

## Critical Commands

- **Frontend Check**: `bun run lint && bun run typecheck`
- **Backend Tests**: `cd src-tauri && cargo test` (Crucial for LaTeX protection
  logic).
- **Dev**: `bun tauri dev` (Starts desktop app).

## Architecture & Entrypoints

- **Tauri Bridge**: Commands in `src-tauri/src/lib.rs`. Main generation service
  in `src-tauri/src/generation.rs`.
- **Generation Orchestration**: `src/lib/generator-batch.ts` (Batching/Variety
  logic).
- **Styling**: Tailwind 4 (CSS-first). Global styles and theme imports in
  `src/themes/index.css`.
- **Fonts**: Spline Sans Variable (interface), JetBrains Mono Variable
  (technical).

## Quirks & Constraints

- **Lockfiles**: `bun.lock` exists. ALWAYS use `bun` for package operations.
- **PDFs**: `reports/` is the reference directory for examiners' reports
  (excluded from git). Exam-paper references have been intentionally removed.
- **Anki**: Native `.apkg` generation via `genanki-rs` in
  `src-tauri/src/anki.rs`.
- **Supabase**: Auth/sync lives in `src/context/SupabaseSyncContext.tsx`; the
  initial database and private image bucket migration is under `supabase/`.
- **CSP**: Disabled in `src-tauri/tauri.conf.json` (`"csp": null`). Required
  because the app loads MathJax 4 from a CDN via `<script>` tag and uses inline
  scripts for theme injection in `index.html`.

---
> Source: [flandyw/questiongen](https://github.com/flandyw/questiongen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
