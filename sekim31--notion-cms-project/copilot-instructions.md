## notion-cms-project

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**노션 견적서 뷰어**는 노션에서 작성한 견적서를 웹에서 확인하고 PDF로 다운로드할 수 있는 시스템입니다.

## Project Context

- PRD 문서: @docs/PRD.md
- 개발 로드맵: @docs/ROADMAP.md

## ⚡ 자주 사용하는 명령어

```bash
# 개발
npm run dev         # 개발 서버 실행 (Turbopack)
npm run build       # 프로덕션 빌드
npm run check-all   # 모든 검사 통합 실행 (권장)

# UI 컴포넌트
npx shadcn@latest add button    # 새 컴포넌트 추가
```

## ✅ 작업 완료 체크리스트

```bash
npm run check-all   # 모든 검사 통과 확인
npm run build       # 빌드 성공 확인
```

## 프로젝트 개요

노션 견적서 뷰어 MVP - Next.js 16 + React 19 기반. 노션 API 연동으로 견적서를 동기화하고, 클라이언트와 공유 가능한 링크 생성 및 PDF 다운로드 기능 제공.

## 아키텍처

### 라우팅 (App Router + Route Groups)
- `app/(marketing)/` - 마케팅 페이지 (홈 랜딩)
- `app/(auth)/` - 인증 페이지 (로그인, 회원가입)
- `app/(dashboard)/` - 대시보드 (견적서 목록, 상세, 설정)
- `app/quote/share/[shareId]/` - 공유 견적서 (공개 페이지)

### 컴포넌트 구조
- `components/ui/` - shadcn/ui 기본 컴포넌트
- `components/layout/` - 레이아웃 컴포넌트 (Header, Footer, AppSidebar 등)
- `components/common/` - 공통 컴포넌트 (Logo, ThemeToggle, LoadingSpinner 등)
- `components/features/` - 기능별 컴포넌트

### 상태 관리
- **테마:** next-themes (시스템 감지 + 수동 전환)
- **서버 상태:** @tanstack/react-query
- **URL 쿼리 상태:** nuqs
- **폼 상태:** react-hook-form + Zod 검증

### Providers 구조 (`providers/index.tsx`)
ThemeProvider → QueryProvider → NuqsProvider 순서로 래핑

## 핵심 설정

### Path Alias
`@/*` → 프로젝트 루트 (예: `@/components`, `@/lib`, `@/hooks`)

### shadcn/ui 설정 (components.json)
- 스타일: new-york
- 아이콘: lucide-react

### 테마 시스템
- CSS 변수 기반 (oklch 색상)
- 다크모드 지원 (class 전략)
- globals.css에서 :root / .dark 정의

## 코드 패턴

### 클라이언트 컴포넌트
```tsx
"use client"
// 상단에 명시적으로 선언
```

### 조건부 클래스 병합
```tsx
import { cn } from "@/lib/utils"
cn("base-class", condition && "conditional-class")
```

### 폼 검증 (Zod 스키마)
`lib/validations/` 디렉토리에 검증 스키마 정의

### 네비게이션 설정
`config/navigation.ts`에서 mainNav, dashboardNav, footerNav 중앙 관리

## 타입 정의

`types/index.ts`에서 공통 타입 정의:
- `PageParams` - 페이지 파라미터
- `ApiResponse<T>` - API 응답
- `PaginatedResponse<T>` - 페이지네이션 응답
- `User` - 사용자 정보

---
> Source: [SEKim31/notion-cms-project](https://github.com/SEKim31/notion-cms-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
