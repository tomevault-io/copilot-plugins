## gobang-linktree

> 이 문서는 상위 [../../CLAUDE.md](../../CLAUDE.md)의 규칙을 상속합니다. 신규 작업(새 슬라이스, 리팩터, 버그 수정) 시작 전 반드시 상위 문서의 4단계 파이프라인(기획·grill-me → 디자인 → 이슈 분할 → 리소스 승인 → 서브에이전트 TDD)을 그대로 따르십시오. 아래는 이 프로젝트에만 적용되는 구체 규칙·함정입니다.

# 프로젝트 규칙 (CLAUDE.md) — 고방 링크트리

이 문서는 상위 [../../CLAUDE.md](../../CLAUDE.md)의 규칙을 상속합니다. 신규 작업(새 슬라이스, 리팩터, 버그 수정) 시작 전 반드시 상위 문서의 4단계 파이프라인(기획·grill-me → 디자인 → 이슈 분할 → 리소스 승인 → 서브에이전트 TDD)을 그대로 따르십시오. 아래는 이 프로젝트에만 적용되는 구체 규칙·함정입니다.

## 문서 맵
- **스펙**: [docs/SPEC.md](docs/SPEC.md)
- **디자인 시스템(SoT)**: [docs/DESIGN_SYSTEM.md](docs/DESIGN_SYSTEM.md)
- **개발 슬라이스 TODO**: [docs/TODO.md](docs/TODO.md)
- **진행원장(로컬, gitignore)**: `.superpowers/sdd/progress.md`

## 프로젝트 기본 정보
- **스택**: Next.js 16 (App Router, Turbopack) + TypeScript + **Tailwind v4**(설정은 `tailwind.config.js` 없이 `src/app/globals.css`의 `@theme`) + Supabase + Vercel. 폰트 Pretendard 단일.
- **로컬 포트**: **3939** (`시작 3939.bat`, `next dev -p 3939` 하드코딩). 새 하위 프로젝트를 만들 때도 이 포트와 중복 금지.
- **배포**: GitHub `NFmkt/gobang-linktree`(public) + Vercel `https://gobang-linktree.vercel.app` (GitHub 연동, `main` push 시 자동 배포).
- **검증 커맨드**:
  - 테스트: `npx vitest run`
  - 린트: `npm run lint` (`next lint`는 Next 16에서 제거됨 — `eslint` 직접 실행)
  - 빌드: `npx next build`
  - 프리뷰 서버명(launch.json): `gobang-linktree` (port 3939)
- **`.env.local` 필요**: Supabase URL/anon/service_role + `ADMIN_PASSWORD` + `GAS_AFFILIATE_WEBHOOK_URL`(제휴 문의 폼 → Google Apps Script 웹훅, 없으면 해당 API가 500). gitignore됨 — 새 세션에서는 `.env.example` 참고해 재구성하거나 기존 값을 전달받을 것.

