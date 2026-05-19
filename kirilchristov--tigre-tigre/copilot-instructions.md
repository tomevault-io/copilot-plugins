## tigre-tigre

> This is a marketing website for **tigre tigre**, a chili crunch brand. The site uses React + Vite with shadcn/ui components, GSAP animations, and Stripe payment links. It supports Bulgarian and English languages.

# Claude Instructions for tigre tigre Project

## Project Context

This is a marketing website for **tigre tigre**, a chili crunch brand. The site uses React + Vite with shadcn/ui components, GSAP animations, and Stripe payment links. It supports Bulgarian and English languages.

## Key Files

- `PLAN.md` - Implementation roadmap with phases and tasks
- `CLAUDE.md` - This file, instructions for AI assistance

## How to Work with the Plan

### Reading the Plan

1. Always read `PLAN.md` first to understand current progress
2. Look for `✅ In Progress` to see active phase
3. Check `[x]` completed tasks and `[ ]` pending tasks
4. Review "Technical Decisions Log" for context on choices made

### Updating the Plan

When completing tasks:

```markdown
- [x] Task completed ← Change [ ] to [x]
```

When starting a new phase:

```markdown
### Phase 2: Internationalization Setup (Simple) ✅ In Progress
```

Remove `✅ In Progress` from previous phase.

### Adding New Tasks

Add under the appropriate phase:

```markdown
- [ ] New task description
```

If task doesn't fit existing phases, add to "Notes" or create sub-section.

### Modifying the Plan

- **Adding phases:** Insert new `### Phase X:` section with difficulty rating
- **Reordering:** Move tasks between phases if dependencies change
- **Removing:** Strike through with reason: `~~Task~~ (removed: reason)`
- **Log decisions:** Add row to "Technical Decisions Log" table

## Project Conventions

### File Structure (Target)

```
tigre-tigre/
├── public/
│   └── locales/
│       ├── en/
│       │   └── translation.json
│       └── bg/
│           └── translation.json
├── src/
│   ├── components/
│   │   ├── ui/          # shadcn components
│   │   ├── layout/      # Header, Footer, Layout
│   │   └── sections/    # Homepage sections
│   ├── hooks/
│   ├── lib/
│   │   └── utils.ts
│   ├── styles/
│   ├── App.tsx
│   └── main.tsx
├── PLAN.md
├── CLAUDE.md
└── package.json
```

### Coding Standards

- TypeScript for all `.ts` and `.tsx` files
- Use functional components with hooks
- Translations: Always use `useTranslation()` hook, never hardcode text
- Styling: Tailwind CSS classes, follow shadcn patterns
- Colors: Only `black` and `white` (Tailwind: `text-black`, `bg-white`, etc.)
- Font: Arial via Tailwind config

### Translation Keys Convention

```json
{
  "nav": {
    "home": "Home",
    "about": "About"
  },
  "hero": {
    "title": "tigre tigre",
    "subtitle": "Premium Chili Crunch"
  }
}
```

### Component Naming

- PascalCase for components: `HeroSection.tsx`
- camelCase for hooks: `useScrollAnimation.ts`
- kebab-case for CSS modules (if used)

## Commands Reference

### Development

```bash
npm run dev      # Start dev server
npm run build    # Production build
npm run preview  # Preview production build
npm run lint     # Run ESLint
```

### Adding shadcn Components

```bash
npx shadcn-ui@latest add button
npx shadcn-ui@latest add card
```

## Important Notes

- No logos yet - use text-based branding
- No images yet - use placeholders or solid colors
- Stripe: Use Payment Links (not full Checkout integration)
- Backend comes later - keep frontend independent
- All text must be translatable (BG + EN)

## When Asked to Implement

1. **Check the Plan** - Read `PLAN.md` for current phase/task
2. **Follow Conventions** - Use patterns defined above
3. **Update Progress** - Mark tasks complete, add decisions to log
4. **Stay Minimal** - Black/white only, Arial font, no extras
5. **Think Translations** - Every user-facing string needs a key

## Prompt Template for Continuing Work

Use this to resume development:

```
Continue working on the tigre tigre project. Read CLAUDE.md and PLAN.md first,
then proceed with the next incomplete task in the current phase. Update the
plan as you complete tasks. Follow all conventions specified in CLAUDE.md.
```

---
> Source: [kirilchristov/tigre-tigre](https://github.com/kirilchristov/tigre-tigre) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
