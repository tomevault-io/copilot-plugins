## campaign-budget

> Nx monorepo with NestJS backend and Angular frontend.

# Campaign Budget Distribution

Nx monorepo with NestJS backend and Angular frontend.

## Commands

```bash
pnpm nx serve api          # Start API at http://localhost:3000/api
pnpm nx serve web          # Start Angular at http://localhost:4200
pnpm nx test api           # Run API unit tests (Vitest)
pnpm nx test web           # Run Web unit tests (Vitest)
pnpm nx lint api           # Lint API
pnpm nx lint web           # Lint Web
docker-compose up -d       # Start MySQL
docker-compose down        # Stop MySQL
```

## Project Structure

- `apps/api/` — NestJS 11 backend (TypeORM, MySQL)
- `web/` — Angular 21 frontend (Signals, Standalone, Zoneless)
- `docker-compose.yml` — MySQL 8.4

## Conventions

- TypeORM entities use decorators, `synchronize: true` for dev
- API prefix: `/api` (configured in main.ts)
- Single shared `package.json` (Nx monorepo pattern)
- Test runner: Vitest for both API and Web
- Package manager: pnpm
- No hardcoded values: use enums for fixed sets (e.g., `EngagementRank`, `CampaignGoal`) and constants for magic numbers (e.g., `FREQUENCY_CAP`)
- `.http` test files live inside each module folder (e.g., `src/currency/currency.http`)

## Database

- MySQL 8.4 via Docker Compose
- Host: localhost, Port: 3306
- Database: `campaign_budget`
- User: root, Password: root

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/peelmicro) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
