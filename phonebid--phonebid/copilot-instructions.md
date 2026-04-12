## phonebid

> - **Entity 수정**: Entity가 수정 및 추가 생성되었을 때 이 파일에도 반드시 수정

# Database Schema Rules

## 📊 데이터베이스 스키마 구조 (정확한 실제 스키마)

### 🔑 기본 원칙

- **Entity 수정**: Entity가 수정 및 추가 생성되었을 때 이 파일에도 반드시 수정
- **UUID 기본키**: 모든 테이블은 UUID 타입의 기본키 사용
- **엔터티 상속 규칙**:
  - **일반 엔터티**: `BaseEntity`를 상속받아 모든 감사 필드 포함 (기본 규칙)
  - **Immutable 엔터티 예외**: 수정되지 않는 엔터티(예: RefreshToken)는 `BaseTimeEntity`를 상속받을 수 있음
- **소프트 삭제**:
  - **BaseEntity 엔터티**: `is_delete = true`, `deleted_at`, `deleted_by` 필드로 논리적 삭제
  - **BaseTimeEntity 엔터티**: `deleted_at` 필드만으로 논리적 삭제 (`is_delete`, `deleted_by` 없음)
- **네이밍 규칙**: 테이블명은 복수형, 컬럼명은 snake_case

### 📋 공통 필드

#### BaseEntity (일반 엔터티용)

일반 엔터티(예: User, Product, Quote 등)에 포함되는 공통 필드:

```sql
created_at      TIMESTAMP    -- 생성 시각
created_by      VARCHAR(255) -- 생성자
updated_at      TIMESTAMP    -- 수정 시각
updated_by      VARCHAR(255) -- 수정자
deleted_at      TIMESTAMP    -- 삭제 시각 (소프트 삭제)
deleted_by      VARCHAR(255) -- 삭제자
is_delete       BOOLEAN      -- 삭제 여부
```

#### BaseTimeEntity (Immutable 엔터티용)

Immutable한 엔터티(예: RefreshToken)에 포함되는 최소한의 공통 필드:

```sql
created_at      TIMESTAMP    -- 생성 시각
deleted_at      TIMESTAMP    -- 삭제 시각 (소프트 삭제)
```

**참고**: `BaseTimeEntity`는 수정되지 않는 엔터티에 사용되므로 `updated_at`, `updated_by`, `created_by`, `deleted_by`, `is_delete` 필드를 포함하지 않습니다. 소프트 삭제는 `deleted_at IS NULL` 조건으로만 확인합니다.

## 🏗️ 실제 테이블 구조

**참고**: 아래 테이블들은 모두 `BaseEntity`를 상속받는 일반 엔터티입니다. `BaseTimeEntity`를 사용하는 테이블(예: `refresh_tokens`)은 별도로 관리됩니다.

### 1. users (사용자)

```sql
-- 기본키
id              UUID PRIMARY KEY    -- 사용자 고유 ID

-- 사용자 정보
username        VARCHAR(255) UNIQUE NOT NULL    -- 사용자명 (로그인 ID)
password        VARCHAR(255) NOT NULL           -- 암호화된 비밀번호
email           VARCHAR(255) UNIQUE NOT NULL    -- 이메일 주소
name            VARCHAR(255) NOT NULL           -- 실명
nickname        VARCHAR(10) NOT NULL            -- 닉네임 (2-10자)
phone           VARCHAR(255)                    -- 휴대폰 번호
role            VARCHAR(255) NOT NULL           -- 사용자 역할 (CONSUMER, SELLER, ADMIN)
provider        VARCHAR(255)                    -- 소셜 로그인 제공자 (KAKAO, NAVER)
provider_id     VARCHAR(255)                    -- 소셜 로그인 고유 ID

-- BaseEntity 필드
```

### 2. sellers (판매자)

```sql
-- 기본키
seller_id       UUID PRIMARY KEY    -- 판매자 고유 ID
user_id         UUID UNIQUE → users.id     -- 연결된 사용자 ID (1:1)

-- 판매자 정보
business_number VARCHAR(255) NOT NULL      -- 사업자등록번호
store_name      VARCHAR(255) NOT NULL      -- 상호명
approval_status VARCHAR(255) NOT NULL      -- 승인 상태 (PENDING, APPROVED, REJECTED)

-- 주소 정보 (임베디드)
postal_code     VARCHAR(255)               -- 우편번호
address         VARCHAR(255)               -- 주소
detail_address  VARCHAR(255)               -- 상세주소

-- BaseEntity 필드
```

### 3. seller_documents (판매자 문서)

```sql
-- 기본키
id              UUID PRIMARY KEY    -- 문서 고유 ID
seller_id       UUID → sellers.seller_id  -- 판매자 ID

-- 문서 정보
type            VARCHAR(255) NOT NULL      -- 문서 종류 (BUSINESS_LICENSE, CONSENT_FORM)
file_url        VARCHAR(255) NOT NULL      -- S3 저장 위치
uploaded_at     TIMESTAMP NOT NULL         -- 업로드 시각

-- BaseEntity 필드
```

