## project-woohyun

> "Project WOOHYUN" — 중학생 아들(우현)의 2학기 중간고사(2026-10-02) 대비 자기주도학습 게이미피케이션 웹앱. React 18 + TypeScript + Vite SPA, 로직 대부분이 `src/app/App.tsx` 한 파일에 있음. Supabase로 가족(엄마·아빠·아이) 간 데이터를 로그인 없이 실시간 공유하고, Vercel(`project-woohyun.vercel.app`)로 배포되어 실제 운영 중.

# CLAUDE.md

## 프로젝트 개요
"Project WOOHYUN" — 중학생 아들(우현)의 2학기 중간고사(2026-10-02) 대비 자기주도학습 게이미피케이션 웹앱. React 18 + TypeScript + Vite SPA, 로직 대부분이 `src/app/App.tsx` 한 파일에 있음. Supabase로 가족(엄마·아빠·아이) 간 데이터를 로그인 없이 실시간 공유하고, Vercel(`project-woohyun.vercel.app`)로 배포되어 실제 운영 중.

세션 단위 진행 상황(구현 완료 사항, 테스트 결과, 미해결 문제, 다음 작업)은 이 파일이 아니라 `docs/WORK_STATUS.md`에 기록한다.

## 아키텍처 / 파일 구조
- `src/app/App.tsx` — 앱의 거의 모든 화면·로직이 이 한 파일에 있음. 새 기능/버그 수정이 대부분 이 파일에 집중됨
- `src/lib/supabaseClient.ts` — Supabase 클라이언트 초기화(`VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY` 사용)
- `src/lib/familySync.ts` — 가족 코드(family id) resolve, RPC 기반 fetch/save
- `public/guide.html` — 아이용 사용법 정적 페이지
- `public/about.html` — 기획~배포 회고 + 운영 매뉴얼 정적 페이지
- `public/manifest.webmanifest`, `public/icon-*.png`, `public/apple-touch-icon.png`, `public/favicon-32.png` — PWA 아이콘
- `.env`(gitignored) / `.env.example`(템플릿) — Supabase 접속 변수
- **미사용 스캐폴딩(건드리지 말 것)**: `src/constants/`, `src/hooks/useTimer.ts`, `src/store/useAppStore.ts`, `src/types/`, `src/styles/globals.css`, `default_shadcn_theme.css` — App.tsx가 이들을 import하지 않음(확인됨). Figma Make/Codex 시절 산출물로 추정되며 실제 앱 동작과 무관

## 중요한 기술적 결정과 제약사항
- **패키지 매니저는 pnpm** — `npm install`은 이 환경에서 실패함. `pnpm-lock.yaml`/`pnpm-workspace.yaml` 존재
- **`vite build`는 esbuild 기반이라 TypeScript 타입 체크를 하지 않음** — tsconfig.json 자체가 없고 별도 typecheck 스크립트도 없음. 빌드 성공이 타입 정확성을 보장하지 않으니 타입 관련 변경 시 각별히 주의
- **GitHub↔Vercel 자동 배포 연결 안 되어 있음** — `git push`만으로는 사이트에 반영되지 않고, 매번 `vercel --prod`를 별도 실행해야 실제 배포됨
- 가족 공유는 계정/로그인이 아니라 URL 파라미터(`?fam=`) + localStorage 기반 "가족 코드" 방식. 링크를 아는 사람은 누구나 읽기/쓰기 가능(의도된 트레이드오프)
- Supabase 접근은 테이블 직접 권한 없이 `get_family_data`/`upsert_family_data` RPC(SECURITY DEFINER)로만 허용 — anon key가 노출돼도 정확한 가족 코드 없이는 데이터 열람 불가하도록 설계됨. 새 테이블/기능 추가 시 이 패턴을 유지할 것
- 사진은 클라이언트에서 압축(가로 최대 640px, JPEG quality 0.6) 후 저장
- 로컬 dev 서버에서도 Supabase 동기화가 실제로 동작함 — Playwright 등으로 로컬 테스트 시 실제 가족 데이터를 건드리지 않으려면 `**/*.supabase.co/**` 라우트를 차단할 것

## 개발 워크플로우
- 기능 요청 → 코드 작성 → 로컬에서 실제 브라우저(Playwright, 시스템 Chrome)로 동작 검증 → git commit/push → `vercel --prod` 배포까지 한 세션에서 처리하는 방식
- 별도 테스트 스위트 없음 — 매 기능마다 Playwright로 실제 시나리오를 만들어 검증. 스크립트는 스크래치패드에 임시 작성 후 삭제(저장소에 커밋 안 함)
- 매 변경 후 `pnpm run build` 성공을 확인하고 배포

---
> Source: [jhkl0514/project-woohyun](https://github.com/jhkl0514/project-woohyun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
