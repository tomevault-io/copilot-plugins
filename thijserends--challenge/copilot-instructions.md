## challenge

> This is a personalized digital puzzle challenge - a web-based labyrinth of 10 interconnected puzzles. Users must solve all puzzles sequentially to reveal a final 4-digit code (the "4-Digit Goat") that unlocks a physical treasure.

# Het Mysterie van Sinterklaas - Cursor AI Rules

## Project Overview
This is a personalized digital puzzle challenge - a web-based labyrinth of 10 interconnected puzzles. Users must solve all puzzles sequentially to reveal a final 4-digit code (the "4-Digit Goat") that unlocks a physical treasure.

## Visual Style: "The Cardboard Arcade"
- **Handmade Feel:** Everything looks like flat colored construction paper cut out with scissors
- **Typography:** Marker-style headings (Permanent Marker) and handwritten body text (Patrick Hand)
- **Animation:** Buttons and elements have a playful "wiggle" to keep the world alive
- **No Textures:** Depth achieved through flat colors and hard, crisp shadows (box-shadow: 4px 4px 0px rgba(0, 0, 0, 0.25))
- **Color Palette:** Kraft brown background (#C4A484), Poofball Red (#C94343), Ushanka Green (#439F47), Teal Toque (#2AB7CA), Parka Orange (#F7931E), Marker Black (#1A1713), Paper White (#F4F1E8)

## Technical Stack
- **React:** Component-based UI framework for interactive puzzles
- **Vite:** Build tool and development server
- **React Router:** Client-side routing (HashRouter for GitHub Pages compatibility)
- **CSS3:** Custom variables, animations, no CSS frameworks
- **Structure:** React components in src/components/ directory, each puzzle is a component

## Code Style Guidelines
- Use React functional components with hooks
- Follow the existing pattern: input field → submit button → validation → navigate to next puzzle
- Use React Router's useNavigate hook for navigation
- Maintain consistent error messaging in Dutch ("Onjuist. Probeer opnieuw.")
- Use CSS custom properties from :root for colors
- All buttons should have the wiggle animation and hard shadows
- Keep code clean, commented, and maintainable
- Use JSX for component structure

## Puzzle Structure
- Puzzles are sequential: solving one unlocks the next
- Each puzzle validates an answer and navigates to the next puzzle on success
- Puzzle 10 reveals the final 4-digit code (1006)
- Navigation: useNavigate() hook from react-router-dom, routes are '/puzzle-X'

## File Organization
- Root: index.html (Vite entry), package.json, vite.config.js, README.md
- src/: React source code
  - main.jsx: React entry point with HashRouter
  - App.jsx: Main app component with Routes
  - index.css: Global styles
  - components/: React components (Layout, LandingPage, Puzzle1-10, PuzzleInput)
- Documentation: PROJECT_INTENT_AND_PROMPT.md (complete project vision and narrative), STYLE_GUIDE.md, PUZZLE_INSPIRATION.md (neal.fun puzzle style reference), puzzle_*.md files

## Puzzle Design Inspiration
- Puzzles are inspired by [neal.fun](https://neal.fun) - aim for interactive, engaging, and visually appealing experiences
- Each puzzle should feel like a mini-game or interactive experience, not just a text-based challenge
- Focus on creative mechanics, satisfying interactions, and clear visual feedback
- Maintain the playful, hands-on feel that makes neil.fun projects so engaging

## Development Notes
- The project was originally designed with Sinterklaas themes, but current implementation uses generic puzzles
- Puzzle 1 has unique interactive canvas drawing functionality using React refs and hooks
- Puzzles 2-9 use the reusable PuzzleInput component
- Puzzle 10 has custom logic for displaying the final code
- All styling should maintain the "Cardboard Arcade" aesthetic
- Use HashRouter for GitHub Pages compatibility (no server-side routing needed)
- Local development: `npm run dev` (Vite dev server)
- Production build: `npm run build` (outputs to dist/)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ThijsErends) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
