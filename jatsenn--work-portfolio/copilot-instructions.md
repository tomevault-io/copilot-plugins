## work-portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm start                 # ng serve — dev server at http://localhost:4200
npm run build             # ng build — production build to dist/portfolio
npm run watch             # ng build --watch --configuration development
npm test                  # ng test — Karma/Jasmine unit tests (watches by default)
```

To run a single test file, use Karma's focus syntax (`fdescribe`/`fit`) in the spec, or:
```bash
node node_modules/.bin/ng test --include='**/home-page.component.spec.ts'
```

There is no lint script configured (no ESLint config in the repo) — don't invoke `npm run lint`.

## Architecture

Angular 19 standalone-components portfolio site, feature-first organization:

```
src/app/
  core/layout/        # navbar, footer — global chrome rendered outside route content
  features/
    home/
      data/home-content.ts     # single source of truth for ALL site copy/content
      pages/home-page/         # single-page flow: Hero, About, Services, Resume, Portfolio, Testimonials, Contact
    project-detail/pages/project-detail-page/   # /project/:id — detail view driven by ProjectItem
    book-call/pages/book-call-page/             # /book-call
    send-message/pages/send-message-page/       # /send-message — uses @emailjs/browser
  shared/models/portfolio.model.ts  # typed interfaces for all content (PortfolioContent, ProjectItem, etc.)
  app.routes.ts        # route table (all routes eagerly loaded, no lazy loading currently)
  app.config.ts        # providers: zone change detection, router, animations
src/styles/
  abstracts/_tokens.scss    # CSS custom properties (colors, spacing)
  base/                     # reset, typography, utilities
  themes/_default.scss
  main.scss                 # imported by top-level src/styles.scss
```

**Content model**: All portfolio content (bio, experience, education, skills, projects, contact info) lives in `src/app/features/home/data/home-content.ts` as one `HOME_CONTENT: PortfolioContent` object, typed against `src/app/shared/models/portfolio.model.ts`. When adding/editing site content, edit this file rather than hardcoding strings in templates. Project entries (`ProjectItem`) carry both summary-card fields (for the home page grid) and detail-page fields (`role`, `timeline`, `sections`, `images`) consumed by `project-detail-page`.

**Routing**: `/` (home), `/project/:id`, `/book-call`, `/send-message` — see `app.routes.ts`.

## Conventions

- Standalone components only (`standalone: true`), `ChangeDetectionStrategy.OnPush` by default.
- Selector prefix `app-`; files kebab-case.
- Import `CommonModule` / `RouterLink` directly in components that need them (no shared module).
- CSS: never hardcode hex colors in component SCSS — use the CSS custom properties defined in `src/styles/abstracts/_tokens.scss` and `src/styles/themes/_default.scss` (`--color-bg`, `--color-surface`, `--color-text`, `--color-muted`, `--color-accent`, `--card-bg`, `--card-border`, `--grad-heading`, etc.).
- Fonts: `Sora` for display/headings, `DM Mono` for mono/labels.
- Tech-stack/skill items in the UI must always pair a name with its icon — prefer Font Awesome brand icons (`fa-brands`) with the technology's official brand hex color; fall back to a small `<span class="tech-badge">` text badge only when no FA icon exists (e.g. TypeScript `TS`, Tailwind `~`, Vercel `▲`). Never render a tech name as bare text.
- Keep components under ~250 lines; split when they grow past that.
- Move data/business logic to services rather than components/templates where practical.

## Design direction

Editorial portfolio aesthetic: warm neutral background, near-black typography, orange accent. 8px spacing scale (8/16/24/32/40/48/64/80). Max content width 1140–1280px, centered. One primary (accent) button style and one secondary (ghost) style. Short, intentional animations (150–250ms). Desktop-first, then responsive down to the required breakpoints: mobile `<=767px`, tablet `768–1023px`, desktop `>=1024px`. Semantic landmarks (`header`, `main`, `section`, `footer`) and visible focus states are required.

---
> Source: [Jatsenn/work-portfolio](https://github.com/Jatsenn/work-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
