## gemini-design-plugin

> This extension provides a complete design exploration workflow for building UI components and pages.

# Design Lab Extension

This extension provides a complete design exploration workflow for building UI components and pages.

## What It Does

Design Lab generates **multiple distinct UI variations** for any component or page, lets you compare them side-by-side in your browser, collects your feedback, and synthesizes a refined version—repeating until you're confident in the result.

## Core Workflow

1. **Preflight** — Detects your framework, styling system, and reads existing design tokens
2. **Interview** — Asks about scope, pain points, inspiration, brand feel, and constraints
3. **Generation** — Creates 5 distinct variants exploring different approaches
4. **Review** — View all variations at `/__design_lab` in your dev server
5. **Feedback & Refinement** — Describe what you like, get synthesized versions
6. **Finalize** — Cleanup temp files, generate `DESIGN_PLAN.md` and `DESIGN_MEMORY.md`

## Available Commands

- `/design:start [ComponentName]` — Start a new design session
- `/design:cleanup` — Manually cleanup temporary files if needed

## Design Principles

All generated variants follow these principles:

### UX & Usability
- **Cognitive Load:** Minimize. Group related items. Use progressive disclosure.
- **Feedback:** Every action needs a reaction (hover, active, focus, success, error).
- **Consistency:** Use the inferred project constraints (colors, spacing, typography).

### Interaction States
- **Hover:** Subtle brightness or elevation change. 150ms transition.
- **Active:** Scale down (0.98) or darken.
- **Focus:** Visible 2px ring with offset.

### Accessibility (Critical)
- **Contrast:** Text must be 4.5:1 minimum.
- **Touch Targets:** 44px minimum for mobile.
- **Semantics:** Use `<button>`, `<nav>`, `<main>`, not `<div>`.
- **Keyboard:** Ensure tab order is logical.

### Layout & Density
- **Compact:** 4-8px spacing, for data-dense interfaces
- **Comfortable:** 12-16px spacing, balanced (default)
- **Spacious:** 20-32px spacing, for marketing/editorial content

### Motion
- **Micro-interactions:** 150-200ms, ease-out
- **Page transitions:** 300-400ms, ease-in-out
- **Loading states:** Use skeleton screens or subtle spinners

### Code Quality
- **Naming:** PascalCase for components, camelCase for props.
- **Typing:** Strict TypeScript interfaces.
- **Structure:** Clean, componentized code.

## Supported Frameworks

- Next.js (App Router & Pages Router)
- Vite (React, Vue)
- Remix
- Astro
- Create React App

## Supported Styling

- Tailwind CSS
- CSS Modules
- Material UI (MUI)
- Chakra UI
- Ant Design
- styled-components
- Emotion

## Tips for Best Results

1. **Be specific in the interview.** The more context about pain points, target users, and inspiration, the more distinct the variations.
2. **Reference products you admire.** "Like Linear's density" or "Stripe's clarity" gives concrete direction.
3. **Don't settle on round one.** The synthesis step is where it gets good—describe what you like about each variant.
4. **Keep your dev server running.** The extension won't start it for you.
5. **Check the DESIGN_PLAN.md.** After finalizing, this contains implementation steps and accessibility checklist.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/akhilesh-w)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/akhilesh-w)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
