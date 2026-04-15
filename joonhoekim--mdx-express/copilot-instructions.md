## mdx-express

> - **커밋 시 Co-Authored-By 메타데이터를 넣지 않는다.** 사용자가 명시적으로 요청한 경우에만 포함한다.

# CLAUDE.md — mdx-express

## 규칙 오버라이드

- **커밋 시 Co-Authored-By 메타데이터를 넣지 않는다.** 사용자가 명시적으로 요청한 경우에만 포함한다.

## 프로젝트 개요

Next.js 16 + MDX 기반 문서/블로그 사이트. 파일 시스템(`content/`)에서 MDX 파일을 읽어 React 컴포넌트로 렌더링한다.

- **프레임워크:** Next.js 16.1.6 (App Router, Turbopack)
- **패키지 매니저:** bun (권장), pnpm (대안)
- **언어:** TypeScript (strict mode)
- **스타일링:** Tailwind CSS 4.2 + Shadcn UI (New York 스타일)
- **테마:** next-themes (다크/라이트 모드)

## 개발 명령어

```bash
bun dev          # 개발 서버 (Turbopack)
bun run build    # 프로덕션 빌드
bun start        # 프로덕션 서버
bun lint         # 린트
bun create-mdx   # 인터랙티브 MDX 파일 생성
```

## 프로젝트 구조

```
app/
  layout.tsx                    # 루트 레이아웃 (ThemeProvider)
  page.tsx                      # / → /docs 리다이렉트
  docs/layout.tsx               # 사이드바 + 상단 내비게이션
  docs/[...slug]/page.tsx       # MDX 동적 라우팅 (핵심)
  api/sibling-files/route.ts    # 형제 파일 API

components/
  mdx-renderer.tsx              # MDX → React 변환 진입점 (compileMDX + 에러 핸들링)
  mdx-components.ts             # 컴포넌트 레지스트리 (Writing UI + Shadcn UI)
  mdx-options.ts                # remark/rehype 플러그인 설정
  mdx-error.tsx                 # MDX 에러 포맷팅 + 개발 모드 UI
  mdx-html-elements.tsx         # HTML 요소 스타일링 + 코드 블록 라우팅
  document-page.tsx             # 단일 문서 뷰
  section-index-page.tsx        # 섹션 인덱스 (디렉토리 뷰)
  sidebar-navigation.tsx        # 사이드바 (클라이언트)
  writing-ui/                   # MDX 전용 커스텀 컴포넌트 (14개)
  ui/                           # Shadcn UI 컴포넌트 (46개)

lib/
  mdx-utils.ts                  # MDX 파일 I/O, 트리 빌드, sanitization
  navigation.ts                 # 내비게이션 데이터 빌드 (서버)
  navigation-utils.ts           # 아이콘/내비게이션 헬퍼
  utils.ts                      # cn(), naturalCompare(), formatTitle()

content/                        # MDX 콘텐츠 (10개 섹션)
hooks/                          # useIsMobile, usePanZoom, useSiblingFiles
scripts/create-mdx.js           # MDX 파일 생성 스크립트
```

## 아키텍처: MDX 렌더링 파이프라인

3개 레이어로 구성:

### 1. 파일 시스템 레이어 (`lib/mdx-utils.ts`)

- `getMDXFile()` — 파일 읽기 + gray-matter frontmatter 파싱
- `buildMDXTree()` — content/ 재귀 순회 → MDXFileNode 트리 구축
- `sanitizeMDXContent()` — CodeBlock JSX 태그 안의 `<>{}` → HTML 엔티티 이스케이프
- `getPathType()` — URL 경로가 파일인지 디렉토리인지 판별

### 2. 라우팅 레이어 (`app/docs/[...slug]/page.tsx`)

- catch-all 라우트로 모든 문서 경로를 처리
- 파일 → `DocumentPage` (MDXRenderer), 디렉토리 → `SectionIndexPage`
- `generateStaticParams()`로 빌드 타임 정적 생성

### 3. 렌더링 레이어 (`components/mdx-renderer.tsx` + 분리 모듈)

- `next-mdx-remote/rsc`의 `compileMDX` 사용 (런타임 RSC 방식)
- remark 플러그인: `remarkCjkFriendly` → `remarkMath` → `remarkGfm` (순서 중요!)
- rehype 플러그인: `rehypeKatex` + `rehypeRaw` (passThrough로 MDX 노드 5종 보존)
- 컴포넌트 매핑: `mdx-components.ts` (Writing UI + Shadcn UI + HTML 요소)
- HTML 요소 + 코드 블록 라우팅: `mdx-html-elements.tsx`
- 플러그인 설정: `mdx-options.ts`
- 에러 핸들링: `mdx-error.tsx` (소스 위치 포함 포맷팅 + 개발 모드 UI)

## MDX 콘텐츠 작성 규칙

### Frontmatter

```yaml
---
title: "페이지 제목"
description: "간단한 설명"
date: "2026-02-25"
order: 1              # 정렬 순서 (기본값 0)
tags: ["tag1", "tag2"]
---
```

