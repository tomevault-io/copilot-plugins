## law-solver

> 이 문서는 Codex 및 다른 AI 코딩 에이전트가 `law-solver` 저장소에서 작업할 때 따라야 할 프로젝트 지침입니다.

# AGENTS.md

이 문서는 Codex 및 다른 AI 코딩 에이전트가 `law-solver` 저장소에서 작업할 때 따라야 할 프로젝트 지침입니다.

## 프로젝트 개요

`law-solver`는 서버 없이 브라우저에서 실행되는 React 기반 로스쿨 문제 풀이 앱입니다. 사용자는 과목별 대시보드에서 CSV 파일을 업로드해 OX, 5지선다, 단답형 문제 세션을 만들고, CBT 화면에서 풀이한 뒤 결과 확인, 오답노트, 재풀이, CSV 내보내기, JSON 백업/복원을 수행합니다.

데이터 저장은 서버나 DB가 아니라 브라우저 localStorage를 사용합니다.

- 세션 저장 key: `law-solver-storage`
- 환경설정 저장 key: `law-solver-settings`

## 기술 스택

- React 18 + Functional Components + Hooks
- TypeScript strict mode
- Vite
- Tailwind CSS
- React Router
- Zustand + persist middleware
- Papa Parse
- Vitest

## 주요 디렉터리별 역할

```txt
src/App.tsx
```

라우팅과 테마/폰트 watcher를 담당합니다. 라우트는 `/`, `/dashboard`, `/dashboard/:subjectId`, `/solve/:sessionId`, `/result/:sessionId`, `/wrong/:sessionId`, `/review/:sessionId`입니다.

```txt
src/pages/
```

라우트 단위 화면입니다. 랜딩, 과목 목록, 과목별 대시보드, 풀이 결과, 오답 확인, 전체 리뷰 페이지가 들어 있습니다.

```txt
src/components/
```

재사용 UI 컴포넌트입니다. CBT 풀이 화면, CSV 업로드 패널, 리뷰용 선택지 표시 컴포넌트, 공통 모달/버튼 UI가 들어 있습니다.

```txt
src/store/
```

Zustand store입니다. 문제 세션, 과목, 세션-과목 매핑 상태는 `useTestStore.ts`, 다크 모드와 글꼴 설정은 `useSettingsStore.ts`에서 관리합니다.

```txt
src/lib/
```

CSV 파싱/다운로드, 채점, 정렬, 답안 표시, ID 생성, 시간 포맷 등 도메인 유틸입니다. 관련 단위 테스트도 이 디렉터리에 있습니다.

```txt
src/types/
```

문제 타입, 세션 타입, 답안 타입 등 공유 TypeScript 타입입니다.

```txt
samples/
```

수동 테스트와 사용자 예시용 CSV 파일입니다.

```txt
public/
```

GitHub Pages용 정적 파일입니다. `404.html`은 SPA 새로고침 대응용이며, `CNAME`은 커스텀 도메인용입니다.

## 코드 스타일과 네이밍 규칙

- TypeScript strict mode를 전제로 작성합니다.
- React 컴포넌트는 함수 컴포넌트와 Hooks를 사용합니다.
- 컴포넌트 파일은 `PascalCase.tsx`를 사용합니다.
- 유틸 함수 파일은 `camelCase` 또는 기능명 기반 짧은 이름을 사용합니다. 예: `csv.ts`, `order.ts`, `session.ts`.
- Zustand store는 `useSomethingStore.ts` 형태를 유지합니다.
- Tailwind CSS 유틸 클래스를 우선 사용하고, 전역 CSS는 `src/index.css`에 한정합니다.
- 서버 API가 없으므로 브라우저 API 사용 시 호환성을 고려합니다. 예: `crypto.randomUUID()` 직접 호출 대신 `src/lib/id.ts`의 `createId()` 사용.
- CSV 헤더 호환성은 `src/lib/csv.ts`의 `normalize`, `getValue` 흐름을 기준으로 확장합니다.
- localStorage 데이터 구조를 바꿀 때는 기존 사용자 데이터와 마이그레이션 영향을 고려합니다.
- `과목 없음`은 저장되는 subject가 아니라 세션-과목 매핑이 없는 상태입니다. `NO_SUBJECT_ID`는 라우팅/UI용 sentinel로만 사용하세요.

## 작업 전 확인해야 할 명령어

저장소 구조 확인:

```bash
rg --files
```

스크립트 확인:

```bash
sed -n '1,220p' package.json
```

라우팅 확인:

```bash
sed -n '1,220p' src/App.tsx
```

주요 타입 확인:

```bash
sed -n '1,260p' src/types/test.ts
```

CSV 관련 작업 전 확인:

```bash
sed -n '1,320p' src/lib/csv.ts
```

상태 저장 관련 작업 전 확인:

```bash
sed -n '1,280p' src/store/useTestStore.ts
sed -n '1,220p' src/store/useSettingsStore.ts
```

## 변경 후 실행해야 할 검증 명령어

기본 검증:

```bash
npm test
npm run build
```

개발 서버 확인:

```bash
npm run dev
```

빌드 결과 미리보기:

```bash
npm run preview
```

린트:

```bash
npm run lint
```

현재 `package.json`에는 `lint` 스크립트가 없습니다. 린트 검증은 “확인 필요”로 표시하거나, 별도 작업으로 ESLint 설정을 추가해야 합니다.

