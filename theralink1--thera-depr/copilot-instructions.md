## thera-depr

> TheraLink is a psychology appointment booking platform migrating from a Node.js/Express + Next.js monolith to a microservices architecture.

# TheraLink — Claude Code Project Instructions

## Project Overview
TheraLink is a psychology appointment booking platform migrating from a Node.js/Express + Next.js monolith to a microservices architecture.

## Current Stack (Monolith — being migrated)
- **Frontend:** Next.js 15 + React 19 + TypeScript (`/frontend`)
- **Backend:** Node.js + Express + TypeScript (`/backend`)
- **ORM:** Prisma + PostgreSQL + PostGIS
- **Auth:** AWS Cognito + AWS Amplify
- **State:** Redux Toolkit + RTK Query

## Target Stack (Microservices)
- **Frontend:** Angular 18+ (`theralink-frontend` — separate repo)
- **Auth:** Keycloak (`theralink-keycloak` — separate repo)
- **API Gateway:** Spring Cloud Gateway (`theralink-api-gateway` — separate repo)
- **Backend services:** Spring Boot + MongoDB (separate repos per service)
- **Message bus:** Apache Kafka
- **Infrastructure:** Docker + Kubernetes + Azure AKS

## Microservice Repositories
| Repo | Purpose |
|---|---|
| `theralink-frontend` | Angular 18+ application |
| `theralink-keycloak` | Custom Keycloak image + themes + realm config |
| `theralink-api-gateway` | Spring Cloud Gateway |
| `theralink-user-service` | Client + Psychologist profile management |
| `theralink-appointment-service` | Appointment lifecycle |
| `theralink-psychologist-service` | Availability slot management |
| `theralink-notification-service` | Event-driven email/SMS (Kafka consumer only) |
| `theralink-payment-service` | Stripe integration (SEPARATE restricted-access repo) |
| `theralink-contracts` | Shared Kafka event schemas + DTOs (Maven library) |
| `theralink-infrastructure` | Kubernetes manifests + Helm charts |

## Migration Documents
- `docs/frontend-migration.md` — Step-by-step Angular frontend migration plan
- `docs/architecture.md` — Full microservices architecture overview

## Current Backend Entities (PostgreSQL / Prisma)
- **Client** — cognitoId, name, email, phoneNumber, history
- **Psychologist** — cognitoId, name, email, phoneNumber, location, hourlyRate, description, specialization, yearsOfExperience
- **Appointment** — clientCognitoId, psychologistId, date, meetingLink, status (enum: Pending/Denied/Approved)
- **Payment** — appointmentId, clientCognitoId, paymentDate, isPaid, amount
- **CalendarAppointment** — psychologistId, date, startHour, patientName

## Critical Security Issues (Must Fix)
1. `backend/src/middleware/authMiddleware.ts` uses `jwt.decode()` — tokens are NEVER cryptographically verified. Anyone can forge a JWT and access the API.
2. `backend/src/routes/chat.ts` has OpenRouter API key **hardcoded in source code**.
3. `frontend/.env.local` and `frontend/public/.env` contain exposed API keys.

## Coding Conventions

### Spring Boot (Backend Services)
- Use constructor injection (not `@Autowired` on fields)
- Use `@RequiredArgsConstructor` (Lombok) for concise constructors
- Return `ResponseEntity<T>` from controllers
- Use DTOs — never expose `@Document` entities directly in responses
- `@Valid` on all `@RequestBody` parameters
- Inject Keycloak user via `@AuthenticationPrincipal Jwt jwt` — `jwt.getSubject()` gives `keycloakId`
- Configuration in `application.yml`, never `.properties`
- All secrets via environment variables — never hardcode

### Angular (Frontend)
- Feature modules are lazy-loaded (see routing config)
- Use NgRx for state shared across 2+ unrelated components
- Use Angular Signals for local component state
- JWT interceptor in `core/interceptors/jwt.interceptor.ts` handles all auth headers
- API base URL always from `environment.apiGatewayUrl`
- Polish UI language where user-facing text exists (app is for Polish users)

### General
- **Never auto-commit** without being explicitly asked
- **Never push** without being explicitly asked
- Kafka topic format: `theralink.{domain}.{event}` (e.g., `theralink.appointment.created`)
- MongoDB document IDs use `String keycloakId` as the user identifier (not numeric)
- Separate database per microservice — never share MongoDB between services

## Local Development Ports
| Service | Port |
|---|---|
| Angular dev server | 4200 |
| Spring Cloud Gateway | 8090 |
| Keycloak | 8080 |
| User Service | 8081 |
| Appointment Service | 8082 |
| Psychologist Service | 8083 |
| Notification Service | 8084 |
| Payment Service | 8085 |
| MongoDB | 27017 |
| Kafka | 9092 |

## User Preferences
- Newbie to Spring Boot — always explain concepts in detail with theory before showing code
- Show full file paths when referencing code
- Polish language for any user-facing UI text (the app is a Polish mental health platform)
- Payment service should always be treated as a separate, restricted-access repository (PCI-DSS)
- No chat service, no recommendation service in scope

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TheraLink1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
