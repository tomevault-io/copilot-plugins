## neo-tidaro

> This document provides essential context for AI models and contributors working in this project. Adhering to these guidelines ensures consistency, modularity, and long-term maintainability.


# GEMINI.md: AI Collaboration Guide

This document provides essential context for AI models and contributors working in this project. Adhering to these guidelines ensures consistency, modularity, and long-term maintainability.

## 1. Project Overview & Purpose

- **Primary Goal:**
  This repository is a **multi-package Flutter/Dart monorepo** managed with **Melos**, whose configuration is integrated directly into **pubspec.yaml**. It enables modular development where apps, features, and core libraries coexist in a unified workspace.

The primary target is **Tidaro** — a housekeeping and service mediation platform designed for Morocco and scalable to global markets. The app is **mobile-first**, with emphasis on **multi-language accessibility** (Arabic, Tifinagh, Berber Latin, English, French).

### Real-World Goal

Tidaro connects households and businesses with vetted housekeeping professionals in Morocco.

- Customers can discover services, get transparent pricing, book and manage appointments, pay securely, and rate experiences.
- Providers (professionals/agencies) can manage availability, assignments, routes, and performance, and receive payouts.
- Operators use the dashboard (TiDash) to oversee supply/demand, staff, bookings, service quality, and support.

Scope across apps:

- `apps/tidaro/`: customer-facing booking experience.
- `apps/tidash/`: operational dashboard for staff, bookings, metrics.
- `apps/tidaro_mini/`: lightweight companion app variant for rapid testing or limited deployments.

Success indicators:

- Reduced booking friction (time-to-book, cancellation rate).
- High fulfillment rate and on-time completion.
- Provider retention and utilization.
- CSAT/NPS and low support load via reliable flows.
- Secure, compliant data handling using Supabase (Auth, Storage, Realtime, Functions).

