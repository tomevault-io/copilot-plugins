## beyond-sw-21th-second-4team

> > Claude Code 작업 가이드 - Time Deal 프로젝트

# CLAUDE.MD

> Claude Code 작업 가이드 - Time Deal 프로젝트

## 프로젝트 개요

**프로젝트명**: Time Deal System (11번가 스타일)
**기술 스택**: Spring Boot 3.5.8, Java 17, Redis, MariaDB, React 19 + TypeScript (Frontend)
**아키텍처**: Monorepo (Backend + Frontend)

### 주요 기능
- 타임딜 프로모션 시스템 (11번가 디자인)
- Redis 기반 대기열 관리
- 재고 관리 (Redis 활용)
- 주문 처리 시스템
- 사용자/관리자 인증
- 관리자 대시보드

---

## 프로젝트 구조

```
.
├── src/main/java/com/ohgiraffers/timedeal/
│   ├── core/
│   │   ├── api/          # REST API 컨트롤러
│   │   ├── domain/       # 도메인 엔티티 및 서비스
│   │   ├── enums/        # 열거형 타입
│   │   ├── messaging/    # 메시징 관련
│   │   └── support/      # 공통 지원 클래스
│   └── storage/          # 레포지토리 계층
├── frontend/             # React 프론트엔드
├── docker-compose.yml    # Redis, MariaDB 설정
└── test-data.sql         # 테스트 데이터

패키지 구조:
- com.ohgiraffers.timedeal.core.api: API 레이어
- com.ohgiraffers.timedeal.core.domain: 도메인 로직
- com.ohgiraffers.timedeal.storage: 데이터 접근 레이어
```

---

## 작업 규칙

### 1. Git 브랜치 전략
- **메인 브랜치**: `dev`
- **현재 워크트리**: `stoic-hertz`
- PR 생성 시 `dev` 브랜치를 base로 사용

### 2. 코딩 컨벤션
- Java 17 기능 활용 (record, switch expression 등)
- Lombok 사용: `@Getter`, `@Builder`, `@NoArgsConstructor` 등
- REST API는 `/api/v1/` prefix 사용
- **중요**: 모든 `@GetMapping`, `@PostMapping` 등의 경로는 반드시 `/`로 시작

### 3. 의존성
```gradle
- Spring Boot 3.5.8
- Spring Data JPA
- Spring Data Redis
- Redisson 3.27.2 (분산 락)
- Springdoc OpenAPI 2.5.0/2.8.13 (Swagger)
- MariaDB
- Lombok
```

### 4. API 설계 원칙
- 응답 포맷: `ApiResult<T>` 래퍼 사용
- Validation: `@Valid`, `@NotNull`, `@NotBlank` 등 활용
- Swagger 문서화 필수: `@Operation`, `@Tag` 사용

---

## 알려진 이슈

### 긴급 수정 완료됨
~~PromotionController의 경로 문제~~ (수정 완료)
- ~~`@GetMapping("api/v1/promotions")` → `@GetMapping("/api/v1/promotions")`~~
- ~~`@GetMapping("api/v1/promtions/...")` → `@GetMapping("/api/v1/promotions/...")`~~

### 프론트엔드 구현 완료 (TD-13)
1. ✅ **프로모션 상세 페이지** (`/promotions/:id`)
   - 상품 정보, 타이머, 재고 현황
   - 수량 선택 및 대기열 입장 버튼

2. ✅ **구매 페이지** (`/checkout/:promotionId`)
   - 대기열 통과 후 접근
   - 수량 선택 및 결제 정보
   - 5분 제한 시간 안내

3. ✅ **주문 완료 페이지** (`/order-complete`)
   - 주문 정보 확인
   - 배송 안내 및 유의사항

4. ✅ **주문 내역 페이지** (`/orders`)
   - 사용자별 주문 목록 조회
   - 주문 통계 및 상태 표시

5. ✅ **관리자 대시보드** (`/admin/dashboard`)
   - 프로모션 통계 및 관리
   - 상태별 필터링 (진행중/대기/종료)
   - 테이블 형식 프로모션 목록

6. ✅ **UI/UX 개선**
   - LoadingSpinner, ErrorMessage, EmptyState 공통 컴포넌트
   - Header에 주문내역 링크 추가

### 사용자 플로우
```
로그인 → 프로모션 목록 → 프로모션 상세 → 대기열 입장
→ 대기열 대기 → 구매 페이지 → 주문 완료 → 주문 내역
```

자세한 내용: `BACKEND_ISSUES.md`, `frontend/README.md` 참조

---

## 주요 도메인 모델

### 핵심 엔티티
1. **User** (사용자)
2. **Admin** (관리자)
3. **Product** (상품)
4. **Promotion** (프로모션/타임딜)
5. **Order** (주문)
6. **OrderDetail** (주문 상세)

