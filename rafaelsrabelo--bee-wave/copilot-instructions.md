## bee-wave

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Bee Wave** is a full-stack surfing school management platform built with Rails 7 + React 19 via Inertia.js. It includes an admin web UI, a REST API for a React Native mobile app, real-time WebSocket updates, push notifications via Expo, and background job processing.

## Development Commands

```bash
# Start both servers (Rails + Vite) via Procfile.dev
./bin/dev

# Or start separately
bin/rails s          # Rails on port 3000
bin/vite dev         # Vite frontend

# Background services (required for jobs/WebSocket)
redis-server
bundle exec sidekiq

# Database
rails db:migrate
rails db:seed
rails db:create

# TypeScript check
npm run check        # tsc on tsconfig.app.json + tsconfig.node.json

# Linting / Security
bundle exec rubocop
bundle exec brakeman

# Tests
rails test
```

## Architecture

### Backend (Rails 7)

- **`app/controllers/api/v1/`** — REST API for the mobile app (JWT authentication via Bearer tokens). Admin-only endpoints under `api/v1/admin/`.
- **`app/controllers/`** — Inertia.js controllers for the web admin UI, rendering React pages server-side.
- **`app/models/`** — Active Record models. Most are scoped by `school_id` for multi-tenancy.
- **`app/services/`** — Business logic: `JwtService`, `ExpoNotificationService`, `NotificationAudienceService`, `GeocodingService`.
- **`app/jobs/`** — Sidekiq background jobs for lesson reminders, payment reminders, and subscription expiry alerts. Scheduled via sidekiq-cron.
- **`app/channels/`** — Action Cable WebSocket channels for real-time lesson updates.

### Frontend (React 19 + Inertia.js)

- **`app/frontend/pages/`** — Inertia page components (one per route). These receive props directly from Rails controllers.
- **`app/frontend/components/`** — Reusable components. `ui/` contains shadcn/ui primitives.
- **`app/frontend/hooks/`** — Custom hooks including `useLessonsChannel.ts` (Action Cable WebSocket).
- **`app/frontend/types/`** — TypeScript type definitions shared across pages.

Inertia.js replaces traditional API calls for the web UI — controllers render React components by calling `render inertia: 'PageName', props: { ... }`.

### Key Data Model

- **School** — top-level multi-tenant unit; most records are scoped by `school_id`.
- **User** — has roles: admin, teacher, student.
- **Lesson** — core scheduling entity (teacher, student, date, status).
- **StudentSubscription** — lesson package with credit tracking.
- **PushNotification** — bulk notification campaigns with audience targeting.
- **AvailabilitySlot** — teacher availability windows.

### Database

SQLite3 in development/test, PostgreSQL in production (via `DATABASE_URL`). Schema is in `db/schema.rb`.

### Push Notifications

Expo Server SDK integration (`ExpoNotificationService`) sends notifications to React Native mobile clients. Push tokens are registered via `/api/v1/push_tokens`. Delivery is async via Sidekiq.

### Real-time

Action Cable over WebSocket. The frontend connects via the `useLessonsChannel` hook to receive live lesson updates.

## Configuration Notes

- Time zone: `America/Fortaleza` (set in `config/application.rb`)
- Active Storage uses Cloudinary in production (configured in `config/storage.yml`)
- Sidekiq queues defined in `config/sidekiq.yml`
- Routes: `config/routes.rb` defines both web UI routes and the `api/v1` namespace

## Docs

`docs/` contains 67 markdown files. Start with `docs/START-HERE.md` for the reading order. API reference for the mobile app is in `docs/API-REACT-NATIVE.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rafaelsrabelo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rafaelsrabelo)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
