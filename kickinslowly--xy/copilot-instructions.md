## xy

> Full autonomous permissions granted. Do not ask, just do:

# CLAUDE.md - Project Intelligence

## Permissions

Full autonomous permissions granted. Do not ask, just do:
- **Web Search**: Search the web for research, competitive analysis, standards, best practices
- **Local Commands**: Run any local commands (npm, pip, python, flask, git, etc.)
- **Search**: Glob, Grep, explore codebase freely
- **Read**: Read any file in the project
- **Write**: Write/edit any file in the project
- **Bash**: Execute any shell commands needed
- **File Creation**: Create new files as needed

## Project Overview

**Mr. A's Math Tools** - An educational math platform for teachers and students.

- **Backend**: Flask 3.1.2 + Flask-SocketIO (real-time multiplayer)
- **Database**: SQLAlchemy (SQLite dev / PostgreSQL prod)
- **Frontend**: Vanilla JS, Canvas rendering, Socket.IO client
- **Auth**: Google OAuth 2.0 + JWT sessions
- **Deploy**: Render (Gunicorn + eventlet)

## Key Files

| File | Purpose | LOC |
|------|---------|-----|
| `app.py` | Flask app, models, routes, socket events | ~1007 |
| `static/main.js` | Coordinate plane mode | ~3901 |
| `static/line_mode.js` | Line graphing mode | ~1564 |
| `static/battleship.js` | Battleship game | ~974 |
| `static/meme_wars.js` | Meme Wars game | ~799 |
| `static/meme_dash.js` | Meme Dash platformer | ~1650 |
| `static/ratios_mode.js` | Ratio puzzles | ~642 |
| `static/bot_ai.js` | Bot opponent logic | ~136 |
| `static/style.css` | Main theme CSS | - |
| `static/css/theme.css` | Dashboard theme | - |

## Templates

`templates/`: base.html, index.html, line_mode.html, battleship.html, meme_wars.html, meme_dash.html, ratios.html, dashboard.html

## Database Tables (20 total)

Users, classes, class_memberships, skills, activities, assignments, submissions, sessions, events, mastery_snapshots, game_results, achievements, user_achievements, teacher_private_names, access_logs, shop_items, user_items

## Game Modes

1. **Coordinate Plane** (`/`) - Interactive 2D canvas, vertex/line challenges, transformations, collaborative
2. **Line Graph** (`/line-mode`) - Data tables, Chart.js visualization, slope/equation calculation
3. **Battleship** (`/battleship`) - Turn-based 2-player, 10x10 grid, bot AI, meme skins
4. **Meme Wars** (`/meme-wars`) - Battleship variant with meme collection mechanic
5. **Meme Dash** (`/meme-dash`) - Platformer, meme collection, power-ups, Terminator bot
6. **Ratios** (`/ratios`) - Visual ratio puzzles, drag-and-drop, multiple challenge types
7. **Shop** (`/shop`) - Cosmetics store: titles, board themes, avatar frames with coin currency
8. **Dashboard** (`/dashboard`) - Analytics, per-mode stats, achievements, recent activity

## Architecture Patterns

- **State**: Client IIFE closures + Socket.IO room state + LocalStorage auth
- **Results Flow**: Game -> `window.recordResult()` -> `/api/results` -> DB + achievements + mastery update -> dashboard refresh
- **Standards Flow**: Game result -> `resolve_standards_for_result()` (mode+challenge_type -> standard codes) -> `update_mastery_for_result()` (Bayesian update per standard) -> mastery_snapshots table
- **Multiplayer**: PIN-based rooms, owner authority model, 20Hz broadcast, 800ms failover
- **Privacy**: Minimal PII, per-class display names, teacher-only private name access, audit logs

## Guiding Documents

- **WORKSTYLE.md** - Prioritization criteria, competitors, pivot rules
- **goals.md** - Detailed project goals (G1-G7+)
- **FEATURES.md** - Authoritative inventory of what exists (routes, models, modes, systems)
- **skills.md** - Repeatable task patterns
- **memory.md** - Session continuity tracking

## Working Rules

1. Follow WORKSTYLE.md prioritization when working autonomously
2. Goals decide success, WORKSTYLE.md decides what to try next
3. Always record progress in memory.md
4. After every major work block, enter Consolidation mode
5. Never brute-force the same failed approach - switch strategies
6. Keep code simple, fun, student-facing - no over-engineering the UX complexity
7. Every feature must serve learning outcomes or engagement

---
> Source: [kickinslowly/xy](https://github.com/kickinslowly/xy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
