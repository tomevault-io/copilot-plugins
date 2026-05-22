## tejaswiquiz

> A React + Vite exam prep app for a high school student (Tejaswi) studying for finals in **Biology**, **Chemistry**, and **Physics**. Each subtopic offers a YouTube video explainer and a multiple-choice quiz with per-topic progress tracking stored in `localStorage`.

# TejaswiQuiz — Project Context for Claude

## What this project is

A React + Vite exam prep app for a high school student (Tejaswi) studying for finals in **Biology**, **Chemistry**, and **Physics**. Each subtopic offers a YouTube video explainer and a multiple-choice quiz with per-topic progress tracking stored in `localStorage`.

## Tech stack

- **React 18** (JSX) + **Vite**
- Single-file CSS (`src/styles.css`) — no CSS-in-JS, no Tailwind
- No backend; all state lives in `localStorage` via `src/hooks/useProgress.js`

## Key files

| File | Purpose |
|------|---------|
| `src/App.jsx` | Top-level router: dashboard → subject → subtopic |
| `src/data/curriculum.js` | All subjects, subtopics, video IDs, and MCQs — **edit here to add content** |
| `src/styles.css` | All styles |
| `src/hooks/useProgress.js` | localStorage-backed progress tracker |
| `src/components/Dashboard.jsx` | Three subject cards + overall stats |
| `src/components/SubjectView.jsx` | Subtopic list for a subject |
| `src/components/SubtopicView.jsx` | Video + quiz for one subtopic |
| `src/components/Quiz.jsx` | Multiple-choice quiz with explanations |
| `src/components/ProgressBar.jsx` | Reusable progress bar |

## Design system

**Always read `DESIGN.md` before making any visual or styling changes.**

`DESIGN.md` defines the complete "Digital Brutalist Prep" design system, including:

- **Color tokens** — dark charcoal background (`#131313`), vibrant orange accent (`#FF5733`), full Material-style palette
- **Typography** — `Syne` for headings (extra-bold, tight leading), `Inter` for body copy, `Space Mono` for labels/metadata
- **Spacing** — 4px base unit, 24px gutters, 3px border widths, 6px hard shadows
- **Elevation** — no blur/gradients/transparency; depth via hard offset shadows and layered borders
- **Shape language** — strictly 0px border-radius (sharp corners only)
- **Components** — buttons (orange fill, 3px border, 4px lift on hover), inputs (dark fill, white→orange border on focus), cards (3px white border, optional striped header), progress bars (16px tall, rectangular), chips/tags (Space Mono, 2px border, flip to orange on select)

The personality is **"Intellectual Punk"** — raw, high-energy, and confident. No soft shadows, no gradients, no rounded corners.

## Conventions

- Questions in `curriculum.js` follow: `{ q, options[], correct (index), explanation }`
- To embed a YouTube video, set `videoId` on the subtopic object; otherwise a search-link button is shown
- Progress key in localStorage: `tejaswi-quiz-progress-v1`
- Run locally: `npm install && npm run dev` → http://localhost:5173

---
> Source: [ericshearer/TejaswiQuiz](https://github.com/ericshearer/TejaswiQuiz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
