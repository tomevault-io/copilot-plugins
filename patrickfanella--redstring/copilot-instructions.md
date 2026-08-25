## redstring

> This repository is `redstring`, an evidence-first litigation workspace built with Go, Vite React TypeScript, Postgres, MinIO, Keycloak/OIDC, local AI, and Docker Compose.

# Agent Instructions

This repository is `redstring`, an evidence-first litigation workspace built with Go, Vite React TypeScript, Postgres, MinIO, Keycloak/OIDC, local AI, and Docker Compose.

## Priorities

1. Keep the application runnable and evidence-first.
2. Preserve Open Pilot issue and PR templates.
3. Keep generated caches, secrets, and local data out of git.
4. Update this file, `README.md`, and `docs/reference/stack.md` when stack conventions change.

## Local ports

Do not occupy host `5432`; it belongs to llama-line on this workstation. Use:

```bash
POSTGRES_PORT=55433 KEYCLOAK_PORT=18081 MINIO_API_PORT=19000 MINIO_CONSOLE_PORT=19001 OIDC_BROWSER_ISSUER_URL=http://localhost:18081/realms/redstring-dev MINIO_BROWSER_ENDPOINT=http://localhost:19000 docker compose up --build postgres keycloak minio api web
```

If `make verify` modifies `backend/app`, restore it before reporting completion:

```bash
rtk git checkout -- backend/app
```

## Verification

Run:

```bash
make verify
```

Use narrower commands only when an issue explicitly asks for a smaller check.

---
> Source: [PatrickFanella/redstring](https://github.com/PatrickFanella/redstring) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
