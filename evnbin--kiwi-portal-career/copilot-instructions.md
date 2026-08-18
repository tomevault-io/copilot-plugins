## kiwi-portal-career

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

프로젝트의 모든 규칙·제약은 @RULES.md 에 정리되어 있다. 이 import 구문으로 RULES.md가 컨텍스트에 자동 로드된다.

## 프로젝트 성격 (ent MVP)

`kiwi-portal-ent`는 원본 `kiwi-portal`에서 **모든 기능 도메인을 제거한 MVP 뼈대**다(이식 계획: `docs/mvp-migration-plan.md`). 인프라·인증·Canvas 연동만 남기고, 기능은 이 위에 새로 얹는다.

- **유지**: 모노레포 인프라, API 파이프라인(캐시·throttle·미들웨어·인터셉터·필터·audit·scheduler·drizzle 코어), auth(Keycloak OIDC + JIT 프로비저닝), `domain/user`, `external/canvas`, web 셸(layout·i18n·auth/signin·공용 컴포넌트).
- **제거됨**: course·calendar·learning-journal·learning-mate·meeting-hub·notice·notification·personal-group·popup·todo·comment·management·file 도메인, panopto·zoom·teams·slack·live-meeting 외부연동, 대응 web 라우트 전부.
- **Canvas Career 재타겟**: 기존 Canvas LMS 학습 메서드는 제거하고 유저 신원·프로비저닝만 유지. Career는 Canvas의 experience 모드(같은 `/api/v1`)로, provider는 account role, learner는 enrollment 기반(상세 §docs/mvp-migration-plan.md §4).
- 현재 `domain/`에는 `user`만, `external/`에는 `canvas`만 존재한다.

## 빌드 · 실행 · 테스트

스크립트 전체 목록은 root `package.json` 참조. 주요 명령:

```bash
pnpm dev          # web:3000 + api:4000
pnpm dev:web      # 프론트엔드만 (Turbopack)
pnpm dev:api      # 백엔드만 (debug + watch)
pnpm loc          # APP_ENV=local (TestModule 활성화)
pnpm build        # 프로덕션 빌드
pnpm lint && pnpm typecheck && pnpm format  # 검증 (Prettier: printWidth 150, singleQuote)

# 테스트 (개별 패키지)
pnpm --filter @kiwi/api test          # Jest --runInBand
pnpm --filter @kiwi/web test          # Vitest
pnpm --filter @kiwi/api test:e2e      # Supertest

# 개별 파일
cd apps/api && pnpm jest src/external/canvas/canvas.service.spec.ts --runInBand
cd apps/web && pnpm vitest run src/hooks/useDebounce.test.ts

# Drizzle
pnpm --filter @kiwi/api db:generate   # 마이그레이션 SQL 생성
pnpm --filter @kiwi/api db:migrate    # 마이그레이션 실행
```

## 모노레포 구조

pnpm 10 + Turborepo 2. TypeScript 6, Zod 4.

| 패키지 | 설명 |
|---|---|
| `apps/web` | Next.js 16 App Router, React 19, Tailwind 4, Shadcn/Radix UI |
| `apps/api` | NestJS 11 on Fastify 5, Drizzle ORM, PostgreSQL, Redis |
| `packages/shared` | 양쪽에서 쓰는 타입·enum·Zod 스키마 (`@kiwi/shared`, `@kiwi/shared/types/*`, `@kiwi/shared/enums/*`, `@kiwi/shared/schemas/*`) |
| `packages/config` | 공유 ESLint·TypeScript 설정 |
| `tests/load` | k6 부하 테스트 |

> **Web 커스텀 브레이크포인트**: `globals.css`의 `@theme`에서 `mini` 360, `tablet` 860, `lg` 1024, `desktop` 1440, `wide` 1704를 Tailwind 기본(`sm` 640, `md` 768, `lg` 1024, `xl` 1280, `2xl` 1536)에 **추가(extend)** 한다. 기본을 제거하지 않으므로 기본·커스텀 모두 사용 가능(`lg`는 양쪽 동일 값).

## ⚠️ 외부 연동을 건드리기 전에 — golden note 먼저 읽어라

**Canvas · Journey · Career** 관련 작업(코드 수정·디버깅·"왜 안 되지" 판단) 전에 **반드시** 먼저 읽는다:

