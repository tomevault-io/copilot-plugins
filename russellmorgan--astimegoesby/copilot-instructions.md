## astimegoesby

> **AsTimeGoesBy** is a "death calendar" application designed to help users visualize the passage of time in their lives. By inputting their birth date and an optional expected lifespan, users are presented with a minimalist, geometric grid representing their life in months. This visualization aims to make the finite nature of time concrete and encourage deliberate living.

# GEMINI.md - AsTimeGoesBy (Death Calendar)

## Project Overview
**AsTimeGoesBy** is a "death calendar" application designed to help users visualize the passage of time in their lives. By inputting their birth date and an optional expected lifespan, users are presented with a minimalist, geometric grid representing their life in months. This visualization aims to make the finite nature of time concrete and encourage deliberate living.

### Key Technologies
- **Bundler:** Vite 7 (configured with `root: 'src'`)
- **Styling:** Tailwind CSS 3 (utility-first) supplemented by custom CSS variables in `src/style.css`
- **Logic:** Vanilla JavaScript (ESM)
- **Data Persistence:** `localStorage` for saving user info and UI state
- **Assets:** Custom icon fonts in `fonts/`

### Architecture
- **Source Root:** All source files (`index.html`, `main.js`, `style.css`) are located in the `src/` directory.
- **Entry Point:** `src/index.html` loads `src/main.js`.
- **Logic:** `src/main.js` manages state, form validation, grid rendering, and storage interactions.
- **Styling:** `src/style.css` handles theme variables (dark/light), animations, and core layout logic that is impractical for utility classes.

---

## Building and Running

### Development
Start the Vite development server:
```bash
npm run dev
```

### Production Build
Build the project into the `dist/` directory:
```bash
npm run build
```

### Testing
Current test script is a placeholder:
```bash
npm run test
```
*Note: Adding automated tests (e.g., using Vitest) is a recommended next step.*

---

## Development Conventions

### Aesthetic & Tone
- **Minimalist & Geometric:** Focus on grids, alignment, and spacing.
- **Restrained:** Avoid excessive decoration, glow effects, or cluttered layouts.
- **Direct & Calm:** The tone should be unsentimental and encourage reflection.
- **Theme:** Dark mode is the default; light mode is supported via `prefers-color-scheme`.

### Coding Standards
- **Native APIs:** Prefer native `Date` and `DOM` APIs over external libraries (e.g., no Moment.js).
- **ESM:** Use ES modules for JavaScript.
- **Accessibility:** Treat accessibility as a priority (e.g., `aria-live`, `aria-invalid`, `role="alert"`).
- **Validation:** Rigorous form validation for birth dates and lifespans.

### Key File Responsibilities
- `src/main.js`: Contains `CONFIG`, `state`, and `refs`. Handles the `init()` sequence, form events, validation logic, and grid rendering.
- `src/style.css`: Defines the `:root` variables for colors and spacing. Includes animations for grid entries and panel transitions.
- `vite.config.js`: Sets the root to `src` and ensures builds output to `../dist`.

---

## Usage
1. **Onboarding:** Enter Name, Birth Month/Year, and an optional expected lifespan (default is 90 years).
2. **Visualization:** View the grid of months. Passed months are filled; the current month pulses.
3. **Reflection:** Read the summary and reflection messages based on progress through the timeline.
4. **Persistence:** Data is saved to `localStorage`, allowing the view to persist across sessions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/russellmorgan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/russellmorgan)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
