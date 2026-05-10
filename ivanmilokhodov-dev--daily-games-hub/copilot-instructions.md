## daily-games-hub

> Development guide for AI assistants working on Scordle.

# CLAUDE.md

Development guide for AI assistants working on Scordle.

## Project Overview

Full-stack web app for tracking and sharing daily puzzle game scores with friends. Users submit scores for games like Wordle and Connections, compete in friend groups, and maintain streaks.

## Tech Stack

- **Backend**: Java 17, Spring Boot 3.2, Spring Security + JWT, Spring Data JPA
- **Frontend**: React 18, Vite 5, React Router 6, Axios, i18next, Recharts
- **Database**: H2 (dev), PostgreSQL 15 (prod)
- **Testing**: JUnit + Mockito (backend), Vitest + React Testing Library (frontend)
- **CI/CD**: GitHub Actions
- **Containerization**: Docker + Docker Compose, Nginx for frontend

## Quick Commands

```bash
# Backend
cd backend && ./mvnw spring-boot:run        # Start dev server (port 8080)
cd backend && ./mvnw test                    # Run backend tests
cd backend && ./mvnw clean package -DskipTests  # Build JAR

# Frontend
cd frontend && npm install                   # Install deps
cd frontend && npm run dev                   # Start dev server (port 5173)
cd frontend && npm test -- --run             # Run tests once
cd frontend && npm run build                 # Production build
```

## Project Structure

```
backend/src/main/java/com/scordle/hub/
  config/          # SecurityConfig, JwtUtil, JwtAuthFilter, CustomUserDetailsService, AdminInitializer
  controller/      # REST controllers (11 files): Auth, Score, FriendGroup, Admin, etc.
  service/         # Business logic (10 files): Score, FriendGroup, User, Rating, PasswordReset, etc.
  repository/      # Spring Data JPA repositories (8 files)
  model/           # JPA entities (9 files): User, Score, GameType (enum), FriendGroup, Streak, etc.
  dto/             # Data transfer objects (17 files)
  util/            # DateUtils

frontend/src/
  pages/           # 15 page components: Home, Login, Register, Dashboard, SubmitScore, Groups, etc.
  components/      # Navbar, Guide, LanguageSelector
  context/         # AuthContext, ThemeContext, GuideContext
  services/        # api.js (Axios instance with JWT interceptor)
  i18n/locales/    # en.json, uk.json
```

## Key Architecture Decisions

- **Authentication**: Stateless JWT with 24h expiry, BCrypt passwords. Token in `Authorization: Bearer` header.
- **Score uniqueness**: One score per user per game per day (composite unique constraint).
- **Admin**: User ID 1 is the primary admin. Admin role stored as `isAdmin` boolean on User entity.
- **Streaks**: Per-game streaks (consecutive days for a specific game) and global day streak (any game).
- **Rating**: ELO-style rating system tracked per game.
- **i18n**: English and Ukrainian via i18next with browser language detection.
- **API proxy**: In dev, Vite proxies `/api` requests to `http://localhost:8080`.

## Supported Games (GameType enum)

WORDLE, CONNECTIONS, CONTEXTO, SEMANTLE, HORSE, TRAVLE, MINUTE_CRYPTIC, COUNTRYLE, SPOTLE, BANDLE

## Environment Variables

```
DB_PASSWORD          # PostgreSQL password (prod)
JWT_SECRET           # JWT signing key, min 256 bits
CORS_ORIGINS         # Comma-separated allowed origins
SPRING_PROFILES_ACTIVE=prod  # Activates PostgreSQL + prod settings
```

## Testing Notes

- Backend tests use `@MockBean` for service mocking and `@WebMvcTest` for controller tests.
- Frontend tests use jsdom environment and React Testing Library.
- CI runs on push to `main`/`develop` and on PRs to those branches.

## Common Pitfalls

- The `isAdmin` field on User must be nullable for PostgreSQL compatibility (existing rows may lack it).
- Password reset has rate limiting. Dev mode skips email sending.
- The frontend package name is `scorle-frontend`.
- H2 console is only accessible in dev profile at `/h2-console`.

## Workflow Orchestration

### 1. Plan Mode Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately - don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution

### 3. Self-Improvement Loop
- After ANY correction from the user: update `tasks/lessons.md` with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for relevant project

### 4. Verification Before Done
- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### 5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes - don't over-engineer
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests - then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

## Task Management

1. **Plan First**: Write plan to `tasks/todo.md` with checkable items
2. **Verify Plan**: Check in before starting implementation
3. **Track Progress**: Mark items complete as you go
4. **Explain Changes**: High-level summary at each step
5. **Document Results**: Add review section to `tasks/todo.md`
6. **Capture Lessons**: Update `tasks/lessons.md` after corrections

## Core Principles

- **Simplicity First**: Make every change as simple as possible. Impact minimal code.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact**: Changes should only touch what's necessary. Avoid introducing bugs.

## PR and Commit Style

- Never include "Generated with Claude Code" or any AI attribution in PR descriptions, commit messages, or code comments.

---
> Source: [ivanmilokhodov-dev/daily-games-hub](https://github.com/ivanmilokhodov-dev/daily-games-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
