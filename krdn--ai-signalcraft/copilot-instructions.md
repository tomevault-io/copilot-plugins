## ai-signalcraft

> 공인 여론 자동 수집·AI 분석 파이프라인 + 웹 대시보드. 소규모 팀(3~10명)이 수동 트리거로 분석 실행.

## AI SignalCraft

공인 여론 자동 수집·AI 분석 파이프라인 + 웹 대시보드. 소규모 팀(3~10명)이 수동 트리거로 분석 실행.

## Architecture

```
apps/web            (Next.js 15 App Router + tRPC + shadcn/ui)
apps/collector      (독립 tRPC API 서비스 — raw_items 쿼리·RAG·임베딩 제공)
apps/whisper-worker (YouTube 음성 전사 워커 — Whisper 기반)
  → packages/core (BullMQ 파이프라인 + Drizzle ORM + AI 분석)
    → packages/collectors (Playwright + Cheerio 수집기 어댑터)
    → packages/ai-gateway (Vercel AI SDK v6 — Claude/GPT/Gemini)
```

**파이프라인 (키워드 단발)**: 수집(5개 소스) → 정규화 → AI 분석(Stage 0→1→2→3→4) → 리포트 생성
**파이프라인 (구독 경로)**: collector RAG → Stage 0→1→2→3→4 → Stage 5(Manipulation, 옵션) → 리포트 생성
**인프라**: PostgreSQL + Redis @ 192.168.0.5 | TimescaleDB(5435) = raw_items | Docker Compose 배포

## Stack

Next.js 15 · TypeScript 5 · React 19 · Drizzle ORM · PostgreSQL 16 · Redis 7
BullMQ 5 · Vercel AI SDK v6 · Zod 3 · tRPC 11 · shadcn/ui · Tailwind 4
Playwright · Cheerio · Recharts · TanStack Query 5 · NextAuth.js 5 · Vitest 3

상세: [docs/tech-stack.md](docs/tech-stack.md)

## AI 모델 설정

분석 모듈별 LLM 모델 추천 (최고/보통/최소 티어, 한국어 성능, 비용 비교):
**[docs/llm-model-recommendations.md](docs/llm-model-recommendations.md)**

- 현재 지원 프로바이더(Claude/Gemini/GPT/DeepSeek) + OpenRouter 확장 모델 포함
- 시나리오 프리셋(A~G) 평가 및 업그레이드 가이드
- 시나리오 G → DeepSeek V4 전환 시 캐시 히트 57% 절감 가능

## Conventions

- **패키지 매니저**: pnpm
- **린팅**: `pnpm lint` (ESLint 9 Flat Config) / `pnpm format` (Prettier)
- **의존성 방향**: web → core → collectors/ai-gateway (역방향 금지)
- **packages 내부 경로 import 금지** — public API(index.ts)만 사용
- **분석 모듈 추가**: `packages/core/src/analysis/modules/` — 기존 모듈 패턴 따를 것
- **수집기 추가**: `packages/collectors/src/adapters/` — `Collector` 인터페이스 구현
- **스크래퍼 오류**: CSS 셀렉터/URL 패턴 변경부터 확인

## Environment Separation (개발/운영 공유 인프라)

개발과 운영이 **같은 ais-prod-postgres(5438) / ais-prod-redis(6385)**를 공유하되 분리됨:

| 구분                           | DB                                           | Redis                      | BullMQ prefix    |
| ------------------------------ | -------------------------------------------- | -------------------------- | ---------------- |
| **운영** (`ais-prod-web:3300`) | `postgres:5432/ai_signalcraft` (Docker 내부) | `redis:6379` (Docker 내부) | `bull` (기본값)  |
| **개발** (`localhost:3000`)    | `192.168.0.5:5438/ai_signalcraft` (외부)     | `192.168.0.5:6385` (외부)  | `ais-dev` (자동) |

- `getBullPrefix()`: `NODE_ENV=production` → `bull`, 그 외 → `ais-dev` (자동 강제)
- 같은 Redis를 써도 큐가 완전히 분리되어 개발/운영 작업이 섞이지 않음
- 모든 `Queue`/`Worker`/`FlowProducer` 생성 시 `getBullMQOptions()` 사용 필수
- DB는 공유하므로 **개발에서 파괴적 SQL 주의** (DROP/TRUNCATE 등)

## Commands

```bash
pnpm dev               # 웹 개발 서버
pnpm dev:all           # 웹 + 워커 동시
pnpm dev:collector     # collector API + 워커 동시
pnpm dev:full          # collector + web 전체 동시
pnpm worker            # BullMQ 워커만
pnpm test              # 전체 테스트
pnpm lint              # ESLint
pnpm format            # Prettier
pnpm build             # 프로덕션 빌드
pnpm db:push           # Drizzle 스키마 동기화
pnpm --filter @ai-signalcraft/collector db:migrate-timescale  # 하이퍼테이블 UNIQUE 제약 적용 (db:push 후 반드시 실행)
pnpm db:studio         # Drizzle Studio
```

### Docker (운영 배포)

```bash
docker compose -f docker/docker-compose.prod.yml up -d --build   # 전체 빌드·배포
docker compose -f docker/docker-compose.prod.yml restart web      # web만 재시작
docker compose -f docker/docker-compose.prod.yml logs -f worker   # 워커 로그
```

## Gotchas

- **`ENCRYPTION_KEY` 운영 필수** — `.env.production`에 설정됨. provider_keys 복호화에 사용. 개발은 폴백 키 자동 사용. 키 변경 시 `scripts/migrate-encryption-key.mjs`로 기존 데이터 재암호화 필수
- **`db:push` 후 `db:migrate-timescale` 필수** — `pnpm --filter @ai-signalcraft/collector db:migrate-timescale`. hypertable에 UNIQUE 제약은 Drizzle이 관리 못 함. 누락 시 ON CONFLICT 에러로 수집 전체 실패
- **`raw_items` 피드 쿼리는 `scope='feed'` 강제** — item_type 필터 없이 ORDER BY time DESC 하면 댓글이 기사를 10:1로 밀어냄
- **collector tRPC schema 먼저 확인** — `items.query.ragOptions.topK` 등 Zod 제약은 분석 측 변경 전 `apps/collector/src/server/trpc/items.ts` 읽기 필수
- **kit(llm-gateway) 수정 금지** — `buildSystemPrompt`를 인자 없이 호출함. domain 등 컨텍스트 주입은 wrapper closure로 바인딩
- **수집 실패 역추적** — `@ais:sub/.../run/<uuid>` 에러 수신 시 timescaledb(5435 `ais_collection`) `collection_runs` 먼저 조회

## Debugging

- 근본 원인(root cause) 먼저 파악. DB → API → Worker → Frontend 전체 경로 추적
- SQL 변경 시 ambiguous column reference 확인
- UI 콜백 변경 후 전체 사용자 플로우 테스트
- 진행 중 run 중지·진단: 모니터 페이지(`/subscriptions/monitor`) 각 row의 [중지]/[진단] 모달 사용. DB 수동 조작 금지 — `run_cancellations` 테이블이 cooperative cancel 플래그의 단일 진실 (executor checkpoint가 이 값을 폴링)

---
> Source: [krdn/ai-signalcraft](https://github.com/krdn/ai-signalcraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
