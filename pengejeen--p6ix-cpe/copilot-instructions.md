## p6ix-cpe

> - Frontend: React 19 + Vite 5 (JavaScript)

# AGENTS.md

## 1) 공통 규칙

### 1.1 프레임워크
- Frontend: React 19 + Vite 5 (JavaScript)
- Backend: Django 5 + Django REST Framework
- DB: PostgreSQL(운영), SQLite(로컬 fallback)
- 배포/빌드: Docker 멀티스테이지(Frontend build + Django runtime)

### 1.2 이 프로젝트 작업 규칙 및 폴더링 규칙
- 변경 범위는 요청 기능과 직접 관련된 파일로 제한한다.
- Frontend 규칙
- `frontend/src/pages`: 라우트 단위 페이지
- `frontend/src/components`: 재사용 UI/도메인 컴포넌트
- `frontend/src/api`: 서버 통신 로직
- `frontend/src/stores`: 전역 상태(zustand)
- `frontend/src/hooks`: 비즈니스 훅
- `frontend/src/utils`: 순수 유틸리티
- Backend 규칙
- `backend/<app>/models`: 데이터 모델
- `backend/<app>/serializers`: DRF 직렬화/검증
- `backend/<app>/views`: API 엔드포인트
- `backend/<app>/urls.py`: 앱 라우팅
- `backend/<app>/tests.py`: 앱 테스트
- 루트 문서
- 요구사항: `공기검토플랫폼_SRS.md`
- 설계: `260306_설계서.md`

### 1.3 변경 원칙
- 최소 수정: 필요한 파일/라인만 변경한다.
- 기존 스타일 유지: 네이밍, 들여쓰기, 구조, 패턴을 우선 따른다.
- 리팩터링은 요청 시에만 수행한다.
- 동작 변경 시 근거(요구사항/버그/성능)를 PR/커밋 설명에 남긴다.

### 1.4 금지사항
- 대규모 구조 변경 금지(폴더 재편, 아키텍처 전환 등).
- 의존성 추가 금지(`package.json`, `requirements.txt` 변경 금지) - 사전 승인 시 예외.
- 무관한 파일 포맷팅/정리 금지.
- 테스트 없이 핵심 로직 수정 후 머지 금지.
- 환경변수/비밀정보 하드코딩 금지.

## 2) 빌드/테스트

### 2.1 유닛 테스트 명령
- Backend 앱 단위(우선):
- `cd backend && SECRET_KEY="ci-secret-key" python manage.py test cpe_module.tests user.tests operatio.tests cpe_all_module.tests`
- Frontend 유닛 테스트:
- 현재 전용 스크립트 미구성(`frontend/package.json`에 `test` 없음)

### 2.2 통합 테스트 명령
- Backend 전체 통합:
- `cd backend && SECRET_KEY="ci-secret-key" python manage.py test`
- 시스템 체크(사전 검증):
- `cd backend && SECRET_KEY="ci-secret-key" python manage.py check`

### 2.3 린트 / 타입체크 명령
- Frontend lint/typecheck: 현재 별도 스크립트 미구성
- Backend lint/typecheck: 현재 별도 스크립트 미구성
- 대체 품질 게이트:
- `cd frontend && npm run build`
- `cd backend && SECRET_KEY="ci-secret-key" python manage.py check`

### 2.4 실행 순서
1. 변경 파일 자체 검토(요구사항 대비 diff 확인)
2. `cd backend && SECRET_KEY="ci-secret-key" python manage.py check`
3. 영향 앱 단위 테스트(가능 시 우선)
4. `cd backend && SECRET_KEY="ci-secret-key" python manage.py test`
5. `cd frontend && npm run build`
6. 릴리즈/배포 직전: Docker 이미지 빌드 검증

### 2.5 느리거나 위험한 테스트 실행 시점
- 전체 통합 테스트(`manage.py test`)는 기능 완료 후, PR 전 최소 1회 실행한다.
- Docker 빌드는 PR 전 최종 검증 또는 릴리즈 직전에 실행한다.
- 외부 연동(Keycloak/Gemini/실DB) 의존 검증은 로컬 기본 테스트와 분리해 수동/스테이징 단계에서 실행한다.

