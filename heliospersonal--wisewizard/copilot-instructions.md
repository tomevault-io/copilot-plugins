## wisewizard

> Single-Owner .NET app: reads an Interactive Brokers portfolio **read-only** and produces a nightly AI research digest delivered via Telegram. Full design lives in `docs/00-overview/` (CONTEXT, idea-brief, sad, adr/) and `docs/features/*`.

# WiseWizard — project conventions

Single-Owner .NET app: reads an Interactive Brokers portfolio **read-only** and produces a nightly AI research digest delivered via Telegram. Full design lives in `docs/00-overview/` (CONTEXT, idea-brief, sad, adr/) and `docs/features/*`.

## Hard rules

- **Test coverage MUST be > 95%** (line + branch), enforced in CI via Coverlet threshold. Excluded: `WiseWizard.Host` (composition root / wiring) and generated code. Every feature is built test-first (TDD).
- **Read-only broker access.** The System never places, modifies, or cancels an order. No order API is wired up. This is a domain invariant (see `docs/00-overview/CONTEXT.md`).
- **Every Verdict cites ≥1 Source.** A Verdict with no evidence is invalid.
- **Single Owner.** No multi-tenant/auth beyond a Telegram chat-id allowlist.

## Stack (fixed by ADRs)

- .NET 10, C#. Single Generic Host process with `BackgroundService` hosted services (ADR-0001).
- Hangfire for nightly job scheduling / persistence / retries (ADR-0004); Hangfire storage is the same PostgreSQL database under a dedicated `hangfire` schema (ADR-0007).
- PostgreSQL via Dapper for domain data (ADR-0007, supersedes the original SQLite decision ADR-0003); Npgsql driver. Money columns are `numeric`, identity keys `BIGINT GENERATED ALWAYS AS IDENTITY`.
- Anthropic Message Batches API for the two-tier model cascade (ADR-0005).
- IBKR Client Portal API (local REST) for read-only positions; keep-alive + manual daily 2FA (ADR-0002, ADR-0006).
- `Telegram.Bot` for the bot.
- .NET Aspire for local-dev orchestration (ADR-0008): `src/Aspire/WiseWizard.AppHost` (provisions Postgres + injects config from user-secrets) and `WiseWizard.ServiceDefaults` (OTel/health/resilience). Local-dev only — excluded from the Docker image and CI coverage. Run: `dotnet run --project src/Aspire/WiseWizard.AppHost`.

## Architecture

Dependency direction: `Host → Bot/Pipeline → Infrastructure → Core`. `Core` has zero external deps and defines all abstractions + domain models. Every external Source/Broker/LLM sits behind a Core interface (add a Source = new impl; Open/Closed). `Pipeline` and `Bot` depend only on Core abstractions, never on concrete Infrastructure types.

## Testing conventions

- xUnit + NSubstitute for mocking Core abstractions.
- Pipeline/LLM logic is unit-tested against **saved fixtures** with zero network.
- Repositories are integration-tested against a real throwaway PostgreSQL database via **Testcontainers** (`TestDatabase` starts one `postgres:17-alpine` container per run and hands out an isolated database per test). **Docker is required** to run the Infrastructure suite locally; CI's ubuntu runner provides it. Core/Bot suites stay network- and Docker-free.
- Real-API (IBKR gateway / Anthropic / Telegram) tests are opt-in and live in `*.Infrastructure.Tests`, excluded from the hermetic PR suite.
- Run coverage: `dotnet test --collect:"XPlat Code Coverage"` (Coverlet); gate at >95%.

## Git

Plain, human-authored commit messages. No AI attribution trailers.

---
> Source: [HeliosPersonal/wisewizard](https://github.com/HeliosPersonal/wisewizard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
