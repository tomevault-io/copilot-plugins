## lofi-j-web

> Lofi J 블로그 프로젝트 AI 작업 규칙 (컨벤션, 기술 스택, 응답 스타일)


# Lofi J Blog - 프로젝트 규칙

## 응답 스타일

- **모든 응답은 반드시 한국어(Korean)로** 작성할 것
- 변경된 코드는 **어디가 바뀌었는지 명확히 하이라이팅** 해줄 것
- 코드에 불필요한 나레이션 주석 금지 (예: `// 상태 업데이트`, `// 함수 정의`)
- 복잡한 로직이나 의도가 불분명한 부분에만 주석 허용

## 기술 스택

| 기술 | 버전 | 비고 |
|------|------|------|
| Next.js | 16 (App Router) | Pages Router 사용 금지 |
| React | 19 | Server Components 우선 |
| TypeScript | 5 | strict 모드 |
| Tailwind CSS | v4 | `@theme` 기반 토큰 시스템 |
| ESLint | 9 | flat config |
| Prettier | 3 | |

### Next.js 규칙

- **App Router만 사용** — `pages/` 디렉토리 절대 사용하지 않는다
- 가능한 한 **Server Component를 기본**으로 하고, 클라이언트 상태/이벤트가 필요한 경우에만 `'use client'` 사용
- `metadata` export를 통해 SEO 메타데이터 관리

### Tailwind CSS v4 규칙

- `tailwind.config.js` 대신 **CSS 파일 내 `@theme inline`** 블록으로 디자인 토큰 정의
- 커스텀 색상은 `globals.css`의 `@theme inline` 블록에 추가
- Tailwind v3 문법(`theme.extend` 등)을 사용하지 않는다

### TypeScript 규칙

- `any` 타입 사용 금지 — 반드시 구체적인 타입 명시
- 컴포넌트 props는 인라인 타입 또는 별도 `type`으로 정의 (`interface` 대신 `type` 선호)
- path alias `@/*`를 사용한 절대 경로 import

## 프로젝트 컨벤션

### 폴더 구조

이 프로젝트는 크게 2가지로 분리되어있음 인증이 필요없는 일반 유저를 위한 Blog 페이지(lofi-j)
블로그 관리자를 위한 admin 페이지(admin)

auth 경로는 오직 관리자의 인증 절차를 위한 경로임

```
app/                  # Next.js App Router (라우팅 + 레이아웃)
  lofi-j/             # 블로그 메인 라우트 그룹
    layout.tsx
    page.tsx
source/               # 비즈니스 로직 및 컴포넌트
  lofi-j/
    components/       # UI 컴포넌트
```

- `app/` — 라우팅과 레이아웃 전용. 비즈니스 로직을 넣지 않는다
- `source/` — 재사용 가능한 컴포넌트, 훅, 유틸리티 배치
- 컴포넌트 import 시 `@/source/...` 경로 사용

### 네이밍 규칙

| 대상 | 규칙 | 예시 |
|------|------|------|
| 컴포넌트 파일 | kebab-case | `topbar.tsx` |
| 컴포넌트 이름 | PascalCase | `Topbar` |
| 훅 파일 | kebab-case (`use-` 접두사) | `use-scroll.ts` |
| 유틸리티 파일 | kebab-case | `format-date.ts` |
| 타입 파일 | kebab-case | `post-types.ts` |

### 컴포넌트 패턴

- **named export** 사용 (`export const Component`, `export default` 지양)
- 단, `page.tsx`, `layout.tsx`는 Next.js 규칙에 따라 `export default` 사용
- 함수형 컴포넌트만 사용 (arrow function 스타일)

```tsx
// 오직 Page, Layout 파일에서만 export default function 방식으로 정의하고 
// Component는 화살표 함수로 정의한다.

// ✅ GOOD
export const MyComponent = ({ title }: { title: string }) => {
  return <div>{title}</div>;
};

// ❌ BAD
export default function MyComponent({ title }: { title: string }) {
  return <div>{title}</div>;
}
```

### 폰트 시스템

- 한글: `font-serif` (Noto Serif KR, CSS 변수 `--font-noto-serif-kr`)
- 영문 디스플레이: `font-display` (Playfair Display, CSS 변수 `--font-playfair-display`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lofi-J-blog) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
