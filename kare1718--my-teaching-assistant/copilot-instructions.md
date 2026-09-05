## my-teaching-assistant

> 학원 관리 SaaS 플랫폼 (멀티테넌트)

# 나만의 조교 - 학원 관리 SaaS 플랫폼

## 프로젝트 개요
학원 관리 SaaS 플랫폼 (멀티테넌트)
- 서버: `server/server.js` (Express, Port 3001)
- 클라이언트: `client/` (React + Vite)
- DB: PostgreSQL (Supabase)
- 슬로건: "나만의 조교로 학원 운영을 더욱 편리하게"

## 기술 스택
- Backend: Express.js, pg (PostgreSQL), bcryptjs, jsonwebtoken
- Frontend: React, Vite, React Router
- DB: Supabase (PostgreSQL)
- 배포: Render (서버) + Supabase (DB)

## 멀티테넌시 구조
- 모든 테이블에 `academy_id` 컬럼
- JWT에 `academy_id` 포함
- 학원별 독립 데이터 격리
- 구독 티어 (확정, 4단 / 모든 가격 VAT 별도):
  - `free` — 0원 / 15명 (성적·출결·공지·자료·Q&A만)
  - `starter` — 49,000원 / 50명 (+수납 기본·SMS·보호자 앱·기본 상담)
  - `pro` — 129,000원 / 100명 (+자동화·상담 CRM·고급 리포트·AI 리포트)
  - `first_class` — 별도 문의 (+게이미피케이션·AI 문제 생성·브랜딩)
  - ※ 신규 가입 시 30일 무료 체험 (Pro 기능 체험, trial_ends_at 기반). 만료 후 Free로 자동 전환.
  - 레거시 호환 매핑: trial→free, basic→starter, standard|growth→pro, premium→first_class
  - 상세: `PLAYBOOK.md` 3절 / 구현: `server/middleware/subscription.js`

## 주요 파일
- `server/db/database.js` — pg Pool, ?→$N 자동 변환
- `server/db/migrations/` — SQL 마이그레이션 파일
- `server/middleware/` — auth, tenant, usage, subscription
- `server/routes/` — 14개 기존 + 4개 SaaS 전용 라우트
- `client/src/contexts/TenantContext.jsx` — 학원별 동적 설정
- `client/src/pages/LandingPage.jsx` — 플랫폼 소개
- `client/src/pages/OnboardingPage.jsx` — 학원 등록 마법사

## 빌드 & 실행
```bash
cd client && npm run build
cd server && node server.js
```

---
> Source: [kare1718/my-teaching-assistant](https://github.com/kare1718/my-teaching-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
