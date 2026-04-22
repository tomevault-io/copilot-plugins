## bcsd-internal-recode

> - **작업 단위마다 커밋+푸시**: 하나의 작업이 끝나면 즉시 커밋하고 push한다. 여러 변경을 모아두지 말 것.

# CLAUDE.md — BCSD_INTERNAL_RECODE (프론트엔드)

## 워크플로우

- **작업 단위마다 커밋+푸시**: 하나의 작업이 끝나면 즉시 커밋하고 push한다. 여러 변경을 모아두지 말 것.
- **커밋 전 lint 필수**: `pnpm lint`를 반드시 실행하고, 에러가 없을 때만 커밋한다.
- 커밋 메시지는 한국어로, conventional commits 형식 사용.
- **shadcn MCP 우선**: UI 컴포넌트/패턴이 필요할 때 shadcn MCP를 먼저 검색하여 활용. 없을 때만 직접 구현.

## 역할 분리

- **이 프로젝트는 프론트엔드 전용**이다. 백엔드(`BCSD_API`)를 직접 수정하지 마라.
- 백엔드에 변경이 필요하면 **요구사항을 정리하여 사용자에게 전달**하라. 사용자가 백엔드 에이전트에 넘긴다.

## 확정된 요구사항

### 인증
- Google OAuth 로그인 (Google ID Token → `POST /v1/auth/login` → httpOnly cookie)
- 회원가입 4단계 위저드: Google 인증 → 정보 입력(이름 수정 + 전화번호) → 학교 이메일 인증(SMTP) → 트랙 선택
- 전화번호: **입력만** (인증 없음). Firebase 전화 인증 제거됨
- **Firebase 사용 안 함** — 프론트엔드에서 Firebase 의존성 완전 제거

### 토큰 관리
- JWT를 **httpOnly cookie**로 관리 (백엔드에서 `Set-Cookie` 헤더로 설정)
- 프론트에서 토큰 직접 접근 불가 (XSS 방어)
- 인증 상태 확인: `GET /v1/auth/me`
- 로그아웃: `POST /v1/auth/logout` (서버에서 쿠키 삭제)
- **Refresh token 없음** — access token 24시간 만료 후 Google 재로그인

### 보안
- CSP 메타 태그 적용 (`index.html`)
- axios 401 인터셉터에서 `/v1/auth/` 엔드포인트는 리다이렉트 제외 (무한 루프 방지)

### 멤버 관리
- 멤버 목록: 테이블 + 필터(트랙/상태/팀/납부) + 이름 검색(디바운스) + 페이지네이션
- 필터 상태는 URL 파라미터(`useSearchParams`)로 관리
- 멤버 상세: 행 클릭 → `/members/:memberId`

### 기술 스택
- React 19, TypeScript, Vite 7, Tailwind CSS v4
- shadcn/ui (base-ui 기반, v4)
- TanStack React Query, React Router DOM 7
- axios, @react-oauth/google, lucide-react, sonner

### 백엔드 API (현재 상태, 수정 요청은 별도 전달)
- `POST /v1/auth/login` — `{ google_token }` → httpOnly cookie 설정 + `{ access_token, token_type }`
- `GET /v1/auth/me` — 쿠키로 인증 → `{ id, email }`
- `POST /v1/auth/logout` — 쿠키 삭제
- `POST /v1/auth/verify-email` — `{ email }` → `{ message }`
- `POST /v1/auth/confirm-email` — `{ email, code }` → `{ verified }`
- `POST /v1/auth/register` — `{ google_token, name, school_email, phone, track }` → httpOnly cookie 설정
- `GET /v1/members/filters` — `{ tracks: string[], statuses: string[], payment_statuses: string[] }`
- `GET /v1/members` — 필터/페이지네이션 → `{ items, total, page, size }`
- `GET /v1/members/{member_id}` → 상세 정보

### 백엔드 수정 요청 (미반영)
- 없음

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BCSDLab) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
