## nihilncunia-dev-rr7

> 이 문서는 Gemini 프로젝트의 핵심 개발 규칙과 가이드라인을 설명합니다. 이 규칙을 준수하면 코드의 일관성, 유지보수성, 효율적인 협업을 보장할 수 있습니다.

# Gemini 프로젝트 - 개발 가이드라인

이 문서는 Gemini 프로젝트의 핵심 개발 규칙과 가이드라인을 설명합니다. 이 규칙을 준수하면 코드의 일관성, 유지보수성, 효율적인 협업을 보장할 수 있습니다.

---

## 1. 일반 규칙

- **언어:** 모든 응답과 문서는 한국어로 작성해야 합니다.
- **패키지 매니저:** 모든 패키지 관련 명령어는 `pnpm`을 사용합니다.
- **코드 지원:** 코드 지원을 위해 다른 문서를 참조할 때는 'Context7 MCP' 서버를 사용합니다.
- **공백:** 가독성을 위해 구문은 한 줄의 공백으로 구분합니다.
- **오류 메시지:** 코드의 모든 오류 메시지는 한국어로 작성해야 합니다.
- **작업 실행:** 여러 작업이 요청되면 다음 작업에 대한 확인을 요청하지 않고 즉시 진행합니다.
- **내보내기 (JS/TS):** `export { Component } from '...'` 형식을 사용합니다. `export * from '...'`는 사용하지 않습니다.
- **가져오기 (JS/TS):** 가져오기 시에는 항상 `@` 별칭을 사용합니다. `import * as`는 사용하지 않습니다.
- **리팩토링:** "개선" 요청은 코드를 직접 리팩토링해야 함을 의미합니다.
- **코드 서식:**
    - 가독성을 위해 긴 코드 줄, 특히 인수가 많은 함수 호출이나 객체 속성은 줄을 바꿉니다.
    - 삼항 연산자는 항상 줄바꿈을 사용합니다.
- **코드 검토:** "확인" 요청은 단순한 검토가 아닌 개선이 필요함을 의미합니다. 수정을 진행합니다.
- **오류 처리:** 오류가 보고되면 문제를 설명한 후 수정을 진행합니다.

---

## 2. 파일 및 폴더 구조

- **파일 이름:** 케밥-케이스(예: `post-list.tsx`)를 사용합니다. 리액트 컴포넌트는 파스칼-케이스(예: `PostCard.tsx`)를 사용할 수 있습니다.
- **폴더 이름:** 케밥-케이스를 사용합니다. `app/` 아래의 8개 주요 폴더는 예외입니다.

- **`app/_data`:** 작은 정적 데이터 세트를 위함. `<name>.data.ts`를 사용하고 `index.ts`에서 명시적으로 내보냅니다.
- **`app/_entities`:** 도메인별 로직을 위함. 각 엔티티는 자체 디렉토리를 가지며 다음을 포함해야 합니다:
    - `index.ts`
    - `<resource>.api.ts`
    - `<resource>.store.ts`
    - `<resource>.keys.ts`
    - `<resource>.types.ts`
    - `<resource>.table.ts` (Drizzle ORM용)
    - `hooks/`
- **`app/_components`:** 라우트별 컴포넌트를 위함. 라우트별로 구성됩니다.
- **`app/_routes`:** 애플리케이션의 라우트를 정의하는 파일을 포함합니다.
- **`app/_styles`:** CSS 또는 SCSS 파일을 위함.
- **`app/_config`:** 설정 파일을 위함. `<name>.config.ts` 형식을 사용합니다.
- **`app/_icons`:** SVG 컴포넌트를 위함.
- **`app/_libs`:** 유틸리티 함수를 위함.

---

## 3. Drizzle ORM

