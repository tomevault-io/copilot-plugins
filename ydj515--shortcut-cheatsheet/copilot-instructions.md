## shortcut-cheatsheet

> 이 문서는 AI 어시스턴트가 `shortcut-cheatsheet` 프로젝트를 이해하고 효과적으로 기여할 수 있도록 돕기 위한 가이드입니다.

# GEMINI 가이드: shortcut-cheatsheet 프로젝트

이 문서는 AI 어시스턴트가 `shortcut-cheatsheet` 프로젝트를 이해하고 효과적으로 기여할 수 있도록 돕기 위한 가이드입니다.

## 1. 프로젝트 개요

이 프로젝트는 다양한 애플리케이션의 단축키를 검색하고 모아볼 수 있는 웹 애플리케이션입니다. 사용자는 카테고리별로 단축키를 보거나, 키워드로 검색하여 원하는 단축키를 빠르게 찾을 수 있습니다.

## 2. 기술 스택

-   **프레임워크:** React (v19) with Vite
-   **언어:** TypeScript
-   **스타일링:** Tailwind CSS
-   **라우팅:** React Router
-   **애니메이션:** Framer Motion
-   **테스팅:** Vitest with React Testing Library
-   **린팅:** ESLint with TypeScript ESLint
-   **패키지 매니저:** npm

## 3. 시작하기

프로젝트를 로컬 환경에서 실행하려면 다음 단계를 따르세요.

1.  **의존성 설치:**
    ```bash
    npm install
    ```

2.  **개발 서버 실행:**
    ```bash
    npm run dev
    ```

## 4. 주요 명령어

-   `npm run dev`: Vite 개발 서버를 시작합니다. (http://localhost:5173)
-   `npm run build`: 프로덕션용으로 프로젝트를 빌드합니다. 결과물은 `dist` 디렉토리에 생성됩니다.
-   `npm run lint`: ESLint를 사용하여 코드 스타일을 검사하고 문제를 찾습니다.
-   `npm run test`: Vitest를 사용하여 단위 및 컴포넌트 테스트를 실행합니다.
-   `npm run preview`: 프로덕션 빌드 결과물을 미리 봅니다.

## 5. 프로젝트 구조

```
/
├── src/
│   ├── components/     # 재사용 가능한 React 컴포넌트
│   ├── data/           # 단축키, 카테고리 등 정적 데이터
│   ├── types/          # TypeScript 타입 정의 (e.g., Shortcut, Category)
│   ├── utils/          # 유틸리티 함수
│   ├── App.tsx         # 메인 애플리케이션 컴포넌트
│   ├── main.tsx        # 애플리케이션 진입점
│   └── index.css       # 글로벌 스타일
└── public/             # 정적 에셋 (favicon, images, etc.)
```

-   **컴포넌트 기반 아키텍처:** 기능은 `src/components` 내의 컴포넌트로 분리하여 관리합니다.
-   **데이터 분리:** 단축키와 카테고리 데이터는 `src/data`에 명확하게 정의되어 있습니다.
-   **타입 정의:** 모든 데이터 구조에 대한 타입은 `src/types`에서 관리하여 타입 안정성을 높입니다.

## 6. 코딩 스타일 및 규칙

-   **언어:** TypeScript를 사용하여 코드를 작성합니다. `strict` 모드가 활성화되어 있으므로 타입 검사를 철저히 준수해야 합니다.
-   **스타일링:** Tailwind CSS의 유틸리티-우선 접근 방식을 사용합니다. 새로운 CSS 클래스를 직접 작성하기보다 기존 유틸리티를 조합하여 UI를 구성합니다.
-   **린팅:** `.eslintrc.js`에 정의된 ESLint 규칙을 따릅니다. 코드를 제출하기 전에 `npm run lint`를 실행하여 오류가 없는지 확인하세요.
-   **AI 상호작용:** 추가적인 AI 상호작용 규칙은 `.gemini/styleguide.md` 파일을 참고해 주세요.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ydj515)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ydj515)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
