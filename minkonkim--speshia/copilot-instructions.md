## speshia

> 이 문서는 Gemini가 Speshia 프로젝트의 코드를 생성하고 수정할 때 일관성을 유지하기 위해 따르는 아키텍처, 구조, 코드 컨벤션 및 프로세스를 정의합니다.

# Speshia 프로젝트 개발 원칙

이 문서는 Gemini가 Speshia 프로젝트의 코드를 생성하고 수정할 때 일관성을 유지하기 위해 따르는 아키텍처, 구조, 코드 컨벤션 및 프로세스를 정의합니다.

## 1. 기본 원칙

- **요구사항 명확화**: 작업을 시작하기 전, 사용자의 요구사항이 모호하다면 반드시 추가 질문을 통해 명확히 합니다.
- **기존 코드 존중**: 새로운 코드를 작성하기 전, 반드시 주변 코드와 프로젝트 전체의 구조 및 스타일을 파악하고 이를 따릅니다.
- **타입 안정성**: TypeScript의 타입 시스템을 최대한 활용하여 타입 안정성을 확보합니다. `any` 타입 사용은 지양합니다.
- **에러 및 리스크 확인**: 잠재적인 에러 케이스(e.g., API 실패, 데이터 부재)를 항상 고려하고 적절히 처리합니다.
- **다음 단계 요약**: 작업 완료 후, 다음 진행할 단계를 간략하게 요약하여 사용자와의 소통을 돕습니다.

## 2. 기술 스택
- **프레임워크**: Next.js 14+ (App Router)
- **언어**: TypeScript
- **스타일링**: Tailwind CSS, shadcn/ui
- **상태 관리**: React Hooks, SWR 또는 React Query (필요시 도입)
- **데이터베이스 ORM**: Prisma
- **백엔드/인증**: Supabase, NextAuth.js
- **API 클라이언트**: Axios
- **테스팅**: Vitest
- **컴포넌트 문서화**: Storybook

## 3. 아키텍처 및 구조 (Architecture & Structure)

이 프로젝트는 **계층형 아키텍처**와 **도메인 중심 설계**를 따릅니다.

### 3.1. 디렉토리별 역할

- **`src/app/`**: **라우팅 및 페이지/API 엔드포인트**
    - 새로운 페이지는 `(user)` 또는 `(admin)`과 같은 라우트 그룹 내에 추가합니다.
    - 백엔드 API는 `src/app/api/{domain}/route.ts` 형태로 도메인별로 파일을 생성하여 관리합니다.

- **`src/components/`**: **UI 컴포넌트**
    - `ui/`: 애플리케이션 전반에서 사용되는 재사용 가능한 기본 컴포넌트 (shadcn/ui 기반).
    - `shared/`: 여러 도메인에서 사용되는 공유 컴포넌트.
    - `{domain}/`: `my-space`, `booking` 등 특정 도메인에 종속적인 컴포넌트.

- **`src/modules/`**: **핵심 도메인 로직**
    - 가장 중요한 비즈니스 로직이 위치합니다.
    - 새로운 도메인(e.g., `payment`) 추가 시, `src/modules/payment` 디렉토리를 생성합니다.
    - `*.model.ts`: 도메인의 핵심 데이터 모델을 정의합니다.
    - `*.dto.ts`: API 통신 등 계층 간 데이터 전송에 사용될 객체(DTO)의 형태를 정의합니다.

- **`src/lib/`**: **코어 로직 및 유틸리티**
    - `api/{domain}-api.ts`: 도메인별 API 클라이언트 함수를 정의합니다. **컴포넌트에서 직접 `axios`나 `fetch`를 호출하지 않고, 반드시 이곳의 함수를 사용합니다.**
    - `utils.ts`: 특정 도메인에 속하지 않는 범용 유틸리티 함수를 정의합니다.
    - `supabase.ts`, `nextauth.ts`: 외부 서비스 설정 및 관련 헬퍼 함수를 관리합니다.

- **`src/hooks/`**: **React 커스텀 훅**
    - 상태 로직, API 호출 등을 캡슐화한 훅을 정의합니다.
    - `use-{action}{domain}` 형태의 네이밍 컨벤션을 따릅니다. (e.g., `useGetUser`, `useGetSpaces`)

- **`prisma/schema.prisma`**: **데이터베이스 스키마**
    - 모든 데이터베이스 테이블과 관계는 이 파일을 통해 정의하고 수정합니다.

### 3.2. 데이터 흐름
1.  **UI (Component)**: 사용자 인터랙션 발생
2.  **Hook**: 컴포넌트는 `src/hooks`의 훅을 호출
3.  **API Client**: 훅은 `src/lib/api`의 API 클라이언트 함수를 호출
4.  **API Route**: API 클라이언트는 `src/app/api`의 엔드포인트로 요청 전송
5.  **Domain/Prisma**: API 라우트는 `src/modules`의 로직 또는 Prisma를 통해 DB 처리 후 응답

## 4. 개발 프로세스 및 컨벤션

- **UI 개발**: 새로운 UI 컴포넌트는 **Storybook**에서 먼저 시각적으로 테스트하고 문서화하는 것을 원칙으로 합니다.
- **테스팅**: 핵심 비즈니스 로직이나 복잡한 유틸리티 함수는 **Vitest**를 사용하여 단위 테스트를 작성합니다.
- **API 통신**: 컴포넌트 내에서 직접 API를 호출하지 않고, `src/lib/api`에 정의된 함수를 통해 중앙에서 관리합니다.
- **네이밍**: 파일명, 변수명, 함수명은 역할을 명확히 알 수 있도록 서술적으로 작성합니다. (e.g., `getHostRegisterForm` -> `HostRegisterForm`)
- **포맷팅**: 코드 작성 후에는 항상 **ESLint**와 **Prettier**를 통해 코드를 검사하고 포맷팅합니다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MinKonKim) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
