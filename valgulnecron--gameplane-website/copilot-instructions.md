## gameplane-website

> Public marketing + docs website for Gameplane, mounted as the `website/` submodule of the main repo (`ValgulNecron/Gameplane`). Static Astro site, TypeScript strict, Tailwind CSS 4, no client framework.

# gameplane-website — guidance for AI coding assistants

Public marketing + docs website for Gameplane, mounted as the `website/` submodule of the main repo (`ValgulNecron/Gameplane`). Static Astro site, TypeScript strict, Tailwind CSS 4, no client framework.

## House rules (inherited from the main Gameplane repo)

- **Design-first**: the visual source of truth is the **Group/Public Website** screen group in the main repo's `design.pen` (Pencil, encrypted — MCP tools only, never Read/edit the file directly). Update the design there before changing this site's visual surface.
- **Tests/lints run in CI, not locally.** Compile checks are fine locally (`npm run build`, `npm run check`); don't run the lint suite locally — push and let `ci.yaml` gate it. Fix what CI flags; never add suppression directives or loosen rules.
- **Commits**: signed (`git commit -s`), conventional prefixes (`feat:`, `fix:`, `chore:`, `docs:`, `ci:`), one logical unit per commit. One branch per unit of work; PR to `main`; delete the branch after merge.
- TypeScript strict; `@typescript-eslint/no-explicit-any` and `no-floating-promises` are errors.

## Architecture rules

- **Semantic tokens only**: components use Tailwind utilities backed by the CSS custom properties in `src/styles/global.css` (`bg-background`, `text-foreground`, `text-primary`, …) — never raw hex. This is what makes the light-themed band (`.theme-light` scope) on the landing page work.
- **Every internal link goes through `withBase()`** (`src/lib/url.ts`). The site deploys under `/gameplane-website` on GitHub Pages; bare root-relative hrefs 404. Markdown cross-doc links are written relative (`./architecture/`).
- Docs pages are a content collection (`src/content/docs/*.md`, schema in `src/content.config.ts`); sidebar order and prev/next both derive from `src/lib/docs-nav.ts` — adding a docs page is frontmatter-only.
- OG/canonical meta need absolute URLs built from `Astro.site`.
- Deployment config (site origin, base path, GitHub URL, displayed version) lives in `src/config.ts`.

## Development

When starting the dev server, use background mode:

```
astro dev --background
```

Manage the background server with `astro dev stop`, `astro dev status`, and `astro dev logs`.

## Documentation

Full documentation: https://docs.astro.build

- [Routing](https://docs.astro.build/en/guides/routing/)
- [Astro components](https://docs.astro.build/en/basics/astro-components/)
- [Content collections](https://docs.astro.build/en/guides/content-collections/)
- [Styling / Tailwind](https://docs.astro.build/en/guides/styling/)

---
> Source: [ValgulNecron/gameplane-website](https://github.com/ValgulNecron/gameplane-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
