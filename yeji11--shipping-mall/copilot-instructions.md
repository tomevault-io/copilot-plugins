## shipping-mall

> 여성 의류 쇼핑몰 프로젝트 (클릭앤퍼니 벤치마킹)

# Shopping Mall Project

## Project Overview
여성 의류 쇼핑몰 프로젝트 (클릭앤퍼니 벤치마킹)

## Tech Stack
- **Frontend**: React + Tailwind CSS + Vite
- **Backend**: Spring Boot 3 + Kotlin + JPA
- **Database**: PostgreSQL (Supabase)
- **Auth**: Kakao OAuth 2.0
- **Payment**: TossPayments
- **Deployment**: Vercel (FE) + Railway (BE)

## Project Structure
```
shopping-mall/
├── docs/           # 기획 문서
├── frontend/       # React 프론트엔드
└── backend/        # Spring Boot 백엔드
```

## Documentation
- `docs/00_팀_구성.md` - 팀 역할 및 기술 스택
- `docs/01_기술스택_검토.md` - 기술 스택 검토 결과
- `docs/02_쇼핑몰_기획서.md` - 기능 정의 및 MVP 범위
- `docs/03_DB_스키마_설계.md` - 데이터베이스 스키마
- `docs/04_API_명세서.md` - REST API 명세
- `docs/05_화면_설계.md` - UI/UX 화면 설계

---

# Agent Team Configuration

## Team: shopping-mall-team

### Planner Agent (쇼핑몰 기획자)
- **Role**: UI/UX 기획 및 화면 설계
- **Focus**: `docs/` 폴더
- **Responsibilities**:
  - 화면 설계 및 와이어프레임 작성
  - 사용자 플로우 정의
  - 기능 명세서 작성
  - 벤치마킹 분석 (클릭앤퍼니 참고)
- **Skills**: 사용자 경험 분석, 정보 구조 설계, 기획 문서 작성

### Frontend Agent (FE 개발자)
- **Role**: 프론트엔드 UI 구현
- **Focus**: `frontend/` 폴더
- **Tech Stack**:
  - React 18+
  - Tailwind CSS
  - Vite
  - React Router
  - TanStack Query
  - Zustand
- **Responsibilities**:
  - 반응형 쇼핑몰 UI 구현
  - 상품 목록/상세 페이지
  - 장바구니 기능
  - 토스페이먼츠 결제 연동 (프론트)
  - 카카오 로그인 연동 (프론트)

### Backend Agent (BE 개발자)
- **Role**: 백엔드 API 개발
- **Focus**: `backend/` 폴더
- **Tech Stack**:
  - Kotlin
  - Spring Boot 3
  - Spring Security
  - Spring Data JPA
  - PostgreSQL
- **Responsibilities**:
  - RESTful API 설계 및 개발
  - 카카오 OAuth 2.0 연동
  - 토스페이먼츠 결제 연동 (서버)
  - 상품/주문/회원 데이터 관리

### Tester Agent (브라우저 테스터)
- **Role**: QA 및 버그 리포트
- **Responsibilities**:
  - Chrome, Safari, Firefox, Edge 크로스 브라우저 테스트
  - 모바일 반응형 테스트
  - 기능 테스트 (회원가입, 결제, 주문 등)
  - UI/UX 이슈 리포트
  - 성능 테스트

---

## Team Workflow

### Phase 1: MVP
1. 상품 전시 (메인, 카테고리, 검색, 상세)
2. 카카오 로그인/회원가입
3. 장바구니
4. 토스페이먼츠 결제
5. 주문 완료/내역 조회

### Phase 2: 운영 기능
- 관리자 상품 관리
- 관리자 주문/배송 관리

### Phase 3: 고객 서비스
- 클레임 관리 (문의 게시판)
- 실시간 채팅 상담
- 리뷰 시스템

---

## Conventions

### Git
- Branch: `feature/{feature-name}`, `fix/{bug-name}`
- Commit: Conventional Commits (feat, fix, docs, style, refactor, test)

### Code Style
- **Frontend**: ESLint + Prettier
- **Backend**: ktlint

### API
- RESTful conventions
- Response format: `{ data, error, meta }`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Yeji11) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