## 이 프로젝트만의 함정 (반드시 지킬 것)
- **`middleware.ts`가 아니라 `src/proxy.ts`**: Next.js 16.2.10부터 "middleware" 컨벤션이 deprecated, "proxy"로 대체됨(파일명·export 함수명 둘 다). 관리자 라우트(`/admin/*`, `/api/admin/*`) 보호 로직이 여기 있음.
- **디자인 토큰은 전부 `src/app/globals.css`**의 `:root`/`@theme`. 컴포넌트는 `bg-[var(--color-...)]` 형태 arbitrary value로만 참조. 색을 바꾸려면 여기부터.
- **아이콘은 stroke 금지, fill 전용 다색**(`@icon-design` 스킬 계약, `viewBox 0 0 20 20`). 브랜드 인지용 소셜 아이콘(네이버 그린/인스타 마젠타 등)만 팔레트 확장 예외.
- **사용자 입력 URL을 `<a href>`로 렌더할 때는 항상 `src/lib/links/isSafeLinkUrl.ts` 같은 스킴 화이트리스트 검증**: React의 자동 escape는 텍스트 콘텐츠에만 적용되고 `javascript:`/`data:` 스킴은 막아주지 않음.
- **`service_role`도 RLS 우회일 뿐 GRANT를 자동으로 갖지 않음**: 새 Supabase 테이블 추가 시 `anon`뿐 아니라 `service_role`에도 명시적 `grant`를 마이그레이션에 포함할 것 (누락 시 로컬·배포 모두에서 `permission denied for table X` 재현됨).
- **Supabase 마이그레이션 적용**: DB 비밀번호가 없어 `supabase db push` CLI 불가 — 대시보드 SQL Editor에 마이그레이션 파일 내용을 직접 붙여넣어 실행.
- **React list `key`는 실제 유일값 사용**: 삭제된 항목을 공통 폴백 라벨("삭제된 링크" 등)로 표시하는 집계에서는 라벨이 중복될 수 있음 — `key={`${label}-${index}`}` 형태로.
- **넓은 날짜 범위/무제한 리스트를 그리는 차트는 포인트 수 상한을 둘 것**: `LineChart`가 `MAX_MARKER_POINTS` 초과 시 개별 마커 렌더를 생략하는 패턴(`src/components/admin/LineChart.tsx`) 참고 — 통계 등 범위가 커질 수 있는 값을 시각화할 때 이 패턴을 재사용.
- **Server Component에서 `try{...}` 블록 안에 JSX를 직접 반환하지 말 것**: ESLint `react-hooks/error-boundaries` 위반. 데이터 fetch/에러 캡처만 try 안에서 하고, JSX 반환은 try/catch 밖에서 조건 분기.
- **App Router에서 폴더명이 `_`로 시작하면 private folder 취급 → 라우트 미생성**(빌드 에러도 없이 조용히 404). 임시 진단 라우트에도 언더스코어 접두 피할 것.
- **OG 이미지 한글 렌더**: Satori 기본 폰트에 한글 없음 → `src/app/fonts/`의 Pretendard `.otf`를 fs로 직접 읽어 넘김(node_modules 직접 참조 금지, 서버리스 번들 누락 위험).
- **Tailwind 스캔 범위**: `globals.css`가 `@import "tailwindcss" source("..")`로 `src/`만 스캔하도록 제한돼 있음. 이 줄을 지우면 리포 루트 `.md` 문서에 적힌 Tailwind 문법 예시(예: `bg-[var(--color-...)]`)까지 후보 클래스로 오인해 CSS 파싱이 깨지고 dev 서버가 500을 낼 수 있다.
- **공개 페이지가 읽는 설정/데이터 조회는 항상 `select("*")`를 쓸 것, 명시적 컬럼 리스트 금지**: Supabase 마이그레이션은 CLI push가 안 되고 대시보드에서 수동 적용하는 구조라, "코드는 배포됐지만 마이그레이션은 아직 안 적용됨" 상태가 실제로 발생한다(`getSiteConfig.ts`가 신규 컬럼을 명시적으로 select하고 있어서 이 순서 문제 하나로 공개 홈페이지 전체가 500 났던 사례 있음, `?? 기본값`으로 폴백). 관리자 전용 API(`select("*")` 이미 관행)는 이 문제에 안전하지만 공개 페이지 쪽 데이터 조회 함수를 새로 만들거나 컬럼을 추가할 때 반드시 이 패턴을 따를 것.
- **recharts 컴포넌트 테스트**: jsdom은 `ResizeObserver`/실측 `getBoundingClientRect`가 없어서 `<ResponsiveContainer>`가 렌더를 포기함 — `LineChart.test.tsx`/`MediumDonutChart.test.tsx`의 `MockResizeObserver` + `getBoundingClientRect` 오버라이드(파일 스코프 `beforeAll`/`afterAll`) 패턴을 그대로 재사용할 것. 새 recharts 컴포넌트를 테스트할 때 매번 새로 고안하지 말 것(현재 파일마다 중복돼 있음 — 공유 헬퍼로 뽑는 건 후속 과제).
- **공개 익명 POST 폼의 스팸 방지 패턴**: 허니팟 hidden input(off-screen 배치, `aria-hidden`+`tabIndex=-1`, `display:none` 대신) + 폼 렌더~제출 사이 최소 시간(현재 3초) 검증. 걸리면 400이 아니라 **조용히 200**을 반환하고 실제 처리(웹훅/DB 등)는 건너뛴다(`/api/affiliate-inquiries` 참고) — 스팸 봇에게 실패를 알려주지 않는 게 의도.

## 알려진 이슈 (해결됨)
- ~~`ADMIN_PASSWORD`(구 값 `nfmkt`)가 과거 `HANDOFF.md` 커밋들을 통해 public repo git 히스토리에 노출됨~~ → **사용자가 비밀번호를 새 값으로 로테이션해서 해결(2026-07-20)**. 옛 값은 git 히스토리에 여전히 남아있지만 이미 무효한 값이라 실질적 위험 없음. 히스토리 자체에서 문자열을 지우는 건(rewrite) 위생 차원의 선택사항으로 남아있음(필요 시 제안).

## 폴더 구조
Next.js App Router 표준(`src/app`, `src/components`, `src/lib`) + `docs/`(스펙·설계·계획) + `supabase/migrations/`(SQL) + `public/`(정적 에셋). 상위 CLAUDE.md의 `src/`·`tests/`·`docs/`·`public/` 일반 원칙보다 이 프레임워크 표준 구조가 우선.

---
> Source: [NFmkt/gobang-linktree](https://github.com/NFmkt/gobang-linktree) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
