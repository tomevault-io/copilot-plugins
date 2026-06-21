## rust-for-frontend

> Guidance for coding agents working in this repository. (Claude Code reads `CLAUDE.md`, which mirrors this file.)

# AGENTS.md

Guidance for coding agents working in this repository. (Claude Code reads `CLAUDE.md`, which mirrors this file.)

## What this is

An interactive Rust tutorial **written for frontend engineers** (content is in Chinese). Every Rust concept is taught by analogy to JavaScript/TypeScript, with hand-built SVG visualizations for the hard parts (ownership, borrowing, lifetimes, iterators). Pure client-side SPA — no backend.

## Commands

```bash
pnpm install            # uses pnpm 11; esbuild needs an approved build script
pnpm dev                # vite dev server
pnpm build              # tsc -b (typecheck) + vite build → dist/
pnpm lint               # tsc -b --noEmit (typecheck only)
pnpm preview            # serve dist/ for production verification
```

There is **no test runner** — `tsc` is the only correctness gate, so always run `pnpm lint` (or `npx tsc -b`) after edits.

**pnpm gotcha:** `pnpm build`/`pnpm install` may fail on this machine with a corepack/dependency-self-check error unrelated to the code. Fall back to `npx tsc -b && npx vite build`. esbuild's build script must be approved (`onlyBuiltDependencies: [esbuild]` lives in `pnpm-workspace.yaml`, not package.json — pnpm 11 moved it).

To verify a chapter actually renders (not just compiles), serve `pnpm preview` and load `http://localhost:4173/#/learn/<slug>` — headless Chrome with `--virtual-time-budget` works for screenshots/DOM dumps.

## Architecture

**Single source of truth: `src/content/chapters.ts`.** The `chapters[]` array drives *everything* — sidebar, homepage outline, prev/next navigation, progress totals, and routing. Each entry has `slug`, `title`, `subtitle`, `group`, `minutes`, `icon`, and a `lazy(() => import(...))` Component. **Array order = learning path.** Chapters are bucketed into sidebar groups by their `group` string via `groupedChapters()`.

**Adding/reordering a chapter:** create `src/content/chapters/MyChapter.tsx` (default-export a component returning JSX fragments), then register it in `chapters.ts`. Nothing else needs touching. When reordering, fix the prose "下一章…" (next-chapter) transition lines at the end of affected chapters — they are hand-written, not derived.

**Routing:** Hash router (`createHashRouter`) for static-host deploy. `Layout` (sidebar + topbar + progress) wraps an `<Outlet/>`; `/learn/:slug` → `ChapterPage`, which looks up the chapter, renders its lazy `Component` in `<Suspense>`, and provides the mark-done button + prev/next nav.

**Progress:** `useProgress` hook persists completed slugs to `localStorage` under key `rff:completed`, with a module-level listener set so all hook instances stay in sync (no context provider).

### Component layers

- **`components/Ui.tsx`** — the teaching primitives every chapter composes from: `Callout` (kinds: tip/info/warn/danger/rust/js), `Compare` (side-by-side JS↔Rust code), `KeyTerm`, `Quiz` (self-contained stateful single-choice), `Figure` (wraps a visualization with title/caption), `Pill`.
- **`components/CodeBlock.tsx`** — syntax highlighting via `react-syntax-highlighter`'s `PrismLight`. **Languages are registered manually at the top of the file** to keep the bundle small (full Prism is ~600kB); add a `registerLanguage` line before using a new `lang`. Supports `highlight` lines and `output`. When `runnable && lang === 'rust'`, it delegates to **`RustRunner`** instead of static rendering.
- **`components/Lab.tsx`** — two exercise primitives. `MicroLab` (动手练习): a `RustRunner` preloaded with deliberately-broken/TODO starter code + collapsible hint + reference solution; learner fixes and runs to verify. `ErrorDrill` (报错训练): broken code + a real pasted `rustc` error snippet + a cause/fix `Quiz`. When adding either, verify on the live API that the starter actually fails (with the cited error code) and the solution produces the stated `expectedOutput` — both have been validated this way.
- **`components/RustRunner.tsx` + `lib/playground.ts`** — inline Rust execution. An overlay editor (transparent `<textarea>` over a `SyntaxHighlighter` highlight layer; both must share `EDITOR_FONT` metrics or the caret drifts) + a Run button that POSTs to the official `https://play.rust-lang.org/execute` API (CORS-open, no backend of ours) and renders stdout/stderr in-page. Code without `fn main` is auto-wrapped. This uses the public Playground's compute — keep it learning-only, don't batch/hammer it.
- **`components/viz/`** — the interactive SVG visualizations. Two patterns:
  - **Stepped animation:** `useSteps` hook + `StepperControls` (prev/next/reset/dots) + `framer-motion` for transitions (e.g. `OwnershipViz`, `RcViz`, `IteratorViz`). Each defines a `frames[]` array of states.
  - **Tabbed scenarios:** local `useState` switching between cases (e.g. `BorrowViz`, `ClosureViz`, `LifetimeViz`), using the `.scenario-tabs`/`.scenario-verdict` classes from `viz.css`.
  - **`Flow.tsx`** is a reusable node/edge flowchart primitive (auto-routes edges, supports box/round/diamond shapes and tone colors) used across chapters for pipelines.

### Styling

Plain CSS, no framework. Design tokens (colors, spacing, fonts) are CSS variables in `src/styles/global.css` — Rust caramel `--rust`, JS yellow `--js`, semantic `--ok/--warn/--err/--info`, dark `--bg-0..3`. Each component has a sibling `.css` file. `.prose` class styles long-form chapter text. Reuse tokens rather than hardcoding colors.

## Authoring conventions (important for content edits)

- Chapter components return a `<>…</>` fragment of `Ui`/`CodeBlock`/`viz` components — no top-level `<h1>`/title (those come from `ChapterPage` via the registry).
- **JSX brace escaping:** Rust format placeholders like `{}`, `{x}`, `{:?}` inside plain JSX text break parsing. Wrap inline mentions as `<code>{'{x}'}</code>`. Inside `CodeBlock` template-literal `code={...}` strings they're fine. Escape literal backticks in template strings as `` \` ``.
- Chapters typically close with a "本章要点 & 下一步" `Callout kind="info"` summarizing and pointing to the next chapter.
- Tutorial content is Chinese; keep the JS-analogy framing and the "对照式" (side-by-side `Compare`) style consistent with existing chapters.

---
> Source: [buynao/rust-for-frontend](https://github.com/buynao/rust-for-frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-21 -->
