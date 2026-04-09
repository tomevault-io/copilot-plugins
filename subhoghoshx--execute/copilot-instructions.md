## execute

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Execute is a web-based code playground with dual modes:
- **Classic Mode**: HTML/CSS/JavaScript split-pane editor with live preview
- **React Mode**: Full React development environment using Sandpack (`?mode=react`)

## Commands

```bash
# Development
npm run dev       # Start Vite dev server at http://localhost:5173

# Code Quality
npm run lint      # Run ESLint with TypeScript rules
npm run build     # TypeScript check + production build

# Production
npm run preview   # Preview production build locally
```

## Architecture

### Tech Stack
- **Frontend**: React 19 + TypeScript + Vite
- **Styling**: TailwindCSS 4.x (new native CSS approach with `@theme inline`)
- **Database**: Convex (real-time sync, projects table)
- **Editor**: CodeMirror 6 with vim mode + Emmet
- **React Mode**: Sandpack from CodeSandbox
- **UI**: shadcn/ui components (Radix UI primitives)

### Key Files & Data Flow

```
src/
├── App.tsx           # Classic editor mode (HTML/CSS/JS panes)
├── ReactEditor.tsx   # React mode with Sandpack
├── main.tsx         # Router setup, mode switching via ?mode=react
└── components/
    ├── HomePage.tsx  # Landing page with project creation
    └── ui/          # shadcn components
    
convex/
├── schema.ts        # projects table definition
└── projects.ts      # CRUD mutations/queries
```

**Data Flow:**
1. Editor changes → local state buffers
2. Save (Ctrl+S or vim `:w`) → Convex mutation
3. Preview updates via iframe src refresh (classic) or Sandpack hot reload (React)
4. Projects accessible at `/:projectId` URLs

### Editor Implementation

**CodeMirror Setup:**
- Language support: HTML, CSS, JavaScript with JSX
- Vim mode with custom `:w` save command mapping
- Emmet expansion for HTML/CSS
- VSCode light/dark themes
- Tab key handling for indentation

**Preview Generation (Classic Mode):**
```javascript
// Combines user code into complete HTML document
const htmlContent = `
  <!DOCTYPE html>
  <html>
    <head>
      ${includeTailwind ? '<script src="https://cdn.tailwindcss.com"></script>' : ''}
      <style>${css}</style>
    </head>
    <body>
      ${html}
      <script>${js}</script>
    </body>
  </html>
`;
```

### State Management

- **Editor State**: Local React state for code buffers
- **Persistence**: Convex database with auto-save on Ctrl+S
- **User Preferences**: localStorage (vim mode, theme, Tailwind toggle)
- **Routing**: React Router with projectId params

### Keyboard Shortcuts

- **Ctrl/Cmd+S**: Save project
- **Alt+1/2/3**: Switch tabs (classic mode)
- **V key**: Create vanilla project (homepage)
- **R key**: Create React project (homepage)
- **Vim `:w`**: Save in vim mode

## Important Implementation Details

### TailwindCSS 4.x Configuration
Uses new native CSS approach without PostCSS:
```css
@import "tailwindcss";
@theme inline { /* custom theme values */ }
```

### Convex Schema
```typescript
projects: defineTable({
  html: v.string(),
  css: v.string(),
  js: v.string(),
  reactFiles: v.optional(v.any()), // For React mode
  createdAt: v.number(),
})
```

### Theme System
- CSS custom properties with OKLCH colors
- Dark mode via `dark` class on root
- System preference detection with manual override

### TypeScript Configuration
- Strict mode enabled
- Path alias: `@/*` → `./src/*`
- Separate configs for app and Node environments

## Development Tips

1. **Adding New Editor Features**: Extend CodeMirror extensions in `App.tsx`
2. **Modifying Preview**: Update iframe generation logic in `App.tsx`
3. **Database Changes**: Update schema in `convex/schema.ts` and run `npx convex dev`
4. **Theme Customization**: Edit CSS variables in `index.css`
5. **React Mode**: Sandpack configuration in `ReactEditor.tsx`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/subhoghoshX)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/subhoghoshX)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