See [[#Business KPIs]] for formal tracking metrics.

## 2. Core Technologies & Stack

- **Languages:** Dart (latest stable supported by Flutter).
- **Frameworks & Platforms:** Flutter (Android, iOS, Web, Linux, macOS, Windows).
- **Backend & Database:** Supabase/PostgreSQL (Auth, Storage, Functions, Realtime).
- **State Management:** Redux (standard across workspace). Existing Redux integrations are the reference; new features should use Redux.
- **Internationalization:** `packages/languist` with `languist.yaml` and Flutter gen-l10n.
- **Containerization:** Docker Compose for local services where applicable.
- **Monorepo Tooling:**

  - [`melos`](https://melos.invertase.dev) → workspace manager (configured in `pubspec.yaml`)
  - `dart pub` → package dependencies
  - `pnpm` → JavaScript package manager for root-level scripts (e.g., seeding)

## 3. Workspace Structure

This project follows a **Melos workspace** layout, with configuration embedded in `pubspec.yaml`.

```shell
.
├── apps/                     # App entrypoints
│   ├── tidaro/               # Main app
│   ├── tidaro_mini/          # Lightweight app variant
│   └── tidash/               # Admin/dashboard app
│
├── packages/                 # Reusable and shared packages
│   ├── core/                 # Core utilities and base functionality
│   ├── shared/               # State mgmt, utilities, shared functions
│   ├── languist/             # Localization (ARB, gen:l10n, config)
│   └── device_sensors/       # Sensors and device integrations
│
├── examples/                 # Minimal runnable examples
│   ├── auth_flow/
│   ├── basic_app/
│   └── ui_kit_showcase/
│
├── supabase/                 # DB schema, seeds, config
│   ├── migrations/
│   └── seeds/
│
├── documentation/            # Architecture, workflow, guides
│   ├── 01-workspace-structure.md
│   ├── 02-coding-conventions.md
│   ├── 03-development-workflow.md
│   ├── 04-packages-overview.md
│   └── 05-platform-support.md
│
├── pubspec.yaml              # Workspace-level dependencies, scripts, and Melos configuration
├── pubspec.yaml              # Workspace-level dependencies and scripts
└── .github/workflows/        # CI pipelines
```

**Philosophy:**

- `/apps/` → Thin entrypoints that compose features and present UI.
- `/packages/` → Independent, reusable modules (domain, data, shared UI).
- Business logic lives in packages; apps wire features and screens.

## 4. Coding Conventions & Style Guide

- **Formatting:**

  - Use `dart format` (2-space indentation, trailing commas).
  - Must pass `flutter analyze`.

- **Naming Conventions:**

  - Variables, functions: `camelCase`
  - Classes, Widgets: `PascalCase`
  - Files: `snake_case.dart`

- **Architecture & State Management:**

  - Clean Architecture and separation of concerns.
  - State management: Use Redux for new and existing features. Keep reducers, actions, selectors, and middleware in `packages/shared/` (or domain-specific packages) and compose them at app level.
  - Networking/services/models live in `packages/core/` or feature packages; UI consumes via abstractions.
  - Localization via `packages/languist` and generated l10n delegates.

- **Error Handling:**

  - Always handle async errors (`try/catch`, `FutureBuilder`, `AsyncSnapshot`).
  - Use `Either/Result` patterns in shared packages where applicable.

## 5. Key Files & Entrypoints

- `apps/tidaro/lib/main.dart` → Main app entrypoint.
- `melos.yaml` → Defines packages and workspace graph.
- `pubspec.yaml` → Declares dependencies at the root level.
- `scripts/` → Contains automation, bootstrap, and CI/CD scripts.

## 6. Development & Testing Workflow

**Setup:**

1. Install Flutter SDK (latest stable).
2. Install Melos: `dart pub global activate melos`.
3. Bootstrap workspace: `melos bootstrap`.
4. Generate localization: `melos run gen:l10n` (uses `packages/languist/languist.yaml`).
5. Run apps via Melos platform scripts, for example:
   - `melos run run:tidaro:linux`
   - `melos run run:tidaro_mini:web`
   - `melos run run:tidash:android`

**Builds:**

- Use Melos build scripts, e.g. `melos run build:linux`, `melos run build:web`, `melos run build:android`.

**Testing:**

- `flutter test` at workspace or package level.
- Each package maintains its own `/test` directory.

**Localization Flow:**

- Edit ARB files in `packages/languist/lib/l10n/`.
- Update `languist.yaml` as needed.
- Run `melos run gen:l10n` to regenerate.
- Apps import generated delegates and use `l10n.*` accessors.

**Seeding Data:**

- **Purpose:** Snaplet is used to generate realistic, synthetic data for development and testing environments. It populates the database based on the defined schema in `seed.ts`.
- **Schema Synchronization:** After any database schema changes (e.g., new migrations), run `npx snaplet generate` to update Snaplet's type-safe client.
- **Execution:** To seed the database, ensure your `DATABASE_URL` environment variable is correctly set (e.g., in a `.env` file), then run: `pnpm run seed`

**CI/CD:**

- Pipelines should include at minimum:

  - `melos bootstrap`
  - `flutter analyze`
  - `flutter test`
  - Optional: platform build matrix for release workflows

**Environment & Secrets:**

- Do not commit secrets or Supabase keys. Use `.env`/CI secrets and `supabase/config.toml`.
- Docker Compose can be used for local services.

## 7. AI Collaboration Guidelines

- **Modularity First:**

  - New features → new package under `/packages`.
  - Keep `/apps` lean, import only what’s needed.

- **State Management:**

  - Use Redux. Add actions, reducers, selectors, and middleware consistently. Prefer selectors over direct state access in UI.

- **Security:**

  - Do not commit secrets or Supabase keys.
  - Use `.env` and secure injection at runtime.

  - Add via `dart pub add <package>` inside the correct package.
  - Sync with `melos.yaml` if package graph changes.

- **Commits:**

  - Follow Conventional Commits:

	- `feat:` → new feature
	- `fix:` → bug fix
	- `chore:` → infra/tooling updates
	- `docs:` → documentation changes

## 8. Business KPIs

This section formalizes success indicators described in [[#Real-World Goal]]. Metrics should be computed regularly (e.g., daily) and visualized in `apps/tidash/`. Data is sourced from Supabase as defined in [[#2. Core Technologies & Stack]].

- Time to Book (TTB)

  - Definition: Median minutes from app open to booking confirmation.
  - Formula: median(`confirmed_at` - `session_start_at`).
  - Source: `events.sessions`, `bookings`.

- Cancellation Rate

  - Definition: Share of bookings canceled by customer or provider.
  - Formula: canceled_bookings / total_bookings over period.
  - Source: `bookings(status in {canceled_by_customer, canceled_by_provider})`.

- Fulfillment Rate

  - Definition: Share of confirmed bookings that complete successfully.
  - Formula: completed_bookings / confirmed_bookings.
  - Source: `bookings(status=completed | confirmed)`.

- On-time Completion Rate

  - Definition: Completions where `completed_at <= scheduled_end_at`.
  - Formula: on_time_completions / completed_bookings.
  - Source: `bookings(completed_at, scheduled_end_at)`.

- Provider Utilization

  - Definition: Share of provider available hours that are booked.
  - Formula: sum(booking_hours) / sum(available_hours).
  - Source: `providers_availability`, `bookings(duration)`.

- Provider Retention (30/90-day)

  - Definition: Providers active in a cohort who return in 30/90 days.
  - Formula: returning_providers / cohort_providers.
  - Source: `providers`, `bookings`.

- Customer Satisfaction (CSAT) and NPS

  - Definition: Average CSAT (1–5) and NPS (promoters − detractors).
  - Formula: `avg(csat_score)`, `(promoters% - detractors%)`.
  - Source: `reviews`, `surveys`.

- Support Ticket Rate

  - Definition: Tickets per 100 bookings.
  - Formula: tickets / bookings \* 100.
  - Source: `support_tickets`, `bookings`.

- Data Security Incidents
  - Definition: Verified incidents impacting confidentiality, integrity, availability.
  - Source: incident registry; see policies in `documentation/`.

Implementation notes:

- Visualize KPIs in [[#3. Workspace Structure]] → `apps/tidash/`.
- Compute with scheduled jobs or Supabase Functions (see [[#2. Core Technologies & Stack]]).
- Store aggregates in `analytics_daily` tables for fast dashboards.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/moughamir) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
