## dungdong

> 이 프로젝트는 **FindingRoomMate**라는 이름의 Vue.js 기반 웹 애플리케이션입니다. Vite를 빌드 도구로 사용하며, Vuetify 프레임워크를 활용하여 UI를 구성하고 있습니다. Firebase를 백엔드로 사용하여 설문 데이터를 저장하고, vue-router를 통해 페이지 라우팅을 관리합니다.

# 프로젝트 분석 보고서: FindingRoomMate

## 1. 프로젝트 개요

이 프로젝트는 **FindingRoomMate**라는 이름의 Vue.js 기반 웹 애플리케이션입니다. Vite를 빌드 도구로 사용하며, Vuetify 프레임워크를 활용하여 UI를 구성하고 있습니다. Firebase를 백엔드로 사용하여 설문 데이터를 저장하고, vue-router를 통해 페이지 라우팅을 관리합니다.

주요 기능은 사용자의 생활 습관 및 선호도에 대한 설문조사를 통해 룸메이트 매칭에 필요한 정보를 수집하고, 이를 바탕으로 결과 이미지를 생성하는 것으로 보입니다.

## 2. 기술 스택

### 2.1. 프론트엔드

*   **Framework**: Vue.js 3
*   **UI Framework**: Vuetify 3
*   **Build Tool**: Vite
*   **Routing**: Vue Router 4
*   **HTTP Client**: Axios
*   **Styling**: Sass/SCSS
*   **Font**: Pretendard, Roboto, Material Design Icons

### 2.2. 백엔드

*   **Database**: Firebase Firestore

### 2.3. 개발 도구

*   **Linting**: ESLint
*   **Package Manager**: npm 또는 yarn

## 3. 프로젝트 구조

*   `src/`: 소스 코드 루트 디렉토리
    *   `main.js`: 애플리케이션 진입점
    *   `App.vue`: 최상위 Vue 컴포넌트
    *   `router/index.js`: 라우팅 설정
    *   `pages/`: 각 페이지 컴포넌트
        *   `Home.vue`: 시작 페이지
        *   `SurveyPage/`: 설문조사 페이지 (1-7)
        *   `End.vue`: 결과 페이지
    *   `components/`: 재사용 가능한 UI 컴포넌트
    *   `assets/`: 이미지, 폰트 등 정적 에셋
    *   `styles/`: SCSS 변수 및 설정
    *   `common/`: Firebase 연동 등 공통 모듈
    *   `plugins/`: Vuetify 등 플러그인 설정
*   `public/`: 정적 파일 (favicon 등)
*   `docs/`: 빌드 결과물이 저장되는 디렉토리
*   `vite.config.mjs`: Vite 설정 파일
*   `package.json`: 프로젝트 의존성 및 스크립트 정의

## 4. 주요 기능 분석

### 4.1. 설문조사 기능

*   총 7단계의 설문조사 페이지(`SurveyPage1` ~ `SurveyPage7`)로 구성되어 있습니다.
*   `App.vue`에서 설문 진행 상태를 추적하고, 페이지 이동(뒤로/다음/완료)을 관리합니다.
*   설문 데이터는 `localStorage`에 `userSurvey`라는 키로 임시 저장됩니다.
*   설문 항목은 다음과 같습니다:
    *   기숙사, 생년, 학번, 단과대, MBTI
    *   흡연 및 음주 습관
    *   기상/소등/취침 시간
    *   잠버릇, 청소, 벌레 민감도, 실내 취식, 소음 민감도, 공유 성향, 귀가 주기
    *   해시태그 및 기타 참고사항

### 4.2. 데이터 관리

*   **Firebase 연동**: `src/common/Firebase.js`에서 Firebase를 초기화하고, `App.vue`에서 설문 결과를 Firestore의 `surveys` 컬렉션에 저장 및 업데이트합니다.
*   **로컬 스토리지**: 사용자의 진행 상황(`userProgress`)과 설문 내용(`userSurvey`)을 로컬 스토리지에 저장하여 페이지를 새로고침해도 데이터가 유지되도록 합니다.

### 4.3. 라우팅

*   `vue-router`를 사용하여 해시 기반(#) 라우팅을 구현했습니다.
*   `/` 경로는 `/home`으로 리다이렉트됩니다.
*   설문조사 페이지, 시작 페이지, 결과 페이지에 대한 경로가 정의되어 있습니다.

## 5. 개선 제안

*   **컴포넌트 분리**: `App.vue`에 많은 로직(상태 관리, 이벤트 핸들링, Firebase 연동 등)이 집중되어 있습니다. 이를 Pinia와 같은 상태 관리 라이브러리를 도입하거나, Composition API의 `provide/inject`를 활용하여 관심사를 분리하면 코드의 유지보수성과 가독성을 높일 수 있습니다.
*   **입력 유효성 검사**: 각 설문 단계에서 사용자 입력에 대한 유효성 검사를 추가하여 데이터의 정합성을 보장할 수 있습니다.
*   **에러 핸들링**: Firebase 연동 실패 또는 API 요청 실패 시 사용자에게 적절한 피드백을 제공하는 에러 핸들링 로직을 강화할 필요가 있습니다.
*   **코드 정리**: `console.log`와 같은 디버깅용 코드는 프로덕션 빌드에서 제거하는 것이 좋습니다.
*   **보안**: Firebase API 키와 같은 민감한 정보는 `.env` 파일을 통해 관리되고 있으나, 클라이언트 사이드 코드에 직접 노출되는 것은 보안상 취약할 수 있습니다. Firebase의 보안 규칙(Security Rules)을 더욱 정교하게 설정하여 데이터 접근을 제어하는 것이 중요합니다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GetOurRI)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/GetOurRI)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