- **테이블 파일:** 파일 이름을 `<resource>.table.ts`로 지정합니다.
- **가져오기:** `drizzle-orm/pg-core`에서 필요한 함수를 가져옵니다.
- **주석:** 각 테이블 열에 용도를 설명하는 주석을 추가합니다.
- **일관성:** 기존 테이블의 스타일을 유지합니다.
- **내보내기:** `index.ts`에서 테이블을 내보내지 않습니다.
- **변수 이름:** 테이블 변수에는 `<resourceInCamelCase>Table`을 사용합니다.
- **열 이름:** 테이블 열에는 스네이크_케이스를 사용합니다.

---

## 4. 폼

- **구조:** `shadcn`의 `<Form>` 컴포넌트로 폼을 감싼 후, 내부에 표준 `<form>` 태그를 사용합니다.
- **`useForm`:** `mode`를 `all`로 설정합니다.
- **유효성 검사:** 스키마 유효성 검사에는 `zod`를 사용합니다.

---

## 5. 가져오기

- **별칭:** `tsconfig.json`에 정의된 대로 항상 `@/` 별칭을 사용합니다. 상대 경로는 사용하지 않습니다.

---

## 6. React Router v7 (파일 기반 라우팅)

### 파일 이름 규칙

- **인덱스 라우트:** `_index.tsx` -> `/`
- **기본 라우트:** `about.tsx` -> `/about`
- **중첩 라우트 (점 구분자):** `concerts.trending.tsx` -> `/concerts/trending`
- **동적 세그먼트:** `concerts.$city.tsx` -> `/concerts/:city`
- **레이아웃:**
    - **중첩:** `concerts.tsx`는 `concerts.*.tsx` 파일의 레이아웃 역할을 합니다.
    - **경로 없는 레이아웃 (앞에 밑줄):** `_auth.login.tsx` -> `/login` (`_auth.tsx`는 레이아웃만 제공).
    - **레이아웃 제외 (뒤에 밑줄):** `concerts_.mine.tsx` -> `/concerts/mine` (`concerts.tsx` 레이아웃을 상속하지 않음).
- **스플랫 라우트:** `$.tsx`는 나머지 모든 경로와 일치합니다.

### 개발 규칙

- **UI 컴포넌트:** Radix UI에서 직접 가져오지 않습니다. 항상 `shadcn/ui`를 사용합니다.
- **이벤트 핸들러:** 이벤트 핸들러 함수의 이름을 명확하게 지정합니다 (예: `onClick`, `onSubmit`).
- **JSX 로직:**
    - 조건부 렌더링에 삼항 연산자를 사용하지 마세요. 참/거짓 확인에는 `&&`를 사용합니다.
    - 간단한 문자열 삼항 연산자는 허용됩니다.
    - 복잡한 논리 표현식에는 괄호를 사용합니다.

### 페이지 컴포넌트 구조

라우트 컴포넌트에서 다음 함수들을 내보냅니다:

- `loader({ request }: Route.LoaderArgs)`
- `action({ request }: Route.ActionArgs)`
- `meta({ data }: Route.MetaArgs)`
- `default function PageComponent({ loaderData, actionData }: Route.ComponentProps)`

### 데이터 처리

- **Props:** `loaderData`와 `actionData` props를 통해 로더 및 액션 데이터에 접근합니다. **`useLoaderData` 또는 `useActionData` 훅을 사용하지 마세요.**
- **반환 값:**
    - `loader`와 `action`에서 일반 객체를 반환합니다. **`json()` 유틸리티를 사용하지 마세요.**
    - 특정 상태 코드와 함께 응답을 반환하려면 `react-router`의 `data()` 유틸리티를 사용합니다.

### 금지 사항

- **`useLoaderData()` 또는 `useActionData()` 훅 사용 금지.**
- **`@remix-run/*`에서 가져오기 금지.**
- **`json()` 유틸리티 함수 사용 금지.**
- **Radix UI에서 직접 가져오기 금지.**
- **함수 이름에 `handle` 또는 `handler` 사용 금지.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nihilapp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
