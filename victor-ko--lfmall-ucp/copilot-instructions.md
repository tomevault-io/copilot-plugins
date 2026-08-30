## lfmall-ucp

> UCP (Universal Commerce Protocol) 연동 프로젝트

# lfmall-ucp 프로젝트

## 개요
UCP (Universal Commerce Protocol) 연동 프로젝트
- Google이 2026년 1월 NRF 컨퍼런스에서 발표한 AI 에이전트용 커머스 표준 프로토콜
- Shopify, Etsy, Wayfair, Target, Walmart 등과 공동 개발한 오픈 스탠다드
- AI 에이전트가 상품 탐색 → 구매 → 구매 후 지원까지 전 과정을 처리할 수 있게 해줌
- MCP(Model Context Protocol), A2A(Agent2Agent), AP2(Agent Payments Protocol)와 호환

## 스택
- Spring Boot 3.2.5 / Java 17 / Gradle
- UCP version: 2026-01-11
- H2 (개발용 인메모리 DB) → 실 환경에서 MySQL/PostgreSQL 교체 필요
- Redis (Idempotency 처리용) → 현재는 ConcurrentHashMap으로 임시 구현

## 프로젝트 구조
```
src/main/java/com/lfmall/ucp/
├── LfmallUcpApplication.java
├── config/
│   ├── UcpProperties.java       # application.yml ucp.* 바인딩
│   └── WebConfig.java           # CORS 설정
├── controller/
│   ├── UcpProfileController.java  # GET /.well-known/ucp (Discovery)
│   └── CheckoutController.java    # Checkout CRUD
├── dto/
│   ├── request/
│   │   ├── CheckoutRequest.java
│   │   ├── ConfirmRequest.java
│   │   └── LineItemRequest.java
│   └── response/
│       ├── CheckoutResponse.java
│       └── UcpProfileResponse.java
├── filter/
│   └── UcpAgentFilter.java      # UCP-Agent 헤더 검증
├── service/
│   └── CheckoutService.java     # 핵심 비즈니스 로직
└── exception/
    ├── GlobalExceptionHandler.java
    ├── CheckoutNotFoundException.java
    └── DuplicateRequestException.java
```

## 구현된 API 엔드포인트
| Method | Path | 설명 |
|--------|------|------|
| GET | `/.well-known/ucp` | AI 에이전트 Discovery (capability 목록 반환) |
| POST | `/ucp/checkout-sessions` | 체크아웃 세션 생성 |
| GET | `/ucp/checkout-sessions/{id}` | 세션 조회 |
| PUT | `/ucp/checkout-sessions/{id}` | 세션 업데이트 |
| POST | `/ucp/checkout-sessions/{id}/confirm` | 결제 확정 및 주문 생성 |
| DELETE | `/ucp/checkout-sessions/{id}` | 세션 취소 |

## 필수 요청 헤더
- `UCP-Agent`: AI 에이전트 식별자 (Discovery 제외 필수)
- `idempotency-key`: 중복 요청 방지 (POST/PUT 필수)
- `request-id`: 요청 추적용 (선택)

## 동작 확인 완료
- `GET http://localhost:8080/.well-known/ucp` → capability 목록 정상 반환

## 다음 작업 (TODO)
1. **실제 이커머스 서비스 연동**
   - `CheckoutService`의 TODO 주석 위치에 기존 서비스 연결
   - 재고 확인: `inventoryService.validateStock()`
   - 가격 계산: `pricingService.calculate()`
   - 결제 처리: `paymentService.process()`
   - 주문 생성: `orderService.createOrder()`

2. **Idempotency Redis 교체**
   - 현재: `ConcurrentHashMap` (인메모리, 서버 재시작 시 초기화)
   - 목표: Redis로 교체하여 TTL 설정 및 다중 인스턴스 지원

3. **결제 핸들러 연동**
   - KakaoPay
   - NaverPay
   - 신용카드 (토스페이먼츠 등)

4. **보안 강화**
   - UCP-Agent 서명 검증 (`request-signature` 헤더)
   - API 인증 추가

## 로컬 실행
```bash
./gradlew bootRun
# 또는 IntelliJ에서 LfmallUcpApplication 실행
# 포트: 8080
```

## 테스트 예시 (Postman)
```
POST http://localhost:8080/ucp/checkout-sessions
Headers:
  Content-Type: application/json
  UCP-Agent: profile="https://agent.example/profile"
  idempotency-key: test-key-001

Body:
{
  "lineItems": [{"itemId": "product_001", "title": "테스트 상품", "quantity": 2}],
  "buyer": {"fullName": "홍길동", "email": "hong@lfmall.com"},
  "currency": "KRW"
}
```

## graphify

This project has a graphify knowledge graph at graphify-out/.

Rules:
- Before answering architecture or codebase questions, read graphify-out/GRAPH_REPORT.md for god nodes and community structure
- If graphify-out/wiki/index.md exists, navigate it instead of reading raw files
- For cross-module "how does X relate to Y" questions, prefer `graphify query "<question>"`, `graphify path "<A>" "<B>"`, or `graphify explain "<concept>"` over grep — these traverse the graph's EXTRACTED + INFERRED edges instead of scanning files
- After modifying code files in this session, run `graphify update .` to keep the graph current (AST-only, no API cost)

---
> Source: [Victor-Ko/lfmall-ucp](https://github.com/Victor-Ko/lfmall-ucp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
