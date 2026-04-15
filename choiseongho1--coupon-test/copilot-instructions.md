## coupon-test

> | Programming Languages | Java |


사용 기술 스택
| Programming Languages | Java |
| --- | --- |
| Frameworks | Spring Boot |
| Databases | MySQL |
| Message Queue | Kafka  |
| Caching & Locking | Redis  |
| Version Control | Git |
| Deployment Tools | Docker, Docker Compose |
| API | RESTful API |
| Monitoring  | Prometheus, Grafana |


ERD
## 🧾 테이블 정의

### 1. `USER`

| 필드명 | 타입 | 제약 조건 | 설명 |
| --- | --- | --- | --- |
| ID | BIGINT | PK, AUTO_INCREMENT | 사용자 고유 ID |
| EMAIL | VARCHAR(100) | UNIQUE | 사용자 이메일 |
| NAME | VARCHAR(50) |  | 사용자 이름 |
| CREATED_AT | DATETIME | DEFAULT now() | 가입 시각 |

---

### 2. `COUPON`

| 필드명 | 타입 | 제약 조건 | 설명 |
| --- | --- | --- | --- |
| ID | BIGINT | PK, AUTO_INCREMENT | 쿠폰 고유 ID |
| TITLE | VARCHAR(100) |  | 쿠폰 이름 |
| TOTAL_QUANTITY | INT |  | 전체 발급 수량 |
| REMAINING_QUANTITY | INT |  | 남은 수량 (캐시 or DB) |
| VALID_FROM | DATETIME |  | 쿠폰 유효 시작일 |
| VALID_TO | DATETIME |  | 쿠폰 유효 종료일 |
| CREATED_AT | DATETIME | DEFAULT now() | 쿠폰 생성일 |

---

### 3. `COUPON_ISSUE`

| 필드명 | 타입 | 제약 조건 | 설명 |
| --- | --- | --- | --- |
| ID | BIGINT | PK, AUTO_INCREMENT | 발급 이력 고유 ID |
| USER_ID | BIGINT | FK → [USER.ID](http://user.id/) | 사용자 참조 |
| COUPON_ID | BIGINT | FK → [COUPON.ID](http://coupon.id/) | 쿠폰 참조 |
| ISSUED_AT | DATETIME |  | 발급 시각 |
| 제약 |  | UNIQUE(USER_ID, COUPON_ID) | 사용자당 중복 발급 방지 |

---

## ✅ 설계 요약

- 사용자와 쿠폰 간 다대다 관계를 `COUPON_ISSUE` 테이블로 관리
- 쿠폰 재고는 `REMAINING_QUANTITY`에서 관리하며 Redis 등 캐시와 연동 가능
- 발급 이력에는 유저당 하나만 발급되도록 UNIQUE 제약 추가

---


개발 기능 목록
| 분류                   | 작업 항목                                                       |
|------------------------|------------------------------------------------------------------|
| 프로젝트 초기 세팅     | Spring Boot, 의존성 설정                                         |
|                        | MySQL & Redis 연동 설정                                          |
|                        | Docker Compose 환경 구성 (MySQL + Redis)                         |
|                        | Dockerfile 작성 (SpringBoot 앱)                                  |
|                        | Swagger 또는 REST Docs 연동                                      |
| 인증 및 사용자 관리    | JWT 로그인 인증 구현                                              |
|                        | 회원가입 API 구현                                                |
|                        | 로그인 API 구현                                                  |
|                        | 인증 필터(JWT → 사용자 추출) 구현                                |
| 도메인 및 데이터베이스 | Coupon, CouponIssue, User 도메인 설계                             |
|                        | 쿠폰 유효기간 만료 처리 스케줄링                                 |
| 쿠폰 관리 기능         | 쿠폰 등록 API (관리자용)                                         |
|                        | 쿠폰 목록 조회 API (모든 사용자)                                 |
|                        | 내 발급 쿠폰 조회 API                                            |
|                        | 발급 수량 통계 API (관리자용)                                    |
|                        | 관리자 대시보드용 쿠폰 현황 API                                  |
| 쿠폰 발급 기능         | Redis에 쿠폰 재고 캐시 저장 로직 구현                            |
|                        | Redis Lua Script로 재고 차감 + 발급 처리                         |
|                        | 쿠폰 발급 API 구현 (`POST /api/coupons/{id}/claim`)             |
|                        | 중복 발급 방지 로직 구현 (유저당 1회)                            |
|                        | 발급 실패 응답 처리 (재고 없음 등)                               |
|                        | 1일 1회 발급 제한 로직 추가                                     |
| 테스트 및 안정성 검증  | 쿠폰 발급 단위 테스트 코드 작성                                  |
|                        | Redis Lua Script 단위 테스트                                     |
|                        | 동시 요청 시나리오 테스트 (JMeter 또는 스크립트)                |
|                        | 대량 트래픽 발급 시 안정성 검증                                 |
| 확장/부가 기능 (선택)  | WebSocket으로 실시간 재고 표시 (선택)                            |
|                        | 이벤트 대기 화면 API (선택)                                     |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/choiseongho1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
