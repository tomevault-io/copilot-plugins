## group-habit-tracker

> <!-- MAINTENANCE RULE: When you modify code in this project, check whether this

<!-- MAINTENANCE RULE: When you modify code in this project, check whether this
file still accurately reflects the codebase. If you added/removed/renamed files,
changed commands, added dependencies, or altered patterns — update this file in
the same commit. When reviewing code you didn't write, verify this file matches
reality before trusting it. -->

# HabitHive — Agent Reference

## Version Control

Repository: `git@github.com:arushawasthi-code/group-habit-tracker.git`

```bash
git status / git log --oneline   # standard workflow
git checkout -b fix/<topic>      # branch naming convention
git push -u origin <branch>      # push without creating a PR yet
```

**Branch convention:** `fix/<topic>` for bug/polish branches, `feat/<topic>` for new features. `master` is the stable baseline — never commit directly to it for anything non-trivial.

**`.gitignore` covers:** `bin/`, `obj/`, `*.db`, `node_modules/`, `dist/`, `test-results/`, `playwright-report/`.

---

## Commands

```bash
# API (run from src/HabitHive.Api/)
dotnet build
dotnet run --launch-profile http      # http://localhost:5000
dotnet run                            # uses default profile

# Frontend (run from src/habithive-client/)
npm install
npm run dev                           # http://localhost:5173
npm run build
npx tsc --noEmit                      # type-check only

# Migrations (run from src/HabitHive.Api/)
dotnet ef migrations add <MigrationName>   # create a new migration
dotnet ef migrations remove                # undo last migration (before it's applied)
# DB is auto-migrated at startup via db.Database.Migrate() in Program.cs
# To reset DB in dev: delete src/HabitHive.Api/habithive.db and restart
```

## Project Structure

```
/
├── HabitHive.sln
├── src/
│   ├── HabitHive.Api/
│   │   ├── Controllers/        REST endpoints (thin — no logic)
│   │   ├── Hubs/               SignalR ChatHub
│   │   ├── Models/             EF entities + Dtos.cs (all DTOs in one file)
│   │   ├── Data/               HabitHiveDbContext.cs
│   │   ├── Services/           All business logic
│   │   ├── wwwroot/uploads/    Local photo storage (served as static files)
│   │   ├── appsettings.json    JWT config + Tenor API key
│   │   └── Program.cs          DI, middleware, EF, SignalR, CORS
│   └── habithive-client/
│       └── src/
│           ├── components/     UI components
│           ├── hooks/          useAuth, useSignalR, jwtDecode
│           ├── pages/          LoginPage
│           ├── services/api.ts Axios client + all API calls
│           └── types/index.ts  All TypeScript types + enums
```

## Tech Stack

| Layer | Tech | Version |
|-------|------|---------|
| Backend runtime | .NET / ASP.NET Core | 10.0 |
| ORM | EF Core + Sqlite provider | 10.0.5 |
| Auth | Microsoft.AspNetCore.Authentication.JwtBearer | 10.0.5 |
| Password hashing | BCrypt.Net-Next | 4.1.0 |
| Real-time | ASP.NET Core SignalR | (built-in) |
| Frontend | React + TypeScript | 19 / 5.9 |
| Build tool | Vite + @tailwindcss/vite | 8.0 |
| CSS | Tailwind CSS v4 (vite plugin, no tailwind.config) | 4.x |
| HTTP client | Axios | 1.x |
| SignalR client | @microsoft/signalr | latest |
| Animations | canvas-confetti | latest |

## Key Patterns & Conventions

**Business logic location:** Services only (`AuthService`, `HabitService`, `GroupService`, `ChatService`). Controllers extract `UserId` from JWT claims and delegate — no logic in controllers.

**User ID extraction (controllers):**
```csharp
private Guid GetUserId() => Guid.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier)!);
```

**User ID extraction (SignalR hub):**
```csharp
private Guid GetUserId() => Guid.Parse(Context.User!.FindFirstValue(ClaimTypes.NameIdentifier)!);
```

**SignalR auth:** Token passed via `?access_token=` query string (not Authorization header). Configured in `JwtBearerEvents.OnMessageReceived` in `Program.cs`. The hub is `[Authorize]`-decorated.

**Chat message types:** `ChatMessage.Content` is either plain text or a JSON payload depending on `MessageType`. JSON is only parsed client-side for rendering — never queried server-side. All types share one table.

