## dividend-logbook

> 이 파일은 dividend-logbook 프로젝트 전용 코딩 규칙입니다.

# Dividend Logbook 프로젝트 코딩 규칙

<!--
  이 파일은 dividend-logbook 프로젝트 전용 코딩 규칙입니다.
  전역 규칙(~/.cursorrules)을 기반으로 하며, 프로젝트 특화 규칙을 추가합니다.
-->

## 📋 메타데이터
- 프로젝트: dividend-logbook
- 버전: 1.0.0
- 마지막 업데이트: 2024-12
- 목적: ETF 배당 수익 추적 프로젝트의 일관된 코드 품질 유지

## 🎯 프로젝트 특화 규칙

### 프로젝트 구조
- **도메인 드리븐 아키텍처**를 따릅니다
- 각 도메인(`etf`, `dividend`, `sale`, `portfolio`)은 독립적으로 관리됩니다
- 도메인별로 `components/`, `store/`, `types/`, `hooks/` 폴더를 가집니다

### 상태 관리
- **Zustand**를 사용하여 도메인별 상태를 관리합니다
- 각 도메인의 store는 `domains/{domain}/store/` 경로에 위치합니다
- Store 파일명은 `use{Domain}Store.ts` 형식을 따릅니다 (예: `useDividendStore.ts`)

### 컴포넌트 구조
- 공통 컴포넌트는 `src/components/common/`에 위치합니다
- 도메인 전용 컴포넌트는 `src/domains/{domain}/components/`에 위치합니다
- UI 컴포넌트는 `src/components/ui/`에 위치합니다

### 스타일링
- **Emotion**을 사용합니다
- 테마는 `src/styles/theme.ts`에서 관리합니다
- 공통 스타일은 `src/styles/common.styles.ts`에 정의합니다
- CSS 변수와 Emotion styled components를 함께 사용합니다

### 타입 정의
- 각 도메인의 타입은 `src/domains/{domain}/types/index.ts`에 정의합니다
- 공통 타입은 필요시 별도 파일로 관리합니다

### Next.js 특화 규칙
- App Router를 사용합니다
- 서버 컴포넌트를 기본으로 사용하고, 필요시 'use client'를 명시합니다
- 페이지는 `src/app/` 디렉토리에 위치합니다

## 🔗 전역 규칙 참조

이 프로젝트는 전역 `.cursorrules` 파일의 다음 규칙들을 따릅니다:
- 핵심 원칙 (커뮤니케이션, 코드 품질)
- 코드 스타일 (네이밍 컨벤션, 포맷팅)
- React 코딩 규칙 (토스 프론트엔드 펀더멘털 기반)
- 컴포넌트 설계 원칙
- Hooks 사용 가이드
- 상태 관리 패턴
- 성능 최적화
- 에러 처리
- 커밋 메시지 규칙
- Pull Request 생성 규칙

## 📝 프로젝트별 추가 가이드라인

### 파일 네이밍
- 컴포넌트 파일: PascalCase (예: `UserProfile.tsx`)
- 유틸리티 파일: camelCase (예: `formatDate.ts`)
- 타입 파일: `types.ts` 또는 `index.ts`
- Store 파일: `use{Domain}Store.ts`

### Import 순서
1. React 및 Next.js 관련
2. 외부 라이브러리
3. 프로젝트 내부 컴포넌트
4. 프로젝트 내부 유틸리티
5. 타입 정의
6. 스타일 관련

### Zustand Store 패턴
```typescript
// 좋은 예
interface DividendState {
  dividends: Dividend[];
  addDividend: (dividend: Dividend) => void;
}

export const useDividendStore = create<DividendState>((set) => ({
  dividends: [],
  addDividend: (dividend) => set((state) => ({
    dividends: [...state.dividends, dividend]
  })),
}));
```

### 도메인 Export 패턴
- 각 도메인은 `index.ts`를 통해 public API를 제공합니다
- 내부 구현은 숨기고 필요한 것만 export합니다

## 🤖 AI 어시스턴트 지침

### 프로젝트 컨텍스트 인식
- 이 프로젝트는 ETF 배당 수익 추적 애플리케이션입니다
- 도메인 드리븐 아키텍처를 따릅니다
- Zustand + Emotion + Next.js 16 (App Router) 스택을 사용합니다

### 코드 생성 시
- 도메인별로 적절한 위치에 파일을 생성합니다
- Zustand store 패턴을 따릅니다
- Emotion styled components를 사용합니다
- TypeScript 타입을 명시적으로 정의합니다

### 리팩토링 시
- 도메인 경계를 존중합니다
- 기존 아키텍처 패턴을 유지합니다
- Store 구조를 변경할 때는 영향 범위를 고려합니다

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hec8897) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