### 4. quotes (견적 요청)

```sql
-- 기본키
id              UUID PRIMARY KEY    -- 견적 요청 고유 ID
user_id         UUID → users.id     -- 견적 요청자 ID

-- 제품 정보
model           VARCHAR(255) NOT NULL      -- 휴대폰 모델명 (예: iPhone 16)
storage         VARCHAR(255) NOT NULL      -- 저장 용량 (예: 128GB)
color           VARCHAR(255) NOT NULL      -- 색상
carrier         VARCHAR(255) NOT NULL      -- 희망 통신사 (SKT, KT, LGU)

-- 경매 정보
status          VARCHAR(255) NOT NULL      -- 견적 상태 (OPEN, CLOSED, CONTRACTED)
expired_at      TIMESTAMP NOT NULL         -- 경매 마감 시각

-- 구매 조건
purchase_method VARCHAR(255)               -- 구매방법 (NUMBER_TRANSFER, DEVICE_CHANGE, NEW_SUBSCRIPTION, ANY)
current_carrier VARCHAR(255)               -- 현재 통신사 (번호이동/기기변경 시)
activation_method VARCHAR(255)             -- 개통방법 (SELECTIVE_SUBSIDY, COMMON_SUBSIDY, ANY)

-- BaseEntity 필드
```

### 5. price_plans (요금제) - **새로 발견된 테이블**

```sql
-- 기본키
id              UUID PRIMARY KEY    -- 요금제 고유 ID

-- 요금제 정보
plan_name       VARCHAR(255)        -- 요금제 이름
plan_price      INTEGER             -- 요금제 가격 (원)
carrier         VARCHAR(255)        -- 통신사 (SKT, KT, LGU)

-- BaseEntity 필드
```

### 6. bids (입찰)

```sql
-- 기본키
id                  UUID PRIMARY KEY    -- 입찰 고유 ID
quote_id            UUID → quotes.id    -- 대상 견적 ID
seller_id           UUID → sellers.seller_id  -- 입찰자 ID
price_plan_id       UUID UNIQUE → price_plans.id  -- 요금제 ID (1:1)

-- 입찰 정보
price               INTEGER NOT NULL    -- 입찰가 (원)
delivery_days       INTEGER NOT NULL    -- 배송 예상일 (일)
rating_snapshot     DOUBLE PRECISION    -- 입찰 당시 판매자 평점 스냅샷

-- 구매 조건
purchase_method     VARCHAR(255) NOT NULL  -- 구매방법 (NUMBER_TRANSFER, DEVICE_CHANGE, NEW_SUBSCRIPTION, ANY)
carrier             VARCHAR(255) NOT NULL  -- 통신사 (이동할/사용할 통신사) 3사 (SKT, KT, LGU)
current_carrier     VARCHAR(255)           -- 기존 통신사 (번호이동/기기변경 시)
activation_method   VARCHAR(255) NOT NULL  -- 개통방법 (SELECTIVE_SUBSIDY, COMMON_SUBSIDY, ANY)

-- 추가 비용 정보
additional_subsidy  INTEGER             -- 추가지원금 (원)
installment_principal INTEGER           -- 할부원금 (원)
additional_services VARCHAR(500)        -- 부가서비스 설명
contract_months     INTEGER             -- 약정개월

-- BaseEntity 필드
```

### 7. bid_history (입찰 수정 이력)

```sql
-- 기본키
id              UUID PRIMARY KEY    -- 이력 고유 ID
bid_id          UUID → bids.id      -- 연관 입찰 ID

-- 이력 정보
version         INTEGER NOT NULL    -- 수정 버전
price           INTEGER NOT NULL    -- 입찰가
delivery_days   INTEGER NOT NULL    -- 배송 예상일

-- BaseEntity 필드
```

### 8. contracts (계약)

```sql
-- 기본키
id              UUID PRIMARY KEY    -- 계약 고유 ID
quote_id        UUID UNIQUE → quotes.id    -- 견적 ID (1:1)
bid_id          UUID UNIQUE → bids.id      -- 입찰 ID (1:1)

-- 계약 정보
status          VARCHAR(255) NOT NULL      -- 계약 상태
signed_at       TIMESTAMP                  -- 서명 시각

-- BaseEntity 필드
```

### 9. payments (결제)

```sql
-- 기본키
id              UUID PRIMARY KEY    -- 결제 고유 ID
contract_id     UUID UNIQUE → contracts.id -- 계약 ID (1:1)

-- 결제 정보
amount          INTEGER NOT NULL    -- 결제 금액
method          VARCHAR(255) NOT NULL -- 결제 방법
status          VARCHAR(255) NOT NULL -- 결제 상태
pg_tid          VARCHAR(255) UNIQUE NOT NULL -- PG 거래 ID
paid_at         TIMESTAMP           -- 결제 완료 시각

-- BaseEntity 필드
```

