## lined

> > Quick reference for Claude Code. For full architectural detail, see AGENTS.md.

# CLAUDE.md — Lined Monorepo

> Quick reference for Claude Code. For full architectural detail, see AGENTS.md.

## Project

**Lined** — schedule sync & task coordination app for couples, families, and friends.
Slogan: *"Where life and quality time meet."*

## Sub-projects at a Glance

| Directory | What it is | Primary language |
|---|---|---|
| `backend/lined/` | Spring Boot REST API | Java 17 |
| `lined-web/` | Vite + React 19 web app | TypeScript |
| `mobile/Lined/` | Expo + React Native app | TypeScript |
| `fitness-metrics-collector/` | CI quality metrics tool | TypeScript |
| `fitness-metrics-analyzer/` | Research analysis scripts | Python |

## Safe Commands to Run Without Asking

```bash
# Backend (from backend/lined/)
./gradlew test
./gradlew checkstyleMain
./gradlew spotbugsMain
./gradlew jacocoTestReport

# Web (from lined-web/)
npm test
npm run lint
npm run lint:fix
npm run build

# Mobile (from mobile/Lined/)
npm run lint

# Metrics collector (from fitness-metrics-collector/)
npm run build
```

## Always Check Before Changing Backend Code

1. **Layering:** Controller → Service → Repository → Entity. No cross-layer shortcuts.
2. **Transactions:** `@Transactional` from `jakarta.transaction`, not Spring's.
3. **Lookups:** `EntityFinder.findOrThrow()` — never bare `Optional.get()`.
4. **Exceptions:** `NotFoundException` (404) or `ConflictException` (409) — never raw exceptions.
5. **Tests:** Every new service behaviour needs a unit test.
6. **Checkstyle:** Methods max 50 lines, 2-space indent, braces required.
7. **Quality gates:** Do not delete tests to raise coverage. Do not suppress SpotBugs without a comment.

## Always Check Before Changing Web Code

1. **Data fetching:** Only through TanStack Query hooks in `src/hooks/` — never direct `ky` calls in components.
2. **State split:** Server data → TanStack Query. UI state → Zustand.
3. **Colours:** Only Tailwind tokens from `tailwind.config.ts` — no hard-coded hex values.
4. **Components:** Never modify `src/components/ui/` (shadcn-owned). Wrap in `src/components/`.
5. **Tests:** Use MSW v2 for API mocking — never mock `ky` directly.
6. **Node version:** 22 LTS (`.nvmrc` — run `nvm use` first).

## Key Conventions (Backend)

- DTO types: `{Domain}CreateDto` / `{Domain}UpdateDto` / `{Domain}Dto` (Java records)
- Auth MVP: `X-User-Id: <Long>` header on every endpoint that needs the caller
- Timestamps: `OffsetDateTime` (UTC) — never `LocalDateTime`
- Enums in DB: `EnumType.STRING` — never `ORDINAL`
- Associations: always `FetchType.LAZY`

## Key Conventions (Web)

- Path alias: `@/` → `src/`
- TypeScript: strict mode, no `any`
- API client: `ky` in `src/api/`, configured with `X-User-Id` interceptor
- Route files: `{Domain}Page.tsx` in `src/pages/`

## CI/CD

Workflow: `.github/workflows/ci-backend.yml`
SonarCloud project key: `Pan14ek_lined`
Required secrets: `SONAR_TOKEN`, `COSMOS_DB_CONNECTION_STRING`

After any backend change, the pipeline runs Checkstyle + SpotBugs + JaCoCo + SonarCloud.
A fitness score (−1 to +1) is computed and stored in Azure Cosmos DB.

## Do Not

- Commit secrets, `.env` files, or credentials
- Change `FetchType.LAZY` to `EAGER` in JPA entities
- Use `LocalDateTime` for persisted timestamps in the backend
- Edit generated directories: `backend/lined/build/`, `lined-web/dist/`, `mobile/Lined/.expo/`, `fitness-metrics-collector/dist/`
- Modify the fitness function weights in `collectMetrics.ts` without updating AGENTS.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Pan14ek)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Pan14ek)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
