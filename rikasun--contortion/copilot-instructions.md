## contortion

> Personal flexibility training app. Single user, single plan. Walks through a timed sequence of exercises with spoken cues, illustrations, and a chime in the final three seconds of each phase.

# CLAUDE.md

Personal flexibility training app. Single user, single plan. Walks through a timed sequence of exercises with spoken cues, illustrations, and a chime in the final three seconds of each phase.

## Layout

```
web/      Vite + React 19 + TS frontend
worker/   Cloudflare Worker + D1 backend (one TS file)
.github/workflows/deploy.yml   CI: builds + Pages deploy + optional Worker deploy
```

## Stack

- **Frontend**: Vite 6, React 19, TypeScript, TanStack Router (browser history) + TanStack Query, Tailwind 4, Radix Themes. Voice cues via Web Speech API; chimes via Web Audio API.
- **Backend**: single Cloudflare Worker + D1 (SQLite). HMAC-SHA256 signed session tokens. Rate-limited writes and unlocks. CORS allowlist via `ALLOWED_ORIGINS` env var.
- **Hosting**: GitHub Pages (frontend) + Cloudflare (Worker + DB). Custom domain via DNS CNAME.

## Auth model

App is anonymous-write: anyone with the URL can POST a finished class. Reading history is gated by a single shared password (configured as a Worker secret); successful unlock returns a 30-day HMAC token stored in `localStorage`. Brute force is throttled by IP-based rate limits at the Worker.

Adding real per-user auth later is intended to be cheap — add a `user_id` column, NULL existing rows.

## Threshold guard ("junk session" rule)

A finished class is only POSTed if both:
- `durationSec > 60`, AND
- `completedAll === true` OR `completedExerciseIds.length / totalExercises >= 0.6`

Logic in `web/src/hooks/useSessionLog.ts` (`isSessionRecordable`). Used from the class screen's `onFinish` callback and the "End class" button (which calls `useSession.endNow()`).

## Plan structure + transitions

Plans live in `web/src/data/plans/<id>.ts`. Each plan is an array of exercises; each exercise is an array of phases; each phase has `seconds` and `cues` (spoken at integer second offsets).

`Exercise.transitionSec` inserts a synthetic "Get ready — X" phase before the exercise (skipped for the very first exercise in a session). Defaults are applied per-section at the bottom of the plan file. Current values in `legs.ts`:
- Warm-up: 5s
- Prep Stretches: 10s
- Splits: 10s
- Active Flex: 8s

## Dev commands

Top-level: **`./dev.sh`** — bootstraps `worker/.dev.vars`, installs deps if missing, applies local D1 schema, and runs both servers in parallel. Single Ctrl-C kills both.

Lower-level (if you need just one piece):

```sh
# frontend
cd web
pnpm dev                                        # http://localhost:5173/
pnpm exec tsc -p tsconfig.app.json --noEmit     # typecheck
pnpm exec vite build                            # production build → web/dist/

# worker
cd worker
pnpm dev                                        # http://localhost:8787 (wrangler --local)
pnpm exec tsc --noEmit                          # typecheck
pnpm wrangler d1 execute contortion --local --file=./schema.sql   # apply schema (idempotent)
```

The frontend's `web/.env.development` points `VITE_API_BASE_URL` at `http://localhost:8787`. Production reads the same env var from a GH Actions repo secret.

## Local vs production isolation

| | Local | Production |
|---|---|---|
| API base | `localhost:8787` | `*.workers.dev` (from GH Actions secret) |
| Runtime | `wrangler dev --local` (Miniflare) | Cloudflare edge |
| Database | SQLite at `worker/.wrangler/state/v3/d1/...` | Cloudflare D1 (remote) |
| Secrets | `worker/.dev.vars` (placeholder) | `wrangler secret put …` (real values) |

Local writes never reach prod; prod writes never reach local. Safe to mash buttons during dev.

## Worker endpoints

| Method | Path | Auth | Notes |
|---|---|---|---|
| POST | `/api/sessions` | none | Write a finished session. Rate-limited per IP. |
| GET | `/api/sessions` | Bearer token | Read history (newest first, limit 200). |
| DELETE | `/api/sessions/:id` | Bearer token | Delete one row. |
| DELETE | `/api/sessions` | Bearer token | Clear all rows. |
| POST | `/api/unlock` | none | Submit password, get token. Rate-limited per IP. |

CORS only allows origins listed in `ALLOWED_ORIGINS`.

## Adding a plan

1. New file `web/src/data/plans/<id>.ts` exporting a typed `Plan`.
2. Register it in `web/src/data/plans/index.ts`.
3. Appears on the home screen at `/class/<id>`.

Phase shape:
```ts
{ label: "Left leg — 60s", seconds: 60, cues: [{ at: 0, say: "Begin." }, { at: 30, say: "Halfway." }] }
```

## Gotchas

- TanStack Router's plugin auto-regenerates `web/src/routeTree.gen.ts` when route files change — don't edit it by hand.
- The class screen has a "Begin gate" before the timer starts. It exists to satisfy mobile Safari's autoplay-unlock requirements (the click is the gesture that unlocks `speechSynthesis` and `AudioContext`). Don't remove it without an alternative gesture-priming strategy.
- `tsconfig.app.json` has `erasableSyntaxOnly` enabled — no parameter properties in constructors (`constructor(public foo: string)` won't compile). Use regular field declarations + assignments.
- `vite.config.ts` `base` and `web/public/404.html` `pathSegmentsToKeep` must stay paired: `base: "/"` ↔ `pathSegmentsToKeep = 0` (custom domain at root); `base: "/contortion/"` ↔ `pathSegmentsToKeep = 1` (github.io path).

---
> Source: [rikasun/contortion](https://github.com/rikasun/contortion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
