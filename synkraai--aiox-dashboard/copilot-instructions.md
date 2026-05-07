## aiox-dashboard

> This is the AIOS Platform UI — a Vite + React + TypeScript dashboard for orchestrating AI agent squads.

# AIOS Platform — Claude Code Instructions

## Project

This is the AIOS Platform UI — a Vite + React + TypeScript dashboard for orchestrating AI agent squads.

## Tech Stack

- **Runtime**: Vite 7 + React 19 + TypeScript
- **Styling**: Tailwind CSS + Liquid Glass design system (CSS custom properties)
- **State**: Zustand stores (`src/stores/`)
- **Testing**: Vitest + React Testing Library

## Design System Squad

The design system squad is available locally at `.squads-design/` (git-ignored).
Use it as reference for tokens, components, accessibility, and design decisions.

### Key resources

| Resource | Path |
|----------|------|
| Squad config | `.squads-design/config.yaml` |
| Agents (Brad Frost, Dave Malouf, Dan Mall) | `.squads-design/agents/` |
| Design tokens spec | `.squads-design/data/design-tokens-spec.yaml` |
| Component quality checklist | `.squads-design/checklists/ds-component-quality-checklist.md` |
| Accessibility checklist (WCAG) | `.squads-design/checklists/ds-accessibility-wcag-checklist.md` |
| Coding standards | `.squads-design/config/coding-standards.md` |
| Atomic design principles | `.squads-design/data/atomic-design-principles.md` |
| Tasks (slash commands) | `.squads-design/tasks/` |

### Design System Rules

- NEVER hardcode color, spacing, radius, shadow, or font values
- ALWAYS use CSS variables from the token system defined in `src/styles/liquid-glass.css`
- Token tiers: **Primitive** → **Semantic** → **Component**
- Before creating ANY new UI element, check existing components in `src/components/ui/`
- EVERY interactive element MUST be keyboard accessible
- EVERY form input MUST have an associated label
- EVERY icon-only button MUST have `aria-label`
- Color contrast MUST meet WCAG AA (4.5:1 normal text, 3:1 large text)

### Available Slash Commands (from squad tasks)

Use these task files as detailed instructions when performing design system work:

- `ds-audit-codebase` — Audit existing codebase for DS adoption
- `ds-build-component` — Build a new design system component
- `ds-compose-molecule` — Compose atoms into a molecule
- `ds-extract-tokens` — Extract design tokens from codebase
- `ds-setup-design-system` — Bootstrap a new design system
- `ds-generate-documentation` — Generate component documentation
- `ds-critical-eye-inventory` — Inventory UI patterns
- `ds-critical-eye-score` — Score component quality
- `ds-critical-eye-compare` — Compare implementations
- `a11y-audit` — Full accessibility audit
- `contrast-matrix` — Color contrast matrix audit
- `bootstrap-shadcn-library` — Bootstrap shadcn component library

For the full list, see `.squads-design/config.yaml` → `tasks:` section.

## AIOX Cockpit Theme

The AIOX Cockpit is a brutalist dark theme inspired by the [AIOX Brandbook](https://github.com/oalanicolas/aiox-brandbook). Activated via `data-theme="aiox"` on `<html>`.

### Brandbook Source

The cloned brandbook lives at `packages/aiox-brandbook/`. It serves as visual SSOT but is NOT directly imported — tokens are mirrored into the platform's CSS var system.

### Theme Files

| File | Purpose |
|------|---------|
| `src/styles/tokens/themes/aiox.css` | Palette primitives + semantic token overrides |
| `src/styles/tokens/themes/aiox-components.css` | Component-level overrides (buttons, cards, inputs, tables) |
| `src/styles/tokens/themes/aiox-animations.css` | Keyframes (aiox-spin, aiox-pulse, aiox-ticker, etc.) + glow utilities |
| `src/styles/tokens/themes/aiox-patterns.css` | Decorative patterns (dot-grid, crosshair, HUD brackets, scanlines) |
| `src/styles/fonts/aiox-fonts.css` | TASAOrbiterDisplay + Roboto Mono font imports |

### Cockpit Components

Located at `src/components/ui/cockpit/`. These use inline styles with `--aiox-*` CSS vars and are **only** styled correctly when `data-theme="aiox"` is active.

| Component | Description |
|-----------|-------------|
| `CockpitBadge` | 5 variants: lime, surface, error, blue, solid |
| `CockpitButton` | 4 variants, 3 sizes, loading state |
| `CockpitSpinner` | 3 sizes with aiox-spin animation |
| `CockpitKpiCard` | Metric card with label/value/trend |
| `CockpitAlert` | 4 variants with border-left accent |

### AIOX Design Rules

- `border-radius: 0` — brutalist, no rounded corners
- `font-family: var(--font-family-mono)` — Roboto Mono for labels/body
- `font-family: var(--font-family-display)` — TASAOrbiterDisplay for headings/values
- `text-transform: uppercase; letter-spacing: 0.08em` — for labels and buttons
- Neon lime (#D1FF00) accent on near-black (#050505) background
- Specificity: `html[data-theme="aiox"]` (0-1-1) beats `.dark` (0-1-0)

### Pattern CSS Classes (aiox-patterns.css)

- `.pattern-dots`, `.pattern-dots-dense`, `.pattern-dots-sparse` — dot grid backgrounds
- `.pattern-crosshair` — crosshair grid overlay
- `.pattern-hud-corners` — HUD corner brackets (clip-path)
- `.pattern-hazard`, `.pattern-hazard-subtle` — hazard stripe borders
- `.divider-tech`, `.divider-dashed`, `.divider-labeled` — section separators
- `.pattern-scanlines` — CRT scanline effect
- `.frame-tech` — chamfered clip-path container
- `.frame-bracket` — bracket-style container border

### Animation CSS Classes (aiox-animations.css)

- `.glow-lime`, `.glow-neon` — box-shadow glow effects
- `.text-gradient-neon` — lime gradient text
- `.anim`, `.anim-left`, `.anim-right`, `.anim-scale` — scroll-reveal (add `.visible` to trigger)
- `.delay-1` through `.delay-5` — stagger delays (100ms increments)

### Token Validation

Run `npx tsx scripts/validate-brandbook-tokens.ts` to verify token parity between brandbook and platform.

## Code Conventions

- Components in `src/components/` organized by feature domain
- Shared UI primitives in `src/components/ui/`
- Hooks in `src/hooks/`
- Stores in `src/stores/`
- API services in `src/services/api/`
- Use `cn()` from `src/lib/utils` for conditional class merging
- Use glass-* CSS classes for the glassmorphism design language

---
> Source: [SynkraAI/aiox-dashboard](https://github.com/SynkraAI/aiox-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
