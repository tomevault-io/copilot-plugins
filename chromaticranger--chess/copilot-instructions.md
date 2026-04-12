## chess

> npm run dev      # Start development server

# CLAUDE.md - Chess App Guidelines

## Commands
```
npm run dev      # Start development server
npm run build    # Build for production
npm run preview  # Preview production build
```

## Code Style
- **Components**: Vue 3 SFCs with `<script setup>` Composition API
- **Naming**: PascalCase for components, camelCase for variables/methods
- **Imports**: Vue imports first, group by type, use specific lodash imports
- **Styling**: TailwindCSS with scoped component styles
- **State**: Use `ref()` and `computed()` for reactive state
- **Events**: Properly clean up listeners with lifecycle hooks
- **Performance**: Use event throttling, computed properties
- **Chess Logic**: Document complex game rules with comments

## Project Structure
- `src/components/` - Vue components (Board, Square, Piece)
- `src/assets/` - Chess piece SVGs and static assets
- Chess pieces naming format: {Type}_{Color}.svg (e.g. K_B.svg, P_W.svg)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ChromaticRanger)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ChromaticRanger)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
