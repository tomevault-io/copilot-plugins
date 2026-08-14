## programpulse

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

ProgramPulse is a `.NET 10` solution with **two projects**:

- **`src/ProgramPulse.Api`** — an ASP.NET Core minimal-API backend. Vertical-slice architecture with a CQRS-flavored command/query + handler pattern, a `Result`/`Error` functional-error model, EF Core + SQL Server, ASP.NET Core Identity with JWT-in-cookie auth, multi-tenancy, and a transactional outbox for async work (e.g. emails).
- **`src/ProgramPulse.Web`** — a Blazor **WebAssembly** SPA styled with Tailwind CSS v4 plus a hand-written design system. Talks to the API over HTTP through typed API clients; auth rides on HttpOnly cookies.

There is no test project yet.

## Commands

```bash
# Start dependencies (SQL Server on 1433, Mailpit SMTP on 1025 / web UI on 8025)
docker-compose up -d

# Run the API (see env note below — a plain `dotnet run` needs DATABASE_CONNECTION_STRING)
dotnet run --project src/ProgramPulse.Api                        # http  -> http://localhost:5017
dotnet run --project src/ProgramPulse.Api --launch-profile https # https -> https://localhost:7093

# Run the Blazor SPA (launchSettings sets ASPNETCORE_ENVIRONMENT=Local)
dotnet run --project src/ProgramPulse.Web                        # http  -> http://localhost:5031
dotnet run --project src/ProgramPulse.Web --launch-profile https # https -> https://localhost:7208

# Build / restore
dotnet build
dotnet restore

# Tailwind CSS (run from src/ProgramPulse.Web)
npm run css:build   # Styles/app.css -> wwwroot/css/app.css (minified)
npm run css:watch   # rebuild on change while developing

# EF Core migrations
dotnet ef migrations add <Name> --project src/ProgramPulse.Api
dotnet ef database update --project src/ProgramPulse.Api
```

Migrations are applied automatically on startup by `DbInitializer.UseInitializeDatabaseAsync` (only when pending), which also seeds Identity roles.

API docs (Scalar UI) and OpenAPI are mapped via `MapApiDocumentation` and available in non-Production environments.

### Environment & configuration gotchas

**API connection string** — `PersistenceConfiguration.AddPersistence` reads it differently per environment:
- **Local** (and any env that isn't Development/Staging/Production): from `ConnectionStrings:DatabaseConnection` in `appsettings.Local.json`.
- **Development / Staging / Production**: from the `DATABASE_CONNECTION_STRING` environment variable (throws if missing).

The API's `launchSettings.json` sets `ASPNETCORE_ENVIRONMENT=Development`, so a plain `dotnet run` expects `DATABASE_CONNECTION_STRING` to be set. To use the Docker SQL Server with the appsettings connection string, run with `ASPNETCORE_ENVIRONMENT=Local`. Local email points at Mailpit (`localhost:1025`).

**CORS + cookies** — the API's CORS policy (`WebCorsPolicy` in `Program.cs`) reads `Cors:AllowedOrigins` and falls back to `["https://localhost:7208", "http://localhost:5031"]` (the SPA's two profiles), with `AllowCredentials()`. Auth cookies will not flow if the SPA runs on a different origin than the one allowed here — update `Cors:AllowedOrigins` when changing ports.

