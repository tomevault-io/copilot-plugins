## k-daejeop

> 카카오맵 API + Supabase를 활용한 한국 지도 기반 **폐쇄형 맛집 리뷰 서비스**.

# K-daejeop — AI Context Guide

## 프로젝트 개요
카카오맵 API + Supabase를 활용한 한국 지도 기반 **폐쇄형 맛집 리뷰 서비스**.
FastAPI 백엔드 + Jinja2 SSR + Vanilla JS 프론트엔드 구조.
사용자가 음식점을 검색하고 별점(1~3)·사진·리뷰를 남기면, 10개 이상 쌓은 뒤 다른 사용자의 추천 맛집을 볼 수 있음.

## 기술 스택 & 버전
- Python 3.9 / FastAPI 0.111 / Uvicorn 0.30
- httpx 0.27 (카카오 REST API 호출)
- Jinja2 3.1 (서버사이드 템플릿)
- pydantic 1.x BaseSettings (환경 변수 관리)
- PyJWT 2.8 (Supabase JWT HS256 검증)
- Supabase JS SDK v2 (프론트엔드 인증 + DB + Storage)
- 카카오맵 JavaScript SDK (지도 렌더링)
- pytest 8.2 + pytest-asyncio 0.23 (테스트)
- uv (패키지 매니저)

## 아키텍처 핵심 원칙

### 보안: 서버 프록시 패턴
- 카카오 REST API 키(`KAKAO_REST_KEY`)는 서버에서만 사용. 클라이언트 → `/api/places` → 서버 → 카카오 API.
- 카카오 JS 키(`KAKAO_JS_KEY`)는 지도 렌더링용으로 브라우저에 노출됨 (정상).
- Supabase `anon key`는 공개(public)키로 브라우저에서 사용. `JWT_SECRET`은 서버에서만 사용.

### 인증 흐름
1. 프론트엔드: Supabase JS SDK → `signInWithOAuth({ provider: 'google' })` → Google 동의 → Supabase 콜백 → 세션 획득
2. 프론트엔드: `onAuthStateChange` → 로그인 화면 ↔ 앱 화면 전환
3. 백엔드 보호 API 호출 시: `Authorization: Bearer <access_token>` → `core/auth.py`의 `verify_supabase_token`이 PyJWT로 HS256/audience=authenticated 검증
4. `get_current_user` (필수 인증) / `get_optional_user` (선택 인증) FastAPI 의존성

### 리뷰 시스템 (Supabase 직접 접근)
- 리뷰 CRUD는 프론트엔드에서 Supabase JS SDK로 직접 수행 (RLS가 보안 담당)
- 사진은 Supabase Storage `review-photos` 버킷에 업로드 (경로: `{user_id}/{timestamp}.{ext}`)
- `reviews.js`에서 `window.__getSupabase()`로 supabase 클라이언트 접근
- DB 스키마: `supabase/migrations/001_reviews_and_profiles.sql`

### 환경 변수 분리
- `APP_ENV` 환경 변수로 `development` / `production` 구분
- 설정 로딩: `.env.{APP_ENV}` → `.env` 순으로 fallback
- `.env.*` 파일은 `.gitignore`에 포함, `.env.*.example`은 추적됨
- Settings는 `@lru_cache`로 싱글턴 캐싱

### 지도 초기화 타이밍
- 로그인 전에는 `#app`이 `hidden` → 카카오맵 컨테이너 크기 0px
- `auth.js`가 로그인 완료 후 `app:visible` 커스텀 이벤트 발행
- `main.js`가 `app:visible`을 수신하여 지도 초기화 또는 `relayout()` + 재센터링
- 이 순서를 바꾸면 지도 중심이 일본으로 밀림 (절대 변경 금지)

## 디렉토리 구조 & 역할

```
app/
├── main.py              # create_app() 팩토리, 라우터 등록, static 마운트
├── core/
│   ├── config.py        # Settings(BaseSettings), get_settings(), _resolve_env_file()
│   └── auth.py          # verify_supabase_token, get_current_user, get_optional_user
├── routers/
│   ├── health.py        # GET /api/health
│   ├── auth.py          # GET /api/auth/me (JWT 필수)
│   ├── pages.py         # GET / (Jinja2 렌더링, 캐시 버스터)
│   └── places.py        # GET /api/places?query= (카카오 프록시)
├── services/
│   └── kakao.py         # KakaoPlacesClient (httpx 기반)
├── static/
│   ├── auth.js          # Supabase 인증, 화면 전환, 튜토리얼 상태 체크
│   ├── main.js          # 지도 초기화, 검색, 자동완성, 마커, 리뷰 버튼
│   ├── reviews.js       # 리뷰 모달 (별점·사진·날짜·텍스트), Supabase CRUD
│   ├── tutorial.js      # 온보딩 튜토리얼 5단계 카드
│   └── styles.css       # 디자인 토큰, 로그인, 앱, 리뷰, 튜토리얼 스타일
└── templates/
    └── index.html       # login-screen + app + review-modal + tutorial
supabase/
└── migrations/
    └── 001_reviews_and_profiles.sql  # reviews, user_profiles, storage 스키마
```

