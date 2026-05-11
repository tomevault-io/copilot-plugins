## poematon-v2

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Poematon 2.0** is an interactive poetry installation application for the "CEROPOÉTICAS" Expanded Poetry Exhibition. Users create "ready-made" poems by selecting and arranging verses from different authors through a drag-and-drop interface within a 3-minute time limit.

**Key Context:**

- This is a complete port from Create React App to Vite
- The original application is at: https://github.com/eberhm/poematon
- Current deployment: https://eberhm.github.io/poematon/
- Complete PRD with requirements, screenshots, and asset URLs in `docs/PRD.md`

## Development Commands

### Primary Commands

```bash
npm install              # Install dependencies
npm run dev              # Start dev server with HMR
npm run build            # Production build
npm run preview          # Preview production build
npm test                 # Run Vitest test suite
npm run test:ui          # Run tests with Vitest UI
npm run data:reload      # Regenerate JSON from CSV (converts src/data/versos.canarias.csv → canarias.json)
npm run format           # Check code formatting with Prettier
npm run format:fix       # Fix code formatting
```

### Quality Validation

```bash
npm run validate             # Run all checks (typecheck + lint + format + tests with coverage)
npm run lint                 # Check ESLint issues
npm run lint:fix             # Fix ESLint issues
npm run format               # Check Prettier formatting
npm run format:fix           # Fix Prettier formatting
npm run typecheck            # Check TypeScript errors
npm run test:coverage        # Run tests with 80% coverage enforcement
```

- **Pre-commit hook** automatically runs Prettier + ESLint on staged files via lint-staged
- **CI** runs all checks (typecheck, lint, format, test:coverage) on every PR and push to main
- Always run `npm run validate` before considering a feature complete

### Running Single Tests

```bash
npm test -- path/to/test.spec.ts           # Run specific test file
npm test -- -t "test name pattern"         # Run tests matching pattern
npm test -- path/to/test.spec.ts --watch   # Watch mode for single file
```

## Technical Stack

- **Build:** Vite 5+ with base path `/poematon/` for GitHub Pages
- **Framework:** React 18+ with TypeScript (strict mode)
- **UI Library:** Material-UI (MUI) v5 with Emotion
- **Drag & Drop:** @dnd-kit/core and @dnd-kit/sortable
- **Testing:** Vitest with @testing-library/react
- **Deployment:** GitHub Actions → GitHub Pages

## Architecture Overview

### Core Application Flow

1. **Welcome Screen** → User clicks "EMPEZAR" → Fullscreen + Timer starts + Music plays
2. **Main Interface** → Two-column layout (VERSOS ← drag → TU POEMA) with 3-minute countdown
3. **Completion** → Print dialog → "Enhorabuena" screen → Auto-reload after 10s

### Key Components Structure

```
PoematonSectionList (main orchestrator)
├── Timer + Print Button (fixed top)
├── VersesSection (left panel, dark #333)
│   └── VerseItem[] (draggable, can regenerate IDs for reuse)
└── PoemSection (right panel, yellow #cfc140)
    └── PoemItem[] (sortable, max 8 verses)
```

### Data Architecture

**Verse Type:**

```typescript
type Verse = {
  id: string // UUID v4
  value: string // Verse text
  autor: string // Author name
  poema: string // Source poem
  poemario: string // Poetry collection
}
```

**Version System:**

- URL parameter `?version=<name>` loads different verse collections
- Default: "v1" → `src/data/data.json`
- Example: "canarias" → `src/data/canarias.json`
- CSV source files in `src/data/` with semicolon delimiter

### Drag & Drop Logic

**Critical ID Management:**

- When verse dragged from VERSOS to TU POEMA, regenerate ID with timestamp suffix
- This allows the same verse to be reused multiple times
- Verses never disappear from VERSOS panel

**Drop Constraints:**

- Maximum 8 verses in TU POEMA
- Show error alert when limit reached (allows reordering/substitution only)
- Can always drag from TU POEMA back to VERSOS
- Keyboard navigation supported via @dnd-kit sortable coordinates

### Timer System

- 3-minute countdown (180s) displayed as MM:SS in 4em white text
- Background music plays throughout (`public/sound/music.mp3`, 40% volume)
- Warning sound at 20s remaining (`public/sound/20s.mp3`, 40% volume)
- On expiration: Auto-print → Stop audio → Show completion → Reload after 10s

### Print Functionality

