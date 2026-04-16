## lift-metrics

> This file defines how all code in this project must be written. Read and follow these instructions before generating any code, files, or comments.

# LiftMetrics — Gemini CLI Project Instructions

This file defines how all code in this project must be written. Read and follow these instructions before generating any code, files, or comments.

---

## Project Overview

LiftMetrics is a full-stack workout analytics platform. Users log exercises, track progress over time, and view interactive charts. It is not a workout planner, nutrition tracker, or social platform — it is purely a data tracking and visualization tool.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Django 5.x |
| Database | PostgreSQL (local port 5433) |
| ORM | Django ORM |
| Data processing | pandas |
| Charts | Plotly.js (frontend, not server-side) |
| Frontend | Bootstrap 5 + Django Templates + vanilla HTML/CSS/JS |
| Exercise data | ExerciseDB API (for exercise names, muscle groups, GIFs) |
| Deployment | Render (free tier, managed PostgreSQL) |
| Secrets | python-decouple (.env file) |
| Environment | Python 3.12.1, venv |

---

## Code Style Rules

These are non-negotiable. Apply them to every file you generate.

### General

- Write code as a real developer would, not as an AI assistant
- No emojis anywhere — not in code, comments, docstrings, README, or commit messages
- No excessive docstrings on simple or obvious functions
- No overly descriptive variable names (use `workout`, not `current_user_workout_object`)
- Comments only where the logic is genuinely non-obvious
- Keep functions focused — one responsibility per function
- Avoid boilerplate filler code

### Python / Django

- Use snake_case for variables, functions, model fields
- Use PascalCase for class names
- Imports: stdlib first, then third-party, then local — one blank line between groups
- No unused imports
- Views should be thin — move logic to models or helper functions if it grows
- Use Django's class-based views where they make sense (ListView, DetailView, etc.)
- Use `get_object_or_404` instead of manual try/except for simple lookups
- Keep settings.py clean — use python-decouple for all secrets and environment-specific values

### HTML / Templates

- Use Django template inheritance (base.html with blocks)
- Keep templates readable — indent properly, no one-liner soup
- Bootstrap 5 classes only — no custom CSS frameworks
- All custom CSS goes in a single `static/css/main.css` file
- No inline styles unless absolutely necessary

### JavaScript

- Vanilla JS only (no jQuery, no React)
- Use `const` and `let`, never `var`
- Keep Plotly chart config in a separate `static/js/charts.js` file
- No `console.log` left in production code

---

## UI and Design

- Dark theme throughout — modern, minimal, professional
- Inspired by: Apple Fitness+, Strava, Stripe Dashboard
- Color palette:
  - Background: `#0f172a`, `#1e293b`
  - Cards/surfaces: `#1e293b`, `#334155`
  - Accent (CTAs, highlights): `#06b6d4`, `#3b82f6`
  - Text: `#f1f5f9`, `#e2e8f0`
- Typography: clean sans-serif, strong contrast, clear hierarchy
- Mobile-first using Bootstrap 5 grid
- No decorative clutter — every UI element should serve a function

---

## Plotly Chart Rules

- Use `mode: 'lines'` — no visible scatter markers by default
- Data and values visible only on hover via tooltips
- Dark background matching the app theme (`#0f172a`)
- Smooth spline curves where appropriate
- Accent colors: cyan `#06b6d4` or blue `#3b82f6`
- Gridlines: subtle, low opacity gray
- No chart titles or axis labels that are redundant with surrounding page context
- Chart types in use:
  - Line chart: weight progression per exercise over time
  - Area chart: total volume (sets x reps x weight) per week
  - Bar chart: weekly or monthly volume comparison

---

## Project Structure

```
lift-metrics/
├── liftmetrics/          # Django project config
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── workouts/             # Main Django app
│   ├── models.py
│   ├── views.py
│   ├── urls.py
│   ├── forms.py
│   └── templates/
│       └── workouts/
├── static/
│   ├── css/
│   │   └── main.css
│   └── js/
│       └── charts.js
├── templates/
│   └── base.html
├── .env                  # never committed
├── .env.example          # committed, no real values
├── .gitignore
├── manage.py
├── requirements.txt
└── README.md
```

---

## Database Models (overview)

- **Exercise**: name, muscle_group, equipment, image_url
- **Workout**: FK to User, FK to Exercise, sets, reps, weight, date, notes
- **Profile**: OneToOne to User, height, weight, age

Relationships: User → many Workouts, Exercise → many Workouts, User → one Profile

---

## Git Rules

- Never commit: `venv/`, `.env`, `__pycache__/`, `*.pyc`, `db.sqlite3`
- Always have `.gitignore` before the first commit
- Commit after every working, self-contained change
- Commit messages: lowercase, descriptive, no punctuation at end
  - Good: `add workout logging form`
  - Bad: `Update files`, `WIP`, `fix stuff`

---

## What This Project is NOT

Do not add features outside this scope:

- No workout planning or scheduling
- No pre-built workout programs
- No nutrition or meal tracking
- No social features, sharing, or public profiles
- No AI-generated recommendations

---

## README Style Guide

Write README.md as a developer would — not as a product marketing page.

- Start with a one-line description of what the app does
- Include: tech stack, local setup instructions, environment variables, how to run
- No feature bullet lists that read like sales copy
- No emojis
- No "This project was built to..." motivational framing
- Screenshots are fine if they add clarity

---

## Summary

Write clean, practical, human-looking code. Follow the stack. Respect the design system. Keep commits small and meaningful. When in doubt, do less and do it well.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anriudh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
