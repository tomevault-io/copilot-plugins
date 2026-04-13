## taptaza

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Tap-Taza is a cleaning services marketplace for Kazakhstan (Almaty/Astana). Monorepo with a React Native/Expo frontend and a Spring Boot backend. Currently in MVP stage — frontend UI is mostly built, backend APIs are defined, integration between them is in progress.

## Commands

### Frontend (run from `frontend/`)

```bash
npm start              # Expo dev server (press i/a/w for platform)
npm run ios            # iOS simulator
npm run android        # Android emulator
npm run web            # Web browser
npm run lint           # ESLint + Prettier check
npm run format         # Auto-fix formatting
npm run prebuild       # Generate native projects
```

### Backend (run from `backend/`)

```bash
mvn spring-boot:run                    # Run locally (needs PostgreSQL on :5432)
mvn clean package                      # Build JAR
mvn clean package -DskipTests          # Build without tests
docker-compose up                      # Start backend + PostgreSQL together
docker build -t taptaza-backend .      # Build Docker image
```

### Backend requires environment variables

See `backend/.env.example` for required vars: `POSTGRES_*`, `JWT_SECRET`, `TWILIO_*`.

## Architecture

### Frontend (`frontend/`)

- **React Native 0.81 + Expo 54** with **TypeScript 5.9**
- **Expo Router** — file-based routing in `app/`. Route groups: `(tabs)` for bottom nav, `(auth)` for login flow, `companies/` for browsing, `booking/` for order flow
- **NativeWind** (Tailwind CSS for RN) — requires `nativewind` Babel JSX import source and `metro.config.js` setup
- **Zustand** for state management (`src/store/bookingStore.ts`)
- **Axios** with JWT interceptors (`src/services/api/client.ts`) — token stored in AsyncStorage under `@taptaza:access_token`
- Path alias: `@/*` maps to `src/*` (configured in tsconfig.json)
- Prettier enforces 100-char width with Tailwind class sorting

### Backend (`backend/`)

- **Spring Boot 3.2 / Java 17 / Maven**
- **PostgreSQL 15** with JPA/Hibernate (DDL: update strategy)
- **Lombok** + **MapStruct** for boilerplate reduction and DTO mapping (annotation processor order matters in pom.xml)
- **Spring Security** with JWT auth — `JwtAuthFilter` extracts Bearer token, `JwtTokenProvider` handles generation/validation
- **Twilio WhatsApp Sandbox** for OTP delivery
- Layered: Controller → Service (interface + impl) → Repository → Model
- DTOs split into `dto/request/` and `dto/response/`
- Custom `@CurrentUser` annotation resolves authenticated user in controllers
- Seed data in `data.sql` (runs on startup, contains sample companies in Russian)

### API Endpoints

| Method | Path | Auth | Purpose |
|--------|------|------|---------|
| POST | /api/auth/send-otp | No | Send 4-digit OTP via WhatsApp |
| POST | /api/auth/verify-otp | No | Verify OTP, returns JWT |
| GET | /api/companies | No | List companies (paginated) |
| GET | /api/companies/{id} | No | Company detail with services |
| POST | /api/bookings | Yes | Create booking |
| GET | /api/bookings | Yes | User's bookings |
| GET | /api/users/me | Yes | Current user profile |
| PUT | /api/users/me | Yes | Update profile |

### Key Patterns

- OTP-only authentication (no passwords) — deliberate UX choice for Kazakhstan market
- Frontend API config in `src/config/index.ts` — uses ngrok tunnel URL for dev
- Backend entities were refactored from `entity/` to `model/` package (uncommitted)
- All UI text is in Russian — no i18n framework yet

## Project Documentation

Detailed docs in `project-doc/`: Features.md (feature specs), Roadmap.md (4-phase plan), Issues.md (known issues with priorities), ProjectArc.md (architecture), Documentation.md (setup guide), UIDesign.md (screen specs).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abo-code-1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/abo-code-1)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