### 10. deliveries (배송)

```sql
-- 기본키
id              UUID PRIMARY KEY    -- 배송 고유 ID
contract_id     UUID UNIQUE → contracts.id -- 계약 ID (1:1)

-- 배송 정보
courier         VARCHAR(255) NOT NULL      -- 택배사
invoice_number  VARCHAR(255) NOT NULL      -- 송장번호
status          VARCHAR(255) NOT NULL      -- 배송 상태
shipped_at      TIMESTAMP                  -- 발송 시각
delivered_at    TIMESTAMP                  -- 배송 완료 시각
delivery_memo   VARCHAR(255)               -- 배송 메모

-- BaseEntity 필드
```

### 11. notifications (알림)

```sql
-- 기본키
id              UUID PRIMARY KEY    -- 알림 고유 ID
user_id         UUID → users.id     -- 사용자 ID

-- 알림 정보
type            VARCHAR(255) NOT NULL      -- 알림 타입
title           VARCHAR(255) NOT NULL      -- 제목
message         VARCHAR(1000) NOT NULL     -- 메시지 (최대 1000자)
channel         VARCHAR(255) NOT NULL      -- 알림 채널
is_read         BOOLEAN NOT NULL           -- 읽음 여부
reference_id    UUID                       -- 참조 ID

-- BaseEntity 필드
```

### 12. phone_models (휴대폰 모델) - **새로 추가된 테이블**

```sql
-- 기본키
id              UUID PRIMARY KEY    -- 모델 고유 ID

-- 브랜드 및 모델 정보
brand           VARCHAR(255) NOT NULL      -- 브랜드 (APPLE, SAMSUNG, LG 등 - Enum)
model           VARCHAR(255) NOT NULL      -- 모델명 예: "iPhone 16", "Galaxy S24"
model_number    VARCHAR(255)               -- 제조사 모델 번호 예: "A3101"(선택)
released_price  INTEGER                    -- 출시가(원)
released_at     DATE                       -- 출시일

-- BaseEntity 필드
```

### 13. phone_options (휴대폰 옵션) - **새로 추가된 테이블**

```sql
-- 기본키
id              UUID PRIMARY KEY    -- 옵션 고유 ID
model_id        UUID → phone_models.id    -- 모델 ID (1:N)

-- 옵션 정보
option_type     VARCHAR(100) NOT NULL     -- 예: 'COLOR', 'STORAGE'
option_value    VARCHAR(255) NOT NULL     -- 예: 'Black', '128'
display_label   VARCHAR(255)              -- 표시명(예: '블랙', '128GB')

-- BaseEntity 필드
```

## 🔗 외래키 관계

### 1:1 관계

- `users` ↔ `sellers` (user_id)
- `bids` ↔ `price_plans` (price_plan_id)
- `quotes` ↔ `contracts` (quote_id)
- `bids` ↔ `contracts` (bid_id)
- `contracts` ↔ `payments` (contract_id)
- `contracts` ↔ `deliveries` (contract_id)

### 1:N 관계

- `users` → `quotes` (user_id)
- `users` → `notifications` (user_id)
- `sellers` → `seller_documents` (seller_id)
- `sellers` → `bids` (seller_id)
- `quotes` → `bids` (quote_id)
- `bids` → `bid_history` (bid_id)

## 📊 인덱스 정보

### Primary Keys (모든 테이블)

- 모든 테이블에 UUID 기본키 인덱스

### Unique Constraints

- `users.email` - 이메일 유니크
- `users.username` - 사용자명 유니크
- `sellers.user_id` - 사용자당 하나의 판매자 계정
- `bids.price_plan_id` - 입찰당 하나의 요금제
- `contracts.quote_id` - 견적당 하나의 계약
- `contracts.bid_id` - 입찰당 하나의 계약
- `payments.contract_id` - 계약당 하나의 결제
- `payments.pg_tid` - PG 거래 ID 유니크
- `deliveries.contract_id` - 계약당 하나의 배송

### Performance Indexes

```sql
-- 조회 성능 최적화 인덱스
idx_bids_quote_id               -- bids(quote_id) - 견적별 입찰 조회
idx_bids_seller_id              -- bids(seller_id) - 판매자별 입찰 조회
idx_bid_history_bid_id          -- bid_history(bid_id) - 입찰 이력 조회
idx_bid_history_version         -- bid_history(bid_id, version) - 버전별 조회
idx_contracts_bid_id            -- contracts(bid_id) - 입찰별 계약 조회
idx_contracts_quote_id          -- contracts(quote_id) - 견적별 계약 조회
idx_contracts_status            -- contracts(status) - 상태별 계약 조회
idx_deliveries_contract_id      -- deliveries(contract_id) - 계약별 배송 조회
idx_deliveries_invoice_number   -- deliveries(invoice_number) - 송장번호 조회
idx_deliveries_status           -- deliveries(status) - 배송 상태별 조회
idx_notifications_user_id       -- notifications(user_id) - 사용자별 알림 조회
idx_notifications_type          -- notifications(type) - 타입별 알림 조회
idx_notifications_channel       -- notifications(channel) - 채널별 알림 조회
idx_notifications_is_read       -- notifications(is_read) - 읽음 상태별 조회
idx_payments_contract_id        -- payments(contract_id) - 계약별 결제 조회
idx_payments_pg_tid             -- payments(pg_tid) - PG 거래 ID 조회
idx_payments_status             -- payments(status) - 결제 상태별 조회
idx_seller_documents_seller_id  -- seller_documents(seller_id) - 판매자별 문서 조회
idx_seller_documents_type       -- seller_documents(type) - 문서 타입별 조회
```