**Habit visibility:** Privacy via presence/absence of `HabitGroupVisibility` rows (composite PK `HabitId + GroupId`). No row = private. Visibility is set via `PUT /api/habits/{id}/visibility` with the full list of target `groupIds`.

**Group soft-delete:** `GroupMember.LeftAt` nullable DateTime. Active members = `LeftAt == null`. Filter pattern: `.Where(gm => gm.LeftAt == null)`.

**DB schema:** EF Core migrations in `src/HabitHive.Api/Migrations/`. `db.Database.Migrate()` in `Program.cs` applies pending migrations on startup. Add new migrations with `dotnet ef migrations add <Name>` from `src/HabitHive.Api/`.

**Streak calculation:** Lives in `HabitService.CalculateStreak()`. Uses UTC dates (`DateOnly`). Daily = consecutive calendar days; Weekly = one completion per Mon–Sun week; Custom = must complete on each specified `DayOfWeek`.

**Tailwind v4 theming:** Custom tokens defined in `src/index.css` via `@theme { --color-* }` — no `tailwind.config.ts`. Use semantic names: `bg-cream`, `text-cocoa`, `border-border-warm`, `bg-bubble-own`, etc.

**Frontend auth:** JWT stored in `localStorage`. Decoded client-side in `hooks/jwtDecode.ts` (manual base64, no library). `AuthProvider` wraps the app in `main.tsx`. Axios interceptor in `services/api.ts` attaches `Authorization: Bearer`.

**DTO location:** All request/response records are in `Models/Dtos.cs` (one file, namespace `HabitHive.Api.Models.Dtos`).

## Common Pitfalls

- **Tailwind v4:** No `tailwind.config.ts` — theme is in `index.css` `@theme {}` block. Adding new colors goes there, not in a config file.
- **EF migrations:** Schema is applied via `db.Database.Migrate()` at startup. Adding a new column requires `dotnet ef migrations add <Name>` from `src/HabitHive.Api/`. In dev, delete `habithive.db` before restarting if you need a clean slate.
- **SignalR group name = `groupId.ToString()`** (Guid string). Match exactly when broadcasting: `Clients.Group(groupId.ToString())`.
- **Double-completion guard:** Unique index on `(HabitId, CompletedDate)` — API returns 409 on duplicate. Frontend should disable the button after success.
- **Habit deletion cascade:** `DeleteHabitAsync` manually removes `HabitCompletions` and `HabitGroupVisibilities` before removing the habit (EF cascade not relied on for these).
- **`HabitSuggestion` FK delete behavior:** `OnDelete(DeleteBehavior.NoAction)` for `TargetUser` and `TargetHabit` — must handle orphan cleanup manually if those entities are deleted.
- **GIF controller returns empty array (not 404)** when Tenor API key is missing. Frontend should check `apiKey` config and hide the GIF button — not handle 404.
- **CORS:** Only `http://localhost:5173` is allowed in dev. Adding a new origin requires updating `Program.cs` `WithOrigins(...)`.
- **Chat pagination:** `GET /groups/{id}/messages?before={ISO8601datetime}` returns newest-first from DB, then reversed before returning. `before` must be a valid `DateTime` string.

## Testing

### E2E — Playwright (active)

```bash
# From src/habithive-client/ — requires API + Vite dev server already running
npm run test:e2e                # headless, chromium + firefox
npm run test:e2e:ui             # interactive Playwright UI
npm run test:e2e:report         # open last HTML report
npx playwright test e2e/auth.spec.ts   # run a single spec file
```

| File | Covers |
|------|--------|
| `e2e/auth.spec.ts` | Register, login, duplicate username, wrong password |
| `e2e/habits.spec.ts` | Create, complete, double-complete guard, delete |
| `e2e/groups.spec.ts` | Create, join via code, invalid code |
| `e2e/visibility.spec.ts` | Private habit not leaked via API, shared habit visible |
| `e2e/helpers.ts` | `uniqueUser()`, `loginAs()` (bypasses UI login via API + localStorage) |

Config: `playwright.config.ts` at `src/habithive-client/` root. `workers: 1`, serial execution.

### Unit / Integration — not yet implemented

Planned: xUnit + Moq (unit), `WebApplicationFactory` (integration).
- API test project: `src/HabitHive.Api.Tests/`
- Naming: `{ServiceName}Tests.cs`, method pattern: `MethodName_Scenario_ExpectedResult`
- Use in-memory SQLite (`Data Source=:memory:`) for integration tests

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arushawasthi-code) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
