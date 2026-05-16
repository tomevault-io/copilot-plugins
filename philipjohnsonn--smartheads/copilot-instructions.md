## smartheads

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- **Dev server**: `npm run dev` (starts on port 8080)
- **Build**: `npm run build` (production) or `npm run build:dev` (development mode)
- **Lint**: `npm run lint`
- **Preview**: `npm run preview`

No test framework is configured.

## Architecture

This is the **SmartHeads** corporate website — a React SPA built with Vite, TypeScript, Tailwind CSS, and shadcn/ui. Originally scaffolded with Lovable.

- **Domain**: `https://smartheads.com.ar/`
- **Sections**: Hero (video bg), About (misión/visión/valores), Solutions, Partners, Industries/Clients, Contact, Footer
- **Solution pages**: system-integration, ai-data, oil-gas-logistics (SAP), cybersecurity, business-consulting

### Routing

React Router v6 in `src/App.tsx`. The homepage is `src/pages/Index.tsx`; solution pages live under `/solutions/*`. Add new routes above the `*` catch-all.

### Key Directories

- `src/components/` — site-wide components (Header, Footer, HeroSection, ContactSection, ClientsSection, SolutionsSection, etc.)
- `src/components/ui/` — shadcn/ui primitives (do not edit directly; managed via `npx shadcn-ui@latest add <component>`)
- `src/pages/` — route-level page components
- `src/hooks/` — custom hooks (`use-mobile`, `use-toast`)
- `src/lib/utils.ts` — `cn()` helper (clsx + tailwind-merge)

### Path Aliases

`@` maps to `./src` (configured in `vite.config.ts` and `tsconfig.app.json`). Use `@/components/...`, `@/lib/...`, etc.

### Styling

Tailwind CSS with CSS variables for theming (defined in `src/index.css`). Colors use `hsl(var(--variable))` pattern. The shadcn/ui config is in `components.json` (style: default, base color: slate). Key custom colors: `--navbar: 220 7% 16%` (#262830), `--primary` (gold/dorado).

### Providers

App wraps content in: `QueryClientProvider` (TanStack Query), `TooltipProvider`, `Toaster` (radix + sonner), `BrowserRouter`.

### Meta / SEO

Meta tags configured in `index.html`. OG image at `public/smartheads-og.jpg` (1200x630, 46KB). All OG/Twitter image URLs must be absolute (`https://smartheads.com.ar/...`).

---
> Source: [PhilipJohnsonn/smartheads](https://github.com/PhilipJohnsonn/smartheads) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
