## gemini-rss-app

> React 19 UI components with Framer Motion animations, specialized for RSS article display and settings management.

## components/

### OVERVIEW
React 19 UI components with Framer Motion animations, specialized for RSS article display and settings management.

### STRUCTURE
- **ArticleCard.tsx**: Reusable article display component with image proxy support
- **SettingsModal.tsx**: Modal-based settings interface with drag-and-drop reordering
- **ui/**: Reusable UI primitives (Card, Dialog, Button, Input, etc.)

### WHERE TO LOOK
- Dual URL handling: Import `getMediaUrl` from `services/rssService.ts` for image rendering
- Performance optimizations: ArticleCard memoization, debounced reordering in SettingsModal

### CONVENTIONS
- **Animations**: Always import variants from animations.tsx, never hardcode durations or easings
- **Media rendering**: Use `getMediaUrl(article.thumbnail)` for all images (handles proxy mode internally)
- **Component style**: Functional components only, no class components

### ANTI-PATTERNS
- ❌ Inline styles - Use Tailwind utility classes exclusively
- ❌ Raw `<img src="...">` without getMediaUrl - Breaks proxy mode support
- ❌ Reordering without debouncing - Causes excessive API calls to database

---
> Source: [Sallyn0225/gemini-rss-app](https://github.com/Sallyn0225/gemini-rss-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
