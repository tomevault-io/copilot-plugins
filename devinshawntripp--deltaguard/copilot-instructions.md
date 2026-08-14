## deltaguard

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run Commands

```bash
npm run dev                    # next dev --turbopack (localhost:3000)
npm run build                  # next build --turbopack
npm run start                  # next start (production)
npm run lint                   # ESLint
npm run test:quality           # tsx --test src/tests/*.test.ts
```

Docker build:
```bash
docker build -t devintripp/deltaguard-ui:latest .
# entrypoint.sh runs prisma migrate then npm start
```

## Architecture

Next.js 16 app (React 19, TypeScript, Tailwind CSS v4) serving as both the web frontend and API gateway for the ScanRook platform.

### Key Patterns

**Auth:** NextAuth v4 with JWT strategy. Providers: Google OAuth + email/password (bcrypt). Sessions stored 7 days, validated against Redis for revocation. Token includes `org_id`, `roles_mask`, `token_version`, `sid`.

**RBAC:** Bitwise role masks in `src/lib/authz.ts`. Roles: VIEWER(1), ANALYST(2), OPERATOR(4), SCAN_ADMIN(8), POLICY_ADMIN(16), BILLING_ADMIN(32), API_KEY_ADMIN(64), ORG_OWNER(128). Checked via `actorHasAnyRole()`. API routes authenticate via session JWT or `Authorization: Bearer <api-key>` header.

**File Upload Flow:** Browser gets presigned S3 POST URL from `/api/uploads/presign` → uploads directly to S3 (bypasses Next.js) → calls `POST /api/jobs` with S3 location → worker picks up.

**Real-time Progress:** `/api/jobs/[id]/events` returns SSE stream. `jobEventsBus.ts` uses PG LISTEN on `scan_events` channel for push-based delivery. Terminal stages: `scan.done`, `scan.summary`, `scan.error`, `worker.stale.fail`.

**Schema Bootstrap:** `src/lib/prisma.ts` calls `ensurePlatformSchema()` on first connection — creates all tables if missing, seeds default plans (FREE/BASIC/PRO/ENTERPRISE), creates default org.

### Module Layout

- `src/app/(dashboard)/dashboard/` — Main scan list, job details, findings, file tree
- `src/app/(auth)/` — Signin, register pages
- `src/app/api/jobs/` — Job CRUD, SSE events, findings, files, packages, reports, cancel
- `src/app/api/uploads/presign/` — S3 presigned URL generation
- `src/app/api/cli/` — Device auth flow + enrichment API for scanner CLI
- `src/app/api/org/` — Members, API keys, invites
- `src/app/api/billing/` — Stripe checkout, portal, webhooks
- `src/app/api/settings/scanner/` — Per-org scanner enrichment settings
- `src/lib/prisma.ts` — Prisma singleton + raw SQL schema bootstrap
- `src/lib/authOptions.ts` — NextAuth config (JWT, providers, callbacks)
- `src/lib/authz.ts` — Role-based authorization helpers
- `src/lib/s3.ts` — Dual S3 clients (`s3Public` for browser, `s3Internal` for server-side)
- `src/lib/jobEventsBus.ts` — SSE event streaming from scan_events
- `src/lib/jobs.ts` — Job CRUD + `pg_notify('job_events')`
- `src/lib/redis.ts` — Session caching + token revocation
- `src/lib/scanner.ts` — Per-org scanner enrichment settings
- `src/lib/usage.ts` — Monthly scan quota tracking

### S3 Configuration

Two S3 clients are initialized: `s3Public` (external HTTPS URL for browser presigned URLs) and `s3Internal` (cluster-internal endpoint for server-side operations). Both use path-style URLs for MinIO compatibility. Env vars support both `S3_*` and legacy `MINIO_*` prefixes.

### Kubernetes Manifests

Located in `k8s/deltaguard/`:
- `deployment.yaml` — UI: 3 replicas, 256Mi-1Gi memory, port 3000
- `worker-deployment.yaml` — Worker: 3 replicas, 1500Mi-5Gi memory, WORKER_CONCURRENCY=1
- `service.yaml` — ClusterIP deltaguard-web:3000
- `ingress.yaml` — scanrook.io, scanrook.sh, deltaguard.apps.onetripp.com
- `redis.yaml` — Single replica, 5Gi PVC
- `pvc.yaml` — Redis storage claim

## CRITICAL: No Polling

NEVER use `setInterval`, `setTimeout`, or recursive timers to poll databases or REST APIs for data changes. This application uses PostgreSQL NOTIFY/LISTEN + SSE for all real-time updates. All data push must flow through:

- **PostgreSQL NOTIFY/LISTEN** for server-side event propagation
- **Server-Sent Events (SSE)** for client-side real-time updates
- `jobsBus.ts` (PG LISTEN `job_events`) for job status changes
- `jobEventsBus.ts` (PG LISTEN `scan_events`) for scan progress events

The only acceptable `setInterval` is for UI-only timers (e.g., updating a clock display with `Date.now()`). Never poll a database table, REST endpoint, or file system for changes.

---
> Source: [devinshawntripp/deltaguard](https://github.com/devinshawntripp/deltaguard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
