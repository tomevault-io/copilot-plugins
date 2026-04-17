## collab-canvas

> - Building a real-time collaborative canvas (Figma-like)

# CollabCanvas Project Rules

## Project Context
- Building a real-time collaborative canvas (Figma-like)
- Tech: React + Vite + Firebase (Firestore + Auth) + Konva.js
- Follow the PRD in docs/PRD.md
- Follow task breakdown in docs/TASKS.md
- Follow Firestore structure in docs/ARCHITECTURE.md

## Code Style
- Use functional React components with hooks
- Use named exports for utilities, default export for components
- Prefer async/await over promises
- Add JSDoc comments for complex functions
- Use descriptive variable names

## Firebase Rules
- Never hardcode Firebase config - use .env.local
- All Firestore writes should handle errors
- Throttle real-time updates (cursors: 50-100ms)
- Use optimistic updates for better UX

## File Organization
- Components in src/components/
- Custom hooks in src/hooks/
- Firebase utilities in src/lib/
- One component per file

## Testing Approach
- Test with 2+ browser windows for multiplayer features
- Console.log key state changes during development
- Test offline/reconnection scenarios

## PR Workflow
- Follow the PR order in docs/TASKS.md
- Complete one PR fully before moving to next
- Commit frequently with descriptive messages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/robglnn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
