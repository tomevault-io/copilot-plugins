## cc101

> Next.js 16 (App Router) 기반의 Claude Code 한국어/영어 입문 가이드 사이트.

# CC101 — Claude Code Korean Beginner Guide

## Project Overview

Next.js 16 (App Router) 기반의 Claude Code 한국어/영어 입문 가이드 사이트.
콘텐츠는 마크다운 파일 + `sections.json`으로 관리되며, 코드 수정 없이 콘텐츠 추가/수정 가능.

- **URL**: https://cc101.axwith.com
- **Routing**: `/ko`, `/en` (path-based i18n, SSG)

## Tech Stack

- Next.js 16.1.6 (App Router, Turbopack, RSC)
- React 19, TypeScript 5 (strict)
- Tailwind CSS v4 + `@tailwindcss/typography`
- MDX: `next-mdx-remote/rsc` + `remark-gfm`
- Content parsing: `gray-matter`
- Package manager: pnpm
- Analytics: GA4 (`G-DXL98QN89J`)

## Commands

```bash
pnpm dev        # 개발 서버 (Turbopack)
pnpm build      # 프로덕션 빌드
pnpm start      # 프로덕션 서버
pnpm lint       # ESLint
npx tsc --noEmit  # 타입 체크
```

테스트 프레임워크는 설정되어 있지 않음. `lint` + `tsc --noEmit`이 유일한 품질 체크.

## Project Structure

```
content/
├── sections.json          # 전체 섹션 메타데이터 (single source of truth)
├── ko/                    # 한국어 마크다운 (21개)
└── en/                    # 영어 마크다운 (21개, ko/ 미러)
src/
├── app/
│   ├── layout.tsx         # Root layout: 폰트, GA4, 글로벌 메타데이터
│   ├── page.tsx           # / → /ko 리다이렉트
│   ├── globals.css        # Tailwind v4 import + 다크모드 + prose 오버라이드
│   └── [lang]/
│       ├── layout.tsx     # LangSetter 렌더링
│       ├── page.tsx       # 서버 컴포넌트: 섹션 로딩 + MDX 렌더링
│       └── LangSetter.tsx # document.documentElement.lang 동기화
├── components/            # UI 컴포넌트 (모두 named export)
├── lib/
│   ├── content.ts         # 서버: sections.json + 마크다운 파일 읽기
│   └── analytics.ts       # GA4 이벤트 래퍼
├── hooks/
│   └── useLang.ts         # 클라이언트 lang 상태
└── types/
    └── index.ts           # Lang, SectionMeta, SectionsConfig 등
```

## Architecture

### Content-Driven Rendering

```
sections.json → lib/content.ts → [lang]/page.tsx → SectionBlock + MDXRemote
```

- `sections.json`: 21개 섹션의 id, order, tier, slug, 제목, 설명, 파일 경로 관리
- `lib/content.ts`: 빌드 타임에 JSON + 마크다운 파일을 읽어 정렬된 배열 반환
- 영문 파일 없으면 한국어로 fallback

### Routing

- `/ko`, `/en`: `generateStaticParams`로 SSG
- 지원하지 않는 언어 경로 → `notFound()` (404)
- 언어 전환: `router.push('/{lang}', { scroll: false })`

### Styling

- Tailwind v4: `@import "tailwindcss"` (v3의 `@tailwind` 아님)
- 다크모드: `html.dark` 클래스 기반, 기본값 dark, localStorage 저장
- 다크 variant: `@variant dark (&:where(.dark, .dark *))`
- 색상: zinc(neutral) + orange-500(브랜드 액센트)
- 타이포: `prose prose-zinc dark:prose-invert`

## Content 추가하는 법

1. `content/sections.json`에 새 섹션 엔트리 추가 (id, order, tier, file paths, 제목 등)
2. `content/ko/{filename}.md`, `content/en/{filename}.md` 생성
3. `tiers` 객체의 `count` 값 업데이트
4. **코드 수정 필요 없음** — 섹션은 JSON에서 자동 디스커버리

## Conventions

- 컴포넌트: named export (Next.js page/layout만 default export)
- 클라이언트 컴포넌트: `'use client'` 디렉티브 필수
- import alias: `@/` → `src/`
- 인라인 TypeScript 인터페이스 사용
- `gray-matter`는 `next.config.ts`에서 `serverExternalPackages`에 반드시 포함
- 워터마크: `PageClient.tsx`에 인라인 SVG data URI로 구현

## Supported Languages

`ko`와 `en`만 지원. 새 언어 추가 시:
1. `src/types/index.ts`의 `Lang` 타입 확장
2. `[lang]/page.tsx`의 `SUPPORTED_LANGS`와 `generateStaticParams` 업데이트
3. `content/{lang}/` 디렉토리에 마크다운 파일 추가
4. `sections.json`에 해당 언어 필드 추가

---
> Source: [fivetaku/cc101](https://github.com/fivetaku/cc101) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
