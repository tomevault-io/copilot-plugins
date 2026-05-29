## mathaidatascience

> This project is an interactive React + Vite + Tailwind course app.

# Project Instructions for Codex

This project is an interactive React + Vite + Tailwind course app.

The file `REFERENCE_APP.jsx` is the preferred reference implementation. Do not replace it with a lighter, generic course app.

## Primary Goal

Rebuild the app so the final project closely follows `REFERENCE_APP.jsx`.

The user prefers this version because it is more math-heavy, practical, and structured around the actual learning modules.

## Must Preserve

Preserve the following from `REFERENCE_APP.jsx`:

- The course title: "Maths & Stats Behind Data Science and AI"
- The module-based structure
- The `modules` array content
- The math concepts:
  - Probability fundamentals
  - Conditional probability
  - Bayes’ Rule
  - Vectors and matrices
  - Recommendation systems
  - Images as numerical arrays
  - PCA and dimension reduction
- Python Lab sections
- Check Yourself questions with show/hide answers
- Mini-projects
- Final 7-question quiz
- Bayes Rule calculator with sliders
- Progress tracking
- localStorage persistence
- Notes box
- Study schedule
- Clean responsive layout

## Allowed Improvements

You may improve:

- File organization
- Component structure
- Styling polish
- Mobile responsiveness
- Accessibility
- README instructions
- Cloudflare deployment readiness
- Minor wording clarity

## Do Not Do

- Do not make the app less mathematical.
- Do not replace the course with generic AI summaries.
- Do not remove Python code labs.
- Do not remove Bayes’ Rule calculations.
- Do not remove module navigation.
- Do not remove the study schedule.
- Do not turn this into a plain static document.
- Do not add a backend, database, paid API, authentication, or server requirement.

## Technical Requirements

Use:

- React
- Vite
- Tailwind CSS
- localStorage for progress and notes
- No backend

The app must build successfully with:

npm run build

Cloudflare Pages settings must be:

Build command: npm run build
Build output directory: dist

## Deliverables

Create or update:

- package.json
- index.html
- src/main.jsx
- src/App.jsx
- src/index.css
- README.md

The README must include:

- how to run in Codespaces
- npm install
- npm run dev -- --host 0.0.0.0
- npm run build
- Cloudflare Pages deployment settings

Open a pull request instead of pushing directly to main.

---
> Source: [nkiliyumwami/mathAIDatascience](https://github.com/nkiliyumwami/mathAIDatascience) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
