## edu-quiz-3

> EduQuizAI is an educational application designed for creating and conducting quizzes, primarily at the school level.

# AI Rules for edu-quiz

EduQuizAI is an educational application designed for creating and conducting quizzes, primarily at the school level.
The system enables the generation of interactive quizzes with AI support. 
The application distinguishes between two user roles: an administrator who manages quizzes and accounts, 
and a student who browses, launches, and completes quizzes, receiving immediate feedback on answer correctness.

## Tech Stack

- React 19 (Core frontend framework)
- Tailwind 4 (Primary styling solution)
- Shadcn/ui (React component library)
- Python + FastAPI (Core backend framework) + unicorn (ASGI server)
- Database Stack: SQLAlchemy + SQLite

## Project Structure

- `/doc/` - Documentation files
- `/data/` - SQLite database file and other data
- `/frontend/` - React frontend
- `/frontend/src` - Source code
- `/frontend/src/assets` - Static assets like images and fonts
- `/frontend/src/components` - Client-side components written in React
- `/frontend/src/components/ui` - Client-side components from Shadcn/ui
- `/frontend/src/lib` - Utility functions and hooks
- `/backend/` - FastAPI backend
- `/backend/app/main.py` - Entry point
- `/backend/app/core/` - Core logic (settings, security, middleware)
- `/backend/app/db/` - Database setup and session management
- `/backend/app/models/` - SQLAlchemy ORM models
- `/backend/app/schemas/` - Pydantic validation schemas
- `/backend/app/crud/` - CRUD database operations
- `/backend/app/routers/` - API endpoints (routes)
- `/backend/app/services/` - Business logic services
- `/backend/app/tests/` - Backend tests
- `/README.md` - Project-level documentation

## Coding practices

### Guidelines for clean code

- Handle errors and edge cases at the beginning of functions.
- Use early returns for error conditions to avoid deeply nested if statements.
- Place the happy path last in the function for improved readability.
- Avoid unnecessary else statements; use if-return pattern instead.
- Implement proper error logging and user-friendly error messages.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marcinmatys)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/marcinmatys)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
