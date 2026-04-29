## diki

> - 기본 Tailwind 색상 클래스(예: `bg-gray-700`, `text-blue-500` 등)를 사용하지 마세요.

# 디키 프로젝트 스타일 가이드

## Tailwind 색상 사용 규칙

- 기본 Tailwind 색상 클래스(예: `bg-gray-700`, `text-blue-500` 등)를 사용하지 마세요.
- 대신 `@tailwind.config.ts`에 정의된 커스텀 색상을 사용하세요:
  - 기본 색상: `bg-primary`, `bg-secondary`, `bg-background`, `bg-accent`
  - 텍스트 색상: `text-main`, `text-sub`
  - 회색 계열: `bg-gray0`~`bg-gray5`, `text-gray0`~`text-gray5`
  - 테두리 색상: `border-light`, `border-extreme-light`
  - 레벨 색상: `text-level-1`~`text-level-5`, `border-level-1`~`border-level-5`

## 애니메이션 사용 규칙

- `@tailwind.config.ts`에 정의된 커스텀 애니메이션을 사용하세요:
  - `animate-slideDown`, `animate-slideDownIn`, `animate-slideDownOut`
  - `animate-intro`, `animate-introSecond`
  - `animate-slideLeftToRight`, `animate-slideRightToLeft`

## 테마 변수 활용

- 모든 색상은 `@globals.css`에 정의된 CSS 변수를 활용합니다:
  - 라이트 모드: `:root` 아래 정의된 변수 사용
  - 다크 모드: `.dark` 아래 정의된 변수 사용

## 코드 스타일링

- ESLint `@.eslintrc.json` 규칙을 따르세요. 특히:
  - 커스텀 클래스명 허용 (`tailwindcss/no-custom-classname: off`)
  - 임의값 사용 자제 (`tailwindcss/no-arbitrary-value: warn`)
  - 모순되는 클래스명 사용 금지 (`tailwindcss/no-contradicting-classname: error`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dxwiki) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
