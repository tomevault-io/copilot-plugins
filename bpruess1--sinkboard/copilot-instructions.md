## sinkboard

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Dev Commands

```bash
npm install                          # Install all workspaces
npm run build                        # Build all workspaces (shared must build first)
npm run build:shared                 # Build shared types/constants (required before backend/frontend)
npm run build:backend                # Compile backend TypeScript
npm run build:frontend               # Compile + Vite build frontend
npm run dev                          # Vite dev server (frontend only, localhost:5173)
npm run lint                         # ESLint across all workspaces
```

**CDK (infra workspace):**
```powershell
cd infra; npx cdk synth              # Synthesize CloudFormation templates
cd infra; npx cdk deploy --all       # Deploy all stacks
```

**Build order matters:** `shared` must be built before `backend` or `frontend` since both depend on `@sink-board/shared`.

## Architecture

This is a gamified task management app where tasks are treasure chests that sink in an ocean. Users submit progress updates scored by AI to raise chests back up.

**Monorepo with 4 npm workspaces:**
- **shared** — Types, constants, and business-hours calculator. This is the source of truth for sink mechanics. Both frontend and backend import from here.
- **backend** — AWS Lambda handlers (one per API route), deployed via CDK's `NodejsFunction` (esbuild ESM bundling). Uses DynamoDB single-table design and Claude Haiku for AI scoring.
- **frontend** — React 19 + Vite SPA with PixiJS v8 for ocean rendering. Auth via raw `amazon-cognito-identity-js` (no Amplify). React Query for data fetching. Sink positions are client-calculated at 60fps from timestamps, not fetched.
- **infra** — AWS CDK stacks: DNS/cert (us-east-1), Auth (Cognito), API (Lambda + API Gateway + DynamoDB), Frontend (S3 + CloudFront).

**Key design constraints:**
- Target cost <$1/month. No WebSockets, no Amplify, SSM Parameter Store instead of Secrets Manager.
- All timestamps UTC. Business hours (Mon-Fri, 24h/day) are the time unit for sinking. `shared/src/business-hours.ts` is the canonical calculator used by both client and server.
- Kraken is client-triggered (requires page visibility), server validates depth >= 95%.
- Lambdas are Node 20, ARM64, ESM format.

## DynamoDB Single-Table Keys

| Entity | PK | SK |
|---|---|---|
| User | `USER#<id>` | `PROFILE` |
| Task | `USER#<id>` | `TASK#<taskId>` |
| Update | `TASK#<taskId>` | `UPDATE#<ts>#<id>` |

GSI1 exists for cross-entity lookups. All DynamoDB operations are in `backend/src/services/dynamo.ts`.

## CI/CD

GitHub Actions deploys on push to `main`. Frontend and backend have separate workflows triggered by path filters. Frontend build requires `VITE_*` env vars (Cognito IDs, API URL) set as GitHub Actions variables. AWS auth uses OIDC role assumption (no stored keys).

## API Routes

All routes are JWT-protected (Cognito). Defined in `infra/lib/api-stack.ts`, handlers in `backend/src/handlers/`. The submit-update handler has a longer timeout (30s) for the AI scoring call.

---
> Source: [bpruess1/sinkboard](https://github.com/bpruess1/sinkboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
