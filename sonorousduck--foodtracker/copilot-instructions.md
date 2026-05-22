## foodtracker

> - Before starting work, do any needed inspection/research to understand the issue, then draft a detailed plan, present it for approval, and wait for confirmation before making changes.

# Repository Guidelines

## Planning & Approval
- Before starting work, do any needed inspection/research to understand the issue, then draft a detailed plan, present it for approval, and wait for confirmation before making changes.
- The plan should be complete enough that the user could execute it themselves step by step.

## Project Structure & Module Organization
- `foodtracker-backend/`: NestJS API server. Core code in `foodtracker-backend/src`, tests in `foodtracker-backend/test`.
- `foodtracker-frontend/`: Expo app. Routes in `foodtracker-frontend/app`, shared UI in `foodtracker-frontend/components`, hooks in `foodtracker-frontend/hooks`, assets in `foodtracker-frontend/assets`, and generated types in `foodtracker-frontend/types`.
- `scripts/` and root scripts: DTO sync tools. `generate-dtos.sh` and `sync-dtos.sh` regenerate frontend types from backend DTOs.

## Build, Test, and Development Commands
Run commands from the appropriate folder and use yarn for both backend and frontend.
- Backend dev server: `cd foodtracker-backend && yarn start:dev`
- Backend build: `cd foodtracker-backend && yarn build`
- Backend lint/format: `cd foodtracker-backend && yarn lint` and `yarn format`
- Backend tests: `cd foodtracker-backend && yarn test` (unit), `yarn test:e2e`, `yarn test:cov`
- Frontend dev server: `cd foodtracker-frontend && yarn start`
- Frontend platform runs: `yarn android`, `yarn ios`, `yarn web`
- Frontend lint: `cd foodtracker-frontend && yarn lint`
- Frontend tests: `cd foodtracker-frontend && yarn test` (watch with `yarn test:watch`)
- Regenerate DTO types (bash required): `bash ./generate-dtos.sh` (outputs to `foodtracker-frontend/types`)

## Coding Style & Naming Conventions
- TypeScript across backend and frontend. Follow the existing ESLint configs.
- Do not cast to `any`. Prefer precise types, generics, type guards, and `unknown` with narrowing.
- Backend formatting via Prettier (`semi: true`, `singleQuote: true`, `trailingComma: all`). Run `yarn format` before committing.
- Frontend linting uses `eslint-config-expo` via `yarn lint`.
- File names must be all lowercase (for example, `haptictab.test.tsx`).
- Keep files aligned to their feature area (Nest modules/controllers/services in backend, file-based routes in `foodtracker-frontend/app`).

## Testing Guidelines
- Backend tests use Jest. Unit tests live in `foodtracker-backend/src` and match `*.spec.ts`. E2E tests live in `foodtracker-backend/test`.
- Run `yarn test` for unit tests and `yarn test:e2e` for E2E coverage.
- Frontend tests use `jest-expo` with `@testing-library/react-native`. Place tests in `foodtracker-frontend/**/__tests__` and name them `*.test.tsx`.
- Always add or update tests for user-facing behavior changes; avoid merging features without functional coverage.
- After writing or updating tests (or any code change), run `cd foodtracker-frontend && yarn test` and fix failures before handing off.

---
> Source: [sonorousduck/FoodTracker](https://github.com/sonorousduck/FoodTracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
