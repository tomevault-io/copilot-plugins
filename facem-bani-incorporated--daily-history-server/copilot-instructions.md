## daily-history-server

> validates via `JwtUtils`, loads `UserDetailsImpl`). Public paths: `/api/v1/auth/**`,

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Spring Boot 3.5 / Java 21 REST backend for a "daily history" mobile app. Serves dated
history content (multilingual events + quizzes), handles auth (local, Google, Apple),
gamification/leaderboards, and RevenueCat-driven Pro subscriptions. Deployed as a single
fat jar (see `Procfile`) against PostgreSQL.

Base package: `com.facem_bani_inc.daily_history_server` (note the underscores).

## Commands

```bash
./mvnw clean package                       # build the fat jar into target/
./mvnw test                                # run all tests (H2 in-memory)
./mvnw test -Dtest=ClassName#methodName    # run a single test
./mvnw spring-boot:run                     # run locally (needs a Postgres + env vars, see below)
java -jar target/daily-history-server-1.0-SNAPSHOT.jar   # run the built jar (Procfile form)
```

There is no linter/formatter configured. Tests use H2 in PostgreSQL-compatibility mode
(`src/test/resources/application.yml`) with Flyway disabled and `ddl-auto: create-drop`,
so they need no external services or secrets.

## Configuration & profiles

All runtime config comes from environment variables resolved in `application.yml`
(DB via `PGHOST`/`PGPORT`/`PGDATABASE`/`PGUSER`/`PGPASSWORD`; plus `SERVER_JWTSECRET`,
`PIPELINE_SECRET`, `GOOGLE_OAUTH_CLIENT_IDS`, `APPLE_BUNDLE_ID`, `CLOUDINARY_*`,
`RESEND_*`, `REVENUECAT_*`, `SUPPORT_ADMIN_EMAIL`). Nothing runs without these set.

- Profiles `dev` and `prod` (`application-{dev,prod}.yml`) both target Postgres with
  `ddl-auto: validate`. Select with `SPRING_PROFILES_ACTIVE`.
- For local secrets, create `src/main/resources/application-local.yml` — it is gitignored.
- The app runs under a tight memory budget (Procfile: `-Xmx192m`, SerialGC,
  `TieredStopAtLevel=1`, and `spring.main.lazy-initialization: true`). Prefer solutions
  that keep the heap and startup footprint small.

## Database: Flyway owns the schema

Because `ddl-auto` is `validate` in every real profile, **Hibernate never creates or
alters tables**. Any schema change (new column, table, index) requires a new Flyway
migration file at `src/main/resources/db/migration/V{n}__description.sql`, matched to the
JPA entity change — otherwise startup fails validation. Existing migrations: V1 schema,
V2 seed data, V3 quiz tables.

`src/main/resources/data.sql` is legacy/embedded-only seeding and does **not** run against
Postgres (Spring's `sql.init.mode` default). Treat Flyway `V2__seed_data.sql` as the
source of truth for seeded rows (roles USER/MODERATOR/ADMIN, and a seed admin user).

## Architecture

Standard layered Spring MVC: `controller` (REST, `/api/v1/**`) → `service` (business logic,
`@Transactional`, caching) → `repository` (Spring Data JPA) → `entity`. Request/response
shapes live in `model/dto` (mostly Java records) and `payload/request|response`; enums in
`model/enums`.

### Content domain
`DailyContent` (one per `dateProcessed`) owns a list of `Event`s; each `Event` has
multilingual `Translation`s (fixed columns `en/ro/es/de/fr`, all non-null) for title and
narrative, an optional `Quiz`, and a gallery. Content is served in tiers:
- `/api/v1/daily-content/guest` — public, top 2 free events for today.
- `/api/v1/daily-content/by-date` — authenticated, free events.
- `/api/v1/daily-content/pro/by-date` — authenticated **and** `isPro` (else 403).

### Content ingestion pipeline (HMAC auth)
`POST /api/v1/daily-content` upserts a day's content and is **not** JWT-protected. It is
guarded by `PipelineHmacAuthFilter`, which grants `ROLE_PIPELINE` when the request carries:
- `X-Timestamp` — epoch seconds, must be within 300s of server time.
- `X-Signature` — `base64(HmacSHA256(PIPELINE_SECRET, "<X-Timestamp>.<raw body>"))`,
  compared in constant time.
The upsert clears and rebuilds all events for that date, and evicts the relevant caches.

### Security
Stateless JWT (`SecurityConfig`, `SessionCreationPolicy.STATELESS`, CSRF disabled, CORS
open). Two custom filters run before `UsernamePasswordAuthenticationFilter`:
`PipelineHmacAuthFilter` (above) and `AuthTokenFilter` (parses `Authorization: Bearer`,
validates via `JwtUtils`, loads `UserDetailsImpl`). Public paths: `/api/v1/auth/**`,
`/api/v1/webhooks/**`, `/api/v1/daily-content/guest`, `/api/v1/test/**`.

Auth providers (`EAuthProvider`): `LOCAL` (username+bcrypt password), `GOOGLE` and `APPLE`
(verify an OS-provided id token in `GoogleAuthService`/`AppleAuthService`, auto-provision
the user). All three paths return the same `JwtResponse`.

### Pro subscriptions (RevenueCat)
`POST /api/v1/webhooks/revenuecat` is authenticated by a static `Authorization` header
match (`REVENUECAT_WEBHOOK_AUTH_HEADER`). Event type maps to setting `User.isPro`
(INITIAL_PURCHASE/RENEWAL/… → true; CANCELLATION/EXPIRATION/REFUND/… → false; TRANSFER moves
it between users). **RevenueCat `app_user_id` must be the numeric DB user id** — non-numeric
ids are skipped. The webhook always returns 200 and swallows processing errors (so RevenueCat
does not retry).

### Caching
Caffeine, wired manually in `config/CacheConfig` with cache names centralized in
`utils/Constants`. Adding a cache means adding the name to `Constants` **and** registering it
(with its own size/TTL) in `CacheConfig`. Services annotate `@Cacheable`/`@CacheEvict` with
those names; content writes evict content/quiz caches, gamification writes evict
gamification + leaderboard.

### Gamification
Client-authoritative sync: the app pushes XP/streaks/saved events via `PUT /api/v1/gamification`
and the server stores them (with light guards like monthly-XP reset and
`longestStreak = max(...)`). Quiz submission (`QuizService`) is the exception — scored
server-side, awarding `XP_QUIZ_PER_CORRECT` per correct answer plus `XP_QUIZ_PERFECT_BONUS`.

### External services
Cloudinary (avatar/image hosting, `AvatarService`), Resend (transactional/support email,
`ResendEmailService`).

---
> Source: [Facem-Bani-Incorporated/daily-history-server](https://github.com/Facem-Bani-Incorporated/daily-history-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
