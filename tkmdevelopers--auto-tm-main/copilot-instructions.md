## auto-tm-main

> This file provides context and guidance for the Gemini CLI agent when working with this repository.

# Gemini CLI Agent Guide

This file provides context and guidance for the Gemini CLI agent when working with this repository.

## Project Overview

**Alpha Motors (auto.tm)** is a full-stack automotive marketplace.
- **Frontend:** Flutter mobile app (iOS/Android) using GetX.
- **Backend:** NestJS API with PostgreSQL and Sequelize.
- **Key Features:** User authentication via phone OTP (Physical SMS Gateway), car listings with photos/videos, comments, favorites, subscriptions, and push notifications.

## Repository Layout

```
.
├── gemini.md              # This file (AI assistant guide)
├── README.md              # Documentation index
├── backend/               # NestJS API (TypeScript, Sequelize, PostgreSQL)
│   ├── src/               # Source code (feature modules)
│   ├── migrations/        # Sequelize migrations
│   ├── scripts/           # Seed scripts
│   ├── docker/            # Entrypoint scripts
│   ├── uploads/           # Media storage
│   └── docs/              # Backend documentation
└── auto.tm-main/          # Flutter mobile app (Dart, GetX)
    ├── lib/               # Dart source
    ├── assets/            # Images, fonts, JSON
    └── .env               # API_BASE config
```

## Access Model

| Access | What | Examples |
|--------|------|----------|
| **Public** | Browse & discover | List/filter posts, view post details, brands, categories, banners, comments, vlogs, OTP send/verify |
| **Token Required** | Create & interact | Post listing, comment, favorites, brand subscribe, profile, auth/me, refresh, logout |

**Rule:** Read-only catalog endpoints = public. Create/modify user data = token required.

## Commands

### Backend Commands
Run from `backend/`:

```bash
# Development
npm run start:dev          # Dev server with hot-reload (port 3080)
npm run lint               # ESLint with auto-fix
npm run format             # Prettier formatting
npm run test               # Jest unit tests (*.spec.ts files)

# Database
npm run db:migrate         # Run Sequelize migrations
npm run db:migrate:undo    # Undo last migration
npm run db:seed:all        # Seed database (currencies, brands, etc.)

# Docker
docker compose up -d       # Start API + PostgreSQL (dev)
docker compose -f docker-compose.build.yml build api
```

### Flutter Commands
Run from `auto.tm-main/`:

```bash
flutter pub get             # Install dependencies
flutter run                 # Run on connected device/emulator
flutter analyze             # Dart linter
flutter test                # Unit tests
flutter build apk --release # Android APK
flutter build ios --release # iOS build (macOS only)
```

## Architecture

### Backend (NestJS)
**Modular Architecture:** `src/{feature}/` containing controller, service, entity, dto, and module.

**Runtime Topology:**
- **HTTP API:** Port `3080` (`/api/v1/...`). Swagger at `/api-docs`.
- **Client Socket.IO:** Port `3090` (Chat/Notifications).
- **SMS Gateway Socket.IO:** Port `3091` (`/sms` namespace). Connects to a physical Android device to send OTPs.
- **Database:** PostgreSQL on port `5432`.

**Key Modules:**
- **Auth:** JWT access (15m) + Refresh (7d) with rotation.
- **OTP:** `POST /otp/send` -> `POST /otp/verify`.
- **Photo/Video:** Uses Sharp and FFmpeg. stored in `uploads/`.

