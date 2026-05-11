## example

> Example application demonstrating library usage - Vite + TailwindCSS 4 + Tweakpane


- Use plain JavaScript (ES modules) - no TypeScript in example directory
- Vite for build tooling (`npm run dev`, `npm run build`)
- TailwindCSS v4 for styling - use utility classes
- Handlebars for templating video cards
- Tweakpane for interactive controls/demos
- Organize code in `lib/` subdirectory with barrel exports
- Constants in `constants.js` at example root
- Use `file:..` dependency reference to local package in package.json
- Main entry point: `main.js` - DOM ready, initialize components
- Functions should be pure and reusable - create factory functions when needed
- Use descriptive variable names and JSDoc comments for complex functions (`@fileoverview` for main files)
- DOM manipulation: use `getElementById` with constants from `constants.js`
- Clean up event listeners and `AmbientGlow` instances properly
- @file example/main.js
- @file example/lib/index.js

---
> Source: [mike-at-redspace/video-ambient-glow](https://github.com/mike-at-redspace/video-ambient-glow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
