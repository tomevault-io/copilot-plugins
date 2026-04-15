## cd-mmp

> **NEVER run any deployment commands without EXPLICITLY asking the user first and waiting for confirmation.**

# Cursor Rules — Car & Driver Marketplace

## DEPLOYMENT RULES (CRITICAL)

**NEVER run any deployment commands without EXPLICITLY asking the user first and waiting for confirmation.**

This includes but is not limited to:
- `npm run deploy-storybook`
- `npx gh-pages`
- `git push`
- `npm run deploy`
- `netlify deploy`
- Any CI/CD pipeline triggers

**Required behavior:**
1. When a task is complete and deployment might be needed, ASK: "Would you like me to deploy these changes?"
2. WAIT for the user to explicitly say "yes" or give approval
3. Only THEN run the deployment command

**Why:** The user wants full control over when changes go live.

See `DEPLOYMENT.md` for the full deployment guide (Netlify app, Storybook, Chromatic).

---

## Design System

This project has a comprehensive design system context layer. The LLM should always follow these sources:

### Auto-loaded Rule Files (`.cursor/rules/`)
These load automatically based on file type:
- **`design-tokens.mdc`** — Colors, typography, spacing, radii, shadows, transitions. Loaded on all CSS/TSX/TS files.
- **`component-patterns.mdc`** — BEM naming, file structure, component inventory, state management. Loaded on component/page files.
- **`layout-system.mdc`** — Grid, containers, breakpoints, responsive patterns. Loaded on CSS/TSX files.

### Machine-Readable Tokens
- **`design-tokens.json`** — Structured JSON of all design tokens, suitable for tooling or programmatic access.

### On-Demand Skill
- **`.cursor/skills/design-system/SKILL.md`** — Complete design system reference with quick-reference tables, code patterns, and a UI code checklist. Use when building or reviewing UI.

### Source of Truth
- **`src/index.css`** `:root` — Canonical CSS custom property definitions
- **Storybook** — Component documentation at https://lenindesign.github.io/cd-mmp/

### Key Principles
1. **Always use CSS variables** — never hardcode hex colors, pixel spacing, or font names
2. **BEM naming** — `.block__element--modifier` for all CSS classes
3. **Colocated CSS** — each component has its own `.css` file imported as a side effect
4. **Semantic fonts** — `var(--font-heading)` for headings, `var(--font-body)` for body, `var(--font-article)` for editorial
5. **4px spacing grid** — use `var(--spacing-*)` tokens exclusively
6. **Mobile-first responsive** — breakpoints at 480px, 768px, 1024px, 1200px

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Lenindesign) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
