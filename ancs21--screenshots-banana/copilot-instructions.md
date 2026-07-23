## screenshots-banana

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
bun dev           # Start dev server at http://localhost:5173/
bun run build     # Production build → dist/
bun run preview   # Test production build at http://localhost:4173/
bun run typecheck # Run TypeScript type checking
bun run lint      # Run ESLint
bun run format    # Format code with Prettier
bun add <pkg>     # Add dependency
```

## Path Aliases

Use `@/` for src imports:

```typescript
import { padding } from '@/store/editor';
import { generateBackground } from '@/lib/ai';
```

## Tech Stack

- **Framework**: Preact with `@preact/signals` for reactive state management
- **Routing**: preact-iso for client-side routing
- **Build**: Vite 7 with `@preact/preset-vite`
- **Styling**: Tailwind CSS v4
- **UI Components**: @base-ui-components/react
- **Export**: html-to-image for PNG generation
- **Storage**: idb-keyval for IndexedDB (API keys, presets)
- **AI**: Google Gemini (configurable: `gemini-2.5-flash-image` or `gemini-3-pro-image-preview`)
- **PWA**: vite-plugin-pwa for installable app

## Architecture

### State Management (Preact Signals)

State lives in `src/store/` using signals:

```typescript
// Reactive state
export const imageData = signal<string | null>(null);
export const padding = signal(64);

// Computed values
export const hasImage = computed(() => imageData.value !== null);

// Actions mutate signals directly
export function setImage(data: string) {
  imageData.value = data;
}
```

Key stores:

- `editor.ts` - Image data, frame styles, transforms, canvas settings
- `ai.ts` - Chat messages, AI mode, presets

### Component Organization

- `src/pages/` - Route pages (Editor, Settings, 404)
- `src/components/Editor/` - Canvas, Frame, DropZone, Toolbar
- `src/components/Controls/` - Sidebar controls (FrameControls, BackgroundPicker, AIChat, etc.)
- `src/components/ui/` - Base components (Tabs, Slider, Tooltip)
- `src/lib/` - Utilities (ai.ts for Gemini API, db.ts for IndexedDB, gradients.ts, patterns.ts)

### Key Features

- **Frame Styles**: 14+ preset frames defined in `Frame.tsx` (FrameStyle type in editor.ts)
- **Canvas Presets**: Auto, Free, Tweet, Instagram, Story, AppStore sizes
- **Backgrounds**: Gradients (12 presets), patterns, AI-generated
- **AI Modes**: background, generate-ui, enhance, edit (multi-turn chat)
- **Export**: PNG, JPG, SVG, WebP at 1x/2x/3x scale with keyboard shortcuts (⌘S/J/G/I)

### Styling Patterns

- Tailwind utilities for most styling
- Inline styles for dynamic values (transforms, shadows, dimensions)
- Custom CSS animations in `style.css` (liquid glass effects, blob morphing)
- Custom theme: `--color-banana`, `--color-banana-dark`

### AI Integration

Direct client-side Gemini API calls in `src/lib/ai.ts`:

- API key and model selection stored in IndexedDB via `src/lib/db.ts`
- Model configurable in Settings page (Gemini 2.5 Flash or Gemini 3 Pro)
- System prompts per mode (background generation, UI creation, enhancement)
- Multi-turn conversation support with image context

### Design Skill

This project uses a `frontend-design` skill (`.claude/skills/frontend-design/SKILL.md`) that guides creation of distinctive, non-generic UI with bold aesthetic choices. Invoke with `/frontend-design` when building new UI components.

---
> Source: [ancs21/screenshots-banana](https://github.com/ancs21/screenshots-banana) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