## 🔄 Enum 값 정의

### Role (사용자 역할)

- `CONSUMER`: 소비자
- `SELLER`: 판매자
- `ADMIN`: 관리자

### Provider (소셜 로그인 제공자)

- `KAKAO`: 카카오
- `NAVER`: 네이버

### ApprovalStatus (승인 상태)

- `PENDING`: 승인 대기
- `APPROVED`: 승인됨
- `REJECTED`: 거부됨

### DocumentType (문서 종류)

- `BUSINESS_LICENSE`: 사업자등록증
- `CONSENT_FORM`: 사전승낙서

### QuoteStatus (견적 상태)

- `OPEN`: 진행중
- `CLOSED`: 마감됨
- `CONTRACTED`: 계약됨

### Carrier (통신사)

- `SKT`: SKT
- `KT`: KT
- `LGU`: LG U+

### PurchaseMethod (구매방법)

- `NUMBER_TRANSFER`: 번호이동
- `DEVICE_CHANGE`: 기기변경
- `NEW_SUBSCRIPTION`: 신규가입
- `ANY`: 상관없음 (Quote만 사용)

### ActivationMethod (개통방법)

- `SELECTIVE_SUBSIDY`: 선택약정
- `COMMON_SUBSIDY`: 공시지원금
- `ANY`: 상관없음 (Quote만 사용)

### ContractStatus (계약 상태)

- `PENDING`: 대기중
- `SIGNED`: 서명완료
- `CANCELLED`: 취소됨

### PaymentStatus (결제 상태)

- `REQUESTED`: 결제 요청
- `PENDING_APPROVAL`: 승인 대기
- `PAID`: 결제 완료
- `FAILED`: 결제 실패

### PaymentMethod (결제 방법)

- `CARD`: 카드
- `BANK`: 무통장입금
- `MOBILE`: 휴대폰 결제

### DeliveryStatus (배송 상태)

- `PREPARING`: 배송 준비중
- `SHIPPED`: 발송됨
- `DELIVERED`: 배송 완료

### Courier (택배사)

- `CJ`: CJ대한통운
- `LOTTE`: 롯데택배
- `HANJIN`: 한진택배

### NotificationType (알림 타입)

- `BID_RECEIVED`: 입찰 도착
- `QUOTE_EXPIRED`: 견적 마감
- `CONTRACT_SIGNED`: 계약 체결

### NotificationChannel (알림 채널)

- `SSE`: Server-Sent Events를 통한 실시간 웹 알림
- `KAKAO`: 카카오톡을 통한 알림톡 발송
- `PUSH`: 브라우저 푸시 알림
- `EMAIL`: 이메일을 통한 알림

## 🎯 쿼리 패턴 가이드

### 자주 사용되는 쿼리 패턴

1. **견적 목록 조회**: `quotes` + `users` 조인
2. **입찰 목록 조회**: `bids` + `sellers` + `users` + `price_plans` 조인
3. **판매자 정보**: `sellers` + `users` 조인
4. **계약 상세**: `contracts` + `quotes` + `bids` + `sellers` + `users` 조인
5. **알림 목록**: `notifications` (user_id 필터)
6. **결제 정보**: `payments` + `contracts` 조인
7. **배송 정보**: `deliveries` + `contracts` 조인

### 성능 고려사항

- LAZY 로딩 기본 사용
- N+1 문제 방지를 위한 JOIN FETCH 활용
- 페이징 처리 필수 (대용량 데이터)
- 페이지네이션 시에는 batch fetching 사용
- 적절한 인덱스 활용
- 복합 쿼리 시 인덱스 힌트 고려

## ⚠️ 주의사항

1. **소프트 삭제**:
   - **BaseEntity 엔터티**: `is_delete = true`, `deleted_at`, `deleted_by` 설정
   - **BaseTimeEntity 엔터티**: `deleted_at`만 설정 (`is_delete`, `deleted_by` 없음)
2. **UUID 사용**: 모든 ID는 UUID 타입
3. **Enum 검증**: 문자열 Enum 값은 애플리케이션에서 검증
4. **외래키 제약**: 데이터 무결성 보장
5. **감사 로그**: BaseEntity/BaseTimeEntity 필드 자동 관리
6. **1:1 관계**: UNIQUE 제약으로 보장
7. **price_plans 테이블**: bids와 1:1 관계로 요금제 정보 분리 저장

