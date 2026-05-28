## nextjs-supabase-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

**GatherEase** — 소규모 그룹 모임 주최자가 참여 관리·공지·정산을 한 곳에서 처리하는 웹 서비스.  
로그인 없이 localStorage 토큰(host_token, participant_token)으로 권한을 식별하는 것이 핵심 설계 원칙이다.

현재 단계: **Phase 0 완료** (Next.js 15 + Supabase 기본 설정, 이메일/Google 인증, 프로필 기능)  
다음 목표: **Phase 1** — GatherEase DB 스키마 마이그레이션 (`docs/ROADMAP.md` 참고)

## 명령어

```bash
npm run dev           # 개발 서버 (localhost:3000)
npm run build         # 프로덕션 빌드
npm run lint          # ESLint 검사
npm run format        # Prettier 포맷
npm run type-check    # npx tsc --noEmit
npm run validate      # lint + type-check 동시 실행
npx shadcn@latest add [component]   # shadcn/ui 컴포넌트 추가
```

작업 완료 전 항상 `npm run validate && npm run build` 실행.

## 환경 변수

`.env.local` 파일에 설정 (`.env.example` 참고):

```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=
```

## 아키텍처

### 라우팅 구조

```
app/
├── page.tsx                    # 랜딩 (공개)
├── auth/                       # 인증 관련 (공개)
│   ├── login/ sign-up/ ...     # 인증 페이지
│   ├── confirm/route.ts        # 이메일 인증 콜백
│   └── callback/route.ts       # OAuth 콜백 (Google)
└── protected/                  # 인증 필요
    ├── layout.tsx              # getClaims()로 세션 보호 + 네비게이션
    ├── page.tsx                # 대시보드
    └── profile/                # 프로필 조회/편집
```

GatherEase 기능 페이지는 `app/event/` 아래에 추가 예정:

- `/event/new` — 모임 생성
- `/event/[id]` — 모임 홈 (인증 불필요, localStorage 토큰으로 권한 구분)
- `/event/[id]/join`, `/event/[id]/notices`, `/event/[id]/settlement`

### Supabase 클라이언트 패턴

- `lib/supabase/server.ts` — Server Component/Route Handler용. **매 요청마다 새로 생성** (전역 변수 금지)
- `lib/supabase/client.ts` — Client Component용 (`createBrowserClient`)
- `lib/supabase/proxy.ts` — 미들웨어: 모든 요청에서 세션 갱신, 미인증 시 `/auth/login` 리다이렉트
- `lib/supabase/types.ts` — Supabase CLI 자동 생성 타입 (`Profile`, `ProfileInsert`, `ProfileUpdate` alias 포함)
- `lib/supabase/profiles.ts` — 프로필 CRUD 서버 함수

세션 확인은 `supabase.auth.getClaims()` 사용 (`getUser()` 아님).

### DB 스키마

**현재 구현된 테이블**:

- `public.profiles` — auth.users와 1:1 연결, RLS 활성화, 신규 유저 생성 트리거로 자동 생성

**Phase 1에서 추가할 테이블** (PRD 데이터 모델 참고):

- `events` — host_token, invite_code(6자리) 포함
- `participants` — status: attending/undecided/absent/waiting
- `notices`, `comments`, `expense_items`, `payment_status`

스키마 변경: `mcp__supabase__apply_migration` → 변경 후 `mcp__supabase__generate_typescript_types`로 `lib/supabase/types.ts` 재생성.  
마이그레이션 명 형식: `동사_대상` (예: `create_events_table`)

### localStorage 토큰 설계

로그인 없이 주최자/참여자를 식별하는 핵심 메커니즘:

- `gg_host_{event_id}` — 모임 생성 시 UUID v4 생성, DB의 `events.host_token`과 비교
- `gg_participant_{event_id}` — 참여 등록 시 UUID v4 생성, DB의 `participants.participant_token`과 비교

### 컴포넌트 구조

- `components/ui/` — shadcn/ui 순수 UI (비즈니스 로직 없음)
- `components/profile/` — `profile-card.tsx`(Server), `profile-edit-form.tsx`(Client, React Hook Form)
- `components/auth-button.tsx` — Server Component (getClaims로 세션 확인 후 렌더링)

## 코딩 컨벤션

- `any` 타입 사용 금지
- 파일명: kebab-case / 컴포넌트명: PascalCase
- import는 항상 `@/` 경로 별칭 사용 (상대 경로 금지)
- Server Component 기본, 상태/이벤트/브라우저 API 필요 시에만 `"use client"`
- 폼: React Hook Form + Zod 스키마 검증 필수
- 스타일: Tailwind CSS, 반응형 필수 (모바일 퍼스트)

## 개발 워크플로우

1. `docs/ROADMAP.md`에서 현재 Phase와 다음 태스크 확인
2. DB 스키마 변경 시 Supabase MCP로 마이그레이션 적용 후 타입 재생성
3. 기능 구현 후 Playwright MCP로 브라우저 테스트 수행
4. 커밋 전 `npm run validate && npm run build` 확인 (Husky가 lint-staged 자동 실행)

---
> Source: [wodud6423/nextjs-supabase-app](https://github.com/wodud6423/nextjs-supabase-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