## 수정 시 주의할 파일과 패턴

- `src/lib/csv.ts`: CSV 업로드/다운로드, Excel 인코딩, 샘플 CSV와 직접 연결됩니다. 헤더 호환성을 깨지 않게 조심하세요.
- `src/types/test.ts`: localStorage에 저장되는 세션 구조와 연결됩니다. 필드 변경 시 기존 저장 데이터 호환성을 검토하세요.
- `src/store/useTestStore.ts`: 세션 생성, 과목 CRUD, 세션-과목 매핑, 답안 저장, 오답노트, 북마크, 백업/복원 동작의 중심입니다.
- `src/components/cbt/CbtSolveScreen.tsx`: 풀이 UX, 타이머, OMR, 단답형 입력, 정답 보기, 책갈피 기능이 모여 있습니다.
- `src/pages/ResultPage.tsx`: 재풀이, CSV 다운로드, 결과 요약 액션이 많아 회귀 가능성이 큽니다.
- `src/pages/SubjectListPage.tsx`: `/dashboard` 과목 목록 화면입니다. 과목 관리, 표지 색상 선택, 과목 카드 드래그 순서 변경, 환경설정, 전체 데이터 백업/복원/초기화가 이 페이지에 있습니다. 과목 삭제는 세션 삭제가 아니라 매핑 삭제로 처리해야 합니다.
- `src/pages/DashboardPage.tsx`: `/dashboard/:subjectId` 과목별 세션 대시보드입니다. 새 문제 등록과 편집 시 세션-과목 매핑이 맞는지 확인하세요. 전체 데이터 백업/복원 UI는 이 페이지에 두지 않습니다.
- `public/404.html` 및 `index.html`: GitHub Pages SPA 새로고침 대응 스크립트가 들어 있습니다. 라우팅/배포 변경 시 함께 확인하세요.
- `vite.config.ts`: GitHub Pages 기본 도메인과 커스텀 도메인 운영 방식에 따라 `base` 설정 영향이 큽니다.
- `public/CNAME`: 현재 커스텀 도메인 `lawsolver.haryun.io`가 설정되어 있습니다. 기본 GitHub Pages URL로 운영할 때는 이 파일과 DNS 상태를 확인하세요.

## 환경변수와 시크릿 취급

현재 필수 환경변수는 없습니다.

이 프로젝트는 프론트엔드 전용 앱이므로 빌드된 JS가 사용자에게 그대로 전달됩니다. API 키, 토큰, 개인정보, 서버용 시크릿을 코드나 `.env`에 넣지 마세요. Vite 환경변수를 추가해야 한다면 클라이언트에 공개되어도 되는 값만 `VITE_` 접두사로 사용하세요.

## 배포

GitHub Actions 워크플로우는 `.github/workflows/deploy-pages.yml`입니다.

- 트리거: `main` 브랜치 push, 수동 실행
- 설치: `npm ci`
- 빌드: `npm run build`
- 산출물: `dist`
- 배포 대상: GitHub Pages

SPA 라우트 새로고침은 `public/404.html`과 `index.html`의 redirect restore 스크립트로 처리합니다.

## localStorage 마이그레이션과 백업

`law-solver-storage`는 Zustand persist `version: 2`를 사용합니다.

- `sessions`: 문제 세션과 답안
- `subjects`: 사용자가 만든 과목 목록, 표지 색상, 표시 순서
- `sessionSubjectMap`: `sessionId -> subjectId` 매핑

기존 v1 데이터는 `sessions`만 있었기 때문에 마이그레이션 시 `subjects: []`, `sessionSubjectMap: {}`로 보정합니다. 즉 기존 문제는 모두 `과목 없음`으로 표시됩니다.

백업/복원은 특정 과목 단위가 아니라 전체 데이터베이스 단위입니다. 백업 복원은 두 형식을 모두 지원해야 합니다.

- 구형: `TestSession[]`
- 신형: `{ app, version, exported_at, sessions, subjects, sessionSubjectMap }`

복원 시 존재하지 않는 sessionId 또는 subjectId를 가리키는 매핑은 버려야 합니다.

## 커밋/PR 작성 권장 형식

커밋 메시지는 변경 의도가 바로 보이게 짧게 작성합니다.

예시:

```txt
docs: update project setup and agent guide
fix: handle csv export encoding for excel
feat: add dashboard backup restore
test: cover question ordering modes
```

PR에는 다음을 포함하는 것을 권장합니다.

- 변경 요약
- 사용자 흐름 영향
- 실행한 검증 명령어와 결과
- 스크린샷 또는 화면 변경 설명
- localStorage 데이터 구조 변경 여부
- 배포 설정 변경 여부

## 작업 원칙

- 문서 작업은 실제 파일과 설정을 읽은 뒤 반영합니다.
- 큰 리팩터링은 문서 정리 작업과 섞지 않습니다.
- README와 샘플 CSV의 설명이 실제 파서 동작과 맞는지 확인합니다.
- 브라우저 저장소 의존 기능은 백업/복원 UX와 데이터 손실 가능성을 함께 고려합니다.
- GitHub Pages 관련 수정은 커스텀 도메인과 프로젝트 경로 배포의 차이를 명확히 확인한 뒤 진행합니다.

---
> Source: [haryunio/law-solver](https://github.com/haryunio/law-solver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
