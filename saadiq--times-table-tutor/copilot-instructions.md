## times-table-tutor

> A calm, anxiety-free web app for learning times tables through adaptive practice and a progressive scene reveal reward system.

# Times Table Tutor

A calm, anxiety-free web app for learning times tables through adaptive practice and a progressive scene reveal reward system.

## Quick Start

```bash
bun install
bun run dev      # Start dev server
bun run build    # Build for production
bun run lint     # Run ESLint
```

## Tech Stack

- **Runtime/Package Manager**: Bun
- **Build Tool**: Vite 7
- **Framework**: React 19 + TypeScript
- **Styling**: Tailwind CSS v4 (CSS-first config in `src/index.css`)
- **Animations**: Framer Motion
- **Icons**: Lucide React (no emojis)
- **State**: Zustand with localStorage persistence
- **PWA**: vite-plugin-pwa
- **Backend**: Cloudflare Pages Functions + D1 (SQLite)

## Backend Development

The app uses Cloudflare Pages Functions with D1 database for user profiles and cloud sync.

### Local Development

```bash
bun run dev       # Start frontend + API concurrently
bun run cleanup   # Kill phantom processes, free ports
```

- Frontend (Vite): `http://localhost:5173`
- API (Wrangler): `http://localhost:8788`

### Database Commands

```bash
bun run db:migrate:local   # Run migrations on local D1
bun run db:migrate         # Run migrations on production D1
```

### Deployment

Before deploying to production:
1. Create D1 database: `bunx wrangler d1 create ttt-db`
2. Update `database_id` in `wrangler.toml` with the real ID
3. Run production migrations: `bun run db:migrate`

## Project Structure

```
src/
├── components/
│   ├── common/       # Button, Modal, ProgressBar, Navigation, Layout, Celebration, SettingsModal
│   ├── practice/     # ProblemDisplay, AnswerInput, HintPanel, MultipleChoice, NumberPad
│   ├── learn/        # FactCard, VisualExplainer
│   ├── progress/     # ProgressView, ProgressScene, MasteryGrid, ActivityCalendar, RevealSequence
│   └── garden/       # (legacy) GardenItem, GardenView
├── views/            # PracticeView, LearnView, ProgressViewPage
├── stores/           # Zustand stores (progress, progressView, garden, session, focusTables)
├── lib/              # Core logic (adaptive, strategies, rewards, sounds, storage)
├── hooks/            # useSound
└── types/            # TypeScript types
```

## Key Concepts

### Three Modes + Settings
1. **Learn** - Visual introduction to facts (no wrong answers)
2. **Practice** - Adaptive problems with hints on mistakes
3. **Progress** - Scene reveal reward system with stats
4. **Settings** - Focus table selection (bottom nav)

### Scene Reveal Reward System (`src/components/progress/`)
- p5.js canvas renders a tree scene that evolves as you master facts
- **4 tiers** based on facts learned (0-36-72-108-144): dawn → morning → afternoon → golden hour
- **Progressive elements**: grass (facts 1-50), flowers (51-90), leaves (91-144)
- **12 animal characters** unlock when you master each times table (1-12)
- Interactive reveal sequences with sparkle animations

### Adaptive Learning (`src/lib/adaptive.ts`)
- Tracks each of 144 facts (1x1 through 12x12) individually
- Confidence levels: `new` → `learning` → `confident` → `mastered`
- Prioritizes: learning facts > trouble spots > spaced review > new facts
- Multiple choice for new facts, number pad for confident facts
- Supports focus tables filter (Settings → select specific tables to practice)

### Strategy Hints (`src/lib/strategies.ts`)
Nine strategies: visual_array, skip_counting, break_apart, use_neighbor, nines_trick, fives_trick, doubles, tens_trick, ones_zeros

### State Stores
- **progressStore**: All 144 fact confidence levels, persisted to localStorage
- **progressViewStore**: Scene reveal state (revealed facts, unlocked characters, tier level)
- **gardenStore**: (legacy) Coins, garden items, themes
- **sessionStore**: Current goal, progress, streak, mode
- **focusTablesStore**: Which tables (1-12) to focus on, with enable/disable toggle

## Tailwind v4 Notes

Uses CSS-first configuration (not `tailwind.config.js`):

```css
/* src/index.css */
@import "tailwindcss";

@theme {
  --color-garden-500: #22c55e;
  --color-warm-500: #f59e0b;
}
```

Custom colors: `garden-*` (greens), `warm-*` (yellows), `sky-*` (blues)

## Design Principles

- **No timers** - Anxiety-free learning
- **ADHD-friendly** - Frequent small wins, clear goals, dopamine-positive feedback
- **Mobile-first** - Large tap targets (48px+), touch-friendly
- **Wrong answers** reframe as learning opportunities with strategy hints

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/saadiq)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/saadiq)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
