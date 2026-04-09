## grap-v2

> +-------------------------+      HTTPS      +--------------------------------------------------------+      DB Connection      +---------------------+


- 프로젝트 아키텍처 구조.
+-------------------------+      HTTPS      +--------------------------------------------------------+      DB Connection      +---------------------+
| 외부 공공 API           | <-------------> | Nuxt.js 애플리케이션 (Nitro 서버 엔진 + Vue 프론트엔드)  | <-------------------> | Supabase 데이터베이스   |
| (행사축제, 공연전시,    |                 |                                                        |                       |                     |
|  복지정보)              |                 |  [Nitro 서버 엔진: server/api/*]                       |                       +---------------------+
+-------------------------+                 |    - 스케줄링된 데이터 수집 API (cron/)                |
                                          |      ┕ XML 파서, HTTP 클라이언트 ($fetch)              |
                                          |      ┕ DAO (DB 저장/업데이트), 로그 DAO                 |
                                          |    - 관리자 인증 API (auth/ - NuxtAuth 또는 직접)       |
                                          |    - 관리자용 데이터/로그 관리 API (admin/)             |
                                          |      ┕ DAO (DB 조회/수정)                             |
                                          |    - 사용자용 데이터 제공 API (public/)                 |
                                          |      ┕ DAO (게시용 DB 조회)                             |
                                          |                                                        |
                                          |  [Vue.js 프론트엔드: pages/, components/, layouts/]    |
                                          |    - 사용자 페이지 (Vue 컴포넌트, 데이터 페칭)          |
                                          |    - 관리자 페이지 (Vue 컴포넌트, 데이터 페칭/관리)     |
                                          |      ┕ UI 라이브러리 (PrimeVue), Pinia 상태 관리        |
                                          +--------------------------------------------------------+
                                                                      ^
                                                                      | HTTP/S (브라우저 요청)
                                                                      |
                                                           +----------+---------+
                                                           | 사용자 / 관리자    |
                                                           | (웹 브라우저)        |
                                                           +--------------------+
- Nuxt.js 프로젝트 공통 개발 규칙

I. 코드 스타일 및 포맷팅 (Code Style & Formatting)

언어: TypeScript 및 Vue.js 3 (Composition API 권장)
모든 .vue 파일 내 <script setup lang="ts"> 사용을 권장합니다.
.ts 파일을 사용하여 컴포저블, 유틸리티, 서버 로직 등을 작성합니다.
any 타입 사용을 최소화하고, 구체적인 타입을 정의하거나 추론하도록 합니다. (전역 타입은 types/ 또는 각 모듈 내에 정의)
포맷팅: Prettier를 사용하여 코드 포맷을 통일합니다.
IDE 저장 시 자동으로 Prettier가 실행되도록 설정합니다.
프로젝트 루트의 .prettierrc.js (또는 .json) 설정을 따릅니다. (PM이 초기 설정 제공: 예 - tabWidth: 2, singleQuote: true, semi: true, trailingComma: 'es5', vueIndentScriptAndStyle: true)
린팅: ESLint (Vue.js 및 TypeScript 플러그인 포함)를 사용하여 코드 품질 및 스타일 오류를 검사합니다.
Nuxt.js 프로젝트에 맞는 ESLint 설정(@nuxtjs/eslint-config-typescript, plugin:vue/vue3-recommended)을 사용합니다. (.eslintrc.js 또는 .json)
주요 규칙 준수 및 커밋 전 ESLint 검사 통과 (Git Hooks - Husky + lint-staged 설정 고려).
네이밍 컨벤션 (Naming Conventions):
변수, 함수명, 컴포저블명: camelCase (예: festivalList, getUserById, useFetchFestivals)
Vue 컴포넌트명 (파일 및 내부): PascalCase (예: FestivalCard.vue, <FestivalCard />). Nuxt.js 페이지 컴포넌트 파일명은 kebab-case 또는 PascalCase 사용 가능 (라우팅 규칙 따름).
Pinia 스토어명: use[StoreName]Store 형식의 camelCase (예: useAuthStore)
타입, 인터페이스, Enum명: PascalCase (예: FestivalData, AdminUserRole)
상수명: UPPER_SNAKE_CASE (예: MAX_RETRY_COUNT, API_BASE_URL)
파일 및 디렉토리명 (컴포넌트 외): kebab-case (PM 결정) (예: festival-dao.ts, api-fetch-logs.ts, user-interface/)
CSS 클래스명 (Tailwind CSS 사용 시): Tailwind CSS 유틸리티 클래스를 우선 사용합니다.
코드 작성 원칙 (Coding Principles) - Nuxt.js/Vue.js 환경 특화

단일 책임 원칙 (SRP): 함수, 컴포저블, 컴포넌트, Pinia 스토어 모듈은 하나의 명확한 책임만 갖도록 설계.
반복 금지 원칙 (DRY): 중복 로직은 컴포저블 함수, 유틸리티 함수, 또는 Vue 컴포넌트로 분리하여 재사용.
가독성 및 명확성: 이해하기 쉬운 코드 작성. Vue 템플릿은 간결하게, 스크립트 로직은 Composition API를 활용하여 논리적으로 그룹화.
Vue.js 베스트 프랙티스 준수:
Composition API 적극 활용: <script setup> 구문을 사용하여 반응형 로직, 생명주기 훅 등을 간결하게 작성.
컴포넌트 설계: 재사용 가능하고 단일 목적을 가진 작은 단위의 컴포넌트로 분리. Props down, events up 원칙 준수. defineProps, defineEmits 사용.
상태 관리: 전역 상태는 Pinia 스토어를 통해 관리. 로컬 컴포넌트 상태는 ref, reactive 사용.
라우팅: Nuxt.js의 파일 시스템 기반 라우팅 활용. 동적 라우트, 중첩 라우트 등 명확히 이해하고 사용. MapsTo, <NuxtLink> 사용.
데이터 페칭: Nuxt 3의 useFetch, useAsyncData 등 내장 컴포저블을 우선적으로 사용하여 서버/클라이언트 데이터 페칭 처리.
서버 API 라우트 (server/api/): Nitro 서버 엔진의 기능을 활용. event 객체, 유틸리티 함수(readBody, getQuery 등) 사용.
에러 처리 (Nuxt.js 환경):
클라이언트 사이드: Vue 컴포넌트 내 try...catch, Nuxt의 onError 훅, 에러 페이지 (error.vue) 활용.
서버 사이드 (Nitro API): try...catch, createError 유틸리티 함수 사용. 정의된 에러 형식으로 응답하고 system_error_logs 테이블에 상세 로그 기록.
환경 변수 사용: nuxt.config.ts의 runtimeConfig를 통해 환경 변수를 안전하게 애플리케이션에 노출하고 사용. 코드에 하드코딩 금지.
Supabase 사용 시 주의사항:
Supabase 클라이언트를 통한 안전한 데이터 접근
Row Level Security (RLS) 정책 활용
Supabase DAO 패턴을 통한 데이터 접근 추상화
 문서화 (Documentation)

README.md: 프로젝트 개요, Nuxt.js 기반 설치/실행 방법, 환경 변수, 주요 기술 결정 사항 등을 지속적으로 업데이트.
API 명세서: (최종본 공유됨) API 엔드포인트, 요청/응답 형식 등을 상세히 기록하고 최신 상태로 유지.
DB 스키마 정의서: (DDL 공유됨) 테이블 구조, 컬럼 설명, 관계 등을 문서화.
코드 내 주석 및 컴포넌트 문서화: Vue 컴포넌트의 props, emits, 주요 로직 및 컴포저블 함수에 대한 설명 주석.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/amaham1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/amaham1)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
