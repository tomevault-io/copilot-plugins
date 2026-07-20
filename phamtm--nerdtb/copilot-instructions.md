## nerdtb

> Source lives under `src/app`, mirroring the Next.js App Router layout: `layout.tsx` for shared frames, `page.tsx` for screens, and `globals.css` for design tokens. Keep UI assets in `public/` (e.g., icons, Open Graph images) and prefer colocating feature-specific components with their hooks and styles inside feature folders. Add reusable utilities under `src/lib` (create the folder if missing) so pages stay lean; aim for straight-line functions instead of deeply nested branches, and split complex logic into small, testable modules.

# Repository Guidelines

## Project Structure & Module Organization
Source lives under `src/app`, mirroring the Next.js App Router layout: `layout.tsx` for shared frames, `page.tsx` for screens, and `globals.css` for design tokens. Keep UI assets in `public/` (e.g., icons, Open Graph images) and prefer colocating feature-specific components with their hooks and styles inside feature folders. Add reusable utilities under `src/lib` (create the folder if missing) so pages stay lean; aim for straight-line functions instead of deeply nested branches, and split complex logic into small, testable modules.

## Build, Test, and Development Commands
- `npm run dev`: launches the local Next 16 server on port 3000 with hot reload.
- `npm run build`: produces the optimized production bundle; run before opening a PR.
- `npm run start`: serves the output of `npm run build` for release verification.
- `npm run lint`: executes Biome checks (style + static analysis).
- `npm run format`: rewrites files using Biome’s formatter (2-space indent, trimmed whitespace).

## Coding Style & Naming Conventions
Follow Biome defaults: 2-space indentation, single quotes in TSX/TS, trailing commas where valid. Prefer descriptive PascalCase component names (`NavSidebar`), camelCase helpers (`buildNavLinks`), and kebab-case file names for pages (`src/app/settings/page.tsx`). Keep components under 80 lines; extract hooks or helpers before logic becomes nested. Document every exported struct or type alias with a high-level comment that explains the data flow it supports.

## Testing Guidelines
A test runner is not yet wired; when adding coverage, introduce Vitest plus React Testing Library and expose it via an `npm run test` script. Colocate specs as `*.test.tsx` beside the component, prefer table-driven cases, and focus on routing logic, accessibility, and pure helpers before styling. If snapshots are necessary, pair them with explicit assertions (text, aria roles) so regressions stay obvious. Document any newly added mocks or fixtures inside the PR description.

## Commit & Pull Request Guidelines
- Follow the Conventional Commits spec for every commit subject (`feat: add hero grid`, `fix: correct theme tokens`) so automated tooling can reason about history.
- Keep commits logically scoped and reference the relevant issue ID in the body when available.
- Pull requests should include: purpose summary, screenshots or recordings for UI tweaks, a checklist of commands executed (`npm run lint`, `npm run build`), and a note about any follow-up tasks for future contributors.

---
> Source: [phamtm/nerdtb](https://github.com/phamtm/nerdtb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
