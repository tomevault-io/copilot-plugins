## peaks-firebase

> - `functions/` — Firebase Cloud Functions (TypeScript, compiled to `functions/lib/`)

# Peaks Firebase

## Project Structure
- `functions/` — Firebase Cloud Functions (TypeScript, compiled to `functions/lib/`)
- `functions/src/` — Source files
- `firestore.rules` — Firestore security rules
- `firebase.json` — Firebase project configuration

## Build & Lint
After making changes, always verify the affected project compiles and lints cleanly before finishing:

**Functions:**
```bash
cd functions && npm run build && npm run lint
```
- **Build**: `npm run build` (runs `tsc`)
- **Lint**: `npm run lint` (runs `eslint --ext .js,.ts .`)
- Lint must pass with zero errors. Warnings for unused `context` params in Firebase function signatures are acceptable.

**Web:**
```bash
cd web && npm run build && npm run lint
```
- **Build**: `npm run build` (runs `next build`)
- **Lint**: `npm run lint` (runs `eslint`)
- Both must pass with zero errors before considering work complete. Pre-existing warnings (e.g. `<img>` vs `<Image />`) are acceptable.

## Deployment
- **CI/CD**: GitHub Actions (`.github/workflows/deploy.yml`) — deploys on push to `main`
  - `deploy-functions` job: builds + deploys Cloud Functions and Firestore rules
  - `deploy-api` job: builds + deploys Cloud Run API (`cloud-sql/api`) via multi-stage Dockerfile
- Deploy uses `--force` to auto-delete stale functions removed from source
- Service account credentials: stored as `FIREBASE_SERVICE_ACCOUNT` GitHub secret
- `firebase.ts` uses application default credentials in CI (falls back when `admin-service-account.json` is absent)

**After every `git push`**: monitor the triggered workflow run with `gh run list --limit 1` and `gh run watch <id>`. If a job fails, check logs with `gh run view <id> --log-failed`, fix the issue, and push again. Do not consider a push complete until CI is green.

**Cloud Run secrets/env vars**: All required env vars and secrets for the Cloud Run API are pinned in `deploy.yml`. **NEVER use `gcloud run services update --set-secrets` or `--set-env-vars`** — these flags REPLACE all existing values, silently dropping any not listed. Instead, update the `env_vars` and `secrets` fields in `deploy.yml` and redeploy via CI. The post-deploy verification step will catch DB connectivity failures.

## GPX Files
When downloading GPX files for the project (e.g. from Hiking Project, Wikiloc, AllTrails), **always verify the files are legitimate GPX** before considering the task complete. Many sources return HTML login pages instead of actual GPX data. Check that files start with `<?xml` and contain `<trkpt>` or `<rtept>` elements. Delete any invalid files immediately.

## React useEffect Rules
When writing or modifying `useEffect` hooks in the web app:
- **NEVER use objects or arrays as dependencies** — they create new references every render, causing infinite re-render loops. Use primitive values (strings, numbers, booleans) instead. For example, use `[userLat, userLng]` not `[userLocation]`.
- **NEVER set state inside an effect that re-triggers that same effect** — e.g. setting `locationStatus` inside an effect that depends on `[locationStatus]`.
- After modifying any page with useEffect, **verify the page doesn't infinite-loop** by loading it in the browser and confirming network requests stop after initial load.

## Key Details
- Uses `firebase-functions` v4 (v1 API) and `firebase-admin` v11
- Node 20 runtime
- Secrets stored via `functions.config()` (not hardcoded) — never commit secrets
- `functions/functions/` is a legacy nested directory — do not use it

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jhmacdon)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jhmacdon)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
