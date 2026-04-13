## trending-topics

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
- `bun run dev` - Start development server with hot reload (runs on port 3000)

### Building
- `bun run build` - Build all assets (CSS and JS)
- `bun run build:css` - Build CSS using Tailwind CLI
- `bun run build:js` - Bundle JavaScript with Bun

### Watching
- `bun run watch:css` - Watch and rebuild CSS on changes
- `bun run watch:js` - Watch and rebuild JavaScript on changes

### Installing
- `bun install` - Install dependencies

### Testing
- `bun run test` - Run tests with hot reload
- `bun run test:run` - Run tests once
- `bun run test:ui` - Run tests with UI
- `bun run test:coverage` - Run tests with coverage report

## Architecture

This is a **server-side rendered (SSR)** web application for displaying trending topics, built with Bun and Hono.

### Technology Stack
- **Runtime**: Bun (JavaScript/TypeScript runtime with built-in bundler)
- **Web Framework**: Hono (lightweight, fast web framework)
- **UI**: Hono JSX for server-side rendering
- **Database**: PostgreSQL (via `postgres` library with SQL template literals)
- **Styling**: Tailwind CSS v4 with @tailwindcss/typography plugin

### Layer Architecture

The application follows a clean layered architecture:

```
Route Handler (Hono) -> Service (Business Logic) -> Repository (Model) -> Database
```

**Route Handlers** (`src/routes/`): Split into two hierarchies:
- `api/` - RESTful API endpoints
- `pages/` - SSR page routes with JSX components

**Services** (`src/services/`): Business logic layer that coordinates between routes and repositories

**Models/Repositories** (`src/models/`): Database access layer using the `postgres` library's tagged template literal syntax

**Views** (`src/views/`): JSX components rendered server-side

### Key Architectural Patterns

**1. Repository Pattern with SQL Template Literals**
All database queries use the `postgres` library's tagged template literal syntax:
```typescript
const users = await db`SELECT * FROM users WHERE id = ${id}`
```

**2. Automatic Case Conversion**
Database uses snake_case, but application uses camelCase. Utility functions in `src/utils/case-converter.ts` handle conversion:
- `toCamelCase()` - Converts query results from snake_case to camelCase
- `toSnakeCase()` - Converts data for inserts/updates from camelCase to snake_case

**3. Service Layer Pattern**
Services provide a clean abstraction between routes and models, isolating business logic from HTTP concerns.

**4. Path Alias**
`tsconfig.json` configures `@/*` to resolve to `./src/*` for cleaner imports.

### File Organization

| Directory | Purpose |
|-----------|---------|
| `src/app.ts` | Main Hono app configuration |
| `src/index.ts` | Bun server entry point |
| `src/config/` | Configuration files |
| `src/lib/` | Core libraries (e.g., database connection singleton) |
| `src/middleware/` | Hono middleware (logger, CORS) |
| `src/models/` | Database models/repositories with type definitions |
| `src/routes/` | Route handlers (API and pages) |
| `src/services/` | Business logic layer |
| `src/utils/` | Utility functions |
| `src/views/` | JSX components (pages and layouts) |
| `src/assets/` | Source files for build process |
| `src/static/` | Built static assets (served at `/static/*`) |

### JSX Configuration

The app uses Hono's JSX renderer configured in `tsconfig.json`:
```json
{
  "jsx": "react-jsx",
  "jsxImportSource": "hono/jsx"
}
```

### Page Routes

Page routes use a custom renderer middleware that wraps content with `BaseLayout`:
```typescript
pages.use('*', async (c, next) => {
  c.setRenderer((content) => c.html(<BaseLayout>{content}</BaseLayout>))
  await next()
})
```

### Database Connection

Database connection is managed as a singleton in `src/lib/db.ts`. The connection is initialized from environment variables (`DATABASE_URL`).

### Building Assets

- CSS is compiled from `src/assets/css/` to `src/static/css/` using Tailwind CLI
- JS is bundled from `src/assets/js/` to `src/static/js/` using Bun's built-in bundler

### Testing

This project uses [vitest](https://vitest.dev/) for testing. Database health can be checked via `GET /api/db/test`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lsl233)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lsl233)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
