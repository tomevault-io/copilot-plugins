## hackathon-team6-6gaejang

> 이 프로젝트에서 작업할 때 항상 따르는 규칙. `feature-impl` 스킬이 "관례가 이미 있는 것"으로

# GEMINI.md

이 프로젝트에서 작업할 때 항상 따르는 규칙. `feature-impl` 스킬이 "관례가 이미 있는 것"으로
간주하고 다시 묻지 않는 기준이 여기다.

## 프로젝트 개요

3일 해커톤 프로젝트 **Delipot** (팀명: 육개장 / Softeer 8기 6팀).
백엔드(Spring Boot) + 프론트(React) 모노레포. 핵심 기능은 미정 —
정해지면 여기에 한 단락으로 적는다. 기능 추가는 그 흐름을 보강하는 것만,
흐름과 무관한 기능은 붙이지 않는다.

현재 구현돼 있는 것: 헬스체크(`GET /api/health`)까지의 골격 — 공통 응답 래퍼,
전역 예외 처리, OpenAPI 스펙 노출, Orval 훅 생성, 헬스 카드 화면, CI.

## 스택

- 백엔드: Spring Boot 4.1 (Java 21, Gradle), Lombok, JPA (JPQL/native query), MySQL 8.4
- 프론트: React 19 + Vite 8 + TypeScript, pnpm, TanStack Router/Query, Orval, shadcn/ui + Tailwind v4
- 인프라: GitHub Actions로 CI(프론트/백엔드 둘 다), SSH+SCP로 간단 자동배포. 모니터링 별도 구축 안 함.

---

## 백엔드 컨벤션

### 네이밍
- 클래스: `PascalCase`, 도메인명 + 역할 접미사 (`RidingService`, `StopEventRepository`)
- 메서드: `camelCase`, 동사로 시작 (`createStopEvent`, `findActiveRidingBy...`)
- DB 컬럼/테이블: `snake_case`

### 패키지 구조
```
com.delipot
├── DelipotApplication.java
├── global/
│   ├── config/      AppConfig(Clock), WebConfig(CORS), OpenApiConfig
│   ├── error/       ErrorCode, BusinessException, GlobalExceptionHandler
│   └── response/    ApiResponse
└── <도메인>/         Controller, Service, Repository, 엔티티 (+ dto/)
```
도메인 단위로 묶는다. 계층 단위(controller/service/repository 전역 폴더)로 흩뿌리지 않는다.
전역 규약만 `global/`에 둔다.

### Lombok
- 엔티티: `@Getter` + `@NoArgsConstructor(access = AccessLevel.PROTECTED)`. `@Setter` 금지 —
  상태 변경은 의미 있는 메서드로 노출 (`riding.markStopped()` 등).
- DTO: record 우선. Lombok 쓸 땐 `@Getter` + `@Builder`.
- `@Data`는 쓰지 않는다 (엔티티에 무분별한 setter/equals 생성 방지).

### 프로파일 / 스키마 (확정 — 매번 묻지 않는다)
- `local`(기본): docker compose MySQL, 호스트 포트 **3307**, `ddl-auto: update`
- `h2`: MySQL 없이 띄울 때 (`create-drop`). 컨텍스트 로딩 테스트는 `@ActiveProfiles("h2")`
- `prod`: `DB_URL`/`DB_USERNAME`/`DB_PASSWORD` 환경변수, `ddl-auto: update`
  (해커톤 기간 한정. 엔티티가 자주 바뀌는데 마이그레이션 도구가 없어 자동 생성이 필요. 스키마 안정화되면 `validate` 로 되돌린다.)
- 마이그레이션 도구는 아직 안 쓴다. 스키마가 안정되면 `validate` + 사람이 DDL 선적용(또는 Flyway 도입)으로 전환한다.

### JPA / 쿼리
- 기본은 JPA 메서드 쿼리 → 복잡해지면 JPQL(`@Query`) → 성능 필요한 곳만 native query.
  native query는 이유를 주석 한 줄로 남긴다.
- N+1 의심되면 fetch join / `@EntityGraph` 우선 고려.
- 동시 자원 접근(같은 라이딩 세션, 같은 이벤트 슬롯)은 낙관적 락(`@Version`) 기본,
  꼭 필요할 때만 비관적 락.

### 응답/예외
- 모든 API는 `ApiResponse<T>`로 래핑. 성공은 `ApiResponse.ok(data)`, 실패는
  `BusinessException(ErrorCode.X)`를 던져 `GlobalExceptionHandler`가 변환하게 한다.
- 새 에러 케이스는 `ErrorCode` enum에만 추가한다. enum 이름이 그대로 응답의 `error.code`로
  나가므로 이름을 바꾸면 프론트가 깨진다.
- 도메인 예외는 `BusinessException`을 상속한 커스텀 예외 + `GlobalExceptionHandler`에서 일괄 처리.
- 기존 패턴에 없는 에러 케이스는 스킬의 계약 확정 게이트에서 사람에게 확인.

