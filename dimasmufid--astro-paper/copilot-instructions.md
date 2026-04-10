## astro-paper

> `src/pages` holds routed Astro pages, while `src/layouts` contains shared page shells and metadata helpers. UI fragments belong in `src/components`, and reusable logic (date helpers, string utilities) lives in `src/utils`. Blog content and configuration data are grouped under `src/data`, with touchpoints referenced through `content.config.ts`. Static assets (images, fonts, favicons) reside in `public`, and design tokens and Tailwind layers sit inside `src/styles`. Update only the files you touch and keep cross-cutting changes small to simplify reviews.

# Repository Guidelines

## Project Structure & Module Organization
`src/pages` holds routed Astro pages, while `src/layouts` contains shared page shells and metadata helpers. UI fragments belong in `src/components`, and reusable logic (date helpers, string utilities) lives in `src/utils`. Blog content and configuration data are grouped under `src/data`, with touchpoints referenced through `content.config.ts`. Static assets (images, fonts, favicons) reside in `public`, and design tokens and Tailwind layers sit inside `src/styles`. Update only the files you touch and keep cross-cutting changes small to simplify reviews.

## Build, Test, and Development Commands
- `pnpm install`: sync dependencies and Astro content collections.
- `pnpm dev`: run the local dev server at the port defined in `astro.config.ts`.
- `pnpm build`: execute `astro check`, compile the site, generate Pagefind search assets, then copy them to `public/pagefind`.
- `pnpm preview`: serve the production build for sanity checks.
- `pnpm lint`: lint Astro, TypeScript, and JSX islands with ESLint.
- `pnpm format` / `pnpm format:check`: apply or verify Prettier formatting with Astro and Tailwind plugins.

## Coding Style & Naming Conventions
Use TypeScript or `.astro` components with four-space indentation (respect existing style). Prefer named exports from utility modules and PascalCase for components (`src/components/Hero.astro`). Page routes mirror filenames (`src/pages/blog/[slug].astro`), and content entries should remain kebab-case to match URL slugs. Tailwind classes power styling; avoid ad-hoc inline styles unless required. Run the formatter before pushing.

## Testing Guidelines
There is no standalone unit test suite today. Rely on `pnpm build` (which runs `astro check`) to catch typing and schema issues, and manually verify key flows via `pnpm preview`. When adding validation or data transforms, consider colocating lightweight tests in a `__tests__` folder under the relevant module so future automation can adopt them seamlessly.

## Commit & Pull Request Guidelines
Follow Conventional Commits (`cz.yaml` config) such as `feat: add reading time badge` or `fix: correct rss feed metadata`. Keep commits scoped to a single concern and include context in the body when touching content collections or config. Pull requests should summarize user-facing changes, link related issues, and include screenshots or URLs for visual updates. Confirm `pnpm lint` and `pnpm build` pass before requesting review.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dimasmufid)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dimasmufid)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
