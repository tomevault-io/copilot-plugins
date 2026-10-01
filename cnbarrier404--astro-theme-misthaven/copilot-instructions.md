## astro-theme-misthaven

> Misthaven is a clean and minimalist Astro blog theme with calm colors, spacious layouts, and a comfortable reading experience. It is designed for sharing, ideas, and everyday life.

# AGENTS.md

Misthaven is a clean and minimalist Astro blog theme with calm colors, spacious layouts, and a comfortable reading experience. It is designed for sharing, ideas, and everyday life.

## Commands

| Command                   | Action                                           |
| :------------------------ | :----------------------------------------------- |
| `npm install`             | Installs dependencies                            |
| `npm run dev`             | Starts local dev server at `localhost:4321`      |
| `npm run build`           | Build your production site to `./dist/`          |
| `npm run preview`         | Preview your build locally, before deploying     |
| `npm run astro ...`       | Run CLI commands like `astro add`, `astro check` |
| `npm run astro -- --help` | Get help using the Astro CLI                     |
| `npm run format`          | Format code                                      |
| `npm run check`           | Check types                                      |

Manage the background server with `npm run astro dev stop`, `npm run astro dev status`, and `npm run astro dev logs`.

After completing work, run `npm run format` and `npm run check`. After source changes, also run `npm run build` and `git diff --check`.

## Repository Structure

| Path                | Purpose                                                                          |
| :------------------ | :------------------------------------------------------------------------------- |
| `.astro/`           | Astro-generated development metadata; should not be edited manually              |
| `dist/`             | Generated production build output for deployment; should not be edited manually  |
| `public/`           | Static assets copied directly to the final build without processing              |
| `src/`              | Main source code for pages, components, layouts, styles, content, and site logic |
| `astro.config.mjs`  | Astro configuration, integrations, Markdown processing, and build settings       |
| `package.json`      | Project metadata, scripts, dependencies, and supported Node.js version           |
| `package-lock.json` | Locked npm dependency graph; update it together with dependency changes          |
| `tsconfig.json`     | TypeScript and Astro compiler configuration                                      |

## Source Layout

`src/` contains the main source code, content, styles, and configuration for the Astro site.

| Path            | Purpose                                                                     |
| :-------------- | :-------------------------------------------------------------------------- |
| `assets/`       | Images, fonts, and other assets processed and optimized by Astro            |
| `components/`   | Reusable UI components grouped by responsibility                            |
| `config/`       | Site metadata, navigation, home page, footer, and integration settings      |
| `content/`      | Markdown content collections for posts and standalone pages                 |
| `i18n/`         | Translation keys, locale strings, language settings, and helpers            |
| `integrations/` | Theme-level Astro integrations, such as the Markdown Negotiation build hook |
| `layouts/`      | Shared document shells and article layout                                   |
| `pages/`        | File-based routes, dynamic routes, and server endpoints                     |
| `styles/`       | Global styles, design tokens, typography, and prose rules                   |
| `types/`        | Shared TypeScript types and interfaces                                      |
| `utils/`        | General-purpose helpers, including post querying and sorting                |

### Component Source Layout

`src/components/`:

| Directory  | Purpose                                                        |
| :--------- | :------------------------------------------------------------- |
| `content/` | Content presentation components such as cards, lists, and tags |
| `layout/`  | Page-wide structure and site shell components                  |
| `search/`  | Search interfaces and interactions                             |
| `ui/`      | Reusable UI primitives and small composed display components   |
| `widget/`  | Standalone page widgets, such as the table of contents         |

## Workflows

### 1. Implementing a Feature

1. Read the user's requirements and inspect the relevant existing code, configuration, content, and documentation.
2. Before editing implementation files, present a concrete plan that describes the intended behavior, affected areas, and validation steps.
3. Do not modify source code or other implementation files, and do not run commands that rewrite or generate files, until the user explicitly approves the plan and authorizes implementation. Read-only inspection is allowed.
4. After approval, implement only the approved scope. If a materially different approach or broader scope becomes necessary, explain the change and request approval again before proceeding.
5. Validate the implementation with the commands required by this document and report the result.

### 2. Fixing a Bug

1. Start from the user's problem description. Inspect the relevant code and, when practical, reproduce the issue using read-only or non-mutating diagnostics.
2. Identify the root cause and support the diagnosis with concrete evidence from the code or reproduction. Do not present an unverified guess as the cause.
3. Explain the root cause, affected behavior, proposed fix, and validation plan to the user. Wait for explicit approval before modifying source code or other implementation files.
4. After approval, implement the fix within the agreed scope, add or update regression coverage when appropriate, and run the required validation commands.

## UI and Styling

- Use Tailwind utilities for component layout and visual styling. Keep global CSS for base styles, design tokens, prose,
  third-party integrations, and selectors that cannot be expressed clearly as utilities.
- Use only semantic theme colors defined in `src/styles/global.css`. Do not use Tailwind's default color palette or raw
  color literals in components.
- Prefer integer spacing utilities so layout, spacing, padding, and gaps remain multiples of `4px`.
- Use only the `tablet` and `wide` responsive variants. Do not add one-off breakpoints in component markup.
- Do not use arbitrary values when a theme token or standard utility exists. Add a reusable token when the same custom
  value is needed more than once.
- Keep letter spacing at `0`. Use font size, weight, and color for hierarchy.
- Reuse repeated visual structures as Astro components. Do not hide copied utility lists behind `@apply` or new CSS
  classes.

## URL Conventions

- The site enforces trailing-slash-free URLs (`trailingSlash: "never"` in `astro.config.mjs`). Every page URL must end
  without a trailing `/`, except the site root `/`.
- When generating or writing links (navigation, footer, cards, RSS, sitemap, redirects, or any `href`), never append a
  trailing `/` to non-root paths. Prefer the project's existing path helpers (e.g. `getPostHref`) over hand-building
  URLs so this convention stays consistent.

## Documentation

Full documentation: https://docs.astro.build

Consult these guides before working on related tasks:

- [Adding pages, dynamic routes, or middleware](https://docs.astro.build/en/guides/routing/)
- [Working with Astro components](https://docs.astro.build/en/basics/astro-components/)
- [Using React, Vue, Svelte, or other framework components](https://docs.astro.build/en/guides/framework-components/)
- [Adding or managing content](https://docs.astro.build/en/guides/content-collections/)
- [Adding styles or using Tailwind](https://docs.astro.build/en/guides/styling/)
- [Supporting multiple languages](https://docs.astro.build/en/guides/internationalization/)

---
> Source: [CnBarrier404/astro-theme-misthaven](https://github.com/CnBarrier404/astro-theme-misthaven) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
