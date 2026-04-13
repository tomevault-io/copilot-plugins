## xstatic

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Extatic is a storage backend for static websites — a hosted service that provides structured data collections, user auth, file uploads, and webhooks via a REST API. Developers register Apps, define Collections with JSON Schema validation, and end-users of those static sites (AppUsers) interact with Items via the Client API.

The repository is in its initial planning stage. The source code has not been written yet. See `PROJECT-REQUIREMENTS.md` for full requirements and `README.md` for architecture and setup details.

## Planned Tech Stack

- **Backend:** .NET 10 (C#), ASP.NET Core Web API
- **ORM:** Entity Framework Core, code-first migrations, Npgsql (PostgreSQL provider)
- **Frontend:** Angular (latest stable), Tailwind CSS, NgRx or Angular Signals
- **Database:** PostgreSQL with JSONB for `Item.data` and `Collection.schema`
- **File Storage:** Azure Blob Storage (local dev: Azurite emulator)
- **Auth (Platform Users):** OAuth2 Proxy sidecar; API trusts `X-Forwarded-User`/`X-Forwarded-Email` headers
- **Auth (AppUsers):** API-managed OAuth 2.0 / OIDC flows issuing per-App JWTs
- **JSON Schema Validation:** JsonSchema.Net (draft 2020-12)
- **Background Jobs:** Hangfire or .NET `BackgroundService` (webhook delivery/retries, orphaned storage cleanup)
- **Hosting:** Azure Container Apps

## Planned Repository Structure

```
extatic/
├── src/
│   ├── Extatic.Api/              # ASP.NET Core Web API
│   │   ├── Controllers/          # Platform API + Client API controllers
│   │   ├── Services/             # Business logic
│   │   ├── Models/               # Entity models and DTOs
│   │   ├── Data/                 # DbContext and EF Core migrations
│   │   ├── Auth/                 # OAuth2 Proxy header parsing, AppUser JWT issuance
│   │   ├── Validation/           # JSON Schema validation logic
│   │   ├── Webhooks/             # Webhook dispatch and retry
│   │   ├── Storage/              # Azure Blob Storage integration
│   │   └── Program.cs
│   └── extatic-dashboard/        # Angular SPA
│       └── src/app/
│           ├── core/             # Auth guards, interceptors, services
│           ├── shared/           # Shared components, pipes, directives
│           └── features/         # apps/, collections/, items/, appusers/, webhooks/, team/
├── infra/
│   ├── oauth2-proxy/
│   └── bicep/ or terraform/
├── tests/
│   ├── Extatic.Api.Tests/
│   └── extatic-dashboard-e2e/
├── docker-compose.yml            # Local dev: API + OAuth2 Proxy + PostgreSQL + Azurite
└── Extatic.sln
```

## Development Commands (once implemented)

### Local Dev (Docker Compose — recommended)
```bash
docker-compose up -d
cd src/extatic-dashboard && yarn install && ng serve
# API: http://localhost:5001 | Dashboard: http://localhost:4200 | OAuth2 Proxy: http://localhost:4180
```

### Backend
```bash
cd src/Extatic.Api
dotnet ef database update     # Apply EF Core migrations
dotnet run                    # Start API
dotnet test tests/Extatic.Api.Tests                    # All tests
dotnet test tests/Extatic.Api.Tests --filter "Name~Foo" # Single test
```

### Frontend
```bash
cd src/extatic-dashboard
yarn install
ng serve                      # Dev server
ng test                       # Unit tests (Jasmine + Karma)
cd tests/extatic-dashboard-e2e && npx playwright test  # E2E tests
```

## Key Architecture Decisions

### Two Distinct API Surfaces
- **Platform API** (`/apps/...`) — for dashboard Users (developers). Authenticated via session headers injected by OAuth2 Proxy.
- **Client API** (`/client/...`) — for static site end-users (AppUsers). Authenticated via `X-Api-Key` (App identifier) + `Authorization: Bearer <appuser-jwt>`.

### Authorization Model
- Platform Users are identified from OAuth2 Proxy headers (`X-Forwarded-User`, `X-Forwarded-Email`).
- Collaborator roles: `owner > admin > editor > viewer`. Owner is implicit (no Collaborator record needed). Collaborators must accept invitations before gaining access.
- AppUsers own their Items and Attachments; platform Users (owner/admin) can override.

### Data Storage
- `Item.data` and `Collection.schema` stored as PostgreSQL JSONB columns.
- Schema validation (JsonSchema.Net) runs on every Item create/update when a Collection has a schema. Returns `422` with field-level errors on failure.
- Schema changes only apply to future Items, never retroactively.

### File Uploads
- Attachments are opt-in per Collection (`attachments_enabled`). Uploading to a disabled Collection returns `403`.
- Validation order: attachments enabled → MIME type against `allowed_attachment_types` → file size → attachment count.
- Default limits (configurable per App): 10 MB max file size, 10 attachments per Item, 1 GB total storage.
- Cascade delete: deleting an Item also deletes its Attachments from Blob Storage (via background job for orphan cleanup).

### Webhooks
- HMAC-SHA256 signed payloads (`X-Extatic-Signature` header).
- Exponential backoff retries: 1 min → 5 min → 30 min → 2 hr → 12 hr (5 attempts). After all fail, webhook auto-deactivated and owner notified.
- Delivery logs retained 7 days; manual re-trigger supported.

### AppUser OAuth Flow
- The API manages AppUser OAuth directly (not OAuth2 Proxy), since each App may use different providers.
- `POST /client/auth/:provider` initiates flow; `GET /client/auth/:provider/callback` exchanges code and issues App-scoped JWT.
- AppUsers are upserted on first login (unique by `app_id + provider + provider_user_id`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/witttness)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/witttness)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
