## construct

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Construct** is a token-based, framework-agnostic design system (`@neuravision/construct`). It provides CSS component styles, design tokens, and accessibility-first UI patterns for use with any framework or vanilla HTML.

## Commands

```bash
npm run build              # Generate token outputs (tokens.css, tokens.json, tokens.ts)
npm run check              # Verify token outputs are up-to-date (CI-friendly)
npm run storybook          # Dev server at http://localhost:6006
npm run storybook:build    # Static Storybook build to storybook-static/
npx vitest --run           # Run tests headless (Playwright/Chromium)
npx vitest                 # Run tests in watch mode
```

## Architecture

### Token Pipeline (Two-Tier)

Source tokens in `tokens/*.json` are built into outputs via `scripts/build-tokens.mjs`:

```
primitives.json + semantic.{light,dark,high-contrast}.json
  → tokens.css   (CSS custom properties)
  → tokens.json  (resolved values with units)
  → tokens.ts    (TypeScript exports)
```

**Never hand-edit generated files** (`tokens/tokens.css`, `tokens/tokens.json`, `tokens/tokens.ts`). Edit the source JSON files and run `npm run build`.

### Themes

Three modes controlled via `data-theme` attribute: `light` (default), `dark`, `high-contrast`. Falls back to `prefers-color-scheme` and `prefers-contrast` system preferences.

### Components

Component styles are modular per-component files in `components/`. Each component has its own CSS file. The bundle entry point is `components/index.css` (imports all files). Legacy `components/components.css` redirects to `index.css`.

```
components/
├── _shared.css          ← :root control variables + shared form control base styles
├── _keyframes.css       ← ct-spin, ct-skeleton, ct-progress-indeterminate
├── button.css
├── field.css            ← Field, Label, Hint, Error, Counter
├── input.css            ← Input-Wrap, Input-Group, Addons
├── select.css           ← Select, Select-Wrap (native fallback)
├── select-menu.css      ← Select-Menu (custom dropdown select)
├── textarea.css
├── checkbox.css         ← Check + Radio
├── switch.css
├── card.css
├── table.css            ← Table + Table-Wrap
├── data-table.css
├── modal.css            ← Modal + Confirmation Dialog
├── toast.css            ← Toast + Toast-Region
├── tooltip.css
├── tabs.css
├── dropdown.css
├── pagination.css
├── breadcrumbs.css
├── datepicker.css       ← Datepicker + Range + Month/Year grid
├── badge.css
├── alert.css
├── banner.css           ← Banner / Callout (persistent page-level notices)
├── chip.css
├── file-upload.css
├── spinner.css          ← Spinner + Loading-Overlay
├── skeleton.css
├── icon.css
├── progress-bar.css
├── toolbar.css
├── sidebar.css          ← Sidebar-Layout + Sidebar + Nav-List
├── toggle-group.css
├── accordion.css
├── avatar.css           ← Avatar + Avatar-Group
├── empty-state.css
├── divider.css
├── slider.css
├── popover.css
├── combobox.css
├── skip-link.css
├── drawer.css
├── navbar.css           ← Navbar / App Header (responsive, dropdown menus, mobile toggle)
├── app-shell.css        ← App Shell V1 (CSS Grid layout shell: Navbar + Sidebar + Main + Panel + Footer)
├── app-shell-v2.css     ← App Shell V2 (Floating Canvas: rounded surfaces, gap-based, glass/branded)
├── list.css             ← List + List Item (rich items, selection, nav, dividers)
└── index.css            ← @import bundle for all components
```

Consumers can import `components/index.css` for everything, or individual files for selective imports.

- **Naming:** `ct-` prefix with BEM modifiers (e.g., `ct-button`, `ct-button__icon`, `ct-button--secondary`)
- **Sizes:** `--sm`, default (md), `--lg`
- **State:** Managed via data attributes (`data-state="open"`) and ARIA attributes (`aria-invalid`, `aria-selected`, `aria-disabled`)

### Storybook & Testing

- Stories in `stories/*.stories.js` using `@storybook/html-vite`
- Tests are story-driven via `@storybook/addon-vitest` (Vitest + Playwright)
- Add test coverage by creating/updating stories
- Accessibility addon (`@storybook/addon-a11y`) is configured to **fail on violations**

## Code Style

- 2-space indentation for JS and CSS
- Single quotes in JS
- CSS custom properties use kebab-case: `--color-brand-primary`
- Commits: Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `test:`)

## Accessibility Requirements

All components must use semantic HTML, support full keyboard navigation, meet WCAG 2.1 AA contrast, include proper ARIA attributes, and have visible focus indicators. See `docs/guidelines.md` for detailed patterns (roving tabindex, focus traps, arrow key navigation, etc.).

---
> Source: [Samyssmile/construct](https://github.com/Samyssmile/construct) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
