## astroquiz

> AstroQuiz is a full-stack astronomy quiz app: **Strapi 5 backend** (TypeScript) + **React Native mobile app**. Deployed on Railway with PostgreSQL, Cloudinary, Firebase Auth, and DeepL translation.

# CLAUDE.md - AstroQuiz

## Project Overview

AstroQuiz is a full-stack astronomy quiz app: **Strapi 5 backend** (TypeScript) + **React Native mobile app**. Deployed on Railway with PostgreSQL, Cloudinary, Firebase Auth, and DeepL translation.

## Quick Reference

```bash
# Backend
npm run develop          # Dev server with auto-reload (port 1337)
npm run build            # Build Strapi
npm run start            # Production start
npm run health           # Health check (curl localhost:1337/api/health)

# Tests
npm test                 # All tests
npm run test:unit        # Unit tests (src/)
npm run test:api         # API integration tests
npm run test:services    # Service tests
npm run test:routes      # Route tests
npm run test:middlewares # Middleware tests
npm run test:coverage    # With coverage report

# Mobile app (from AstroQuizApp/)
npx react-native start   # Metro bundler
npx react-native run-ios  # iOS
npx react-native run-android # Android
```

**Node version**: 22 LTS (see `.nvmrc`)

## Architecture

```
/                          # Strapi backend (root)
  src/
    index.ts               # Bootstrap: Firebase init, route registration, rate limiting
    routes/                # Custom API routes (quiz, questions, user-profile, i18n, debug)
    services/              # Business logic (quiz-session, quiz-logic, firebase-auth, validation)
    middlewares/            # Auth (Firebase), rate-limit, cache, performance-monitor
    api/question/           # Strapi content-type schema
    plugins/deepl-translation/ # Custom Strapi plugin for DeepL
  config/
    database.ts            # DB config (SQLite dev, PostgreSQL prod)
    game-rules.js          # Scoring, phases, difficulty progression
  scripts/                 # Integration & performance tests
  docs/                    # API docs, deployment guides, troubleshooting

AstroQuizApp/              # React Native mobile app
  src/
    App.tsx                # Entry point (Sentry, Firebase, i18n, AppContext)
    screens/               # HomeScreen, QuizScreen, QuizResultScreen, LoginScreen, etc.
    services/              # api.ts, authService, quizService, analyticsService
    contexts/AppContext.tsx # Global state (user, quiz session, rules, locale)
    navigation/            # React Navigation setup
    components/            # Reusable UI components
    i18n/                  # i18next translations (pt, en, es, fr)
    config/                # theme.ts, sentry.ts
```

## Key Technical Details

### Database
- **Dev**: SQLite (`.tmp/data.db`)
- **Prod**: PostgreSQL via `DATABASE_URL` (Railway)
- Custom routes use **Knex.js** directly (not Strapi entity service)
- Connection pool: 2-10 connections

### Authentication
- **Firebase Admin SDK** for token verification
- Three middleware levels: `createAuthMiddleware()` (required), `createOptionalAuthMiddleware()` (guest ok), `createAdminMiddleware()` (admin role)
- Quiz play works without auth (guest mode)

### API Routes (all under `/api/`)
- `quiz/*` - Session-based gameplay (start, question, answer, finish, pause, resume)
- `questions/*` - CRUD + bulk import + i18n import
- `user-profile/*` - Profile management with Firebase UID
- `i18n-setup/*` - Language configuration
- `debug/*` - Dev-only tools

### API Response Format
```typescript
{ success: boolean, message?: string, data: any, error?: { status, name, message } }
```

### Game Mechanics
- 50 phases, 10 questions each, 30s per question
- Difficulty scales: Phase 1-3 = Level 1 only, Phase 46-50 = Level 5 only
- Scoring: base points (10-50 by level) x speed multiplier (1.0-2.0x) + streak bonus
- Sessions expire after 6 hours
- Locales: en, pt, es, fr

### Testing
- **Jest 29** + **ts-jest** + **Supertest**
- Test files: `src/**/*.test.ts`, `src/**/__tests__/**`, `scripts/**/*.test.js`
- Coverage collected from `src/**/*.ts`

### Deployment
- **Railway** with Nixpacks (no Dockerfile)
- Build: `npm run railway:build` / Start: `npm run railway:start`
- Health check: `GET /api/quiz/health`

## Code Conventions

- **Backend**: TypeScript (gradual migration from JS). Kebab-case files (`quiz-routes.ts`). Factory functions for middleware (`createAuthMiddleware()`). Async/await everywhere.
- **Frontend**: TypeScript. PascalCase components (`QuizScreen.tsx`). Functional components + hooks. Context API for state. React Native StyleSheet (no CSS libs).
- **Naming**: Services end in `-service.ts`, routes in `-routes.ts`, tests in `.test.ts` or `__tests__/` dirs.

## Security Rules

**Nunca introduzir vulnerabilidades de seguranca criticas ou altas.** Ao escrever ou modificar codigo, verificar:

- **Sem segredos expostos**: nunca logar, retornar em responses ou hardcodar tokens, senhas, chaves API ou credenciais. Debug endpoints so existem em dev (`NODE_ENV !== 'production'`).
- **Sem injection**: sempre usar queries parametrizadas (Knex query builder). Nunca interpolar input do usuario em SQL, comandos shell ou HTML.
- **Sem vazamento de erros**: catch blocks devem retornar mensagens genericas ao cliente (`'Internal server error'`). Detalhes vao para `strapi.log.error()`, nunca para `ctx.throw(500, error.message)`.
- **IDs e tokens seguros**: usar `crypto.randomBytes()` para gerar IDs. Nunca `Math.random()`. Comparar tokens com `crypto.timingSafeEqual()`, nunca com `===`.
- **Validar input**: validar e sanitizar toda entrada do usuario nas bordas da API. Usar as funcoes de `src/services/validation.ts`.
- **Rate limiting**: todo endpoint publico deve estar coberto pelo rate limiter. Nao confiar em headers `X-Forwarded-For` do cliente — usar `ctx.request.ip`.
- **Auth por padrao**: novos endpoints devem exigir autenticacao. Usar `auth: false` apenas quando explicitamente necessario (health, rules, quiz play).
- **Dependencias**: manter `npm audit` com 0 vulnerabilidades criticas. Usar `overrides` no package.json para forcar versoes seguras em dependencias transitivas quando necessario.

## Environment Variables

See `env.example` for full list. Key ones:
- `DATABASE_CLIENT` (sqlite/postgres), `DATABASE_URL` (prod)
- `APP_KEYS`, `API_TOKEN_SALT`, `ADMIN_JWT_SECRET`, `JWT_SECRET` (security)
- `DEEPL_API_KEY`, `FIREBASE_PROJECT_ID`, `STRAPI_WRITE_TOKEN` (integrations)
- `CLOUDINARY_NAME/KEY/SECRET` (file uploads)

---
> Source: [roberto-muricy/AstroQuiz](https://github.com/roberto-muricy/AstroQuiz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