```
~/golden-brain/notes/apps/kiwi-portal-ent/career-journey-api.md   # Canvas/Journey/Career 실측
~/golden-brain/notes/apps/kiwi-portal-ent/auth-clients.md         # Keycloak 클라이언트 구성
```

이 노트들은 **라이브 실측**이다. 여기 있는 걸 코드만 보고 추측하면 틀린다. 실제로 반복된 사고:

- Canvas는 **User-Agent 없는 요청을 403**으로 막는다 → 멀쩡한 토큰을 "폐기됐다"고 오진
- `/api/v1/jwts`는 `canvas_audience=false` 없이도 **200**을 주지만 Journey가 거부하는 JWE다
- Journey는 **인증 실패를 HTTP 200 + `errors[]`**로 준다 → status만 보면 조용히 통과
- 팀 위젯(`learning_status_details`)은 **Journey가 아는 사용자만** 담는다 → 갓 만든 계정은 안 보임

**규칙**: 외부 API 동작을 단정하기 전에 노트를 확인하고, 노트에 없으면 **실측한 뒤 노트에 올린다**(팀 규칙: `~/golden-brain/docs/team-rules.md`).

## 아키텍처 핵심

### 인증 (Keycloak OIDC)
Keycloak OIDC/JWKS(RS256) + PKCE. realm role → 내부 RoleCode 매핑.

인증 흐름: `BearerTokenMiddleware`(토큰 추출·블랙리스트·L1/L2 캐시) → `LxpAuthGuard`(DB 사용자+역할 조회, `request.currentUser` 설정) → `PermissionGuard`(@Permission OR 로직)

### 캐시 아키텍처
L1(인프로세스 LRU, 1000항목/30s) → L2(Redis) → DB. `@Cacheable(prefix, ttl)` 데코레이터.
Redis Pub/Sub(`cache:invalidate`)로 멀티 인스턴스 L1 동기화.

## API 핵심 패턴

### 모듈 구조
Module → Controller → Service. 도메인: `apps/api/src/domain/`, 외부 연동: `apps/api/src/external/`.

### 전역 파이프라인 (실행 순서)
1. **Middleware**: `ConcurrencyLimiterMiddleware`(동시 요청 150, IP당 50) → `RequestContextMiddleware`(AsyncLocalStorage requestId) → `BearerTokenMiddleware`(토큰 추출·캐시)
2. **Guard**: `ThrottlerBehindProxyGuard`(전역 100 req/60s, IP 기준; local 바이패스, 라우트별 `@Throttle` 오버라이드) → `LxpAuthGuard`(인증) → `PermissionGuard`(인가)
3. **Interceptor**: `MetricsInterceptor`(Prometheus) → `HttpLogInterceptor` → `ResponseInterceptor`(응답 래핑 해제) → `ETagInterceptor`
4. **Filter**: `GlobalExceptionFilter`(RFC 7807)

## CI/CD

> 브랜치 역할·승급 규칙(feature → `dev` → `stg`/`main`)은 RULES.md "Git · 배포" 참조.

### 배포 트리거
| 브랜치 | 트리거 | 배포 워크플로 |
|---|---|---|
| `dev` | push | `cd-dev.yml` |
| `stg` | push | `cd-stg.yml` |
| `main` | 수동 | `release-prod.yml` |
> stg 자동 배포는 외부 GitOps 저장소에 `apps/<app>/stg/values.yaml`과 k8s stg 환경이 준비돼야 실제 동작한다. 그 전까지 `cd-stg.yml`은 GitOps 단계에서 의도적으로 실패한다.

- CI(`ci.yml`): `dev`·`stg`·`main` 대상 PR → Prettier → lint → typecheck → test → build (`--affected`만, Draft PR 스킵)
- CD-Dev(`cd-dev.yml`) / CD-Staging(`cd-stg.yml`): Docker matrix 빌드 (API+Web 병렬) → GitOps values.yaml 갱신
- Docker: multi-stage, `node:24-alpine`, standalone, tini, non-root(UID 1001)
- Release-Prod: dev 이미지를 **digest 복사로 프로모션** (재빌드 없음), semantic versioning

---
> Source: [evnbin/kiwi-portal-career](https://github.com/evnbin/kiwi-portal-career) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-18 -->
