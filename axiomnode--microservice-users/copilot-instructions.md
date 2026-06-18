## microservice-users

> Identity/session and gameplay analytics microservice with leaderboard/monitoring APIs.

# AGENTS

## Repo purpose
Identity/session and gameplay analytics microservice with leaderboard/monitoring APIs.

## Key paths
- src/: auth/session/profile/stats logic and routes
- docs/: architecture, guides, operations
- .github/workflows/ci.yml: CI + infra dispatch

## Local commands
- cd src && npm install
- cd src && npm run dev
- cd src && npm test && npm run lint && npm run build

## CI/CD notes
- Push to main dispatches platform-infra build-push with service=microservice-users.
- Platform-infra handles rollout to dev.

## LLM editing rules
- Keep strict vs dev auth modes explicit.
- Preserve leaderboard/stats contracts for BFF consumers.
- Mirror auth/header behavior changes in docs.

---
> Source: [AxiomNode/microservice-users](https://github.com/AxiomNode/microservice-users) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