## 🚀 확장 계획

### 향후 추가될 테이블

- `phone_brands`: 휴대폰 브랜드
- `phone_models`: 휴대폰 모델
- `phone_options`: 모델별 옵션 (색상, 용량)

이 스키마는 휴대폰 역경매 플랫폼의 모든 비즈니스 요구사항을 지원하도록 설계되었으며, 실제 데이터베이스와 정확히 일치합니다.


## 🏗️ 실제 테이블 구조 (11개 테이블)

### 1. users (사용자)

```sql
-- 기본키
id              UUID PRIMARY KEY    -- 사용자 고유 ID

-- 사용자 정보
username        VARCHAR(255) UNIQUE NOT NULL    -- 사용자명 (로그인 ID)
password        VARCHAR(255) NOT NULL           -- 암호화된 비밀번호
email           VARCHAR(255) UNIQUE NOT NULL    -- 이메일 주소
name            VARCHAR(255) NOT NULL           -- 실명
nickname        VARCHAR(10) NOT NULL            -- 닉네임 (2-10자)
phone           VARCHAR(255)                    -- 휴대폰 번호
role            VARCHAR(255) NOT NULL           -- 사용자 역할 (CONSUMER, SELLER, ADMIN)
provider        VARCHAR(255)                    -- 소셜 로그인 제공자 (KAKAO, NAVER)
provider_id     VARCHAR(255)                    -- 소셜 로그인 고유 ID

-- BaseEntity 필드
is_delete       BOOLEAN
created_at      TIMESTAMP
updated_at      TIMESTAMP
deleted_at      TIMESTAMP
created_by      VARCHAR(255)
updated_by      VARCHAR(255)
deleted_by      VARCHAR(255)
```

### 2. sellers (판매자)

```sql
-- 기본키
seller_id       UUID PRIMARY KEY    -- 판매자 고유 ID
user_id         UUID UNIQUE → users.id     -- 연결된 사용자 ID (1:1)

-- 판매자 정보
business_number VARCHAR(255) NOT NULL      -- 사업자등록번호
store_name      VARCHAR(255) NOT NULL      -- 상호명
approval_status VARCHAR(255) NOT NULL      -- 승인 상태 (PENDING, APPROVED, REJECTED)

-- 주소 정보 (임베디드)
postal_code     VARCHAR(255)               -- 우편번호
address         VARCHAR(255)               -- 주소
detail_address  VARCHAR(255)               -- 상세주소

-- BaseEntity 필드
is_delete       BOOLEAN
created_at      TIMESTAMP
updated_at      TIMESTAMP
deleted_at      TIMESTAMP
created_by      VARCHAR(255)
updated_by      VARCHAR(255)
deleted_by      VARCHAR(255)
```

### 3. seller_documents (판매자 문서)

```sql
-- 기본키
id              UUID PRIMARY KEY    -- 문서 고유 ID
seller_id       UUID → sellers.seller_id  -- 판매자 ID

-- 문서 정보
type            VARCHAR(255) NOT NULL      -- 문서 종류 (BUSINESS_LICENSE, CONSENT_FORM)
file_url        VARCHAR(255) NOT NULL      -- S3 저장 위치
uploaded_at     TIMESTAMP NOT NULL         -- 업로드 시각

-- BaseEntity 필드
is_delete       BOOLEAN
created_at      TIMESTAMP
updated_at      TIMESTAMP
deleted_at      TIMESTAMP
created_by      VARCHAR(255)
updated_by      VARCHAR(255)
deleted_by      VARCHAR(255)
```

### 4. quotes (견적 요청)

```sql
-- 기본키
id              UUID PRIMARY KEY    -- 견적 요청 고유 ID
user_id         UUID → users.id     -- 견적 요청자 ID

-- 제품 정보
model           VARCHAR(255) NOT NULL      -- 휴대폰 모델명 (예: iPhone 16)
storage         VARCHAR(255) NOT NULL      -- 저장 용량 (예: 128GB)
color           VARCHAR(255) NOT NULL      -- 색상
carrier         VARCHAR(255) NOT NULL      -- 희망 통신사 (SKT, KT, LGU)

-- 경매 정보
status          VARCHAR(255) NOT NULL      -- 견적 상태 (OPEN, CLOSED, CONTRACTED)
expired_at      TIMESTAMP NOT NULL         -- 경매 마감 시각

-- 구매 조건
purchase_method VARCHAR(255)               -- 구매방법 (NUMBER_TRANSFER, DEVICE_CHANGE, NEW_SUBSCRIPTION, ANY)
current_carrier VARCHAR(255)               -- 현재 통신사 (번호이동/기기변경 시)
activation_method VARCHAR(255)             -- 개통방법 (SELECTIVE_SUBSIDY, COMMON_SUBSIDY, ANY)

-- BaseEntity 필드
is_delete       BOOLEAN
created_at      TIMESTAMP
updated_at      TIMESTAMP
deleted_at      TIMESTAMP
created_by      VARCHAR(255)
updated_by      VARCHAR(255)
deleted_by      VARCHAR(255)
```

