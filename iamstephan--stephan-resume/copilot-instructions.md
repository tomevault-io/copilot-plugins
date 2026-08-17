## stephan-resume

> This repository builds Stephan Burger's resume as a PDF. It is a Bun + React project that uses `@react-pdf/renderer` to render a static React component tree into `dist/stephan-burger-resume.pdf`.

# Repository Guidelines

## Project Overview

This repository builds Stephan Burger's resume as a PDF. It is a Bun + React project that uses `@react-pdf/renderer` to render a static React component tree into `dist/stephan-burger-resume.pdf`.

It is not a browser app: there is no `index.html`, CSS entrypoint, web server, or client-side runtime state.

## Architecture & Data Flow

Build flow:

1. `index.tsx` runs under Bun.
2. `registerFonts()` from `src/helpers/font.ts` registers remote Inter and Roboto Mono fonts.
3. Bun ensures `dist/` exists.
4. `renderToBuffer(<Resume />)` renders the React PDF tree.
5. Bun writes `dist/stephan-burger-resume.pdf`.

Source/data flow:

- `src/main.tsx` composes the resume document.
- `src/helpers/**` provides font registration, style conversion, metadata, and shared data.
- Static resume content and assets flow into the React PDF render tree.
- Styling flows through `cn(...)` in `src/helpers/tw.ts`, which merges Tailwind-like class strings and converts them into React PDF style objects.

Async work is limited to the build/render boundary and asset loading. Most source files should stay stateless, declarative, and data-down.

## Key Directories

- `src/` - all application source.
- `src/sections/` - resume section modules; document the folder, not individual section names.
- `src/components/` - reusable PDF UI primitives; document the folder, not individual component names.
- `src/helpers/` - font registration, style conversion, metadata, and shared data.
- `src/assets/` - static assets consumed by PDF rendering.
- `.agents/skills/` - vendored resume/job-search assistant skills; useful for resume-content work, not application architecture.
- `dist/` - generated PDF output; ignored by git.

No `tests/`, `scripts/`, or `.github/` directories are present.

## Development Commands

Use Bun. The lockfile is `bun.lockb`; do not switch package managers unless explicitly asked.

```bash
bun install
bun --watch index.tsx  # package script: bun run dev
bun index.tsx          # package script: bun run build
```

Package scripts from `package.json`:

- `bun run dev` - watches and reruns `index.tsx`.
- `bun run build` - generates `dist/stephan-burger-resume.pdf`.

There are no configured `test`, `lint`, or typecheck scripts.

## Code Conventions & Common Patterns

Formatting:

- Prettier config: 2 spaces, spaces over tabs, no semicolons.
- TypeScript is strict (`tsconfig.json`) with `noEmit` and Bun types.
- Imports use path aliases from `tsconfig.json`; keep new aliases aligned there instead of relying on relative path chains.

React/PDF patterns:

- Prefer `@react-pdf/renderer` primitives (`Document`, `Page`, `View`, `Text`, `Link`, `Image`) over DOM elements.
- Keep the render tree declarative and mostly stateless.
- Keep resume content text-first and easy to review.
- Avoid browser CSS assumptions. React PDF style support is not identical to web CSS.

Styling:

- Always pass class-like styling through `cn(...)` from `@/helpers`.
- `cn(...)` handles `clsx`, `tailwind-merge`, React PDF Tailwind conversion, empty styles, and line-height normalization to `pt` strings.
- `tailwind.config.js` is IntelliSense-only (`content: []`); runtime style behavior comes from `src/helpers/tw.ts`.

Assets/fonts:

- Register fonts before rendering via `registerFonts()` in `index.tsx`.
- Add static render assets under `src/assets/` and keep TypeScript declarations in `src/types.d.ts` aligned with imported asset types.

## Important Files

- `package.json` - Bun scripts, dependencies, package entry metadata.
- `bun.lockb` - Bun dependency lockfile.
- `index.tsx` - build entrypoint and PDF writer.
- `src/main.tsx` - resume document composition.
- `src/helpers/tw.ts` - Tailwind-like class merging and React PDF style conversion.
- `src/helpers/font.ts` - Inter and Roboto Mono font registration.
- `src/helpers/info.ts` - personal/contact data.
- `src/helpers/document.ts` - PDF metadata object.
- `src/types.d.ts` - ambient asset module declarations.
- `tsconfig.json` - strict TypeScript, Bun types, path aliases.
- `.prettierrc` - formatting rules.
- `tailwind.config.js` / `postcss.config.js` - tooling support, not a browser CSS pipeline.

## Runtime/Tooling Preferences

- Required runtime/package manager: Bun.
- Use `bun run build` to create the PDF artifact.
- Use Bun APIs already present in the codebase (`Bun.$`, `Bun.write`, `Bun.file`) instead of replacing them with Node `fs` APIs.
- Keep ESM style (`"type": "module"`).
- Keep TypeScript path aliases aligned with `tsconfig.json`.
- Do not add web-app tooling unless the task explicitly changes the project type.
- Generated output belongs in `dist/`, which is gitignored.

## Git

- Never include AI tools or assistants as commit co-authors.
- Use semantic commit messages, such as `fix: correct PDF line-height handling` or `feat: add resume project entry`.
- Group commits logically by intent; do not mix unrelated content, tooling, and cleanup changes in one commit.

## Testing & QA

No automated test framework is configured. No test/spec files or test-runner configs are present.

Practical QA for changes:

1. Run `bun run build`.
2. Confirm `dist/stephan-burger-resume.pdf` is regenerated.
3. Visually inspect the PDF for layout, page breaks, typography, links, and icon rendering.

High-risk areas for manual checks:

- Remote font loading from `fonts.gstatic.com`.
- Line-height and PDF style conversion.
- Layout and page-break regressions after copy changes.
- Link and asset rendering in the generated PDF.

If adding tests later, prefer behavior-level tests around PDF generation or pure helpers. Do not add shallow tests that only restate the current component tree.

---
> Source: [IamStephan/stephan-resume](https://github.com/IamStephan/stephan-resume) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