- `title` 없으면 파일명이 제목이 됨
- `order`로 사이드바 정렬 제어

### 사용 가능한 Writing UI 컴포넌트

```mdx
import { Callout, CodeBlock, Tabs, TabsList, TabsTrigger, TabsContent } from '@/components/writing-ui';

<Callout type="info|warning|error|success|note" title="제목">내용</Callout>
<CodeBlock language="ts" title="파일명">코드</CodeBlock>
<Steps><Step title="1단계">내용</Step></Steps>
<Tabs defaultValue="tab1">
  <TabsList><TabsTrigger value="tab1">탭1</TabsTrigger></TabsList>
  <TabsContent value="tab1">내용</TabsContent>
</Tabs>
<Icon name="AlertTriangle" size={20} />
```

- Mermaid 다이어그램: 표준 코드 펜스 ` ```mermaid ` 사용 (자동 감지)
- 코드 블록: 표준 코드 펜스로 작성하면 highlight.js 자동 적용

### 컴포넌트 이름 충돌 규칙

- Writing UI 버전 우선: `Card`, `Badge` → Writing UI 컴포넌트
- Shadcn UI 버전: `ShadcnCard`, `ShadcnBadge` 접두사 사용

## 핵심 주의사항

### CJK 호환성

- `remark-cjk-friendly`가 한글 **볼드**/`*이탤릭*` 렌더링 문제 해결
- 플러그인 순서: `remarkCjkFriendly` → `remarkMath` → `remarkGfm` (반드시 이 순서)

### Sanitization

- `sanitizeMDXContent()`는 `<CodeBlock>` 태그 안의 `<>{}` 만 이스케이프
- 표준 마크다운 코드 펜스(```)는 별도 sanitization 불필요
- HTML 엔티티 방식 사용 (템플릿 리터럴 방식은 백슬래시 문제로 불가)

### rehype-raw passThrough

`rehypeRaw`의 `passThrough`에 MDX 노드 5종 필수 명시:
`mdxFlowExpression`, `mdxJsxFlowElement`, `mdxJsxTextElement`, `mdxTextExpression`, `mdxjsEsm`
— 이 설정 없으면 MDX import/JSX 컴포넌트가 깨짐

### pre 요소 라우팅

`mdx-html-elements.tsx`의 `pre` 매핑이 코드 블록 처리의 핵심:
1. `language-mermaid` → Mermaid 컴포넌트
2. `math-display` → 패스스루 (rehype-katex가 처리)
3. `language-*` → CodeBlock 컴포넌트 (highlight.js)
4. 언어 미지정 → plaintext CodeBlock
5. 그 외 → 순수 pre 태그

## 파일 경로 ↔ URL 매핑

```
content/dev/react/hooks.mdx        → /docs/dev/react/hooks
content/javascript/closures.mdx    → /docs/javascript/closures
```

## 핵심 TypeScript 인터페이스

```typescript
interface MDXFile { slug, title, description?, order?, path, content }
interface MDXFileNode { type: 'file'|'directory', name, slug, title, description?, order?, path, content?, children?, fullPath: string[] }
interface MDXFrontmatter { title, description, order, author, date, tags }
interface NavigationItem { title, href, icon?, children?, isActive? }
```

## 코드 퀄리티 규칙

### 파일 크기

- 컴포넌트/유틸리티 파일은 **200줄 이하** 유지 (Shadcn UI 제외)
- 초과 시 관심사별로 분리 (렌더링 / 인터랙션 / 데이터 로직)

### 컴포넌트 설계

- **단일 관심사 원칙:** 하나의 컴포넌트는 하나의 역할만 담당
- **선언적 코딩:** 조건부 렌더링은 early return 또는 맵 객체 활용, 명령형 DOM 조작 최소화
- Named Export 사용, 모든 컴포넌트는 `className` prop 수용
- 복잡한 인터랙션 로직은 커스텀 훅으로 분리

### 재사용성

- 아이콘/색상/스타일 맵은 `constants.ts`로 중앙 관리
- 2회 이상 사용되는 유틸리티 함수는 `lib/`로 추출
- 도메인 특화 로직은 커스텀 훅으로 격리

### lib/ 분할 기준

- 파일 I/O → `mdx-file.ts`
- 트리 빌드 → `mdx-tree.ts`
- 인접 글/형제 파일 → `mdx-navigation.ts`
- 공통 타입/상수 → `mdx-types.ts`

## 주요 의존성

- `next-mdx-remote` 6.0.0 — MDX 렌더링 (compileMDX, RSC 모드)
- `gray-matter` — frontmatter 파싱
- `highlight.js` — 190+ 언어 구문 강조
- `remark-math` + `rehype-katex` — 수식 렌더링 (KaTeX)
- `mermaid` — 다이어그램
- `lucide-react` — 아이콘 (선별 import, LUCIDE_ICONS 매핑)
- `@radix-ui/*` + Shadcn UI — UI 컴포넌트
- `react-resizable-panels` — 드래그 가능 사이드바

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joonhoekim)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/joonhoekim)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