### 5. price_plans (요금제) - **새로 발견된 테이블**

```sql
-- 기본키
id              UUID PRIMARY KEY    -- 요금제 고유 ID

-- 요금제 정보
plan_name       VARCHAR(255)        -- 요금제 이름
plan_price      INTEGER             -- 요금제 가격 (원)
carrier         VARCHAR(255)        -- 통신사 (SKT, KT, LGU)

-- BaseEntity 필드
is_delete       BOOLEAN
created_at      TIMESTAMP
updated_at      TIMESTAMP
deleted_at      TIMESTAMP
created_by      VARCHAR(255)
updated_by      VARCHAR(255)
deleted_by      VARCHAR(255)
```

### 6. bids (입찰)

```sql
-- 기본키
id                  UUID PRIMARY KEY    -- 입찰 고유 ID
quote_id            UUID → quotes.id    -- 대상 견적 ID
seller_id           UUID → sellers.seller_id  -- 입찰자 ID
price_plan_id       UUID UNIQUE → price_plans.id  -- 요금제 ID (1:1)

-- 입찰 정보
price               INTEGER NOT NULL    -- 입찰가 (원)
delivery_days       INTEGER NOT NULL    -- 배송 예상일 (일)
rating_snapshot     DOUBLE PRECISION    -- 입찰 당시 판매자 평점 스냅샷

-- 구매 조건
purchase_method     VARCHAR(255) NOT NULL  -- 구매방법 (NUMBER_TRANSFER, DEVICE_CHANGE, NEW_SUBSCRIPTION, ANY)
carrier             VARCHAR(255) NOT NULL  -- 통신사 (이동할/사용할 통신사) 3사 (SKT, KT, LGU)
current_carrier     VARCHAR(255)           -- 기존 통신사 (번호이동/기기변경 시)
activation_method   VARCHAR(255) NOT NULL  -- 개통방법 (SELECTIVE_SUBSIDY, COMMON_SUBSIDY, ANY)

-- 추가 비용 정보
additional_subsidy  INTEGER             -- 추가지원금 (원)
installment_principal INTEGER           -- 할부원금 (원)
additional_services VARCHAR(500)        -- 부가서비스 설명
contract_months     INTEGER             -- 약정개월

-- BaseEntity 필드
is_delete           BOOLEAN
created_at          TIMESTAMP
updated_at          TIMESTAMP
deleted_at          TIMESTAMP
created_by          VARCHAR(255)
updated_by          VARCHAR(255)
deleted_by          VARCHAR(255)
```

### 7. bid_history (입찰 수정 이력)

```sql
-- 기본키
id              UUID PRIMARY KEY    -- 이력 고유 ID
bid_id          UUID → bids.id      -- 연관 입찰 ID

-- 이력 정보
version         INTEGER NOT NULL    -- 수정 버전
price           INTEGER NOT NULL    -- 입찰가
delivery_days   INTEGER NOT NULL    -- 배송 예상일

-- BaseEntity 필드
is_delete       BOOLEAN
created_at      TIMESTAMP
updated_at      TIMESTAMP
deleted_at      TIMESTAMP
created_by      VARCHAR(255)
updated_by      VARCHAR(255)
deleted_by      VARCHAR(255)
```

### 8. contracts (계약)

```sql
-- 기본키
id              UUID PRIMARY KEY    -- 계약 고유 ID
quote_id        UUID UNIQUE → quotes.id    -- 견적 ID (1:1)
bid_id          UUID UNIQUE → bids.id      -- 입찰 ID (1:1)

-- 계약 정보
status          VARCHAR(255) NOT NULL      -- 계약 상태
signed_at       TIMESTAMP                  -- 서명 시각

-- BaseEntity 필드
is_delete       BOOLEAN
created_at      TIMESTAMP
updated_at      TIMESTAMP
deleted_at      TIMESTAMP
created_by      VARCHAR(255)
updated_by      VARCHAR(255)
deleted_by      VARCHAR(255)
```

### 9. payments (결제)

```sql
-- 기본키
id              UUID PRIMARY KEY    -- 결제 고유 ID
contract_id     UUID UNIQUE → contracts.id -- 계약 ID (1:1)

-- 결제 정보
amount          INTEGER NOT NULL    -- 결제 금액
method          VARCHAR(255) NOT NULL -- 결제 방법
status          VARCHAR(255) NOT NULL -- 결제 상태
pg_tid          VARCHAR(255) UNIQUE NOT NULL -- PG 거래 ID
paid_at         TIMESTAMP           -- 결제 완료 시각

-- BaseEntity 필드
is_delete       BOOLEAN
created_at      TIMESTAMP
updated_at      TIMESTAMP
deleted_at      TIMESTAMP
created_by      VARCHAR(255)
updated_by      VARCHAR(255)
deleted_by      VARCHAR(255)
```

