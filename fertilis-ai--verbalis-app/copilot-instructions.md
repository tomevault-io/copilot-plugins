## verbalis-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Verbalis-app is a Turborepo/Bun monorepo (`apps/web` = React + Vite frontend and Tauri
desktop runtime; `packages/*` = shared config and env). Read `package.json`, `turbo.json`,
and `apps/web/vite.config.ts` for the current stack and versions.

**Vision and design principles live in `SPEC.md`** — the product goals (local-first,
privacy by default, progressive autonomy, multi-agent) are specified there, not here.

Domain skills for this repo: `pi-skill` (pi-mono AI features), `shadcn-skill` (UI
components), `tanstack-skill` (Router and Form), `tauri-skill` (desktop runtime).

App logs are written to `~/.verbalis/logs/`.

## Code Editors & Syntax Highlighting

**Editor text must never depend on syntax highlighting.** `components/files/file-editor.tsx`
(Workspace) and `components/toolbox/toolbox-editor.tsx` (Toolbox) layer a transparent
`<textarea>` over a Shiki HTML overlay injected with `dangerouslySetInnerHTML`. The
line-number gutter is independent DOM, so when the overlay is empty the panel renders
**only line numbers** — a bug this project has hit more than once. Rules:

- The textarea is `text-transparent` **only** when `highlightedHtml` is non-empty; otherwise
  `text-foreground`. Never make it unconditionally transparent.
- Every `highlightCode()` call needs a `.catch` that falls back to
  `<pre><code>${escapeHtml(content)}</code></pre>`.
- Never cache a rejected highlighter or grammar promise (`lib/highlighter.ts`) — one failure
  would otherwise blank every editor for the rest of the session.
- Keep `leading-5` on the gutter, the overlay (`[&_code]:leading-5`), and the textarea, or
  the numbers drift out of alignment. In a `cn()` call `leading-5` must come **after**
  `text-sm`; tailwind-merge drops a preceding `leading-*` because Tailwind's text-size
  utilities also set line-height.
- Regression coverage lives in `components/editor-highlight-fallback.test.tsx`.

**Shiki must use the JavaScript regex engine, not Oniguruma WASM.** The packaged app's CSP
(`src-tauri/tauri.conf.json`) is `script-src 'self'` with no `'wasm-unsafe-eval'`, so WASM
compilation is blocked and the WASM engine fails at runtime. Two places depend on this:

- `lib/highlighter.ts` passes `createJavaScriptRegexEngine({ forgiving: true })` explicitly.
  Do not remove it. `lib/highlighter.csp.test.ts` blocks `WebAssembly` globally and asserts
  highlighting still works — it fails if the engine option is dropped.
- Chat code blocks go through `@streamdown/code`, which builds its **own** highlighter.
  Versions **< 1.0.3 use the WASM engine** and break in the packaged app. Keep it at
  `>= 1.0.3`. Note `>= 1.1.0` changes the plugin's `themes` type to `ThemeInput` and needs a
  `streamdown` newer than 2.1.0 — bump both together or stay on 1.0.3.

Do not introduce other WASM dependencies without also updating the CSP.

**`devCsp` is `null`, so dev mode does not enforce the CSP.** Any webview-dependent change
(WASM, inline scripts, external origins, `connect-src` targets) must be verified in a
packaged build — `cd apps/web && bunx tauri build --bundles dmg` — not just `bun run dev:web`.

## Notes

- Tests use Vitest (jsdom). `bun run quick_test` runs a fast subset, not the whole suite
- Biome is linter-only — the **formatter is disabled**, so do not reach for `biome format`
- Bun is the package manager - do not use npm/yarn/pnpm
- The `routeTree.gen.ts` file is auto-generated - never edit manually
- Use `bunx tauri icon ~/assets/fertilis_logo_white.png` to generate app icons
- Use `bunx tauri build --bundles dmg` to bundle a new mac release (.dmg)

---
> Source: [fertilis-ai/verbalis-app](https://github.com/fertilis-ai/verbalis-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
