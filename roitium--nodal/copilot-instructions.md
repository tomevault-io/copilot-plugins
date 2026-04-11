## nodal

> Nodal is a full-stack, cross-platform microblogging-style note-taking application. It is managed as a monorepo using **Bun workspaces**, supporting a modern tech stack across backend, web, and mobile.

# GEMINI.md - Nodal Monorepo Context

## Project Overview
Nodal is a full-stack, cross-platform microblogging-style note-taking application. It is managed as a monorepo using **Bun workspaces**, supporting a modern tech stack across backend, web, and mobile.

### Architecture
- **Monorepo Manager**: [Bun](https://bun.sh/) (v1.3.5, configured via `packageManager` in `package.json`)
- **Backend (`apps/backend`)**: [ElysiaJS](https://elysiajs.com/) (Bun framework) + [Drizzle ORM](https://orm.drizzle.team/) + [PostgreSQL](https://www.postgresql.org/)
- **Web Frontend (`apps/web`)**: [React](https://react.dev/) + [React Router 7](https://reactrouter.com/) + [Tailwind CSS 4](https://tailwindcss.com/)
- **Android App (`apps/android`)**: [Kotlin](https://kotlinlang.org/) + [Jetpack Compose](https://developer.android.com/jetpack/compose) + [Retrofit](https://square.github.io/retrofit/)
- **Data Migration (`apps/recover`)**: Scripts for migrating data from Memos.

## Key Directories
- `apps/backend`: REST API implementation.
  - `src/controllers`: API route handlers (auth, memos, resources).
  - `src/db/schema.ts`: Drizzle database schema definitions.
- `apps/web`: Web application implementation.
  - `app/routes.ts`: Configuration-based routing (React Router 7).
  - `app/routes/`: Route components.
- `apps/android`: Native Android application.
  - `app/src/main/java/com/roitium/nodal/ui`: Compose UI components and screens.
  - `app/src/main/java/com/roitium/nodal/data`: Repositories and network models.
- `docs/spec.md`: Shared project specifications (e.g., API date formatting).

## Building and Running

### Root Environment
Install all dependencies for all workspaces:
```bash
bun install
```

### Backend (`apps/backend`)
Run in development mode (with watch):
```bash
cd apps/backend
bun dev
```

### Web Frontend (`apps/web`)
Run in development mode:
```bash
cd apps/web
bun dev
```
Build for production:
```bash
bun build
```
Type check:
```bash
bun run typecheck
```

### Android (`apps/android`)
Open the `apps/android` directory in **Android Studio**. Use Gradle to build and run the application on a simulator or physical device.

## Development Conventions
- **Language**: Use TypeScript for all web and backend development.
- **API Standards**:
  - Follow the ISO 8601 format for all `createdAt` / `updatedAt` fields, including timezone (e.g., `2023-01-01T00:00:00.000Z`).
  - API versioning is prefixed with `/api/v1`.
- **Database**:
  - Use Drizzle Kit for migrations: `bunx drizzle-kit generate` and `bunx drizzle-kit push`.
- **Storage**:
  - Use Supabase for file/resource storage (configured in backend `.env`).
- **Styling**:
  - Web: Use Tailwind CSS 4 utility classes.
  - Android: Use Jetpack Compose Material 3 components.
- **Workflow**:
  - Avoid staging or committing changes unless explicitly requested.
  - Reproduce bugs before fixing them.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/roitium)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/roitium)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
