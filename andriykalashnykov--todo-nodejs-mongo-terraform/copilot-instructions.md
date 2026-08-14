## todo-nodejs-mongo-terraform

> Full-stack Todo application with Node.js/Express API, React frontend, and Azure infrastructure managed by Terraform.

# CLAUDE.md

## Project Overview

Full-stack Todo application with Node.js/Express API, React frontend, and Azure infrastructure managed by Terraform.

## Tech Stack

- **API:** Node.js + Express 5 + TypeScript + Mongoose (MongoDB)
- **Web:** React 19 + Vite + TypeScript + Fluent UI
- **Infrastructure:** Terraform on Azure (Cosmos DB, App Service, Key Vault, APIM)
- **E2E Tests:** Playwright
- **Package Manager:** pnpm (via corepack)
- **CI/CD:** GitHub Actions (lint -> build + test -> deploy) + Azure Developer CLI (azd)

## Project Structure

```
src/api/       # Express API (port 3100)
src/web/       # React frontend (port 3000)
infra/         # Terraform modules
tests/         # Playwright E2E tests
.github/       # GitHub Actions workflows
```

## Build & Run

```bash
make deps      # Install required tools (idempotent)
make lint      # Lint API, Web, and Dockerfiles
make build     # Build API and Web
make test      # Run API unit tests
make run       # Start API and Web locally
make ci        # Full local CI pipeline
```

## Key Variables

| Variable | Value | Purpose |
|----------|-------|---------|
| `NVM_VERSION` | 0.40.4 | nvm version for Node management |
| `NODE_VERSION` | 24 | Node.js major version |
| `ACT_VERSION` | 0.2.87 | act for local CI runs |
| `HADOLINT_VERSION` | 2.14.0 | Dockerfile linter |

## Environment Variables

**API** (`.env`): `AZURE_COSMOS_CONNECTION_STRING`, `AZURE_COSMOS_DATABASE_NAME`, `APPLICATIONINSIGHTS_CONNECTION_STRING`

**Web** (`.env.local`): `VITE_API_BASE_URL`, `VITE_APPLICATIONINSIGHTS_CONNECTION_STRING`

## Notes

- API tests require a running MongoDB instance (integration tests, not unit tests)
- Uses pnpm (not npm) — install via `corepack enable pnpm`
- ESLint uses flat config format (`eslint.config.mjs` / `eslint.config.js`)

## Upgrade Backlog

Last reviewed: 2026-04-03

- [ ] Monitor `eslint-plugin-react-hooks` for eslint 10 compatible release (v8) — peer dep override in place

## Skills

Use the following skills when working on related files:

| File(s) | Skill |
|---------|-------|
| `Makefile` | `/makefile` |
| `renovate.json` | `/renovate` |
| `README.md` | `/readme` |
| `.github/workflows/*.yml` | `/ci-workflow` |

When spawning subagents, always pass conventions from the respective skill into the agent's prompt.

---
> Source: [AndriyKalashnykov/todo-nodejs-mongo-terraform](https://github.com/AndriyKalashnykov/todo-nodejs-mongo-terraform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
