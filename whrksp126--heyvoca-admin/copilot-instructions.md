## heyvoca-admin

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md — heyvoca_admin

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

heyvoca 어드민 (Flask 3 백엔드 + React SPA 프론트). 자체 비즈니스 로직은 거의 없고, heyvoca_back 의 `/admin/*` API 를 호출하는 thin client (Flask 프록시가 `X-Admin-API-Key` 주입 → 키 브라우저 비노출). 배포는 ghmate 홈서버 Docker (dev/stg/prod) — `./deploy.sh dev|stg|prod`.

---

## Commands

```bash
# 로컬 (docker) — 컨테이너 heyvoca_admin_local (포트 5101)
#   프론트는 host 에서 `cd frontend && npm run build` 후 볼륨 마운트
docker compose -f docker-compose.local.yml up --build -d

# 배포 (홈서버 docker — heyvoca_service 와 동일 방식: SSH → git pull → up --build)
./deploy.sh dev
./deploy.sh stg
./deploy.sh prod
```

---

## Architecture

Flask 3.0 + SQLAlchemy 2.0 + Flask-Login (인증/세션) + React SPA(`frontend/`, Vite + React + Tailwind). Flask 는 얇은 게이트웨이 역할:
1. 세션 인증 (`/auth/*`, Admin 테이블 + Flask-Login)
2. 보안 프록시 (`/api/<path>` → heyvoca_back `/admin/<path>`, `X-Admin-API-Key` 주입)
3. SPA 서빙 (catch-all → `app/static/spa/index.html`)

### 진입점

| 파일 | 용도 |
|------|------|
| `run.py` | gunicorn/로컬 진입점. Dockerfile·compose CMD = `gunicorn ... run:app` |

### 디렉토리 구조

```
heyvoca_admin/
├── app/
│   ├── __init__.py          # Flask 앱 팩토리 (create_app), Blueprint 등록, login_manager.user_loader
│   ├── extensions.py        # db, login_manager 싱글톤
│   ├── models/models.py     # Admin / User / Level 등 (BinaryUUID PK, heyvoca_back 과 schema 공유)
│   ├── routes/
│   │   ├── auth.py          # JSON 로그인 /auth/login, /auth/logout, /auth/me (세션 쿠키)
│   │   ├── api_proxy.py     # 제너릭 /api/<path> → heyvoca_back /admin/<path> (X-Admin-API-Key 주입, multipart 지원)
│   │   ├── ai.py            # /api/ai/generate_words (admin 내부 OpenAI 단어 생성)
│   │   └── spa.py           # catch-all → app/static/spa/index.html
│   └── static/spa/          # vite 빌드 산출물 (frontend/ outDir)
├── frontend/                # React SPA (Vite + Tailwind). base '/static/spa/'
│   └── src/
│       ├── features/        # auth / bookstore / overview / voca / vocaBooks
│       ├── lib/             # api.js, endpoints.js, useInfiniteList.js, listCache.js, scrollContext.js, toast.jsx
│       └── components/ui/   # 공용 UI (primitives, overlays)
├── config.py                # Config (DATABASE_URL, BACKEND_URL, ADMIN_API_KEY, *_API_KEY)
├── run.py
├── requirements.txt
├── Dockerfile / Dockerfile.local
├── docker-compose.{local,dev,stg}.yml / docker-compose.yml(prod)
└── deploy.sh                # 홈서버 배포 (dev/stg/prod)
```

### 인증 흐름

React → `/auth/login`(세션 쿠키) → `/api/*`(같은 출처) → Flask 프록시가 `X-Admin-API-Key` 주입 → heyvoca_back `/admin/*`. **API 키는 브라우저에 노출 안 됨.**
`Admin` 테이블(BinaryUUID PK, `user_id` + `password` 해시) + Flask-Login. `auth.py` 에서 `Admin.query.filter(Admin.user_id == username).first()` → `check_password_hash` → `login_user`.

### heyvoca_back 호출 패턴

