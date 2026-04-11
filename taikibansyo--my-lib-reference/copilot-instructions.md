## my-lib-reference

> - Source: `src/` with `app/` (Next.js App Router), `components/` (UI + shared), and `lib/` (utilities).

# Repository Guidelines

## Project Structure & Module Organization
- Source: `src/` with `app/` (Next.js App Router), `components/` (UI + shared), and `lib/` (utilities).
- Assets: `public/` for static files.
- Tests: `test/` with `*.test.ts[x]` and `test/vitest.setup.ts`.
- Config: root `tsconfig.json` (paths: `@/*` → `src/*`), `next.config.js`, `tailwind.config.js`, `vitest.config.ts`.
- Legacy snapshots: `charanko/`, `koudakaojisan/`, `yukiguni_shiitake/` are historical; not part of the current build.

## Build, Test, and Development Commands
- `npm run dev`: Start Next.js dev server at `http://localhost:3000`.
- `npm run build`: Production build (`.next/`).
- `npm start`: Serve the production build.
- `npm run lint`: Lint with `eslint-config-next`.
- `npm test`: Run Vitest suite (jsdom + Testing Library).
Examples: run a single test file `npx vitest run test/example.test.ts`; watch mode `npx vitest`.

## Coding Style & Naming Conventions
- Language: TypeScript (strict). Prefer 2-space indentation, semicolon-free style, and ES Modules.
- React: Functional components; export named or default as appropriate.
- Files: utilities/hooks kebab-case (e.g., `use-thing.ts`), components in `components/` may be PascalCase or kebab-case; prefer kebab-case for new files for consistency.
- Imports: use `@/` alias for `src` (e.g., `import { cn } from "@/lib/utils"`).
- CSS: Tailwind in JSX and `src/app/globals.css`. Dark mode via `class` strategy.

## Testing Guidelines
- Framework: Vitest + @testing-library/react + jsdom.
- Location: `test/` directory; name tests `*.test.ts` or `*.test.tsx`.
- Setup: extend matchers via `test/vitest.setup.ts`.
- Patterns: test behavior and accessibility; avoid implementation details. Example: `render(<Component />)` and assert via roles/text.

## Commit & Pull Request Guidelines
- Commits: Prefer Conventional Commits (`feat:`, `fix:`, `chore:`). Keep messages imperative and scoped.
- PRs: concise description, rationale, and scope; link issues; include screenshots/GIFs for UI changes; list breaking changes; ensure `npm run lint` and `npm test` pass.

## Security & Configuration Tips
- Environment variables: prefix client-exposed vars with `NEXT_PUBLIC_`; keep secrets out of VCS.
- Review `next.config.js` and `tailwind.config.js` when adding new tooling or paths.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/taikibansyo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/taikibansyo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
