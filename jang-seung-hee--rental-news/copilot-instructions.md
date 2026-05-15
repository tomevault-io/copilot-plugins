## typescript-standards

> - **모든** 파일은 `.tsx` 또는 `.ts` 확장자 사용

# TypeScript 코딩 표준

## 파일 확장자
- **모든** 파일은 `.tsx` 또는 `.ts` 확장자 사용
- **반드시** 타입 정의를 `src/types/` 폴더에 작성
- **모든** 컴포넌트 props는 인터페이스로 정의

## 명명 규칙
- **컴포넌트**: PascalCase (예: `PromotionCard.tsx`)
- **함수/변수**: camelCase (예: `getPromotionData`)
- **상수**: UPPER_SNAKE_CASE (예: `API_ENDPOINTS`)
- **파일명**: 컴포넌트는 PascalCase, 유틸은 camelCase

## 타입 정의 예시
```typescript
interface PromotionCardProps {
  title: string;
  content: string;
  imageUrl?: string;
  createdAt: Date;
}

interface CustomerData {
  id: string;
  name: string;
  email: string;
  phone?: string;
}
```

## React 컴포넌트 규칙
- **함수형 컴포넌트**만 사용
- **Hooks**: useState, useEffect, useContext 활용
- **상태 관리**: Context API 또는 Zustand 사용
description:
globs:
alwaysApply: false
---

---
> Source: [jang-seung-hee/rental-news](https://github.com/jang-seung-hee/rental-news) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
