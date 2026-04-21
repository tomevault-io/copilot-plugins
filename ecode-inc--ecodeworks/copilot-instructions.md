## ecodeworks

> 통합 사내 솔루션 (일정관리 + 문서화 + 칸반 + 비밀번호 금고 + QA 연동)

# ecode-internal

통합 사내 솔루션 (일정관리 + 문서화 + 칸반 + 비밀번호 금고 + QA 연동)

## 작업 규칙

### 팀 에이전트 기반 작업 프로세스

모든 작업은 반드시 **팀 에이전트 기능**으로 각각의 파트를 나눠서 병렬로 작업한다.
각 에이전트가 작업을 완료한 후, **리뷰어 에이전트가 웹 검색을 통해 검증**한 뒤 최종 반영한다.

#### 에이전트 역할 분담

| 에이전트 | 역할 | 담당 영역 |
|----------|------|-----------|
| **백엔드 에이전트** | API 구현 | Hono 라우트, D1 쿼리, 미들웨어, 마이그레이션 |
| **프론트엔드 에이전트** | UI 구현 | React 컴포넌트, Zustand 스토어, Tailwind 스타일링 |
| **리뷰어 에이전트** | 검증/QA | 웹 검색으로 최신 문서/패턴 대조, 코드 리뷰, 타입 체크 |
| **문서화 에이전트** | 문서 관리 | CLAUDE.md 업데이트, API 문서, 가이드 문서 작성 |

#### 작업 프로세스

1. 요청 접수 → 작업을 에이전트별로 분배
2. **백엔드 + 프론트엔드 에이전트가 병렬로** 구현
3. **리뷰어 에이전트**가 결과를 검증 (타입 체크, 웹 검색 기반 패턴 검증)
4. 리뷰어 승인 → 배포
5. **문서화 에이전트**가 변경사항 문서에 반영

- 리뷰어가 확인하지 않은 코드는 머지하지 않는다
- 배포 전 반드시 `npx tsc --noEmit` 통과 확인

#### 문서화 에이전트 필수 규칙

**API 엔드포인트가 추가/변경될 때마다** 반드시 다음 파일을 업데이트해야 한다:

1. **`packages/web/src/components/ai/AIGuidePage.tsx`** — AI 가이드 페이지
   - `getActions` 배열: GET Action 엔드포인트 (web_fetch 호환)
   - `getReadEndpoints` 배열: GET 조회 엔드포인트
   - `postEndpoints` 배열: POST/PATCH 엔드포인트
   - `mcpTools` 배열: MCP 도구 목록
   - 이 페이지는 `https://work.e-code.kr/?key=API_KEY`로 접근되며, 오픈클로(AI봇)가 참조하는 유일한 문서이다
2. **`packages/api/src/routes/mcp.ts`** — MCP 도구 정의 (`TOOLS` 배열 + switch case)
3. **`CLAUDE.md`** — 이 파일의 AI API 섹션

**이 가이드가 최신이 아니면 오픈클로가 새 기능을 사용할 수 없다.**

### 기술 스택

- **Backend**: Cloudflare Workers + Hono + D1 (SQLite) + R2
- **Realtime**: Durable Objects + WebSocket
- **Frontend**: React 18 + Vite + Tailwind CSS + Zustand
- **Desktop**: Tauri v2
- **Mobile/Web**: PWA

### 개발 명령어

```bash
pnpm dev          # API + Web 동시 실행
pnpm dev:api      # API만 (wrangler dev, :8787)
pnpm dev:web      # Web만 (vite, :3000)
pnpm dev:landing  # Landing만 (vite, :3001)
pnpm build        # 빌드
pnpm db:migrate:local  # 로컬 DB 마이그레이션
```

### 배포 명령어

```bash
# API 배포
cd packages/api && npx wrangler deploy

# Web 빌드 + 배포 (packages/web 디렉토리에서 실행)
cd packages/web
npx vite build
CLOUDFLARE_ACCOUNT_ID=eb202b6b79a065f953071ee9b2bc6865 npx wrangler pages deploy dist --project-name=ecode-internal --branch=master --commit-dirty=true --commit-message=deploy

# ⚠️ 중요: VITE_API_URL은 packages/web/.env.production 파일에 설정됨
# 빌드 시 별도로 환경변수를 전달하지 않아도 됨
# .env.production 파일: VITE_API_URL=https://ecode-internal-api.aws-eb2.workers.dev/api
# 이 파일이 없으면 API 호출이 /api 로 가서 Cloudflare Pages에서 405 에러 발생!

# Landing 빌드 + 배포 (packages/landing 디렉토리에서 실행)
cd packages/landing
npx vite build
CLOUDFLARE_ACCOUNT_ID=eb202b6b79a065f953071ee9b2bc6865 npx wrangler pages deploy dist --project-name=ecode-landing --branch=master --commit-dirty=true --commit-message=deploy
# .env.production 파일: VITE_API_URL=https://ecode-internal-api.aws-eb2.workers.dev

# D1 마이그레이션 (원격)
cd packages/api && npx wrangler d1 execute ecode-db --remote --file=./migrations/XXXX.sql
```