### Frontend (Flutter + GetX)
**Architecture:** Layered Clean Architecture (UI -> Domain -> Data), following the [official Flutter architecture recommendations](https://docs.flutter.dev/app-architecture).
**State Management:** GetX (`Rx<T>`, `Obx()`, `GetxController`).
**Networking:** Dio with interceptors for JWT injection and refresh.
**Structure:**
- `lib/screens/`: UI features (Widgets + Controllers).
- `lib/domain/`: Business logic.
    - `/models/`: Pure entities (e.g., `Post`).
    - `/repositories/`: Abstract definitions of data operations.
- `lib/data/`: Data implementation.
    - `/dtos/`: Raw API response objects (e.g., `PostDto`).
    - `/mappers/`: Converters from DTO -> Domain Model.
    - `/repositories/`: Concrete implementations (e.g., `PostRepositoryImpl`).
- `lib/services/`: Global services (Auth, Network, Notifications).

## Architectural Transition Roadmap

To align fully with the [official Flutter architecture recommendations](https://docs.flutter.dev/app-architecture/recommendations), which emphasize **Separation of Concerns** (UI vs Data layers) and the **Repository Pattern**, the following roadmap is established:

### Core Principles
1.  **Separation of Concerns:** Distinct separation between the **UI Layer** (displaying data, user events) and the **Data Layer** (business logic, data access).
2.  **Repository Pattern:** Use Repositories to abstract data sources (API, DB) from the business logic.
3.  **UI Logic Separation:** Keep widgets "dumb". Move state management and logic into Controllers (ViewModels).

### Execution Phases

1.  **Phase 1: Data Layer Consolidation (The "Data" Layer)**
    -   **Goal:** Centralize all data access logic.
    -   **Action:** Migrate shared DTOs to `lib/data/dtos/`.
    -   **Action:** Encapsulate all raw API/DB interactions within **Repositories** (`lib/data/repositories/`) and **Data Sources** (`lib/data/datasources/`). Services (`lib/services/`) should generally be consumed by Repositories, not directly by UI.

2.  **Phase 2: Domain Layer Definition (The Contract)**
    -   **Goal:** Define the pure business rules and contracts independent of external frameworks.
    -   **Action:** Standardize pure Business Entities in `lib/domain/models/` (free of JSON serialization logic).
    -   **Action:** Define abstract **Repository Interfaces** in `lib/domain/repositories/`. These are the contracts the UI layer will depend on.

3.  **Phase 3: UI Layer Decoupling (The "UI" Layer)**
    -   **Goal:** Ensure UI components only interact with the Domain layer.
    -   **Action:** Refactor GetX Controllers (`lib/screens/**/controller/`) to depend *only* on **Domain Repository Interfaces**.
    -   **Action:** Remove direct API calls from Controllers.
    -   **Action:** Use **Mappers** (`lib/data/mappers/`) to convert DTOs to Domain Models *before* data leaves the Data layer.
    -   **Action:** Ensure Widgets are purely presentational and reactive to Controller state.

4.  **Phase 4: Cleanup & Validation**
    -   **Goal:** Enforce the architecture physically and programmatically.
    -   **Action:** Remove redundant directories (e.g., `lib/models/` once empty).
    -   **Action:** Update `test/architecture_test.dart` to strictly enforce these import boundaries (e.g., UI cannot import Data).

## Authentication Flow

1.  **Send OTP:** `POST /api/v1/otp/send` -> Backend hashes OTP, sends via SMS Gateway (Port 3091).
2.  **Verify OTP:** `POST /api/v1/otp/verify` -> Returns Access + Refresh tokens.
3.  **Refresh:** `POST /api/v1/auth/refresh` -> Rotates tokens.
4.  **Handling 401:** Client interceptor catches 401. If `USER_DELETED`, forces logout. If expired, attempts refresh.

## Database Strategy

### Native UUIDs
- **Primary Keys:** All main entities use **Native PostgreSQL UUIDs** (`uuid` column).
- **Foreign Keys:** All relationships use UUIDs.
- **Migration:** `20260209000000-convert-string-to-uuid.js` handles the conversion from legacy STRING types to Native UUID.

### Schema Management
- **Sequelize Migrations:** Source of truth.
- **Entities:** Must explicitly define `@Column({ type: DataType.UUID, defaultValue: DataType.UUIDV4 })`.
# Rules
Before beginning any implementation, perform thorough step-by-step reasoning and validation of the approach. Do not assert correctness, completeness, or functionality without explicit logical justification, verifiable evidence, or alignment between the design and the actual implementation. Every claim about behavior, performance, or correctness must be demonstrably supported by analysis, constraints, or testable proof.
## "Android-as-a-Service" SMS Gateway

**Pattern:** The backend acts as a command center for physical Android devices that perform the actual SMS delivery.
- **Why?** Cost efficiency for high-volume OTPs in the target region (Turkmenistan).
- **Component:** `SmsGateway` (Socket.IO port `3091`).
- **Device Role:** Connects as a client, listens for `sms:send` events, dispatches via GSM, and reports `sms:ack`.
- **Reliability:** The system depends on the physical device being online.
- **Security:** Devices authenticate via `SMS_DEVICE_AUTH_TOKEN`.

## Key Conventions

- **API Responses:** `{ message: string, data: any, status: boolean }`
- **IDs:** UUIDs used for primary keys.
- **File Uploads:** `multipart/form-data`.
- **Environment:**
    - Backend: `.env` (needs `DATABASE_HOST`, `JWT_SECRETS`, `SMS_DEVICE_AUTH_TOKEN`, etc.).
    - Flutter: `.env` (needs `API_BASE`).

## Useful Documentation References

- **Backend Architecture & Risks:** [backend/docs/ARCHITECTURE.md](backend/docs/ARCHITECTURE.md)
- **API Reference:** [backend/docs/API_REFERENCE.md](backend/docs/API_REFERENCE.md)
- **Database Schema:** [backend/docs/DATABASE.md](backend/docs/DATABASE.md)
- **Deployment:** [backend/docs/PRODUCTION_DEPLOYMENT.md](backend/docs/PRODUCTION_DEPLOYMENT.md)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tkmdevelopers)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tkmdevelopers)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
