## front

> app  →  (processes)  →  features  →  shared

# FSD based 5 Layered 아키텍처 헌법 (필수 조항)

## 1) 의존 방향(단방향)
```
app  →  (processes)  →  features  →  shared
                         ↘ api (codegen)
```
- 역방향 금지. 같은 레벨끼리 직접 의존 금지.
- **shared**는 어떤 상위도 import하지 않는다.
- **features**는 **services**만 통해 네트워크/스토리지 접근(직접 api 호출 금지 — 엄격 모드).

## 2) shared는 "도메인 무취"
- 디자인 시스템 래퍼와 범용 컴포넌트(`shared/ui`), 범용 훅/유틸(`shared/hooks|lib`), 환경설정(`config/`)만.
- 앱 고유 정책/흐름/명칭/타입이 **드러나는 순간** shared에서 하향 이동(→ features/processes).
- 예외적으로, **인증/인가**(auth)와 같이 범용 인프라는 shared로 격상한 바 있다.

## 3) app은 "조립만"
- `app/layout`, `app/providers`, `app/pages`(or screens)에서 **오케스트레이션·배치만** 하고 **비즈니스 로직 금지**.
- 상태 주입/라우팅/전역 가드(Orientation 등)만 허용.

## 4) processes는 "횡단 오케스트레이션"
- 라우터 없이도 필요한 **전역 플로우(온보딩/게임 진행 등)**는 `processes/*`로.
- 프로세스 상태는 여기서 관리(예: `processes/game-flow/model/useGameFlowStore.ts`).
- 커지면 FSM(XState) 등으로 승격 가능.

## 5) features는 "사용자 시나리오 단위의 응집"
- 구조: `features/<feat>/{ui, model, lib, mocks}`
- **공개 표면**만 `index.ts`로 배럴 export. 내부 구조는 외부에서 직접 접근 금지.
- feature 내부 유틸/훅은 **공용처럼 보여도** 우선 여기에 둔다(진짜 범용이면 승격하여 shared로 이동).

## 6) services와 api의 분리
- `src/api/` = **openapi-typescript-codegen 생성물(편집 금지)**
- `src/services/` = **우리가 만든 어댑터/래퍼**(토큰주입, 에러표준화, 리트라이/캐싱, 로깅).
- features는 services만 의존(엄격 모드).

## 7) 네이밍/코로케이션 규약
- 폴더: kebab-case / 컴포넌트: PascalCase / 훅·유틸: camelCase.
- 테스트/스토리/스타일은 **같은 폴더에 코로케이션**.
- 루트에 **barrel** 두어 외부 노출 최소화(`index.ts`).

## 8) 환경/설정의 경계
- `src/config/`에 환경 래퍼(`env.ts`), 전역 상수/플래그, API 기본설정.
- 어디서나 읽을 수 있지만 **쓰기/부작용은 금지**(쓰기/사이드이펙트는 services에서).

## 9) Mock/테스트 자산의 위치
- 기능 전용 mock: `features/<feat>/mocks/*`(co-location).
- 여러 곳에서 쓰는 공통 테스트 인프라: `src/test/*` (MSW server, 공통 factories, RTL render 등).
- 런타임 데모용 정적 JSON은 `public/mocks/*`.

## 10) 변경의 신호와 승격/분리 기준
- 어떤 유틸이 3개 이상 feature에서 중복 → shared로 승격.
- 반대로 shared가 도메인 냄새 나기 시작 → 해당 feature/process로 **하향 이동**.

---

# 권고 조항(운영 노하우)

### A. ESLint로 강제
- `eslint-plugin-boundaries`로 **단방향 의존**과 **테스트/목 예외**를 강제.

### B. 공개면(Stable API) 관리
- 각 feature는 **외부에 약속된 것만** `index.ts`로 export.
- 외부에 드러난 타입/함수는 **semver 마인드**로 변경(브레이킹 시 마이그레이션 가이드).

### C. 상태 범위 스코핑
- 전역 상태는 **processes**에서만(필요 최소한).
- feature 내부 상태는 feature에 격리.
- 전역 store에 feature 비즈니스 로직 섞지 않기.

### D. 서비스 단일화
- 인증/에러/로깅/리트라이/백오프/캐싱(React Query·Zustand persist)은 **services**에서 한 번에.
- 이렇게 하면 모든 feature가 공짜로 혜택을 받음.

### E. 경로 별칭
- `@app/*`, `@processes/*`, `@features/*`, `@shared/*`, `@api/*`, `@config/*`
- 깊은 상대경로 금지.

---