### 프로젝트 구조

- `packages/api/` - Cloudflare Workers API (Hono)
- `packages/web/` - React 프론트엔드
- `packages/landing/` - 랜딩 페이지 (ecode-landing-8gh.pages.dev, 커스텀 도메인 예정)
- `packages/desktop/` - Tauri v2 래퍼 (예정)

### 보안 주의사항

- JWT Secret, VAULT_KEY 등은 반드시 `wrangler secret`으로 관리
- 비밀번호 금고는 AES-256-GCM으로 암호화, VAULT_KEY는 64자 hex string
- 모든 금고 접근은 감사 로그에 기록됨

### Cloudflare 계정 관리

Wrangler 토큰 위치: `C:\Users\ionar\AppData\Roaming\xdg.config\.wrangler\config\`

| 파일 | 계정 |
|------|------|
| `default.toml` | 현재 활성 계정 |
| `aws-eb2.toml` | aws@e-code.kr (운영 계정) |

전환: `cp .../config/aws-eb2.toml .../config/default.toml`

### 계정 정보

**CEO 계정**: `ecode@e-code.kr` / `ecode2026!` (조직 슬러그: `ecode`)
**슈퍼어드민**: `super@e-code.kr` / `ecode2026!` (경로: `/super`)
**도메인**: `work.e-code.kr` → ecode-internal.pages.dev

### Super Admin (SaaS 관리)

플랫폼 레벨의 관리자 시스템. 조직(org) 관리, 구독 플랜, 감사 로그를 제공한다.

- **DB**: `super_admins`, `org_subscriptions`, `platform_audit_log` 테이블
- **API**: `/api/super/*` 라우트
- **Frontend**: `/super` 경로로 접근, 별도 로그인
- **구독 플랜**: free(5명), starter, business, enterprise(무제한)

### AI API

- **경로**: `/api/v1/*` (API 키 인증)
- **안전 제한**: DELETE 차단, 금고 비밀번호 미노출, 사용자/조직 수정 차단
- **가이드**: `https://work.e-code.kr/?key=API_KEY`로 접근 시 가이드 페이지 표시
- **스코프**: calendar, kanban, docs, vault(메타만), members, departments, telegram

### 징계 관리

- **DB**: `disciplines` 테이블
- **API**: `/api/v1/action/create-discipline`, `/api/v1/action/list-disciplines` (API 키 인증)
- **유형**: 감봉, 연차삭감, 대표면담, 반성문
- **MCP**: `list_disciplines`, `create_discipline`
- **스코프**: members:read (조회), members:write (등록)

### 텔레그램 연동

- **DB**: `telegram_chats`, `telegram_user_mappings`, `telegram_command_log`
- **API**: `/api/telegram/*` (JWT 인증), `/api/v1/telegram/*` (API 키 인증)
- **기능**: 채팅방 등록, 텔레그램↔이코드 사용자 매핑, 명령 히스토리

### 법인계좌 (오픈뱅킹)

- **DB**: `banking_connections`, `banking_oauth_states`
- **API**: `/api/banking/*` (JWT 인증, CEO/관리자 전용)
- **금융결제원**: 테스트 환경 (`testapi.openbanking.or.kr`), 프로덕션 심사 신청 중
- **적용 API**: 잔액조회, 잔액조회(법인), 거래내역조회, 거래내역조회(법인)
- **보안**: 접근토큰·갱신토큰 AES-256-GCM 암호화 저장, 만료 시 자동 갱신
- **OAuth state**: 32자 hex 랜덤 생성 → `banking_oauth_states` 테이블에 user/org 매핑 저장

### 메일 시스템 (예정)

- **상태**: 보류 — 설계 완료, 구현 미착수
- **구조**: 수신 Cloudflare Email Routing + Workers → D1/R2, 발신 AWS SES
- **멀티테넌트**: 고객사별 커스텀 도메인 지원 (AWS SES API로 도메인 자동 등록/검증)
- **첨부파일**: 수신 시 MIME 파싱 → R2 저장, 발신 시 SendRawEmail MIME 멀티파트
- **제한**: SES 메일당 최대 40MB, 초과 시 R2 다운로드 링크 삽입

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ecode-inc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