### 테스트
- 단위 테스트는 항상 작성 (도메인 규칙, 상태 전이).
- 통합 테스트는 동시성/정합성 걸리는 부분만 선별.
- 테스트 클래스명: `{대상}Test` (단위), `{대상}IntegrationTest` (통합).

---

## 프론트엔드 컨벤션

### 디렉토리 구조 — Route-based
```
src/
├── routes/
│   ├── __root.tsx
│   └── ridings/
│       ├── index.tsx
│       ├── -components/     # -prefix: 라우트 트리 제외
│       └── $ridingId/
│           ├── index.tsx
│           └── -components/
├── components/ui/           # shadcn, 전역 공유만
├── lib/                     # axios customInstance, queryClient
├── styles/globals.css       # @theme 토큰
└── api/generated/           # Orval 자동 생성, 손대지 않음
```

### 네이밍
- 컴포넌트 파일: `PascalCase.tsx` (`RidingMap.tsx`)
- 훅: `use` + `camelCase` (`useStopEvents`)
- 그 외 유틸/타입: `camelCase.ts`

### API 연동
- 항상 Orval 생성 훅(`src/api/generated/`)만 소비. 직접 axios 호출 금지.
- 백엔드 스펙 바뀌면 `pnpm generate:api`로 재생성 후 사용.
- 훅 이름은 백엔드 `operationId`(= 컨트롤러 메서드명)에서 나온다. 메서드명 변경은 프론트를 깨뜨린다.
- 응답이 `ApiResponse` 래핑이므로 실데이터는 `data?.data`로 꺼낸다.
- 인증 헤더/에러 인터셉터는 `lib/axios.ts`의 mutator 레이어에서만 처리.

### 스타일
- `src/styles/globals.css`의 `@theme` 토큰 우선. 색상 하드코딩 지양, 반복되면 토큰으로 승격.
- 레이아웃은 flex/grid. 간격 조정용 빈 `<div>` 금지 (`gap`/`padding` 사용).
- `position: absolute`는 오버레이/장식 등 명확한 이유 있을 때만.

### 컴포넌트 설계
- `components/ui/` 바깥은 과한 공용화보다 화면 맞춤 코드 우선.
- 인터랙티브 UI는 `components/ui/`의 shadcn 조합 우선, raw `<button>`/`<input>` 재구현 지양.

### 접근성
- 페이지 루트: `<main aria-label="...">`
- 에러/상태 변화: `role="alert"` 또는 `aria-live`

---

## CI/CD (프로젝트 초기 세팅, 한 번만)

### 브랜치
- `main` — 배포용. 직접 커밋하지 않는다.
- `dev` — 통합 브랜치. 기능 브랜치는 여기로 PR 한다.
- 기능 브랜치 — `feat/...`, `fix/...`, `chore/...`

### CI (PR + push 트리거, 경로 필터로 분리)
- `backend-ci.yml` — `./gradlew test` + `bootJar`. 테스트가 h2 프로파일이라 CI에 MySQL 서비스가 필요 없다.
- `frontend-ci.yml` — `pnpm lint` + `typecheck` + `build`.

### CD
- **아직 없다.** 배포 방식(EC2 SSH+SCP / S3+CloudFront 등)이 정해지면 워크플로우를 추가한다.
- 배포용 브랜치는 `main` 이다. CD 를 붙일 때 `on: push: branches: [main]` 으로 건다.
- 백엔드는 `prod` 프로파일이 `DB_URL`/`DB_USERNAME`/`DB_PASSWORD`/`CORS_ALLOWED_ORIGINS` 를
  환경변수로 받게 되어 있다. 배포 환경에서 이 값만 주입하면 된다.
- 모니터링 별도 구축 없음 — 문제 생기면 서버 로그 직접 확인.
- 이 설정은 기능 구현 스킬(`feature-impl`) 범위 밖. 워크플로우 파일 수정은 별도로 요청할 것.

---

## 스킬

`.agents/skills/feature-impl/` (또는 `.claude/skills/feature-impl/`) — 기능 하나를 백엔드+프론트 끝까지 구현하는 파이프라인.
상세 구현 규칙은 이 스킬의 `references/backend.md`, `references/frontend.md`에 있다.
규칙을 고칠 때는 references 쪽을 고친다 — 같은 규칙을 두 곳에 두지 않는다.

## 자주 쓰는 명령

백엔드:
```bash
docker compose up -d                          # MySQL (호스트 포트 3307)
cd backend && ./gradlew bootRun                # 기본 프로파일 local
SPRING_PROFILES_ACTIVE=h2 ./gradlew bootRun    # MySQL 없이 띄우기
./gradlew test
```

프론트엔드:
```bash
pnpm -C frontend dev          # 5173, /api는 8080으로 proxy
pnpm -C frontend generate:api # 백엔드 띄운 뒤 스펙 재생성
pnpm -C frontend typecheck
pnpm -C frontend lint
```

---
> Source: [softeerbootcamp-8th/HACKATHON-TEAM6-6GaeJang](https://github.com/softeerbootcamp-8th/HACKATHON-TEAM6-6GaeJang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
