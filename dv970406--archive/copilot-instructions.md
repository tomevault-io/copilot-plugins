## archive

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

MDX 기반 개인 기술 블로그 플랫폼. Next.js 16 App Router + Supabase + Claude AI 자동 요약.

## 개발 명령어

```bash
pnpm run dev              # 개발 서버 (http://localhost:3000)
pnpm run build            # 프로덕션 빌드
pnpm run biome            # Biome lint + format (자동 수정)
pnpm run supabase:type-gen  # Supabase DB 타입 재생성 → src/types/database.types.ts
pnpm run supabase:db-sync   # 원격 DB → 로컬 DB 동기화
```

## 코드 스타일 (Biome)

- **인덴트**: 탭 사용 (스페이스 아님)
- **따옴표**: 더블 쿼트 (`"`)
- **console 사용 금지**: `noConsole: "error"` — `console.log` 등 사용 시 lint 에러
- **import 자동 정렬**: `organizeImports: "on"`
- **pre-commit**: Husky + lint-staged가 staged 파일에 `biome check --write` 자동 실행

## 경로 별칭

`@/` → `src/` (예: `import { cn } from "@/lib/style/tailwind"`)

## 아키텍처

### 라우트 그룹 (App Router)

- `app/(with-header)/` — 공개 페이지 (헤더 포함: 피드, 글 상세, 카테고리)
- `app/(with-auth)/` — 인증 필요 페이지 (글 작성/수정, 클라이언트 auth 가드)
- `app/(with-no-auth)/` — 비인증 페이지 (로그인)
- `app/api/` — Route Handlers (AI 요약 등)

### 데이터 페칭 계층

1. **`src/api/`** — Supabase 쿼리 순수 함수 + `unstable_cache` 래핑 (ISR용)
2. **`src/hooks/queries/`** — TanStack Query `useQuery`/`useInfiniteQuery` 훅 (클라이언트)
3. **`src/hooks/mutations/`** — TanStack Query `useMutation` 훅
4. **서버→클라이언트 hydration**: `prefetchQuery` + `HydrationBoundary` + `dehydrate` 패턴

**주의**: `unstable_cache`로 래핑된 함수는 TanStack Query의 `queryFn`에 직접 전달 불가. 별도 함수로 감싸야 함.

### 상태 관리

- **Zustand**: 클라이언트 전역 상태 (`src/store/`)
  - `usePostStore`: 포스트 작성/수정 임시 상태
  - `useAlertModalStore`: 알림 모달 상태
  - 패턴: `actions`를 store 내부에 모으고, 선택자 훅을 개별 export

### 스타일링

- **Tailwind CSS v4** + CSS 변수 기반 테마 (oklch 컬러)
- **Shadcn UI** (New York 스타일) — `src/components/ui/`
- **`cn()` 유틸리티**: `clsx` + `tailwind-merge` (`@/lib/style/tailwind`)
- **다크/라이트 테마**: `next-themes`, 기본값 `dark`

### Query Key 관리

`src/lib/query-keys.ts`에 모든 TanStack Query 키 상수가 정의되어 있음 (`QUERY_KEYS`).

### MDX 렌더링

- `next-mdx-remote-client` 사용
- 커스텀 컴포넌트: `Callout`, `ImageGallery` — `src/components/mdx/`

## 환경변수

```env
NEXT_PUBLIC_SUPABASE_API_URL=
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_DEFAULT_KEY=
NEXT_PUBLIC_SUPABASE_STORAGE_URL=
NEXT_PUBLIC_SITE_URL=
ANTHROPIC_API_KEY=          # 서버 전용
```

## 주요 패턴

- **이미지 업로드**: Canvas API로 WebP 변환 후 Supabase Storage(`post-uploads` 버킷)에 저장
- **AI 요약**: 클라이언트 → `/api/ai/summarize` Route Handler → Anthropic SDK (`claude-sonnet-4-5-20250929`)
- **인증**: Supabase Auth (이메일/패스워드), 미들웨어 없이 클라이언트 레이아웃에서 auth 가드
- **캐싱**: 포스트 `revalidate: 300` (5분), 카테고리 `revalidate: false` (무기한)
- **포스트 상태**: `DRAFT` / `PUBLISHED`

---
> Source: [dv970406/archive](https://github.com/dv970406/archive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-11 -->