### 10. deliveries (배송)

```sql
-- 기본키
id              UUID PRIMARY KEY    -- 배송 고유 ID
contract_id     UUID UNIQUE → contracts.id -- 계약 ID (1:1)

-- 배송 정보
courier         VARCHAR(255) NOT NULL      -- 택배사
invoice_number  VARCHAR(255) NOT NULL      -- 송장번호
status          VARCHAR(255) NOT NULL      -- 배송 상태
shipped_at      TIMESTAMP                  -- 발송 시각
delivered_at    TIMESTAMP                  -- 배송 완료 시각
delivery_memo   VARCHAR(255)               -- 배송 메모

-- BaseEntity 필드
is_delete       BOOLEAN
created_at      TIMESTAMP
updated_at      TIMESTAMP
deleted_at      TIMESTAMP
created_by      VARCHAR(255)
updated_by      VARCHAR(255)
deleted_by      VARCHAR(255)
```

### 11. notifications (알림)

```sql
-- 기본키
id              UUID PRIMARY KEY    -- 알림 고유 ID
user_id         UUID → users.id     -- 사용자 ID

-- 알림 정보
type            VARCHAR(255) NOT NULL      -- 알림 타입
title           VARCHAR(255) NOT NULL      -- 제목
message         VARCHAR(1000) NOT NULL     -- 메시지 (최대 1000자)
channel         VARCHAR(255) NOT NULL      -- 알림 채널
is_read         BOOLEAN NOT NULL           -- 읽음 여부
reference_id    UUID                       -- 참조 ID

-- BaseEntity 필드
is_delete       BOOLEAN
created_at      TIMESTAMP
updated_at      TIMESTAMP
deleted_at      TIMESTAMP
created_by      VARCHAR(255)
updated_by      VARCHAR(255)
deleted_by      VARCHAR(255)
```

## 🔗 외래키 관계

### 1:1 관계

- `users` ↔ `sellers` (user_id)
- `bids` ↔ `price_plans` (price_plan_id)
- `quotes` ↔ `contracts` (quote_id)
- `bids` ↔ `contracts` (bid_id)
- `contracts` ↔ `payments` (contract_id)
- `contracts` ↔ `deliveries` (contract_id)

### 1:N 관계

- `users` → `quotes` (user_id)
- `users` → `notifications` (user_id)
- `sellers` → `seller_documents` (seller_id)
- `sellers` → `bids` (seller_id)
- `quotes` → `bids` (quote_id)
- `bids` → `bid_history` (bid_id)

## 📊 인덱스 정보

### Primary Keys (모든 테이블)

- 모든 테이블에 UUID 기본키 인덱스

### Unique Constraints

- `users.email` - 이메일 유니크
- `users.username` - 사용자명 유니크
- `sellers.user_id` - 사용자당 하나의 판매자 계정
- `bids.price_plan_id` - 입찰당 하나의 요금제
- `contracts.quote_id` - 견적당 하나의 계약
- `contracts.bid_id` - 입찰당 하나의 계약
- `payments.contract_id` - 계약당 하나의 결제
- `payments.pg_tid` - PG 거래 ID 유니크
- `deliveries.contract_id` - 계약당 하나의 배송

### Performance Indexes

```sql
-- 조회 성능 최적화 인덱스
idx_bids_quote_id               -- bids(quote_id) - 견적별 입찰 조회
idx_bids_seller_id              -- bids(seller_id) - 판매자별 입찰 조회
idx_bid_history_bid_id          -- bid_history(bid_id) - 입찰 이력 조회
idx_bid_history_version         -- bid_history(bid_id, version) - 버전별 조회
idx_contracts_bid_id            -- contracts(bid_id) - 입찰별 계약 조회
idx_contracts_quote_id          -- contracts(quote_id) - 견적별 계약 조회
idx_contracts_status            -- contracts(status) - 상태별 계약 조회
idx_deliveries_contract_id      -- deliveries(contract_id) - 계약별 배송 조회
idx_deliveries_invoice_number   -- deliveries(invoice_number) - 송장번호 조회
idx_deliveries_status           -- deliveries(status) - 배송 상태별 조회
idx_notifications_user_id       -- notifications(user_id) - 사용자별 알림 조회
idx_notifications_type          -- notifications(type) - 타입별 알림 조회
idx_notifications_channel       -- notifications(channel) - 채널별 알림 조회
idx_notifications_is_read       -- notifications(is_read) - 읽음 상태별 조회
idx_payments_contract_id        -- payments(contract_id) - 계약별 결제 조회
idx_payments_pg_tid             -- payments(pg_tid) - PG 거래 ID 조회
idx_payments_status             -- payments(status) - 결제 상태별 조회
idx_seller_documents_seller_id  -- seller_documents(seller_id) - 판매자별 문서 조회
idx_seller_documents_type       -- seller_documents(type) - 문서 타입별 조회
```

