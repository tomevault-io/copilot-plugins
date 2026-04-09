## smart-training-api

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development
pnpm dev              # Run with hot reload (tsx --watch)

# Linting
pnpm dlx eslint .     # Lint the project

# Database
pnpm dlx prisma migrate dev    # Create and apply a migration
pnpm dlx prisma generate       # Regenerate Prisma client after schema changes
pnpm dlx prisma studio         # Open Prisma Studio
```

There are no tests configured yet.

## Architecture

**Fastify + Zod + Prisma REST API** for a workout tracking application.

### Request lifecycle

Routes (`src/routes/`) → Use Cases (`src/useCases/`) → Prisma (`src/lib/db.ts`)

- **Routes** handle HTTP concerns: auth session validation via `better-auth`, request/response schema, error mapping.
- **Use Cases** contain business logic. Each use case is a class with an `execute(dto)` method.
- **Schemas** (`src/schemas/index.ts`) define Zod schemas shared between route validation and response serialization.

### Key patterns

- `fastify-type-provider-zod` wires Zod schemas directly into Fastify for type-safe request/response handling. Always use `app.withTypeProvider<ZodTypeProvider>().route(...)`.
- All routes register via plugin functions and are mounted with a prefix in `src/index.ts`.
- Prisma client is generated to `src/generated/prisma/` (not the default location). Import types and enums from there, e.g. `../generated/prisma/enums.js`.
- Use `.js` extensions in imports (required for ESM).

### Authentication

`better-auth` handles auth at `/api/auth/*`. All workout routes check the session using:

```ts
const session = await auth.api.getSession({
  headers: fromNodeHeaders(request.headers),
});
```

### Data model

`User` → `WorkoutPlan` (one active at a time) → `WorkoutDay` → `WorkoutExercise`
`WorkoutDay` also has `WorkoutSession` records (completed sessions).
All relations use cascade deletes. Creating a new workout plan automatically deactivates the previous active one (ACID transaction in `CreateWorkoutPlan`).

### Error handling

Throw typed errors from `src/errors/index.ts` in use cases; catch and map them to HTTP status codes in routes.

### API docs

Available at `http://localhost:8080/docs` (Scalar UI) when running locally.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/danielangelo1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/danielangelo1)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