**Print Layout (CSS print media query):**

1. Hide main UI (timer, buttons, drag panels)
2. Show two sections:
   - "POEMATÓN. Tu Poema ready-made:" with verse list (text only)
   - "Poema confeccionado con los versos de los autores:" with attributions (author, poem, collection)

**Optional Video Server Integration:**

- URL param `?videoServer=<url>` enables external display
- On print, POST poem data to server URL
- Silent failure, doesn't block print

### Visual Design

**Color Palette:**

- Primary accent: `#cfc140` (yellow/gold) for TU POEMA panel and buttons
- Dark background: `#000` (black for body)
- Container background: `#333` (dark gray for VERSOS panel)
- Text: `#fff` (white)

**Key Assets:**

- Background: `background.portada.png` (body and modal screens)
- Logo: `corona.png` (welcome screen)
- Fonts: Roboto (300, 400, 500, 700)
- Typography: Large title 80px, timer 4em, section headers MUI h6

**Layout:**

- Material-UI Grid 12-column system
- Two equal columns (6/6 split) with spacing: 4 units
- Both panels: 75vh height, 65vh scrollable content area
- Rounded corners: 20px border-radius
- Top margin: 50px

## Important Implementation Notes

### Verse Reusability

When a user drags a verse from VERSOS to TU POEMA, the verse must remain visible and draggable in VERSOS. Implement this by regenerating the verse ID with a timestamp suffix when added to poem. This is critical for the user experience.

### Maximum Verse Limit

The 8-verse limit is enforced only for adding new verses. Users can still:

- Reorder verses within TU POEMA
- Remove verses from TU POEMA
- Substitute verses (drag from VERSOS to TU POEMA replaces existing)
  Display error alert above verse list when limit reached.

### Fullscreen Mode

Triggered when user clicks "EMPEZAR" button. Use browser Fullscreen API. Handle gracefully if not supported.

### Audio Management

All audio in `public/sound/` directory. Volume set to 40%. Stop all audio on timer expiration and before auto-reload.

### Testing Requirements

- Target >80% code coverage
- Unit tests for utilities (`src/utils/board.ts`, `src/utils/verse.ts`)
- Component tests for VerseCard, VersesSection, PoemSection
- Integration tests for complete user flow
- Visual regression tests using `/docs/screens/` screenshots as baseline

## Build & Deployment

**Production Build:**

- Base path: `/poematon/` configured in `vite.config.ts`
- Output: `dist/` directory
- GitHub Actions workflow: `.github/workflows/deploy.yml`
- Deploys to: https://eberhm.github.io/poematon/

**CI/CD Pipeline:**

1. Trigger on push to main or manual dispatch
2. Setup Node.js 20
3. Install with `npm ci`
4. Run tests
5. Build application
6. Deploy to GitHub Pages

## Migration Reference

All assets from the original repository are publicly accessible. See `docs/PRD.md` Appendix C for complete list of URLs for:

- Images (background, corona logo, favicon)
- Audio files (music.mp3, 20s.mp3)
- Data files (data.json, canarias.json, CSV sources)
- Screenshots (intro, main, print-dialog, end)
- Source code browser links

## TypeScript Configuration

Use strict mode. All data structures must have type definitions in `src/types/index.ts`. No implicit any types allowed.

## Code Organization

```
src/
├── components/          # React components (VerseCard, VersesSection, PoemSection, etc.)
├── data/               # JSON verse collections and CSV sources
│   └── index.ts        # Data loading with version selection
├── types/              # TypeScript type definitions
├── utils/              # Utility functions (board.ts, verse.ts)
├── App.tsx             # Root component
├── main.tsx            # Entry point
└── theme.ts            # MUI theme configuration

scripts/
└── loadData.js         # CSV to JSON conversion script

public/
├── background.portada.png
├── corona.png
└── sound/              # Audio assets
```

## Performance Requirements

- Initial load: <3 seconds on standard broadband
- Drag operations: <16ms frame time (60fps)
- Bundle size: <500KB gzipped
- Smooth animations throughout (no layout shifts)

## Browser Support

Last 2 versions of Chrome/Edge, Firefox, Safari. No IE11 support required.

**Required APIs:**

- Fullscreen API
- Web Audio API
- Fetch API
- Modern drag and drop (via @dnd-kit)
- Print API (window.print)

---
> Source: [eberhm/poematon-v2](https://github.com/eberhm/poematon-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
