## saas-template

> SaaS project template generator. Creates a fully configured project with:

# CLAUDE.md

## Project Overview

SaaS project template generator. Creates a fully configured project with:
Cloudflare Workers (API) + Supabase Auth + Stripe + Cloudflare Pages (SPA).

## Repository Structure

```
saas-template/
├── init.sh              # Template initializer script
├── my-template/         # Template source files
│   ├── apps/api/        # Hono + GraphQL Yoga + Pothos + Drizzle
│   ├── apps/web/        # React + Vite + Relay + Tailwind
│   ├── docker-compose.yml
│   ├── Dockerfile
│   └── scripts/         # Badge generation scripts
└── .github/workflows/   # CI that tests the template
```

## Usage

```bash
./init.sh <project-name> <project-number>
# Example: ./init.sh my-cool-app 1
# Creates ./my-cool-app/ with unique ports (31xx, 32xx, 33xx, 34xx, 35xx)
```

## Port Scheme

Each project gets a unique number N (1-99). Host ports:
- Web (Vite): 3100 + N
- API (Wrangler): 3200 + N
- Auth (GoTrue): 3300 + N (internal stays 9999)
- DB (PostgreSQL): 3400 + N (internal stays 5432)
- Stripe Mock: 3500 + N (internal stays 12111)

## Development Workflow

### Testing template changes

When modifying files under `my-template/`, **no need to re-run init.sh**.
Just copy the changed file directly into a running test project:

```bash
# Copy a modified source file into the running container
docker cp my-template/apps/api/src/lib/db.ts <container>:/workspace/apps/api/src/lib/db.ts

# Then re-run the relevant check inside the container
docker exec <container> bash -c "cd /workspace/apps/api && npm run typecheck"
docker exec <container> bash -c "cd /workspace/apps/api && npm run lint"
docker exec <container> bash -c "cd /workspace/apps/api && npm run test:coverage"
```

Only re-init when testing init.sh itself or verifying the full flow (port substitution, naming, etc.).

### Full integration test

```bash
./init.sh test-app 10 /tmp/test-app
cd /tmp/test-app
docker compose up -d
docker exec <container> bash -c "cd /workspace && pnpm install"
docker exec <container> bash -c "cd /workspace/apps/api && npm run db:push && npm run dev:seed"
docker exec -d <container> bash -c "cd /workspace/apps/api && npm run dev > /tmp/wrangler.log 2>&1"
# Wait ~10s, then:
curl http://localhost:3210/api/health
# Cleanup:
docker compose down -v && rm -rf /tmp/test-app
```

## CI

GitHub Actions workflow (`.github/workflows/ci.yml`) runs on push to main:
1. `init.sh ci-test 10` — creates project from template
2. `docker compose up -d --build` — starts all services
3. `pnpm install` → `db:push` → `dev:seed`
4. Runs typecheck, lint, unit tests (100% coverage), integration tests, e2e tests
5. Tests health check and GraphQL from host
6. Tests frontend (Vite) reachable from host
7. Generates coverage + test status badges → commits to `badges/`

## Conventions

- All development runs **inside the devcontainer** (not host)
- Services communicate via Docker DNS: `db:5432`, `auth:9999`, `stripe-mock:12111`
- Host ports are only for browser/external access
- API uses `wrangler dev` (actual Workers runtime, not Node.js dev server)
- DB migrations: `drizzle-kit push` only (no SQL migration files)
- 100% unit test coverage required on service layer

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/soya-miyoshi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/soya-miyoshi)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
