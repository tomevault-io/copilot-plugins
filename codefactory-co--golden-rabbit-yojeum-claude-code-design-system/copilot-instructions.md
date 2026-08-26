## golden-rabbit-yojeum-claude-code-design-system

> 이 파일은 Claude Code (claude.ai/code)가 이 저장소에서 작업할 때 필요한 가이드라인을 제공합니다.

# CLAUDE.md

이 파일은 Claude Code (claude.ai/code)가 이 저장소에서 작업할 때 필요한 가이드라인을 제공합니다.

## 꼭 기억할 요소!!!
- UI를 제작할때는 꼭 Shadcn MCP를 사용해줘.
- UI를 제작할때 **절대로** emoji를 사용하지 말고 Lucide Icon을 사용해줘.
- **공통 컴포넌트 활용**: UI 구현시 반드시 `src/components/`의 기존 공통 컴포넌트를 먼저 확인하고 import해서 사용해줘.
- **새 컴포넌트 생성 전 확인**: 존재하지 않는 새로운 컴포넌트가 필요할 때는 사용자에게 요구사항을 물어보고 공통 컴포넌트로 새로 생성할지 확인해줘.

## 프로젝트 개요

TypeScript, Tailwind CSS v4, shadcn/ui 컴포넌트를 사용하여 구축된 Next.js 15 디자인 시스템 프로젝트입니다. App Router 아키텍처를 사용하는 모던 React 애플리케이션으로 설정되었으며 디자인 시스템 구축을 위한 컴포넌트 유틸리티를 포함하고 있습니다.

## 개발 명령어

**개발 서버 시작:**
```bash
npm run dev
```
더 빠른 개발 빌드를 위해 Turbopack과 함께 Next.js를 사용합니다.

**프로덕션 빌드:**
```bash
npm run build
```
Turbopack을 사용하여 최적화된 프로덕션 빌드를 생성합니다.

**프로덕션 서버 시작:**
```bash
npm start
```

**코드 린트:**
```bash
npm run lint
```
Next.js 구성으로 ESLint를 실행합니다.

## 아키텍처 및 주요 컴포넌트

**파일 구조:**
- `src/app/` - Next.js App Router 페이지와 레이아웃
- `src/lib/` - 유틸리티 함수와 공유 로직
- `src/components/ui/` - 재사용 가능한 UI 컴포넌트 (shadcn/ui 패턴)
- `components.json` - shadcn/ui 구성 파일

**주요 파일:**
- `src/lib/utils.ts` - clsx와 tailwind-merge를 사용한 className 병합을 위한 `cn()` 유틸리티 포함
- `src/app/layout.tsx` - Geist 폰트가 적용된 루트 레이아웃
- `components.json` - 별칭과 스타일 기본 설정이 포함된 shadcn/ui 구성

**기술 스택:**
- **프레임워크**: Next.js 15 with App Router
- **언어**: TypeScript
- **스타일링**: Tailwind CSS v4 with CSS variables
- **컴포넌트**: shadcn/ui (New York 스타일)
- **아이콘**: Lucide React
- **폰트**: Geist Sans and Geist Mono

## 경로 별칭

프로젝트는 `tsconfig.json`과 `components.json` 모두에 구성된 TypeScript 경로 별칭을 사용합니다:

- `@/components` → `src/components`
- `@/lib` → `src/lib`
- `@/utils` → `src/lib/utils`
- `@/ui` → `src/components/ui`
- `@/hooks` → `src/hooks`

## 디자인 시스템 구성

**shadcn/ui 설정:**
- 스타일: "new-york"
- 기본 색상: "neutral"
- CSS 변수: 활성화됨
- RSC: 활성화됨
- 아이콘 라이브러리: lucide

**컴포넌트 설치:**
새로운 shadcn/ui 컴포넌트를 추가할 때, 구성된 별칭과 스타일 기본 설정으로 `src/components/ui/`에 설치됩니다.

## 공통 컴포넌트 가이드

**중요**: `src/components/CLAUDE.md` 파일에 모든 공통 컴포넌트에 대한 상세한 가이드가 정리되어 있습니다.

**컴포넌트 사용 규칙:**
1. **기존 컴포넌트 우선** - UI 구현시 `src/components/ui/`와 `src/components/navigation/`의 기존 컴포넌트를 먼저 확인
2. **일관된 import 패턴** - `@/components/ui/button`, `@/components/navigation/header` 등의 별칭 사용
3. **새 컴포넌트 생성시** - 사용자와 요구사항 논의 후 공통 컴포넌트로 추가할지 결정

**주요 공통 컴포넌트:**
- **폼 관련**: Button, Input, Label, Checkbox, Select, PasswordInput, FormField
- **레이아웃**: Card, Badge, Separator, Tabs
- **네비게이션**: Header, Logo, UserMenu, SearchBar, BreadCrumb

## 스타일링 접근법

- PostCSS 구성과 함께 **Tailwind CSS v4** 사용
- 테마 커스터마이징을 위한 **CSS 변수**
- **className 유틸리티**: 조건부 및 병합된 className을 위해 `src/lib/utils.ts`의 `cn()` 사용
- **컴포넌트 변형**: 컴포넌트 API 디자인을 위해 `class-variance-authority` 사용

## 개발 참고사항

- 성능 향상을 위해 개발과 프로덕션 빌드 모두에서 **Turbopack** 사용
- Next.js 전용 규칙으로 **ESLint** 구성됨
- 기본적으로 **서버 컴포넌트** 지원 (RSC 활성화됨)
- Geist 폰트 패밀리로 `next/font`를 통한 폰트 최적화 처리

---
> Source: [codefactory-co/golden-rabbit-yojeum-claude-code-design-system](https://github.com/codefactory-co/golden-rabbit-yojeum-claude-code-design-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