### 주요 열거형 (Enums)
- `PromotionStatus`: 프로모션 상태
- 기타 도메인별 상태 열거형

---

## 개발 환경 설정

### 로컬 실행
```bash
# Docker 컨테이너 시작 (Redis, MariaDB)
docker-compose up -d

# Windows 환경
docker-start.bat

# 백엔드 실행
./gradlew bootRun

# 프론트엔드 실행 (별도 터미널)
cd frontend
npm install
npm start
```

### 접속 정보
- **백엔드 API**: http://localhost:8080
- **Swagger UI**: http://localhost:8080/swagger-ui/
- **프론트엔드**: http://localhost:5173 (Vite 개발 서버)

---

## 작업 시 참고사항

### Redis 사용
- **대기열 관리**: Redis Sorted Set 활용
- **재고 차감**: Redis 트랜잭션 사용
- **분산 락**: Redisson 라이브러리 활용

### 테스트
```bash
# 전체 테스트 실행
./gradlew test

# 특정 테스트 실행
./gradlew test --tests "QueueServiceTest"
```

### 데이터베이스
- 테스트 데이터: `test-data.sql` 참조
- JPA Config: 엔티티 스캔, 트랜잭션 설정 확인

---

## 문서

- **변경 이력**: `CHANGELOG.md`
- **백엔드 이슈**: `BACKEND_ISSUES.md`
- **CI/CD 설정**: `.github/` 디렉토리
- **Changelog 생성**: `cliff.toml` (git-cliff 사용)

---

## 주의사항

### 절대 하지 말 것
1. **경로에 슬래시(`/`) 누락 금지**
   - 잘못: `@GetMapping("api/v1/...")`
   - 올바름: `@GetMapping("/api/v1/...")`

2. **엔티티 직접 반환 금지**
   - Controller에서 Entity를 직접 반환하지 말고 DTO/Response 객체 사용

3. **트랜잭션 주의**
   - Redis 재고 차감과 DB 주문 저장 간의 일관성 유지
   - 분산 락 사용 시 데드락 주의

### 권장 사항
1. **API 변경 시 Swagger 문서 업데이트**
2. **새로운 도메인 추가 시 테스트 코드 작성**
3. **Redis 키 네이밍 컨벤션 준수**
4. **로깅 적절히 활용** (SLF4J)

---

## 최근 작업 (Git Log 기준)

```
543b011 Merge branch 'dev' into feature/TD-13
62f74f5 feat(order): Redis를 이용한 재고 차감 로직 구현
4478a33 feat(order): Swagger 문서화 작업
ca86885 feat(order): validation 추가
```

현재 브랜치는 주문(Order) 기능 개발 중인 것으로 보입니다.

---

## 문제 해결 가이드

### API 404 에러
1. Controller의 `@RequestMapping`, `@GetMapping` 경로 확인
2. 슬래시(`/`) 누락 여부 확인
3. Swagger UI에서 실제 등록된 경로 확인

### Redis 연결 문제
1. `docker-compose up -d` 실행 확인
2. `application.yml` 또는 `application.properties`의 Redis 설정 확인
3. Redis 포트 충돌 확인 (기본 6379)

### 테스트 실패
1. Docker 컨테이너 실행 상태 확인
2. 테스트 데이터베이스 설정 확인
3. `test-data.sql` 로드 여부 확인

---

## Claude Code 작업 체크리스트

새로운 기능 추가 시:
- [ ] 엔티티/DTO 정의
- [ ] Repository 작성
- [ ] Service 레이어 구현
- [ ] Controller 작성 (경로 `/` 확인!)
- [ ] Validation 추가
- [ ] Swagger 문서화 (`@Operation`, `@Tag`)
- [ ] 단위 테스트 작성
- [ ] 통합 테스트 작성
- [ ] CHANGELOG.md 업데이트 (git-cliff 자동 생성)

---

---

## 프론트엔드 페이지 목록 (TD-13)

### 사용자 페이지
- `/` - Role Selection (역할 선택)
- `/login` - 로그인
- `/signup` - 회원가입
- `/promotions` - 타임딜 목록
- `/promotions/:id` - 타임딜 상세
- `/queue/:promotionId` - 대기열 상태
- `/checkout/:promotionId` - 구매 페이지
- `/order-complete` - 주문 완료
- `/orders` - 주문 내역

### 관리자 페이지
- `/admin/login` - 관리자 로그인
- `/admin/dashboard` - 관리자 대시보드

---

**작성일**: 2025-12-04
**최종 업데이트**: 2025-12-04 (TD-13 프론트엔드 완료)
**버전**: 1.1.0
**작성자**: Claude Code (Frontend Specialist)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/20250918-beyond-SW-Camp-21th) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
