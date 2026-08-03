## design-ai

> mkdir design-md/[company-name]/

# AGENTS.md - design-ai

## How to Add a New Company

```bash
# 1. Create directory
mkdir design-md/[company-name]/

# 2. Fetch target website
webfetch https://[company].com

# 3. Create DESIGN.md (see format below)

# 4. Commit and push
git add design-md/[company-name]/
git commit -m "Add [Company] design system docs"
git push
```

## DESIGN.md Format (9 sections)

Every file must follow this structure:

1. **Visual Theme & Atmosphere** - Mood, density, aesthetic philosophy
2. **Color Palette & Roles** - Token name + hex + functional role
3. **Typography Rules** - Font stack, type scale table
4. **Component Stylings** - Buttons, cards, inputs with CSS snippets
5. **Layout Principles** - Spacing scale, grid, responsive approach
6. **Depth & Elevation** - Shadow system, surface hierarchy
7. **Do's and Don'ts** - Design guardrails
8. **Responsive Behavior** - Breakpoints, touch targets
9. **Agent Prompt Guide** - Quick reference + prompt snippet

Use the webfetch tool to collect colors, typography, and components directly from each company's live website.

## Directory Structure

```
design-md/
├── company-1/DESIGN.md
├── company-2/DESIGN.md
└── ... (38 companies total)
```

## Git Conventions

- Commit message: `"Add [Company] design system docs"`
- No branch protection - push directly to main

---
> Source: [Khalidabdi1/design-ai](https://github.com/Khalidabdi1/design-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