데이터(단어/단어장/서점)는 직접 SQLAlchemy 로 만지지 말고, 프론트에서 `/api/*` 로 호출 → `api_proxy.py` 가 heyvoca_back `/admin/*` 으로 프록시한다. 새 백엔드 기능이 필요하면 heyvoca_back 의 admin blueprint 에 추가하고, 프론트는 `frontend/src/lib/endpoints.js` 에 호출 함수만 추가하면 된다 (프록시가 제너릭이라 admin 레포에 라우트 추가 불필요).

---

## 환경변수 (`.env`)

```
SECRET_KEY=...
DATABASE_URL=mysql+pymysql://voca:voca!@34@localhost:3310/heyvoca
# 또는 DB_USER/DB_PASSWORD/DB_HOST/DB_PORT/DB_NAME 개별 지정 (config.py 가 둘 다 지원)
BACKEND_URL=http://localhost:5100        # heyvoca_back
ADMIN_API_KEY=...                        # 백엔드 /admin/* 인증 헤더값 (백엔드와 일치 필요)
OPENAI_API_KEY=...
GEMINI_API_KEY=...
ANTHROPIC_API_KEY=...
```

- 로컬 DB 포트 기본 `3310` (heyvoca_service local docker-compose 와 동일)
- `.env*` 는 gitignore. 서버는 각 환경 `.env.{dev,stg}` / `.env`(prod) 를 서버에 배치

---

## DB

- heyvoca_back 과 **동일한 schema 공유** (heyvoca_user / heyvoca_dict)
- 스키마 변경은 반드시 `heyvoca_service/heyvoca_back/migrations*/` 에서 Flask-Migrate 로 (루트 `.claude/rules/db-migration.md` 참조)
- admin 레포에서는 schema 변경 금지. `admin` 테이블만 어드민 전용

---

## 배포 (홈서버 Docker)

heyvoca_service 와 동일하게 ghmate 홈서버 Docker 로 dev/stg/prod 운영. `deploy.sh` 가 SSH → git pull → 빌드까지 자동.

```bash
./deploy.sh dev    # SSH → /srv/projects/heyvoca_admin git pull → docker compose -p heyvoca_admin_dev up --build -d admin → nginx reload → 부팅 헬스체크
./deploy.sh stg
./deploy.sh prod   # compose 파일은 docker-compose.yml
```

- 서버 경로: `/srv/projects/heyvoca_admin` (git `main`)
- 컨테이너: `heyvoca_admin_{dev,stg,prod}` (gunicorn `run:app`, 내부 :5000), 글로벌 `nginx_proxy` 네트워크 조인
- 도메인: `dev-heyvoca-admin` / `stg-heyvoca-admin` / `heyvoca-admin.ghmate.com` (nginx-proxy/conf.d/heyvoca.conf)
- 멀티스테이지 Dockerfile 이 **서버에서** node 로 vite 빌드 후 python 런타임 구성 (로컬 npm 빌드 불필요)
- SSH: `ssh -i ~/.ssh/ghmate_server -p 222 ghmate@ghmate.iptime.org`

---

## 알아두면 좋은 것 (non-obvious)

- 비즈니스 로직 99% 는 `heyvoca_back /admin/*` 호출 — 라우트 파일이 얇음. 데이터 변경 로직은 백엔드 admin blueprint 에서 찾을 것
- `/api/*` 프록시는 제너릭(`api_proxy.py`) — 백엔드에 admin 엔드포인트만 있으면 admin 레포 라우트 추가 없이 `endpoints.js` 한 줄로 호출 가능
- `Admin` 모델은 `is_active`/`is_authenticated` 를 메소드로 정의 (Flask-Login 은 보통 property 기대) — 동작은 하지만 손볼 일 있으면 인지
- `config.py` 의 `_build_db_url()` 은 `DATABASE_URL` 비밀번호에 미인코딩 `@` 가 있으면 None 반환하고 개별 env 로 폴백
- dev compose 는 `./app` 마운트(소스 reload) + SPA 산출물 익명 볼륨(`/app/app/static/spa`)으로 덮어쓰임 방지
- Overview 대시보드 데이터는 heyvoca_back `app/routes/admin_dashboard.py` (`/admin/study/metrics`, `/admin/fsrs/health`, `/admin/progress` 등)

---
> Source: [whrksp126/heyvoca_admin](https://github.com/whrksp126/heyvoca_admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
