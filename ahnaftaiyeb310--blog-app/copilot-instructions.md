## blog-app

> A full-stack blog application featuring a Django backend and a frontend (framework to be confirmed).

# Blog App Project Context

## Project Overview
A full-stack blog application featuring a Django backend and a frontend (framework to be confirmed).

## Tech Stack
- **Backend:** Django, Django Rest Framework (DRF)
- **Database:** (To be confirmed, likely PostgreSQL or SQLite)
- **Task Queue:** Celery (identified in `backend/Blog/celery.py`)
- **Environment Management:** Pipenv (`Pipenv` and `Pipenv.lock` present)
- **Frontend:** Next.js (App Router), Tailwind CSS, JavaScript
    - **UI Library:** shadcn/ui
    - **State Management:** (Pending selection, likely Zustand or Context API)
    - **Icons:** Lucide React (standard with shadcn)
    - **Standards:** WCAG AA Accessibility, SEO Optimized, Performance Best Practices (LCP, FID, CLS)

## Architecture & Modules
The backend is structured into several Django apps:
- `blog`: Core blogging functionality (posts, comments).
- `users`: User management and authentication.
- `likes`: Interaction system for posts/comments.
- `notifications`: User notification system.
- `tags`: Tagging system for blog posts.

## Key Files
- `backend/manage.py`: Django entry point.
- `backend/Blog/settings.py`: Project configurations.
- `backend/Blog/urls.py`: Main routing.
- `backend/AI_CONTEXT.md`: Existing AI-related context.

## Current State
The project has a defined backend structure with migrations and basic app logic implemented.
The frontend is being initialized with Next.js, Tailwind CSS, and shadcn/ui to provide a modern, responsive, and accessible user interface.
# Skill Priority Rules
1. **FRONTEND:** Use `next-best-practices` skill for all Next.js logic, file conventions, and performance patterns.
2. **UI COMPONENTS:** Use `shadcn-ui` (via `ui-ux-pro-max` skill with `--stack shadcn` or standard library) for all component structure.
3. **THEMING & POLISH:** Use `ui-ux-pro-max` skill for colors, spacing, typography, and visual polish.
   - *Conflict Resolution:* If `ui-ux-pro-max` suggests a pattern that breaks `shadcn-ui` accessibility, `shadcn-ui` defaults win.
4. **CACHING:** Refer to `.gemini/skills/next-cache-components.md` for Next.js 16+ caching strategies (PPR, `use cache`).
5. CORE LOGIC: Use 'superpowers:brainstorming' for requirements gathering.


<!-- Not needed now -->
<!-- #. BACKEND: (Django) Prioritize standard Django patterns + Django Ninja. -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AhnafTaiyeb310)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AhnafTaiyeb310)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
