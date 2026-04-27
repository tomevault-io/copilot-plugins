## girls-in-tech-brazil

> Last updated: 2026-03-10

# girls-in-tech-brazil Development Guidelines

Last updated: 2026-03-10

## Quick Context

- **Stack**: TypeScript 5.x, Node.js >= 22.x, Next.js 15.x App Router, React 19.x
- **Styling**: Tailwind CSS 4.x — design system "Pulso BR Tech" (violet/rose/gold, Outfit font)
- **Animations**: Framer Motion — parallax, 3D tilt spring, scroll reveals, animated counters
- **Data layer**: Zod 4 schema + JSON files in `src/data/creators/`
- **Validation flow**: `npm run validate` → `npm run lint` → `npm run type-check` → `npm run build`
- **Deploy**: Static export (`output: 'export'`) for GitHub Pages

## Active Technologies

- TypeScript 5.x, Node.js >= 22.x
- Next.js 15.x (App Router, static export), React 19.x
- Tailwind CSS 4.x (`@theme` block, no `tailwind.config.js`)
- Framer Motion (motion components, `useScroll`, `useSpring`, `useMotionValue`)
- ESLint 9.x, Prettier 3.x
- Zod 4.x for data contracts
- lucide-react, clsx, tailwind-merge
- TSX for TypeScript script execution

## Progressive Disclosure

Default reading order for new tasks:

1. Read this file first.
2. **Creator data** → `src/schemas/creator.schema.ts`, `src/lib/creators.ts`, `src/data/creators/`, `scripts/validate-data.ts`
3. **UI components** → `src/components/`, `src/styles/globals.css`, `src/app/layout.tsx`
4. **Animations** → `src/lib/motion-variants.ts`, `src/hooks/use-count-up.ts`, `src/components/scroll-progress.tsx`, `src/components/section-reveal.tsx`
5. **Contribution / CI** → `CONTRIBUTING.md`, `.github/PULL_REQUEST_TEMPLATE.md`, `.github/workflows/ci.yml`
6. **Feature planning** → active spec under `specs/`
7. Only go deeper when the task actually needs it.

## Project Structure

```text
.github/
  workflows/           # CI + deploy workflows (PR validation and GitHub Pages publish)
docs-spec/             # Epic documentation and PRDs
public/
  images/creators/     # optional local avatars
scripts/
  validate-data.ts     # Zod dataset validator
specs/                 # feature specs per epic
src/
  app/                 # Next.js App Router pages
  components/          # React UI components (16 total)
    avatar.tsx
    badge.tsx
    category-filter.tsx
    creator-card.tsx    # 3D tilt with spring physics
    cta-banner.tsx
    discovery-experience.tsx
    empty-state.tsx
    footer.tsx
    header.tsx          # scroll-shadow via useMotionTemplate
    hero.tsx            # parallax + stagger animations
    index.ts
    pagination.tsx
    scroll-progress.tsx # rainbow gradient scroll bar
    search-bar.tsx
    section-reveal.tsx  # whileInView fade-slide
    spotlight-grid.tsx  # stagger container for card grids
    stats-banner.tsx    # animated counters
  data/creators/       # JSON profiles — source of truth
  lib/
    avatar.ts
    cn.ts
    content-types.ts
    creators.ts         # centralized dataset access
    discovery.ts
    motion-variants.ts  # shared Framer Motion variants
  hooks/
    use-count-up.ts     # animated counter hook
  schemas/
    creator.schema.ts   # Zod schema — single validation source
  styles/
    globals.css         # @theme tokens, design system CSS
```

## Commands

- `npm run dev`: start local development
- `npm run validate`: validate creator dataset locally
- `npm run validate:ci`: validate dataset in CI mode
- `npm run lint`: run ESLint
- `npm run type-check`: run TypeScript checks (no emit)
- `npm run build`: verify production static export
- `npm run format`: format with Prettier

## Data Layer Notes

- Source of truth for creators is `src/data/creators/*.json`
- Files starting with `_` are non-public and ignored by dataset loading
- `contentTypes` is derived from `links`, not manually authored in JSON
- Dataset access is centralized in `src/lib/creators.ts`
- Validation rules are centralized in `src/schemas/creator.schema.ts`
- `featured: true` marks creators highlighted in the spotlight section

## UI / Design System Notes

- Design system name: **Pulso BR Tech**
- Palette: Violet primary `#8b5cf6`/`#7c3aed`, Rose accent `#f43f5e`, Gold `#f59e0b`, base `#fdfcff`
- Font: `Outfit` via `next/font/google`, variable `--font-outfit`
- Tailwind tokens live in the `@theme {}` block inside `globals.css` — no `tailwind.config.js`
- Background: morphing CSS blobs (`.bg-blob-violet/rose/gold`) + apex ceiling glow — all in `globals.css`
- All `'use client'` components use Framer Motion; server components stay animation-free

## Contribution Flow Notes

- New creator contributions should follow [CONTRIBUTING.md](CONTRIBUTING.md) guidelines
- PRs should use [PULL_REQUEST_TEMPLATE.md](.github/PULL_REQUEST_TEMPLATE.md)
- PR validation runs through [ci.yml](.github/workflows/ci.yml)
- Deploy to GitHub Pages runs through [deploy-pages.yml](.github/workflows/deploy-pages.yml) on push/merge to `main`
- Review ownership for critical infra is enforced by [CODEOWNERS](.github/CODEOWNERS)
- Branch protection expectations are documented but configured in GitHub settings

---
> Source: [Bullas/girls-in-tech-brazil](https://github.com/Bullas/girls-in-tech-brazil) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