## 🔄 Enum 값 정의

### Role (사용자 역할)

- `CONSUMER`: 소비자
- `SELLER`: 판매자
- `ADMIN`: 관리자

### Provider (소셜 로그인 제공자)

- `KAKAO`: 카카오
- `NAVER`: 네이버

### ApprovalStatus (승인 상태)

- `PENDING`: 승인 대기
- `APPROVED`: 승인됨
- `REJECTED`: 거부됨

### DocumentType (문서 종류)

- `BUSINESS_LICENSE`: 사업자등록증
- `CONSENT_FORM`: 사전승낙서

### QuoteStatus (견적 상태)

- `OPEN`: 진행중
- `CLOSED`: 마감됨
- `CONTRACTED`: 계약됨

### Carrier (통신사)

- `SKT`: SKT
- `KT`: KT
- `LGU`: LG U+

### PurchaseMethod (구매방법)

- `NUMBER_TRANSFER`: 번호이동
- `DEVICE_CHANGE`: 기기변경
- `NEW_SUBSCRIPTION`: 신규가입
- `ANY`: 상관없음 (Quote만 사용)

### ActivationMethod (개통방법)

- `SELECTIVE_SUBSIDY`: 선택약정
- `COMMON_SUBSIDY`: 공시지원금
- `ANY`: 상관없음 (Quote만 사용)

### ContractStatus (계약 상태)

- `PENDING`: 대기중
- `SIGNED`: 서명완료
- `CANCELLED`: 취소됨

### PaymentStatus (결제 상태)

- `REQUESTED`: 결제 요청
- `PENDING_APPROVAL`: 승인 대기
- `PAID`: 결제 완료
- `FAILED`: 결제 실패

### PaymentMethod (결제 방법)

- `CARD`: 카드
- `BANK`: 무통장입금
- `MOBILE`: 휴대폰 결제

### DeliveryStatus (배송 상태)

- `PREPARING`: 배송 준비중
- `SHIPPED`: 발송됨
- `DELIVERED`: 배송 완료

### Courier (택배사)

- `CJ`: CJ대한통운
- `LOTTE`: 롯데택배
- `HANJIN`: 한진택배

### NotificationType (알림 타입)

- `BID_RECEIVED`: 입찰 도착
- `QUOTE_EXPIRED`: 견적 마감
- `CONTRACT_SIGNED`: 계약 체결

### NotificationChannel (알림 채널)

- `SSE`: Server-Sent Events를 통한 실시간 웹 알림
- `KAKAO`: 카카오톡을 통한 알림톡 발송
- `PUSH`: 브라우저 푸시 알림
- `EMAIL`: 이메일을 통한 알림

## 🎯 쿼리 패턴 가이드

### 자주 사용되는 쿼리 패턴

1. **견적 목록 조회**: `quotes` + `users` 조인
2. **입찰 목록 조회**: `bids` + `sellers` + `users` + `price_plans` 조인
3. **판매자 정보**: `sellers` + `users` 조인
4. **계약 상세**: `contracts` + `quotes` + `bids` + `sellers` + `users` 조인
5. **알림 목록**: `notifications` (user_id 필터)
6. **결제 정보**: `payments` + `contracts` 조인
7. **배송 정보**: `deliveries` + `contracts` 조인

### 성능 고려사항

- LAZY 로딩 기본 사용
- N+1 문제 방지를 위한 JOIN FETCH 활용
- 페이징 처리 필수 (대용량 데이터)
- 페이지네이션 시에는 batch fetching 사용
- 적절한 인덱스 활용
- 복합 쿼리 시 인덱스 힌트 고려

## ⚠️ 주의사항

1. **소프트 삭제**:
   - **BaseEntity 엔터티**: `is_delete = true`, `deleted_at`, `deleted_by` 설정
   - **BaseTimeEntity 엔터티**: `deleted_at`만 설정 (`is_delete`, `deleted_by` 없음)
2. **UUID 사용**: 모든 ID는 UUID 타입
3. **Enum 검증**: 문자열 Enum 값은 애플리케이션에서 검증
4. **외래키 제약**: 데이터 무결성 보장
5. **감사 로그**: BaseEntity/BaseTimeEntity 필드 자동 관리
6. **1:1 관계**: UNIQUE 제약으로 보장
7. **price_plans 테이블**: bids와 1:1 관계로 요금제 정보 분리 저장

## 🚀 확장 계획

### 향후 추가될 테이블

- `phone_brands`: 휴대폰 브랜드
- `phone_models`: 휴대폰 모델
- `phone_options`: 모델별 옵션 (색상, 용량)

이 스키마는 휴대폰 역경매 플랫폼의 모든 비즈니스 요구사항을 지원하도록 설계되었으며, 실제 데이터베이스와 정확히 일치합니다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/phonebid)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/phonebid)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
