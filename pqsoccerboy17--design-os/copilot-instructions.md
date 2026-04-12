## design-os

> Product planning and design tool for defining products before implementation. Exports handoff packages for developers/AI agents.

# Design OS

Product planning and design tool for defining products before implementation. Exports handoff packages for developers/AI agents.

## Dual-Context Architecture

**Context 1: Design OS Application**
- The tool itself (what you're working on)
- Stack: React 19 + Vite 7 + Tailwind CSS v4 + Radix UI
- Design tokens: Stone palette + Lime accent + DM Sans typography

**Context 2: Product Designs**
- What gets exported (what users create with the tool)
- Lives in `/product/` and `/src/sections/`
- Uses custom design tokens defined in product specs

**Critical:** When editing, know which context you're in. Don't mix Design OS UI styling with product export styling.

## Four-Pillar Product Structure

All products follow this schema:
1. **Product Overview** - Vision, problems, solutions, features ([product-overview.md](product/product-overview.md))
2. **Data Model** - Entities and relationships ([data-model/](product/data-model/))
3. **Design System** - Colors (Tailwind) + Fonts (Google Fonts) ([design-system/](product/design-system/))
4. **Application Shell** - Persistent nav/layout ([shell/spec.md](product/shell/spec.md))

## Build-Time Constraints

**Module loading happens at Vite build:**
- Uses `import.meta.glob()` for dynamic imports
- Changes to `/product/*.md` files require rebuilds
- Screen designs must be in `/src/sections/[section-id]/`
- Missing files silently degrade (no errors)

**Markdown as source of truth:**
- Parsing is regex-based and fragile
- Exact format required (see [loaders/](src/loaders/) for patterns)
- Product roadmap drives section generation

## Component Conventions

**Compound component pattern:**
- All components use `data-slot` attributes (e.g., `data-slot="button"`)
- CVA (class-variance-authority) for variant management
- Radix UI primitives + shadcn/ui patterns (with custom slots)

**Export philosophy:**
- Components are props-based for portability
- No direct imports between product sections
- Designed to hand off to external codebases via `product-plan.zip`

## Workflow

1. Define product in `/product/*.md` (markdown specs)
2. Build screen designs in `/src/sections/[section-id]/` (React components)
3. Export via "Export Product Plan" button
4. Implement elsewhere (Design OS only exports, doesn't implement)

See [components.json](components.json) for shadcn/ui schema, [src/loaders/](src/loaders/) for parsing logic, and [agents.md](agents.md) for comprehensive workflow documentation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pqsoccerboy17)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pqsoccerboy17)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
