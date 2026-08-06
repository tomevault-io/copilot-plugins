## ids-it-helpdesk

> - **Frontend:** Next.js 16.2.6 (App Router), TypeScript 5, Tailwind CSS v4, shadcn/ui (Radix), Zod 4 + react-hook-form, Axios, Recharts, @tanstack/react-query 5, @microsoft/signalr 10, sonner (toast), js-cookie, lucide-react, tw-animate-css

# AGENTS.md — IDS IT Helpdesk

## Stack

- **Frontend:** Next.js 16.2.6 (App Router), TypeScript 5, Tailwind CSS v4, shadcn/ui (Radix), Zod 4 + react-hook-form, Axios, Recharts, @tanstack/react-query 5, @microsoft/signalr 10, sonner (toast), js-cookie, lucide-react, tw-animate-css
- **Backend:** ASP.NET Core net10.0 Web API, EF Core 10.0.9 + Npgsql 10.0.2, JWT Bearer auth (token versioning for revocation), AutoMapper 16.1.1, BCrypt.Net-Next 4.0.3, Swashbuckle 6.6.2, QuestPDF, ClosedXML 0.104.2, SignalR, DotNetEnv 3.2.0
- **DB:** PostgreSQL (EF Core migrations in `backend/Migrations/`)

## Commands

| Task | Command (from `backend/`) |
|------|--------------------------|
| Dev server | `dotnet run` (port 5055) |
| Watch mode | `dotnet watch run` |
| Migrations | `dotnet ef database update` (after `dotnet tool restore`) |
| New migration | `dotnet ef migrations add <Name>` |
| Test | `dotnet test tests\HelpdeskApi.Tests.csproj` |
| Build test | `dotnet build tests\HelpdeskApi.Tests.csproj` |

| Task | Command (from `frontend/`) |
|------|--------------------------|
| Dev server | `npm run dev` (port 3000) |
| Build | `npm run build` |
| Lint | `npm run lint` |

## Backend architecture

- **`Program.cs`** wires everything. Loads `backend/.env` via DotNetEnv (never commit — gitignored). Enforces JWT secret >= 32 chars at startup. Invokes `DbSeeder.SeedAsync` in a startup scope (~line 200).
- **Auth policies:** `AdminOnly`, `AgentOrAbove`, `ManagerOrAbove`, `AllAuthenticated`.
- **Rate limiting:** 10 req/min fixed window on `AuthPolicy` — applied to auth endpoints via `[EnableRateLimiting("AuthPolicy")]`.
- **SignalR hub:** `/hubs/notifications` — tokens accepted via query string (`access_token`) for WebSocket connections.
- **Test quirk:** `backend/tests/` excluded from main compilation via `<Compile Remove="tests\**" />`. Uses xUnit + Moq + EF Core InMemory. If you hit duplicate assembly attribute errors, clean all `obj/` and `bin/` dirs.
- **Settings pattern:** `SystemSettings` table (key-value), `EmailTemplate` table with placeholders (`{Name}`, `{ReferenceNumber}`, `{TicketUrl}`, `{NewStatus}`).

## Frontend architecture

- **Auth token** stored in cookie named `token` (js-cookie, sameSite: strict, 1-day expiry). Auto-refreshed via Axios 401 response interceptor (`src/lib/api.ts`) — POST `/auth/refresh`, retries original request.
- **Route protection:** `src/proxy.ts` provides the guard logic but is **not** auto-registered as Next.js middleware (no `middleware.ts` exists — it must be manually wired if used). Matches all routes except `_next/static|_next/image|favicon.ico|api/`. Public paths: `/login`, `/forgot-password`, `/reset-password`. Admin-only: `/admin`, `/users`, `/settings`, `/activity-logs`, `/monitoring`. Reports: Admin + Manager only.
- **API clients:** per-entity modules in `src/lib/api/` (tickets.ts, users.ts, auth.ts, etc.) — use these instead of raw Axios.
- **Env:** `NEXT_PUBLIC_API_URL=http://localhost:5055` in `frontend/.env.local` (fallback to `http://localhost:5055/api` in `api.ts`).
- **shadcn/ui** components in `src/components/ui/`. Aliases per `components.json`: `@/components`, `@/lib`, `@/hooks`, `@/components/ui`.
- **Theme:** dark mode via `src/lib/theme-provider.tsx`.
- **Roles (4, no "Super Admin"):** Admin, Agent, Manager, Employee — typed as `Role` in `src/types/index.ts`.

## Test data

`appsettings.Development.json` sets `SeedTestData: true`, which triggers `SeedTestDataAsync` in `DbSeeder`. Creates 12 users, 20 tickets, comments, attachments, notifications, activity logs, system settings, escalation rules, and refresh tokens. Key login:

| Email | Password | Role |
|-------|----------|------|
| `admin@test.com` | `Test@1234` | Admin |
| `bob.agent@test.com` | `Test@1234` | Agent |
| `eve.manager@test.com` | `Test@1234` | Manager |
| `diana@test.com` | `Test@1234` | Employee |

## Seeded lookup data

### Statuses (IDs used in DB and API)
| ID | Name |
|----|------|
| 1 | Open |
| 2 | In Progress |
| 3 | Resolved |
| 4 | Closed |
| 5 | Cancelled |
| 6 | Pending |

### SLA targets (by priority)
| Priority | Target hours |
|----------|-------------|
| Critical | 4 |
| High | 8 |
| Medium | 24 |
| Low | 72 |

### Priorities
1=Low, 2=Medium, 3=High, 4=Critical (seeded in `AppDbContext.OnModelCreating`).

## Backend .env setup

Create `backend/.env` (already gitignored) with:
```
JwtSettings__Secret=<at least 32 chars>
ConnectionStrings__DefaultConnection=Host=localhost;Database=helpdesk;Username=postgres;Password=...
```

## Role system

4 roles seeded by `DbSeeder` (idempotent — returns if roles exist). Self-registration disabled — only Admin-role users can create accounts via `UsersController` (`POST /api/users`). Lookup data (categories, priorities, statuses, email templates, SLA targets) seeded in `AppDbContext.OnModelCreating`.

No "Super Admin" role exists — the codebase uses "Admin" as the top role.

## Discrepancies from README

- README mentions "Super Admin" as a role — actual code has 4 roles (Admin, Agent, Manager, Employee). The README is aspirational; `DbSeeder` is the source of truth.
- README claims "User self-registration is disabled" — correct, user creation is via `UsersController` (Admin-only policy), not `AdminController` (which only has activity-logs endpoint).

---
> Source: [Domenico-Labaki/IDS-IT-helpdesk](https://github.com/Domenico-Labaki/IDS-IT-helpdesk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
