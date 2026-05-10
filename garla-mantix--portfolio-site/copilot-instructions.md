## portfolio-site

> Personal portfolio website built with React, TypeScript, and Tailwind CSS.

# Portfolio Site

Personal portfolio website built with React, TypeScript, and Tailwind CSS.

## Tech Stack

- **Framework**: React 19 + Vite
- **Language**: TypeScript (strict mode, verbatimModuleSyntax)
- **Styling**: Tailwind CSS v4 (CSS-based config via `@theme`)
- **Animations**: Framer Motion
- **Build**: Vite 7

## Commands

```bash
npm run dev      # Start dev server (localhost:5173)
npm run build    # Type-check + production build
npm run preview  # Preview production build
npm run lint     # ESLint check
```

## Project Structure

```
src/
├── components/
│   ├── layout/          # Header, Footer, Section wrapper
│   ├── sections/        # Hero, Projects, Skills, Experience, TypingGame
│   ├── ui/              # Badge, Button, Card, Timeline (reusable)
│   └── typing-game/     # GameDisplay, GameInput, GameStats, GameResults
├── hooks/
│   ├── useTypingGame.ts # Game state with useReducer
│   └── useLocalStorage.ts
├── data/                # Static content (projects, skills, experience, words)
├── types/index.ts       # TypeScript interfaces
└── index.css            # Tailwind theme + global styles
```

## Code Conventions

- Use `import type` for type-only imports (required by verbatimModuleSyntax)
- Components use named exports, grouped by barrel files (`index.ts`)
- Tailwind classes use custom theme colors: `dark-50` to `dark-950`, `accent`, `accent-hover`
- Framer Motion for all animations (scroll reveal, hover effects, staggered lists)
- Data files contain typed arrays for easy content updates

## Theme

Dark theme with custom color palette defined in `src/index.css`:
- Background: `dark-950` (#0d0d0f)
- Text: `dark-100` to `dark-500`
- Accent: `accent` (#3b82f6 blue)
- Fonts: Inter (sans), JetBrains Mono (mono)

## Typing Game

State machine: `idle → countdown → playing → finished`

- 15 developer-related words
- Tracks WPM (words per minute) and accuracy
- High scores persist in localStorage
- Character-by-character highlighting (green/red)

## Customization

1. **Personal info**: Edit `src/components/sections/Hero.tsx`
2. **Projects**: Edit `src/data/projects.ts`
3. **Skills**: Edit `src/data/skills.ts`
4. **Experience**: Edit `src/data/experience.ts`
5. **Social links**: Edit Hero.tsx and Footer.tsx

---
> Source: [Garla-mantix/portfolio-site](https://github.com/Garla-mantix/portfolio-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