## 3) 테스트 작성 원칙

### 3.1 우선순위
- 1순위: 이번 변경으로 동작이 바뀌는 도메인 로직/계산 로직
- 2순위: 직전 버그 재발 방지 회귀 테스트
- 3순위: 권한, 입력 검증, 예외 처리(에러 코드/메시지)

### 3.2 Snapshot 갱신 정책
- 스냅샷 테스트는 명시적 UI 변경 요청이 있을 때만 갱신 허용
- 단순 "테스트 통과" 목적의 일괄 snapshot 갱신 금지

### 3.3 Mocking 원칙
- 외부 의존성(SSO, Gemini, 외부 API, 시간/난수)은 mock/fake 사용
- 핵심 비즈니스 로직은 과도한 mocking 지양(실제 입력/출력 중심)
- DB 연동은 Django TestCase 트랜잭션 격리를 기본으로 사용

### 3.4 Flaky 테스트 처리
- 첫 대응: 동일 조건 3회 재실행으로 재현성 확인
- 원인 미확인 상태의 장기 skip 금지
- 임시 완화(skip/조건부 비활성)는 이슈 번호, 만료기한, 복구 조건을 반드시 기록

### 3.5 버그 수정 시 회귀 테스트
- 버그 수정 PR에는 재현 테스트(실패 -> 수정 후 통과) 추가를 기본 요구로 한다.

## 4) 리뷰 기준
- correctness: 요구사항과 실제 동작이 일치하는가
- security: 인증/인가, 입력검증, 비밀정보 노출 위험이 없는가
- regression: 기존 API/화면/데이터 흐름에 부작용이 없는가
- edge case: null/빈값/경계값/동시성/권한예외를 다뤘는가
- performance: 불필요한 반복 연산/쿼리/API 호출이 없는가
- migration risk: 스키마/환경변수/배포 절차 변경 시 롤백 가능성이 확보됐는가

## 5) 문서화 규칙

### 5.1 기능 명세서 기본 목차 (`공기검토플랫폼_SRS.md` 참조)
1. 소개(목적, 범위, 용어/약어, 참고문헌)
2. 일반사항(사용자 특성, 제약사항, 가정, 위험요소)
3. 기능요구사항(요구사항 ID, 우선순위, 수용기준)
4. 비기능 요구사항(성능, 신뢰성, 보안, 유지보수 등)
5. 부록

### 5.2 API 문서 스타일
- 엔드포인트별 고정 템플릿 사용
- Method / Path / Auth / Request / Response / Error / Example
- Request/Response는 실제 필드명 기준으로 작성
- 에러는 HTTP 코드 + 발생 조건을 함께 기록

### 5.3 설계 문서 구분 원칙 (`260306_설계서.md`, 요청 명칭: `설계서.md`)
- 사실(Fact): 현재 코드/운영에 이미 반영된 내용
- 제안(Proposal): 아직 반영되지 않은 변경안
- 오픈 이슈(Open Issue): 의사결정이 필요한 항목, 대안, 결정기한
- 문서 내 섹션 또는 라벨로 세 범주를 명확히 분리한다.

### 5.4 Changelog 작성 방식
- 파일: `CHANGELOG.md` (없으면 생성)
- 형식: 날짜(`YYYY-MM-DD`) + 변경유형(`Added/Changed/Fixed/Removed`)
- 항목마다 영향 범위(Frontend/Backend/API/DB)와 마이그레이션 필요 여부를 표시

## 6) 계획문서 규칙
- 파일 위치: `docs/plans/`
- 파일명: `YYYYMMDD_<주제>_plan.md`
- 기본 목차
1. 목표/배경
2. 범위(In) / 범위(Out)
3. 작업 단위(최대 반나절 단위)
4. 검증 방법(테스트/수동검증)
5. 리스크/대응
6. 롤백 계획
7. 완료 기준(Definition of Done)
- 계획은 "무엇을/왜/어떻게 검증할지"가 있어야 하며, 구현 세부코드는 설계 문서로 분리한다.

---
> Source: [PengEjeen/p6ix-CPE](https://github.com/PengEjeen/p6ix-CPE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
