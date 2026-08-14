## personal-portfolio-astro

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev          # Start dev server at localhost:4321
npm run build:node   # Build for Node (local preview)
npm run preview      # Preview Node build locally
npx astro check      # TypeScript type check
```

No lint or test scripts are configured.

## Architecture

**Astro 5 SSR portfolio** deployed to Vercel (adapter switches to Node when `--node` flag is passed to build). Single-page app with React 19 islands for interactive components.

### Routing & i18n

- Every page has one URL — no `/en/` or `/es/` prefixes
- Middleware (`src/middleware.ts`) resolves language per request: (1) `lang` cookie, (2) `Accept-Language` header, (3) `'en'` default — no redirects
- Resolved lang stored in `Astro.locals.lang` (`'en' | 'es'`) — typed via `App.Locals` in `src/types/env.d.ts`
- All copy lives in `src/i18n/en.json` and `src/i18n/es.json` (flat key-value maps)
- `src/i18n/ui.ts` wraps JSON imports and re-exports `ui`, `languages`, `defaultLang`
- Translation helper: `useTranslations(lang)` from `src/i18n/utils.ts`
- Each component co-locates its i18n keys in a `.const.ts` file (e.g., `Header.const.ts` exports `HEADER_I18N_KEYS`)
- Language switcher sets `lang` cookie + `window.location.reload()` — URL never changes
- Blog post body content renders as authored (no translation); only UI chrome follows `Astro.locals.lang`

### Component hierarchy (atomic design)

```
src/components/
├── buttons/     # Atoms
├── form/inputs/ # Atoms
├── cards/       # Molecules
├── form/        # Molecules
└── sections/    # Organisms (page sections)
```

Each component lives in its own directory with `ComponentName.tsx` (React) or `ComponentName.astro`, a `ComponentName.module.scss`, and an `index.ts` re-export. The top-level `src/components/index.ts` barrel-exports everything.

### Styling

- **Tailwind CSS v4** (via `@tailwindcss/vite`) for utility classes
- **SCSS modules** (`.module.scss`) per component for scoped styles
- Theme variables in `src/styles/theme/` — import with `@use "../../../styles/theme/main.scss" as main`
- Responsive mixins: `@import "../../styles/theme/mixins"` then `@include mobile {}` / `@include tablet {}`
- Global styles entry: `src/styles/global.scss`

### SVG icons

Import as React components using the `?react` query (handled by `vite-plugin-svgr`):

```typescript
import IconName from "../../../assets/icons/icon-name.svg?react";
<IconName className={styles["icon-class"]} />
```

### Path aliases (tsconfig.json)

`@assets`, `@components`, `@layouts`, `@pages`, `@styles`, `@consts`, `@types`, `@data`, `@schemas`, `@utils` — all resolve to their respective `src/` subdirectories.

### Contact form

Built with `react-hook-form` + `zod` (schema in `src/schemas/contact-form.ts`) + SendGrid (`@sendgrid/mail`). Toast notifications via `react-hot-toast`.

### TypeScript conventions

- Interface names use `I` suffix (e.g., `ExperienceI`)
- Props interfaces extend HTML element types when applicable
- Strict mode enabled (`astro/tsconfigs/strict`)

### Astro client directives

Use strategically to control hydration:
- `client:load` — critical interactive components
- `client:visible` — below-fold components (performance)

<!-- SPECKIT START -->
## Active Feature Plan

**Current feature**: i18n JSON Migration & Browser Language Detection
**Plan**: [specs/001-i18n-json-migration/plan.md](specs/001-i18n-json-migration/plan.md)
**Spec**: [specs/001-i18n-json-migration/spec.md](specs/001-i18n-json-migration/spec.md)
<!-- SPECKIT END -->

---
> Source: [hmmatus/personal-portfolio-astro](https://github.com/hmmatus/personal-portfolio-astro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
