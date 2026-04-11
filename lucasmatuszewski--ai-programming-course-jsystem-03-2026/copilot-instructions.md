## ai-programming-course-jsystem-03-2026

> This file is the root instruction set for all agents working in this repository. Keep changes aligned with the product, architecture, and design documents, and do not treat a task as done until the changed scope is verified.

# Repository Guidelines

This file is the root instruction set for all agents working in this repository. Keep changes aligned with the product, architecture, and design documents, and do not treat a task as done until the changed scope is verified.

## Primary References
- `docs/loan-decision-copilot-prd.md` - product requirements and user flows
- `docs/loan-decision-copilot-adr-and-implementation-plan.md` - architecture decisions and implementation direction
- `docs/nbp-design-system.md` - UI language, brand rules, and design tokens

Read the relevant sections before implementing. If code, comments, config, or docs contradict these files, bring them back into alignment.

## Specialist Instructions
- `backend/AGENTS.md` - backend-specific rules
- `frontend/AGENTS.md` - frontend-specific rules

## Current Stack
- Backend: Spring Boot, Java 21, Maven, Spring AI, LangGraph4j, AG-UI Java SDK
- Frontend: Next.js, React, TypeScript, CopilotKit, AG-UI client, Tailwind CSS
- Runtime config: OpenRouter settings come from `.env` and `backend/src/main/resources/application.yml`

## Project Structure
- `backend/` - Spring Boot app, with code in `src/main/java`, config in `src/main/resources`, tests in `src/test/java`
- `frontend/` - Next.js App Router app, currently centered in `src/app`
- `docs/` - PRD, ADR/implementation plan, design system
- `scripts/` - setup and local run scripts for Windows and Bash
- `docker/` - PostgreSQL init assets and seed SQL for `compose.yaml`
- `ag-ui/` - Git submodule with the AG-UI community Java SDK used by the newer `langgraph4j-copilotkit` template; it is external source code, not part of this app, and should not be modified unless the task explicitly targets the SDK integration itself

The AG-UI behavior should come from the community Java SDK, so do NOT add custom in-repo AG-UI protocol implementation. That SDK exposes the agent/UI integration layer, including the agent-driven UI control model used by the app.
This project uses PostgreSQL in Docker under WSL for local backend data. Do not introduce file-based, memory-only, or substitute databases as the default project direction.

Frontend structure will grow beyond the current starter shape. Keep new code organized and predictable as the app expands: separate route files, reusable UI components, domain logic, and tests instead of letting everything accumulate inside `src/app/page.tsx`.

## Workflow
1. Read the relevant PRD, ADR, and design system sections first.
2. Read the area-specific `AGENTS.md` if the task touches `backend/` or `frontend/`.
3. Define the expected behavior from the specification before changing code.
4. Use TDD for every feature and bug fix.
5. Verify the changed scope completely before committing.
6. Commit only after the scope is in a working state.

Keep commits focused and granular. Do not push to remote unless the user explicitly asks.

## TDD Rules
For every feature or bug fix:
1. Start from the specification, not from the existing implementation.
2. Write or extend tests before production code.
3. Run the new tests and confirm they fail for the expected reason.
4. Implement the minimum code needed to make them pass.
5. Run the relevant full verification for the changed scope.
6. Refactor only with tests still green.

If the project does not yet have suitable test infrastructure for the changed area, add it as part of the task instead of silently skipping tests.

## Test Honesty Rules
Do not make tests pass by weakening them.

Every new or modified test must be specific enough that it would fail if the related production behavior breaks. After tests pass, verify that by intentionally breaking or removing the relevant implementation and confirming the test fails, then restore the code.

Forbidden shortcuts:
- changing assertions to match broken behavior
- removing assertions that expose a real defect
- adding fallback assertions that allow multiple incompatible outputs
- mocking away the unit or flow that is supposed to be tested
- skipping a failing test without a justified reason in the code and task report

If a test fails, fix the production code or rewrite the test because the requirement was wrong or changed. Do not edit the test to accept incorrect behavior.

## Verification Requirements
Before finishing any task, verify all relevant items for the changed scope:
- tests for the changed behavior pass
- existing relevant tests still pass
- lint passes with no errors
- build passes
- the affected app starts correctly if the change impacts runtime behavior
- no warnings, broken flows, or obvious regressions remain

Use the real commands that exist in the repo today:
- Backend tests: `cd backend; .\mvnw.cmd test`
- Backend run check: `cd backend; .\mvnw.cmd spring-boot:run`
- Frontend install: `cd frontend; npm install`
- Frontend lint: `cd frontend; npm run lint`
- Frontend build: `cd frontend; npm run build`
- Frontend unit/integration tests: `cd frontend; npm run test`
- Frontend E2E tests: `cd frontend; npm run test:e2e`
- Frontend run check: `cd frontend; npm run dev`
- Full app scripted setup (use only to install/build dependencies): `powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\setup-copilot-template.ps1` or `bash ./scripts/setup-copilot-template.sh`
- Full app scripted start: `powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\start-copilot-template.ps1` or `bash ./scripts/start-copilot-template.sh`

Frontend work is expected to use Vitest for unit/integration coverage and Playwright for E2E coverage. If the required test scripts or configuration are missing for the task, add them instead of skipping tests.

## Coding Standards
- Follow existing conventions in each area instead of inventing a new style.
- Keep secrets in environment variables only. Never commit credentials, tokens, `.env` secrets, or local database artifacts.
- Remove outdated or misleading instructions when you find them; do not leave contradictory project files behind.
- When you learn from a mistake, update the most specific relevant `AGENTS.md` file so the same mistake is less likely to happen again. Prefer nested specialist files over expanding the root file.
- For database changes, keep schema evolution and seed automation explicit and repeatable. Prefer migrations plus seed scripts that work with the project PostgreSQL setup.

## Completion Criteria
A task is complete only when:
- the implementation matches the relevant PRD, ADR, and design system guidance
- tests were added first and verified honestly
- verification for the changed scope passed
- the repository remains consistent and reviewable
- the change is committed with a focused message

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LucasMatuszewski)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/LucasMatuszewski)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
