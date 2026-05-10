## birdeye-design-system

> Read this file at the start of every session. Follow it strictly.

# Aero Design System — AI Instructions

Read this file at the start of every session. Follow it strictly.

---

## Project Structure

This project follows a strict tiered spec + token architecture.
**Never deviate from this structure.** When adding anything new,
place it in the correct tier and create the matching spec file.

```
aero-design-system/
├── specs/
│   ├── foundations/           # Tier 1: Visual primitives
│   │   ├── color.md
│   │   ├── typography.md
│   │   ├── spacing.md
│   │   ├── radius.md
│   │   ├── elevation.md
│   │   ├── motion.md
│   │   ├── z-index.md
│   │   ├── iconography.md      ← add when icons are introduced
│   │   ├── accessibility.md    ← add when a11y audit happens
│   │   ├── breakpoints.md      ← add when responsive tokens land
│   │   ├── grid.md             ← add when layout system is built
│   │   ├── borders.md
│   │   └── opacity.md
│   │
│   ├── tokens/               # Tier 1: CSS variable reference
│   │   ├── token-reference.md  (master map of ALL tokens)
│   │   ├── color-tokens.md
│   │   ├── spacing-tokens.md
│   │   ├── typography-tokens.md
│   │   ├── elevation-tokens.md
│   │   └── motion-tokens.md
│   │
│   ├── atoms/                # Tier 2: Smallest UI components
│   │   └── (one .md per component: button.md, input.md, etc.)
│   │
│   ├── molecules/            # Tier 2: Composed components
│   │   └── (one .md per component: tabs.md, modal-dialog.md, etc.)
│   │
│   ├── organisms/            # Tier 2: Product-level assemblies
│   │   └── (one .md per component: table.md, navigation.md, etc.)
│   │
│   └── patterns/             # Tier 3: Layout & composition rules
│       └── (one .md per pattern: form-layout.md, list-detail.md, etc.)
│
├── src/
│   ├── tokens/
│   │   ├── tokens.css          ← unified 3-layer token system
│   │   ├── colors.css          ← color CSS variables
│   │   ├── typography.css      ← type utility classes
│   │   ├── elevation.css       ← shadow utility classes
│   │   ├── index.ts            ← TypeScript token exports
│   │   ├── typography.ts
│   │   └── elevation.ts
│   ├── components/
│   │   └── [Name]/
│   │       ├── [Name].tsx
│   │       ├── [Name].css       ← Layer 3: ONLY var(--token) refs
│   │       ├── [Name].stories.tsx
│   │       └── index.ts
│   └── index.ts
│
├── tokens.css                  ← root copy of token system
├── scripts/
│   └── token-audit.cjs         ← CI-ready hardcoded-value scanner
├── .storybook/
└── CLAUDE.md                   ← THIS FILE
```

---

## Rules

### Rule 1: Spec before code

Before writing or modifying any UI code, **read the relevant spec
file** in `specs/`. If no spec exists for what you're building,
**create it first**, then write the code.

### Rule 2: Token-only CSS

Component CSS (Layer 3) must **only** reference Layer 2 alias
tokens from `tokens.css`. Never hardcode:

| Property | Use token |
|---|---|
| Colors (hex, rgb, rgba) | `var(--color-*)` |
| Spacing (padding, margin, gap) | `var(--space-*)` |
| Font sizes | `var(--text-*)` |
| Font weights | `var(--font-weight-*)` |
| Font families | `var(--font-family)` or `var(--font-family-mono)` |
| Border radius | `var(--radius-*)` |
| Box shadows | `var(--shadow-*)` |
| Z-index | `var(--z-*)` |
| Transition durations | `var(--duration-*)` |
| Easing | `var(--easing-*)` |
| Opacity | `var(--opacity-*)` |
| Transforms | `var(--transform-*)` |

### Rule 3: Audit before commit

```bash
node scripts/token-audit.cjs
```
**Zero errors required.** Warnings should be addressed when practical.