**SPA → API base URL** — `src/ProgramPulse.Web/wwwroot/appsettings.json` sets `ApiBaseUrl` (default `https://localhost:7093`, the API's https profile). Read in `Program.cs` into the single `HttpClient`'s `BaseAddress`.

**`Frontend:BaseUrl`** (API side, bound to `FrontendOption`) is the SPA's public origin, used to build user-facing links in emails (e.g. the password-reset link) so they point at the SPA rather than the API. Set per environment.

**Generated CSS** — `src/ProgramPulse.Web/wwwroot/css/app.css` is Tailwind output and is **gitignored**. Never edit it; edit `Styles/app.css`. The Web `.csproj` has a `TailwindBuild` target that runs `npm install` (when `node_modules` is missing) and `npm run css:build` before every build, so `dotnet build` regenerates it.

---

## API architecture (`src/ProgramPulse.Api`)

### Vertical slices (`Features/`)
Each feature folder (e.g. `Features/Kpis/Create`) is self-contained and typically holds:
- **`*Command.cs` / `*Query.cs`** — a `record` request, its `*Response` record(s), plus the `*CommandHandler`/`*QueryHandler` class. Handlers take dependencies via primary constructors and expose `HandleAsync(request, ct)` returning `Result` / `Result<T>`.
- **`*Endpoint.cs`** — a class implementing `IEndpoint`. `MapEndpoint` maps the route, then chains `.HasApiVersion(ApiVersions.V1)`, `.WithValidation<TCommand>()`, `.RequireAuthorization(...)`, `.WithName(...)`, `.WithTags(...)`. The handler is injected as a route parameter and the result is returned via `result.ToHttpResult()`.
- **`*CommandValidator.cs`** — a FluentValidation `AbstractValidator<TCommand>`.

Route ids that identify a **parent** travel in the route and are merged into the command in the endpoint, e.g. `command with { ObjectiveId = objectiveId }` in `CreateKpiEndpoint`. Nested collections are routed under the parent (`objectives/{objectiveId}/kpis`) while single-resource update/delete use the flat route (`kpis/{id}`).

### Wiring (`Program.cs`)
- Endpoints are discovered by reflection (`AddEndpoints` scans for `IEndpoint`) and mapped under `api/v{version:apiVersion}` with a default IP fixed-window rate-limit policy (`MapApiEndpoints` in `SharedKernel/EndpointExtensions.cs`).
- **Handlers are NOT auto-registered** — each new handler must be added with `builder.Services.AddScoped<THandler>()` in `Program.cs`. Validators ARE auto-registered (`AddValidatorsFromAssemblyContaining<Program>`).

### Multi-tenancy (`Infrastructure/Authentication`)
Every tenant-owned resource is scoped **manually** — there is no global tenant query filter. The pattern in essentially every feature handler:

```csharp
var tenant = await _currentTenant.GetTenantIdAsync(cancellationToken);
if (tenant.IsFailure)
    return Result<T>.Failure(tenant.Error);
// ...then constrain the query, walking up to the tenant via the aggregate chain:
.Where(k => k.Id == id && k.Objective.Programme.TenantId == tenant.Value)
```

`ICurrentTenant`/`CurrentTenantService` resolves the tenant from `ICurrentUser.UserId` → `ApplicationUser.TenantId`, returning a `Result<Guid>` (failure when unauthenticated or the user has no tenant). A cross-tenant id must surface as the resource's **not-found** error, never a 403 — see `GetObjectiveKpisQueryHandler`. Forgetting this filter is a data-leak bug, so add it to every new tenant-scoped handler.

`ICurrentUser`/`CurrentUserService` reads the caller from the current request's JWT claims; both are registered by `AddCurrentUserService()`.

### Result / Error model (`SharedKernel/Primitives`)
Handlers never throw for expected failures — they return `Result.Failure(error)` / `Result<T>` with an `Error` (code, message, `ErrorType`). `ToHttpResult()` maps success to 200/201 (`Result<T>.Created` carries a `Location`)/202 (`Accepted`)/204, and failure to RFC-7807 `ProblemDetails` via `Error.ToProblemDetails()` (`ErrorType` → status code). Define feature-specific errors as static factory methods in a `*Errors` class next to the entity (`FaqErrors`, `KpiErrors`, `ObjectiveErrors`, `ProgrammeErrors`, `MeasurementErrors`, `MeasurementCommentErrors`, `TenantErrors`, `AuthenticationErrors`).

Paged reads return `Result<PagedList<T>>` (`SharedKernel/Primitives/PagedList.cs`); see `GetProgrammesQueryHandler`, which clamps `pageSize` to `MaxPageSize`.

### Domain model (`Domain/Entities`)
The aggregate chain is **Tenant → Programme → Objective → Kpi → Measurement → MeasurementComment**, all under `Domain/Entities/Tenants/`:

- `Programme` belongs to a `Tenant` and can nest one level via the optional `ParentProgrammeId` self-reference (top-level programmes have `null`). `Status` (`Active`/`Archived`) is **derived from `EndDate`, not persisted**.
- `Objective` belongs to a `Programme`; an objective owns **many** `Kpi`s.
- `Kpi` carries `Category` (Operational/Strategic/Priority), `Direction` (Increase/Decrease), baseline/target/current values, `DueDate`, `Status` (`KpiStatus`), an optional `MeasurementFrequency` cadence, and free-text results-framework notes (`Strategies`, `Activities`, `KeyOutputs`, `PerformanceMeasure`).
- `Measurement` owns `MeasurementComment`s.

**Child entities are created only through their parent.** Their `Create` factories are `internal`; callers use the parent's mutator — `Programme.AddObjective`, `Objective.AddKpi`, `Kpi.AddMeasurement` (which also syncs `Kpi.CurrentValue` to the new reading), `Measurement.AddComment`. Handlers load the parent (tenant-filtered), call the mutator, then `SaveChangesAsync`.

Aggregates derive from `AuditableEntity<T>` (→ `AggregateRoot<T>` → `BaseEntity<T>`), use private setters and static factory / mutator methods with guard clauses (`ArgumentException.ThrowIfNullOrWhiteSpace`); EF materializes via a private parameterless ctor. IDs are `Guid.CreateVersion7()`.

- **Auditing & soft-delete are automatic** in `ApplicationDbContext.SaveChangesAsync`: `IAuditableEntity` gets created/modified stamps (current user or `"system"`), and deletes of `ISoftDeletable` entities are converted to `IsDeleted = true` updates. A global query filter excludes soft-deleted rows. Do not set audit fields or hard-delete soft-deletable entities manually.

**Measurement cadence** — when a KPI has a `MeasurementFrequency`, `CreateMeasurementCommandHandler` rejects a reading that falls within ±one interval of an existing one (`MeasurementFrequencyExtensions.AddInterval`) with `MeasurementErrors.MeasurementTooSoon`. A `null` frequency means no limit.

### Persistence (`Infrastructure/Persistence`)
`ApplicationDbContext` extends `IdentityDbContext<ApplicationUser>` and is the single read/write surface. Entity mappings live in `Configurations/` and are auto-applied via `ApplyConfigurationsFromAssembly`. Handlers depend on the `IApplicationDbContext` interface, not the concrete context — so a new `DbSet` must be added to **both**. `IApplicationDbContext` also exposes explicit transaction control (`BeginTransactionAsync`/`Commit`/`Rollback`) and `CreateExecutionStrategy()`; use the strategy + transaction combo for multi-step writes (see `RegisterCommandHandler`).

### Transactional outbox (`Infrastructure/Messaging/Outbox`)
For async side-effects (e.g. emails), handlers publish a domain event via `IOutboxPublisher.Add(type, payload)`, which adds an `OutboxMessage` to the context but does NOT save — it rides along with the handler's own `SaveChangesAsync` (same transaction). The `OutboxProcessor` background service polls every 15s and dispatches unprocessed messages through `OutboxDispatcher` to the matching `IDomainEventHandler<T>` (events in `Features/Notifications/Events`, handlers in `Features/Notifications/EventHandlers`). Failures are recorded on the message and retried.

### Auth
Login issues a JWT that is stored in an **HttpOnly cookie** (`AuthCookieService`, names/flags under the `AuthCookies` config section) alongside a refresh-token cookie; `TokenService` mints and validates tokens. Because the token is HttpOnly, the SPA cannot read it — it learns who is signed in only via `GET api/v1/auth/me`.

Authorization uses named policies in `Domain/Authorization/AuthorizationPolicies.cs` (`AdminOnly`, `Authenticated`) — reference these constants in `.RequireAuthorization(...)` rather than inlining role checks. Roles (`Administrator`, `User`) are defined in `Roles.cs` and seeded at startup.

**Registration onboards a whole organization**: `RegisterCommandHandler` creates a `Tenant` (with a generated slug) and its first `Administrator` user in one transaction, auto-confirms the email, and enqueues a welcome email via the outbox.

### Cross-cutting infrastructure (`Infrastructure/`, registered in `Program.cs`)
Serilog logging (with sensitive-data masking) + request-performance logging, global exception handling → ProblemDetails, FluentValidation filter, security headers, IP rate limiting, health checks (`/health`), API versioning + OpenAPI, JWT + cookie auth, CORS for the SPA, and FluentEmail/Razor email templating.

---

## Web architecture (`src/ProgramPulse.Web`)

Blazor WebAssembly, no server-side rendering. `Program.cs` registers one `HttpClient` (base address = `ApiBaseUrl`) plus a scoped typed client per API area.

### Typed API clients (`Services/`)
One client per API area (`AuthApiClient`, `UsersApiClient`, `FaqsApiClient`, `ProgrammesApiClient`, `ObjectivesApiClient`, `KpisApiClient`, `MeasurementsApiClient`, `MeasurementCommentsApiClient`). The consistent conventions:

- Build an explicit `HttpRequestMessage` and call **`message.SetBrowserRequestCredentials(BrowserRequestCredentials.Include)`** on every authenticated call — without it the browser won't send the HttpOnly auth cookie.
- **Reads** return `null` on any failure (401, 404, unreachable server, bad JSON) rather than throwing.
- **Writes** return `AuthResult` (`Success`, `GeneralError`, `FieldErrors`), produced by parsing the API's RFC-7807 body into the internal `ProblemResponse` record, so dialogs can render general and per-field messages directly. `AuthResult`/`ProblemResponse` live in `AuthApiClient.cs` and are reused by all clients.
- Request/response DTOs are `sealed record`s declared in the client file, mirroring the API's command/response shapes. Enums cross the wire as numbers, so the Web copies in `Models/KpiEnums.cs` **must keep the same ordinals** as the API enums.

### Pages, layouts, components
- `Pages/*.razor` declare `@page` and pick one of two layouts: `MarketingLayout` (landing + all auth pages) or `DashboardLayout` (the signed-in app shell — sidebar, user block, which fetches the current user via `AuthApiClient.GetCurrentUserAsync`). `App.razor` defaults to `MarketingLayout` and routes unmatched paths to `Pages/NotFound`.
- Routes: `/`, `/login`, `/register`, `/forgot-password`, `/reset-password`, `/dashboard`, `/programmes`, `/programmes/{Id:guid}`, `/programmes/{Id:guid}/objectives/{ObjectiveId:guid}`, `/users`, `/notifications`, `/settings`, `/help`.
- `Components/` holds dialogs (`AddKpiDialog`, `EditKpiDialog`, `MeasurementDialog`, `MeasurementCommentsDialog`, …) and list/card components; `Components/Dashboard/` holds the dashboard widgets.
- `Models/ViewModels.cs`, `DashboardVm.cs`, `NotificationVm.cs` hold UI-facing view models. `_Imports.razor` already brings in the `Components`, `Layout`, `Models`, and `Services` namespaces.

### Still mocked
`Services/SampleData.cs` is a singleton mock source. `Pages/Dashboard.razor`, `Pages/Notifications.razor`, and `Components/NotificationsDialog.razor` still read from it. Note the API **does** have `GET api/v1/dashboard/summary` (`Features/Dashboard/GetSummary`) — wiring the dashboard to it is outstanding work, and there is no notifications API yet.

### Design system (`Styles/app.css`)
`Styles/app.css` (~1900 lines) is the design system: Tailwind v4 `@import "tailwindcss"` plus a `:root` token block and hand-written `pp-*` component classes. Rules that the file states explicitly:

- **Never hard-code a hex, radius, or shadow in a component** — use an existing token or add one.
- Tokens are grouped as surface (`--canvas`, `--surface`, `--inset`, `--ink`), text (`--text`, `--text-2`, `--text-3`, `--on-dark`), accent (`--violet*`, used sparingly), status (`--mint`, `--danger`, `--amber` — confined to pills, dots, stripes and bar fills, never a card background or body text), line (`--line` on canvas, `--line-2` on white, `--line-dark` on ink), radius (`--r-sm/md/lg/xl/btn/pill`), elevation (`--sh-1`…`--sh-4`), and layout (`--gut`, `--max`).
- `.tabnum` (tabular numerals) is mandatory on any figure that can change or align in a column.
- Focus is a 2.5px violet outline at 3px offset; never `outline: none` without a replacement.
- The `.rise` scroll reveal (driven by `wwwroot/js/reveal.js`) is for marketing and auth pages only.

The file cites a `DESIGN_SYSTEM.md` as authoritative, **but that document is not checked into this repo**. Treat `Styles/app.css` and its inline comments as the in-repo source of truth; if the document turns up, it wins.

---

## Conventions for adding a feature

### API
1. Create `Features/<Area>/<Action>/` with `<Action>Command.cs` (record + response records + handler), `<Action>Endpoint.cs` (`IEndpoint`), and `<Action>CommandValidator.cs` if it has input.
2. Register the handler in `Program.cs` (`AddScoped`).
3. If the resource is tenant-owned, inject `ICurrentTenant`, resolve the tenant id first, and filter every query by it — returning the resource's not-found error for out-of-tenant ids.
4. Return `Result`/`Result<T>`; add feature errors to a `*Errors` class.
5. For async side-effects, publish via `IOutboxPublisher` and add an `IDomainEventHandler<T>` under `Features/Notifications/EventHandlers`.
6. For new entities: derive from `AuditableEntity<T>`, make the factory `internal` and expose an `Add*` mutator on the parent if it is a child entity, add a `DbSet` to **both** `ApplicationDbContext` and `IApplicationDbContext`, add an `IEntityTypeConfiguration` in `Configurations/`, then add a migration.

### Web
1. Add or extend the typed client in `Services/`, following the credentials / `null`-on-read-failure / `AuthResult`-on-write conventions, and register it in `Program.cs` if new.
2. Mirror any API enum into `Models/KpiEnums.cs` with matching ordinals.
3. Style with existing `pp-*` classes and tokens from `Styles/app.css`; add new tokens there rather than inlining values.

---
> Source: [marlonajgayle/programpulse](https://github.com/marlonajgayle/programpulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