# 짧은 폴더 예시
```
src/
  app/
    App.tsx
    layout/
      RootLayout/
        index.tsx
        OrientationGuard/
          index.tsx
    pages/
      LandingPage/
        index.tsx
      MainMenu/
        index.tsx

  processes/
    game-flow/

  features/
    shelf-selection/
      ui/
        ShelfSelection/
      model/
        types.ts
        useShelfSelectionStore.ts
      __mocks__/
      index.ts

  api/                        # codegen 산출물 (편집 금지)
    services/
      AppService.ts
      AuthService.ts

  shared/
    ui/
      Button.tsx
      ...
    lib/
      utils.ts
      ...
    styles/
      globals.css
      ...
    assets/
    auth/                     # (격상된 인증, 범용 인프라로 취급)
      model/
        useAuthStore.ts
        types.ts
      index.ts

  config/
    env.ts
```

## 코딩 스타일 가이드

### React 컴포넌트
- 함수형 컴포넌트 사용
- TypeScript 필수
- Props 인터페이스는 컴포넌트 위에 정의
- 컴포넌트명은 PascalCase

### 상태 관리
- 전역 상태: Zustand + persist
- 로컬 상태: useState, useReducer
- 서버 상태: TanStack Query

### 스타일링
- Tailwind CSS 사용
- 커스텀 스타일은 `shared/styles/`에 정의
- 반응형 디자인 우선

### 에러 처리
- try-catch로 에러 처리
- 사용자 친화적 에러 메시지
- 로깅은 services에서 통합 관리

### 성능 최적화
- React.memo, useMemo, useCallback 적절히 사용
- 코드 스플리팅 고려
- 이미지 최적화

### 테스트
- 단위 테스트: Jest + React Testing Library
- 통합 테스트: MSW 사용
- E2E 테스트: Playwright 고려

### Tailwind CSS
- 항상 **정규화된 Tailwind 스케일**을 사용한다. (ex: h-17, w-131, px-10.75, py-0)
  - 모바일 가로모드 환경(1024px 미만)에서는 spacing을 0.1rem (기본값 대비 0.4배)으로 사용함에 유의한다. (ex: `h-10` -> `16px`)
  - ex. Figma 디자인이 72px인 경우 `h-18`로 반영 -> 모바일 가로모드에서는 `28.8px`로 계산되어 반영된다.
- 여러 className을 병합하는 경우 `cn` 함수를 사용한다.
  - ex. `cn('h-18', 'w-131', 'px-10.75', 'py-0')`

### 화면 컨텐츠 영역 종횡비 고정 및 배경 관리

#### UI 레이아웃 구조
- 모든 페이지 컨텐츠는 **16:9 고정 비율 컨테이너** 안에 렌더링됩니다 (RootLayout의 z-10 레이어)
- 페이지/컴포넌트 루트는 **반드시 `w-full h-full`** 또는 **`absolute inset-0`**으로 컨테이너를 채워야 합니다
- `fixed` 포지셔닝은 16:9 영역을 벗어나므로 주의 (z-10 컨테이너가 relative이므로 이를 기준으로 `absolute` 사용 권장)

#### 배경 변경
- 페이지/컴포넌트의 배경(색상/이미지/그라디언트)을 변경할 때는 **`useSetBackground` 훅**을 사용합니다
- 인라인 스타일이나 className으로 배경을 직접 설정하지 마세요 (16:9 영역 밖까지 배경 적용 필요)
- 사용법: `useSetBackground({ color: '#000' })` 또는 `useSetBackground({ className: 'bg-black' })`
- 또는 명령형으로 `setBackgroundColor`, `setBackgroundImage`, `setBackgroundGradient`, `setBackgroundClassName` 함수를 사용할 수 있습니다. (useSetBackground 훅에서 반환되는 함수)
- 상세 문서: `/docs/BACKGROUND_USAGE.md` 참조

#### BackgroundPortal을 통한 전체화면 컴포넌트 배치
- 16:9 고정비율 컨텐츠 영역 바깥의 전체화면 영역으로 컴포넌트를 이동시키고 싶을 때는 **`BackgroundPortal`** 컴포넌트로 감싸서 사용합니다
- 전체화면을 기준으로 고정위치 포지셔닝하거나 전체화면을 채워야 하는 컴포넌트에 사용 (예: 전체화면 오버레이, 글로벌 모달 등)
- 사용법: `<BackgroundPortal>{children}</BackgroundPortal>`

#### 주의사항
- GlobalBackground(z-0)가 전체 화면 배경을 담당하므로, 페이지 컴포넌트는 배경 스타일 불필요
- 페이지 전환 시 배경은 자동으로 초기화됩니다 (cleanup 옵션으로 제어 가능)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/the-promise-lab) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
