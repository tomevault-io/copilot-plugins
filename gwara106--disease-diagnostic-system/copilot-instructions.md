## disease-diagnostic-system

> Disease Diagnosis and Hospital Recommendation System for Nepal. Full-stack

# Swastha — working notes for agents

Disease Diagnosis and Hospital Recommendation System for Nepal. Full-stack
TypeScript: a React 18 SPA and an Express 5 API that share one port in
development, backed by MongoDB through Mongoose.

Read `README.md` first for setup and architecture. This file covers the
conventions to follow when changing the code.

## Tech stack

- **Frontend**: React 18 + React Router 6 (SPA) + TypeScript + Vite + TailwindCSS 3
- **Server state**: TanStack Query; the session lives in `client/context/AuthContext.tsx`
- **Backend**: Express 5, mounted into the Vite dev server by `vite.config.ts`
- **Database**: MongoDB + Mongoose, with an in-memory fallback for local work
- **Validation**: Zod on the server, mirrored in `client/lib/validation.ts`
- **Testing**: Vitest — unit tests plus end-to-end HTTP tests against the real app
- **UI**: Radix primitives in `client/components/ui/`, Lucide icons

Path aliases: `@/*` → `client/`, `@shared/*` → `shared/`.

> Server files import shared types with a **relative** path (`../../shared/api`),
> not the `@shared` alias. `vite.config.ts` loads the server module graph and
> Vite's config loader does not apply aliases — the relative path keeps
> `tsc`, both Vite configs, Vitest and plain Node all working.

## Project layout

```
client/     React SPA — pages/, components/{ui,common,layout,admin,hospitals}/, lib/, context/, hooks/
server/     Express API — config/, db/, middleware/, models/, routes/, seed/, services/, validation/
shared/     api.ts — the contract both sides import
netlify/    Serverless adapter
```

## Conventions

### Every endpoint returns the same envelope

```ts
{ success: true,  data: T, message?: string }
{ success: false, message: string, errors?: Record<string, string> }
```

Never write a bare `res.json(payload)` — the client unwraps `data` and turns a
failure into a typed `ApiError`.

### Adding an API route

1. Add the response type to `shared/api.ts` if the client needs it.
2. Add a Zod schema to `server/validation/schemas.ts` for the body and/or query.
3. Create the handler in `server/routes/<name>.routes.ts`:

```ts
router.post(
  "/",
  requireAuth, // or optionalAuth / requireAdmin
  validate(createThingSchema), // replaces req.body with the parsed value
  asyncHandler(async (req, res) => {
    const thing = await db().things.create(req.body);
    res
      .status(201)
      .json({ success: true, message: "Thing added", data: thing });
  }),
);
```

4. Mount it in `server/index.ts` and document it in `server/routes/docs.routes.ts`
   (that array drives both the HTML docs and the OpenAPI document).
5. Add a typed wrapper to `client/lib/endpoints.ts`.

Rules that matter:

- Wrap async handlers in `asyncHandler` so rejections reach the error middleware.
- Throw `ApiError.badRequest/unauthorized/forbidden/notFound/conflict` — never
  hand-roll an error response.
- Read validated query strings with `validatedQuery<T>(req)`; Express 5 exposes
  `req.query` through a getter, so `validate(schema, "query")` stores the parsed
  value separately.
- `req.params.id` is typed `string | string[]` in Express 5 — wrap it in `String()`.
- Go through `db()`, never `mongoose` directly. Both backends implement the same
  `Repository<T>` interface, so anything the in-memory query engine cannot express
  must not be used against MongoDB either.

### Adding a page

1. Create `client/pages/MyPage.tsx` and wrap the content in `PageLayout`.
2. Register it in `client/App.tsx` **above** the catch-all `*` route, inside
   `RequireAuth` / `RequireAdmin` when it needs a session.
3. Fetch with TanStack Query through `client/lib/endpoints.ts`, and render
   `LoadingState` / `ErrorState` / `EmptyState` from `components/common/States`
   rather than inventing new ones.
4. Use `TextField` / `SelectField` / `TextAreaField` and `buttonStyles` from
   `components/common/Field` so forms stay consistent.
5. Report outcomes with `toast` from `sonner`; confirm destructive actions with
   `ConfirmDialog`.

### Environment variables

`server/config/env.ts` is the only file that reads `process.env`. Importing it must
never throw — production requirements are enforced by `assertEnvIsUsable()`, which
`initializeApp()` calls once at boot. Add new variables there and document them in
`.env.example` and the README table.

### Styling

TailwindCSS utilities with the brand palette from `tailwind.config.ts`
(`brand-navy`, `brand-coral`, `brand-lilac`, `brand-mint`, `brand-ink`). Combine
conditional classes with `cn()` from `@/lib/utils`.

## Commands

```bash
npm run dev        # client + API on http://localhost:8080
npm run build      # dist/spa + dist/server
npm start          # run the production build
npm run seed       # load reference data into MongoDB (one time, idempotent)
npm run seed:reset # clear and reload the reference collections
npm test           # Vitest
npm run typecheck  # tsc across client, server and shared
```

### Reference data is read-only at runtime

Symptoms, diseases, hospitals and health tips are written **once** by
`npm run seed`. `prepareDatabase()` (called from `initializeApp()`) only creates
indexes and reports the counts — it never inserts, unless the process is on the
in-memory store or `SEED_ON_BOOT=true`. If you add a new reference collection,
extend `server/seed/index.ts` rather than writing to it from a route or a boot
hook.

Before finishing a change: `npm run typecheck && npm test && npm run build`.

## Medical-safety rules

These are product requirements, not preferences:

- Never present a result as a diagnosis. Confidence is capped at 95%.
- Red-flag symptoms (`RED_FLAG_SYMPTOMS` in `server/services/naiveBayes.ts`) must
  raise the emergency banner regardless of the ranking.
- Every results surface keeps the "consult a clinician" caveat and the 102
  emergency number reachable.

---
> Source: [Gwara106/Disease-Diagnostic-System](https://github.com/Gwara106/Disease-Diagnostic-System) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
