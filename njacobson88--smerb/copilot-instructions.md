## smerb

> This project uses a **single Firebase project** (`r01-redditx-suicide`) with **collection prefixes** to separate dev and prod data.

# SocialScope Development Guide

## IMPORTANT: Dev vs Prod Environments

This project uses a **single Firebase project** (`r01-redditx-suicide`) with **collection prefixes** to separate dev and prod data.

- **prod** (default): Collections use standard names (`participants`, `valid_participants`, etc.)
- **dev**: Collections are prefixed with `dev_` (`dev_participants`, `dev_valid_participants`, etc.)

Subcollections (events, ema_responses, safety_alerts, etc.) are NOT prefixed — they are already scoped by their parent document.

**The default is ALWAYS prod. Dev requires explicit opt-in.**

---

## How to Switch Environments

### Flutter App
```bash
# Dev mode (orange banner visible, writes to dev_ collections)
flutter run --dart-define=ENVIRONMENT=dev

# Prod mode (default — no banner, writes to production collections)
flutter run --dart-define=ENVIRONMENT=prod
flutter run  # same as prod

# Build for distribution
flutter build apk --dart-define=ENVIRONMENT=prod
flutter build ipa --dart-define=ENVIRONMENT=prod
```

### Dashboard Backend (local development)
```bash
ENVIRONMENT=dev python -m uvicorn main:app --reload    # Dev
ENVIRONMENT=prod python -m uvicorn main:app --reload   # Prod (default)
```

### Dashboard Backend (Cloud Run deploy)
```bash
# Dev
gcloud run deploy socialscope-dashboard-api-dev --source . \
  --set-env-vars="ENVIRONMENT=dev" ...

# Prod
gcloud run deploy socialscope-dashboard-api --source . \
  --set-env-vars="ENVIRONMENT=prod" ...
```

### Cloud Functions
```bash
ENVIRONMENT=dev firebase deploy --only functions    # Deploys dev_ prefixed triggers
ENVIRONMENT=prod firebase deploy --only functions   # Deploys prod triggers
```

### Seed Scripts
```bash
ENVIRONMENT=dev node scripts/seed_test_users.js
ENVIRONMENT=prod node scripts/seed_test_users.js
```

---

## How to Tell Which Environment You're In

| Component | Dev Indicator |
|-----------|--------------|
| Flutter app | Bright orange "DEV ENVIRONMENT" banner at top of screen |
| Dashboard | Orange banner below the header: "DEV ENVIRONMENT — Data in dev_ collections" |
| Backend logs | Startup line: `Environment: dev (prefix: 'dev_')` |
| Firestore | Collections named `dev_participants`, `dev_valid_participants`, etc. |

---

## Key Architecture Decisions

- **EnvConfig** (`lib/core/config/environment_config.dart`): Singleton that reads `--dart-define=ENVIRONMENT`. Use `EnvConfig.col('participants')` to get the correct collection name.
- **config.col()** (`dashboard/backend/config.py`): Backend equivalent. Use `config.col("participants")` in Python.
- **Cloud Functions**: Use `col("participants")` helper. Trigger paths include the prefix. Function names are prefixed in dev (`dev_onSafetyAlert`).
- **Firestore Rules**: Both `participants` and `dev_participants` (and all other dev_ collections) have matching rules.

---

## Firebase Project Info
- Project ID: `r01-redditx-suicide`
- Dashboard: `https://socialscope-dashboard.web.app`
- Backend API: `https://socialscope-dashboard-api-436153481478.us-central1.run.app`
- Bundle ID (iOS/Android): `com.dartmouth.smerb`

---

## Safety-Critical Code

The check-in flow (`lib/features/checkin/screens/checkin_screen.dart`) handles SI screening and safety alerts. Changes to this file require extra care:

1. Safety alerts are persisted locally first (LocalSafetyAlerts table), then synced to Firestore
2. Four trigger questions: desire_intensity, intention_strength, thoughts_intent, ability_safe
3. Walk-away detection: if participant exceeds threshold but leaves, 5/10/15 min follow-ups fire
4. All notifications are logged to `notification_log` subcollection

**Never deploy changes to the check-in flow without testing the full safety alert pipeline.**

---
> Source: [njacobson88/smerb](https://github.com/njacobson88/smerb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
