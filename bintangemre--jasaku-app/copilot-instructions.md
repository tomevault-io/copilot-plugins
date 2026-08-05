## jasaku-app

> Jasaku is a home-service marketplace with two packages in this monorepo.

# AGENTS.md — Jasaku

Jasaku is a home-service marketplace with two packages in this monorepo.

## Packages

| Directory | Tech | Role |
|---|---|---|
| `jasaku_app/` | Flutter 3.7+ / Riverpod / Dio | Mobile app (customer + provider "Mitra") |
| `jasaku-backend/` | Express 5 / TypeScript / Prisma 7 / PostgreSQL (Supabase) | REST API |

## Flutter app — `jasaku_app/`

**Entry points** (three separate apps sharing the same codebase):
- `lib/main_customer.dart` — customer app, blue theme
- `lib/main_provider.dart` — provider (Mitra) app, teal theme  
- `lib/main.dart` — stale boilerplate (ignore, unmodified template)

**Bootstrap flow**: `core/bootstrap.dart` → Firebase init → `runApp`.

**Networking**: Singleton Dio client in `core/network/api_client.dart` with JWT interceptor. Base URL is configured in `core/constants/api_endpoints.dart` via `--dart-define=BASE_URL=...`, default `10.0.2.2:3000` (Android emulator → host).

**Auth token**: `flutter_secure_storage` via `core/utils/storage.dart` (key: `jwt_token`).

**Commands**:
| Action | Command |
|---|---|
| Run customer (emulator) | `flutter run --target lib/main_customer.dart` |
| Run customer (HP fisik) | `flutter run --target lib/main_customer.dart --dart-define=BASE_URL=IP_LAPTOP:3000 -d DEVICE_ID` |
| Run provider (emulator) | `flutter run --target lib/main_provider.dart` |
| Run provider (HP fisik) | `flutter run --target lib/main_provider.dart --dart-define=BASE_URL=IP_LAPTOP:3000 -d DEVICE_ID` |
| Analyze | `flutter analyze` |
| Test | `flutter test` |
| Codegen | `dart run build_runner build` |
| Build APK | `flutter build apk` (or `--target lib/main_customer.dart` / `main_provider.dart`) |
| Run with ORS | Add `--dart-define=ORS_API_KEY=YOUR_KEY` for route directions |

**Maps & Routing**:
- **Tile provider**: CartoDB Voyager (`light_all`) — gratis, no API key
- **Route directions**: OpenRouteService (ORS) — gratis tier, butuh API key via `--dart-define=ORS_API_KEY=...`
- **RoutingService** di `lib/services/routing_service.dart` — panggil ORS API, return `List<LatLng>`
- **Provider location tracker** di `lib/features/location/presentation/providers/location_tracker_provider.dart` — kirim lokasi ke `PUT /api/locations/update` tiap 30 detik via `Geolocator.getPositionStream()`
- **Provider dashboard** menampilkan map dengan: marker biru (provider live), marker merah (customer), polyline rute
- **Order tracking page** menampilkan rute antara provider dan customer

**Test**: Single smoke test at `test/widget_test.dart` hitting the stale `main.dart`. Real tests not yet written.

## Backend — `jasaku-backend/`

**Stack**: Express 5 + TypeScript (ESNext modules) + Prisma 7 with `@prisma/adapter-pg`.

**Commands**:
| Action | Command |
|---|---|
| Dev server (hot reload) | `npm run dev` |
| Build (tsc) | `npm run build` |
| Prisma generate | `npx prisma generate` (outputs to `src/generated/prisma/`) |
| Prisma migrate | `npx prisma migrate dev` |
| Prisma studio | `npx prisma studio` |

**No tests, no linter, no typecheck script configured.**

**Module structure** (flat under `src/modules/`):
- `auth/` — register/login (email + Google OAuth), JWT
- `services/` — service categories & listings
- `orders/` — CRUD + status transitions
- `locations/` — geospatial (PostGIS geometry)
- `provider/` — provider-specific services & locations
- `reviews/` — ratings & reviews
- `payments/` — payment methods & status
- `notifications/` — FCM push
- `custom-tasks/` — custom task proposals
- `admin/` — admin endpoints

Each module follows the pattern `*.routes.ts` → `*.controller.ts` → `*.service.ts`.

**Middleware**: `auth.middleware.ts` (JWT verify), `role.middleware.ts` (role guard), `upload.middleware.ts` (multer).

**API docs**: Swagger UI at `http://localhost:3000/api-docs` (spec driven by `src/docs/`).

**Prisma quirks**:
- Client is generated **in-repo** at `src/generated/prisma/` (not `node_modules/.prisma`). Import path: `../generated/prisma/client`.
- Adapter: `@prisma/adapter-pg` wrapping Supabase pooler URL.
- All models use UUIDs (`uuid_generate_v4()`) and are annotated with Row-Level Security (managed in DB, not Prisma).
- PostGIS `geometry` type used in `custom_tasks`, `order_locations`, `provider_locations`.

## Security / Commits

- `.env` has live Supabase, JWT, and Google OAuth credentials — **never commit**.
- Firebase Admin SDK JSON excluded via `.gitignore` (`jasaku-backend/src/config/firebase/*.json`).
- CORS wide-open (`origin: '*'`) in `src/app.ts`.
- Do not commit `generated/prisma/` contents (gitignored).

## Firebase Admin Setup

Backend push notifications need a Firebase service account JSON:
1. Go to [Firebase Console](https://console.firebase.google.com) → Project Settings → Service Accounts
2. Click "Generate New Private Key" → download JSON
3. Save as `jasaku-backend/src/config/firebase/service-account.json`
4. The file is gitignored — safe to place there
5. Restart backend — `admin.ts` auto-detects the file and initializes Firebase Admin SDK

If the file doesn't exist, notifications silently fail (no crash).

---
> Source: [bintangemre/Jasaku-App](https://github.com/bintangemre/Jasaku-App) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