### Rule 4: Correct tier placement

When adding a new component, place it in the right tier:

| Tier | Directory | What goes here | Example |
|---|---|---|---|
| Atom | `specs/atoms/` | Single-purpose, no children | Button, Input, Badge, Avatar |
| Molecule | `specs/molecules/` | Composed of atoms | Tabs, Modal, Dropdown, Tooltip |
| Organism | `specs/organisms/` | Product-level assemblies | Table, Navigation, Dashboard Card |
| Pattern | `specs/patterns/` | Layout & composition rules | Form Layout, List-Detail, Grid |

### Rule 5: Only spec what exists

**Do not create placeholder or empty spec files.** Every spec file
must describe something that is implemented in `src/`. When a new
component is added to `src/`, its spec is created at the same time.
When a foundation is added to `tokens.css`, its spec is created at
the same time. Empty tiers keep their directory but no files.

### Rule 6: Spec file template

Every component spec must follow this structure:

```markdown
# [Component Name]

## Metadata
- **Name:** ComponentName
- **Category:** Atom | Molecule | Organism | Pattern
- **Status:** Stable | Beta | Deprecated
- **Source:** `src/components/[Name]/[Name].tsx`

## Overview
### When to use
### When NOT to use

## Anatomy
[Named parts of the component]

## Tokens Used
| Token | What it controls |
|---|---|

## Props / API
| Prop | Type | Default | Description |
|---|---|---|---|

## States
| State | Appearance | Token changes |
|---|---|---|

## Code Example
[TSX usage]

## Cross-References
[Related specs]
```

### Rule 7: Token architecture (3 layers)

- **Layer 1** (`--ds-*`): Primitive values. Never use in components.
- **Layer 2** (semantic aliases): `--color-text-primary`, `--space-4`, etc. Components use ONLY this layer.
- **Layer 3**: Component CSS files — `var()` references to Layer 2.

When adding a new token:
1. Add primitive in Layer 1 of `src/tokens/tokens.css`
2. Add semantic alias in Layer 2 referencing Layer 1 with fallback
3. Update root `tokens.css`
4. Update `specs/tokens/token-reference.md`
5. Run the audit

### Rule 8: Foundation spec template

Every foundation spec must follow this structure:

```markdown
# [Foundation Name]

## Overview
[What this covers]

## Tokens
| Token | Value | Description |
|---|---|---|

## Usage Guidelines
[When and how to use]

## Do's and Don'ts

## Related
[Links to related specs]
```

---

## What exists today

### Foundations (from Figma)
- **Color** — 6 families (Blue, Red, Purple, Green, Gray, Yellow), 18 shades each
- **Typography** — Roboto (web/Android), SF Pro (iOS), scale from 12px–32px
- **Elevation** — 4 shadow levels (xs, sm, md, lg) + hover shadow

### Derived foundations (from token audit)
- Spacing, Radius, Motion, Z-index, Borders, Opacity

### Components
- **Button** — primary, secondary, outline, danger × sm/md/lg
- **ColorPalette** — swatch grid for documenting colors
- **Typography** — text rendering component with level/weight
- **ElevationShowcase** — shadow level documentation cards

---

## Checklist for adding anything new

- [ ] Identify the correct tier (foundation / atom / molecule / organism / pattern)
- [ ] Create the spec file in the right `specs/` subdirectory
- [ ] If new tokens are needed, add to both layers of `tokens.css`
- [ ] Update `specs/tokens/token-reference.md`
- [ ] Write component code in `src/components/[Name]/`
- [ ] CSS uses ONLY `var(--token)` references — zero raw values
- [ ] Add Storybook stories
- [ ] Export from `src/index.ts`
- [ ] Run `node scripts/token-audit.cjs` — zero errors

---
> Source: [raynilkumar-bs/Birdeye-Design-System](https://github.com/raynilkumar-bs/Birdeye-Design-System) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
