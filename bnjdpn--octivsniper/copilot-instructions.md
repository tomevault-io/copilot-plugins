## octivsniper

> OctivSniper auto-books classes on any gym using the Octiv Fitness platform the instant the booking window opens. Classes typically open a configurable number of days before at the class end time. The app pre-fetches class IDs, starts attempts 30s early, and retries aggressively. Compatible with any Octiv Fitness tenant (CrossFit, yoga, HIIT, etc.).

# CLAUDE.md

## Project overview

OctivSniper auto-books classes on any gym using the Octiv Fitness platform the instant the booking window opens. Classes typically open a configurable number of days before at the class end time. The app pre-fetches class IDs, starts attempts 30s early, and retries aggressively. Compatible with any Octiv Fitness tenant (CrossFit, yoga, HIIT, etc.).

## Stack

- **Runtime:** Bun (TypeScript, no transpile step)
- **Dependencies:** None at runtime. `bun-types` dev only.
- **No framework** - pure CLI with `process.argv` routing
- **Interactive UI** built with raw ANSI escape codes (no dependency)

## Commands

After `bun link`, the `octiv` command is available globally:

| Command | Description |
|---------|-------------|
| `octiv` | **Interactive mode** - login, pick classes, launch scheduler |
| `octiv login` | Direct login (for scripting) |
| `octiv add <day> <time> <name>` | Add a slot manually |
| `octiv list` | Show configured slots |
| `octiv remove <index>` | Remove a slot by index |
| `octiv next` | Display upcoming booking attempts with timing |
| `octiv test` | Dry-run: validate JWT, fetch today's classes |
| `octiv run` | Start scheduler daemon (no interactive menu) |

## Architecture

```
src/
  index.ts       # Entry point, arg routing (no args → interactive mode)
  cli.ts         # Interactive mode + legacy command handlers
  ui.ts          # Terminal UI: multiSelect, spinner, password input, colors
  api.ts         # Octiv REST client (login, refresh, getUserInfo, getClassDates, bookClass)
  scheduler.ts   # Opening time calculation, pre-fetch, retry loop, token refresh, reschedule
  config.ts      # config.json read/write, validation helpers
  types.ts       # Shared TypeScript interfaces
```

## Key files

- `config.json` — **gitignored**, contains JWT + refreshToken + credentials + slot list
- `src/ui.ts` — interactive terminal UI (multiSelect with arrow keys, spinner, password masking)
- `src/api.ts` — all Octiv API calls, base URL: `https://api.octivfitness.com`
- `src/scheduler.ts` — core booking logic: 30s anticipation, 2min pre-fetch, auto token refresh

## API details

Base: `https://api.octivfitness.com`

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/login` | POST | Auth (`{username, password}` → accessToken + refreshToken) |
| `/oauth/token` | POST | Refresh token (`{grant_type, refresh_token, client_id}`) |
| `/api/users/me` | GET | User info (userId via `data.id`, tenantId via `data.userTenants[0]`) |
| `/api/class-dates` | GET | List classes (response: `{data: [...]}`, fields: `name`, `startTime`, `endTime`, `limit`, `bookings[]`) |
| `/api/class-bookings` | POST | Book a class (`{classDateId, userId}`) |
| `/api/class-bookings/{id}/cancel` | PUT | Cancel a booking |

Required headers: `Authorization: Bearer <JWT>`, `X-CamelCase: true`, `Accept: application/json, text/plain, */*`, `bypass-tunnel-reminder: *`

## API response quirks

- Login: field is `username` (not `email`), response has `accessToken` + `refreshToken` + `expiresIn`
- `/api/users/me`: flat response (no `data` wrapper). tenantId at `userTenants[0].tenantId`, locationId at `userTenants[0].tenant.locations[0].id`
- `/api/class-dates`: wrapped in `{data: [...]}`. Class name in `name` field, time in `startTime` ("HH:MM:SS"). Availability = compare `bookings.length` to `limit`
- Booking error "too far in advance" → HTTP 400 `{message: "You cannot book this far in advance"}`

## Conventions

- No external dependencies — use Bun built-ins (`Bun.file`, `Bun.write`, `Bun.sleep`, `fetch`)
- Config defaults in `src/config.ts` (`DEFAULT_CONFIG`)
- Days are lowercase English strings (`monday`..`sunday`)
- Times are `HH:MM` 24h format
- French locale for date display and UI text
- Logging format: `[ISO timestamp] message`
- UI colors via ANSI escape codes in `src/ui.ts`

## Config structure

```json
{
  "auth": { "email", "jwt", "refreshToken", "expiresAt", "userId", "tenantId", "locationId" },
  "advanceBookingDays": 4,
  "slots": [{ "day": "monday", "time": "07:00", "className": "WOD" }],
  "retryIntervalMs": 100,
  "maxRetries": 600
}
```

## Token refresh

- JWT expires in ~1 year (`expiresIn: 31536000`)
- `refreshToken` + `expiresAt` stored in config
- Scheduler auto-refreshes 7 days before expiry via `/oauth/token` (Laravel Passport)
- "Too early" errors: no delay, spam immediately
- Full class: keeps retrying (slower) in case of cancellation
- Interactive mode tries refresh before prompting re-login

## Booking logic

1. For each slot, calculate next class date occurrence
2. Opening = classDate - advanceBookingDays + classDuration (default 60min)
3. Pre-fetch classDateId 2 minutes before (with token refresh check)
4. Start attempts 30 seconds before opening
5. Retry every 500ms up to 20 times
6. On completion (success or fail), schedule next week automatically

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bnjdpn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
