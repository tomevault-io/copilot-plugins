## white-povar

> - Backend env file at `backend/.env` contains real Supabase URL, anon key, service key, OpenAI key, and Firebase project ID.

# Project Intelligence (.cursorrules)

## Credentials & Configuration
- Backend env file at `backend/.env` contains real Supabase URL, anon key, service key, OpenAI key, and Firebase project ID.
- Frontend reads `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `API_BASE_URL` via `String.fromEnvironment` in `frontend/lib/core/config/app_config.dart`.
- For local dev, pass values with `--dart-define` when running Flutter.

## Recent Changes (2025-08-11)
- Added `supabase_flutter` and enabled `firebase_core`, `firebase_auth` in `frontend/pubspec.yaml`.
- Updated `frontend/lib/core/config/app_config.dart` defaults to match current Supabase project for development.
- Fixed `Uint8List` upload in `frontend/lib/core/services/supabase_service.dart`.
- Verified `flutter build web` succeeds.

## Preferred Workflow
- Keep secrets out of source. Use `--dart-define` for builds and CI secrets for deployments.
- Regenerate Firebase configs with `flutterfire configure` when project/app IDs change.
- Backend settings: use `backend/app/core/settings.py` with `.env` loaded via pydantic-settings.

## Pending/Watchouts
- Replace `SECRET_KEY` placeholder in backend `.env` for production.
- Ensure RLS policies in Supabase are applied as documented.
- Update CI/CD to inject dart-defines for web/mobile builds.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/igskak) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
