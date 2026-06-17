## speedometer

> Speedometer는 Vue 3와 Vite를 기반으로 구축된 현대적인 웹 애플리케이션 프로젝트임.

# Project GEMINI Context: Speedometer

## Project Overview
Speedometer는 Vue 3와 Vite를 기반으로 구축된 현대적인 웹 애플리케이션 프로젝트임.

- **Framework:** Vue 3 (Composition API, `<script setup>`)
- **Build Tool:** Vite
- **Routing:** Vue Router (Web History mode)
- **State Management:** Pinia
- **Styling:** CSS (Scoped styles, `assets/base.css`, `assets/main.css`)
- **Linting & Formatting:** ESLint, Oxlint, Prettier

## Directory Structure Highlights
- `src/`: 소스 코드 루트
    - `main.js`: 애플리케이션 엔트리 포인트
    - `App.vue`: 루트 컴포넌트
    - `components/`: 재사용 가능한 UI 컴포넌트
    - `views/`: 페이지 단위 컴포넌트 (Home, About 등)
    - `router/`: 라우팅 설정
    - `stores/`: Pinia 상태 저장소
    - `assets/`: 정적 자산 및 CSS 파일
- `public/`: 정적 파일 저장소 (Favicon 등)

## Building and Running
이 프로젝트에서 사용 가능한 주요 명령어는 다음과 같음:

- **의존성 설치:** `npm install`
- **개발 서버 실행:** `npm run dev`
- **프로덕션 빌드:** `npm run build`
- **빌드 결과물 미리보기:** `npm run preview`
- **린트 체크 (Oxlint + ESLint):** `npm run lint`
- **코드 포맷팅 (Prettier):** `npm run format`

## Development Conventions
- **최소 폰트 사이즈:** 프로젝트의 모든 텍스트는 최소 `text-sm` (14px) 이상이어야 함. `text-xs`나 `text-[10px]` 등 더 작은 사이즈는 절대 사용 금지.
- **파일 경로 별칭:** `vite.config.js`에 설정된 대로 `@` 기호를 사용하여 `src/` 디렉토리에 접근할 수 있음 (예: `import HelloWorld from '@/components/HelloWorld.vue'`).
- **컴포넌트 스타일:** Vue SFC(Single File Component) 내에서 `<style scoped>`를 사용하여 스타일 충돌을 방지함.
- **상태 관리:** 복잡한 상태 관리는 `src/stores/` 하위의 Pinia 스토어를 활용함.
- **라우팅:** 새로운 페이지 추가 시 `src/router/index.js`에 경로를 등록하며, 가급적 Lazy Loading(`import(...)`)을 권장함.
- **코드 품질:** Oxlint를 통해 빠른 정적 분석을 수행하고, ESLint와 Prettier를 통해 코드 스타일과 잠재적 버그를 관리함.

---
> Source: [R00neyj/Speedometer](https://github.com/R00neyj/Speedometer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
