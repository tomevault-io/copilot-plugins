## findr

> - Always use Tailwind CSS utility classes for all styling

# Findr Project Rules

## Styling
- Always use Tailwind CSS utility classes for all styling
- Never use inline styles unless absolutely no Tailwind alternative exists
- Scoped CSS (<style scoped>) is allowed only for animations, keyframes, or styles impossible in Tailwind
- Custom design tokens defined in src/style.css under @theme
- Available custom colors: bg-app-base, bg-app-card, bg-app-input, bg-app-hover, text-app-content, text-app-subtle, text-app-faint, border-app-border, border-app-border-hover, text-app-accent, bg-app-accent, text-app-error, text-app-success, text-app-warning, bg-app-warning

## Tech Stack
- Frontend: Vue 3 + Vite + Tailwind CSS v4
- Backend: Node.js + Express
- Database: PostgreSQL + PostGIS
- Cache: Redis (Redis Cloud)
- AI: Claude API with tool calling
- Maps: Google Maps SDK + Google Places API

## Code Style
- Vue 3 Composition API with script setup
- No Options API
- Files generated as downloadable .vue files, not pasted in chat

## Project Structure
- Frontend: findr/findr-frontend/src
- Backend: findr/findr-backend/src

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kingsley-coder-prog)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Kingsley-coder-prog)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