## 모듈 간 통신

### JS 모듈 공유 패턴
- `auth.js`: supabase 클라이언트 생성 → `window.__getSupabase()` 노출
- `auth.js`: 튜토리얼 저장 → `window.__markTutorialSeen()` 노출
- `main.js`: 리뷰 모달 열기 → `window.__openReviewModal(place)` 호출
- 커스텀 이벤트: `app:visible`, `tutorial:show`, `review:saved`

### Supabase 테이블 (RLS 적용)
- `reviews`: 별점(1~3), 사진URL, 리뷰텍스트, 방문일자
- `user_profiles`: tutorial_seen 플래그, 자동 생성 트리거
- Storage `review-photos`: 유저별 폴더, 공개 읽기

## 코딩 규칙

### Python
- `from __future__ import annotations` 모든 파일 상단에
- pydantic 1.x 스타일 (`Field(env=...)`, `class Config`)
- FastAPI 의존성 주입: `Depends(get_settings)`, `Depends(get_current_user)`
- 비동기 우선 (`async def`, `httpx.AsyncClient`)
- 테스트: 변경 시 반드시 관련 테스트 추가/수정, 테스트 실행 후 전체 통과 확인

### JavaScript (프론트엔드)
- Vanilla JS (프레임워크 없음), ES module (`type="module"`)
- 카카오맵 SDK는 전역 `window.kakao` 사용 (ES module 아님)
- `escapeHtml()` 필수 사용 (XSS 방지)
- 디바운스 200ms (`DEBOUNCE_MS`)로 자동완성 API 호출 제어
- `searchSeq` 카운터로 stale 응답 무시 (race condition 방지)
- 토스트는 한 번에 하나만 표시, `clearTimeout`으로 이전 타이머 정리

### CSS
- 디자인 토큰은 `:root` CSS 변수로 관리
- BEM-like 클래스 네이밍 (`block__element`, `block--modifier`)
- `.is-*` 상태 클래스 (`is-visible`, `is-open`, `is-active`, `is-leaving`)
- z-index: 헤더 9000, 자동완성 9999, 로그인 화면 10000, 리뷰 모달 10001~10002, 튜토리얼 10010

### 테스트
- pytest + pytest-asyncio (asyncio_mode = "auto")
- conftest.py에서 mock 환경변수 설정 (`os.environ.setdefault`)
- `get_settings.cache_clear()` 매 테스트마다 자동 실행 (autouse fixture)
- 외부 API 호출 테스트(Supabase 연결)는 `test_supabase_connection.py`에 분리

## 주의사항

1. **pydantic 버전**: 1.x 사용 중. `BaseSettings`는 `pydantic` 패키지에서 직접 import. 2.x의 `pydantic-settings`가 아님.
2. **Supabase JWT**: HS256 알고리즘, audience="authenticated". RS256 아님.
3. **지도 init 순서**: `app:visible` 이벤트 후에만 initMap 호출. DOMContentLoaded에서 호출하면 안 됨.
4. **캐시 버스팅**: `pages.py`의 `_CACHE_BUSTER`(서버 시작 타임스탬프)를 CSS/JS URL에 `?v=` 파라미터로 붙임.
5. **한글 IME**: `keydown`에서 `keyCode === 229` 체크 필수. `input` 이벤트는 IME 완료 후에도 발생하므로 자동완성 트리거로 적합.
6. **테스트 필수**: 모든 변경 후 `uv run pytest tests/ -v` 실행. `test_supabase_connection.py`는 실제 네트워크 필요 시 `--ignore`로 제외 가능.
7. **리뷰 모달 z-index**: 오버레이 10001, 모달 10002. 로그인 화면(10000)보다 위.
8. **Supabase 직접 접근**: 리뷰 CRUD와 사진 업로드는 프론트엔드에서 Supabase JS SDK로 직접 수행. RLS가 보안 담당.

## 향후 확장 예정
- 리뷰 10개 달성 시 다른 사용자 추천 맛집 노출 기능
- 즐겨찾기, 검색 기록 기능
- 리뷰 목록 / 편집 / 삭제 UI

---
> Source: [Anturex/K-daejeop](https://github.com/Anturex/K-daejeop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
