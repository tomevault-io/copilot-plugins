## styling-standards

> - **우선순위**: TailwindCSS 클래스 사용

# 스타일링 표준

## TailwindCSS 우선순위
- **우선순위**: TailwindCSS 클래스 사용
- **커스텀 스타일**: `src/globals.css`에만 작성
- **반응형**: 모바일 최우선 (480px → 768px → 1024px)

## 색상 시스템
- **Primary**: 파란색 계열 (#87ceeb ~ #4682b4)
- **Secondary**: 연한 파란색 계열 (#f0f8ff ~ #b0e0e6)
- **Text**: 흰색 및 회색 계열

## 반응형 브레이크포인트
```css
/* 모바일: 480px 이하 */
/* 태블릿: 768px 이하 */
/* 데스크톱: 768px 이상 */
```

## Shadcn UI 컴포넌트
- **우선 사용**: Shadcn UI 컴포넌트 활용
- **커스터마이징**: TailwindCSS 클래스로 스타일 조정
- **일관성**: 프로젝트 전체에서 동일한 디자인 시스템 적용

## 컴포넌트 스타일 예시
```tsx
// 올바른 예시
<div className="bg-blue-500 text-white p-4 rounded-lg shadow-md hover:bg-blue-600 transition-colors">
  <h2 className="text-xl font-bold mb-2">프로모션 제목</h2>
  <p className="text-sm opacity-90">프로모션 내용</p>
</div>

// 반응형 예시
<div className="w-full md:w-1/2 lg:w-1/3 p-4">
  <div className="bg-gradient-to-r from-blue-400 to-blue-600 rounded-lg p-6">
    {/* 내용 */}
  </div>
</div>
```
description:
globs:
alwaysApply: false
---

---
> Source: [jang-seung-hee/rental-news](https://github.com/jang-seung-hee/rental-news) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
