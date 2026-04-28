## gateway-api

> API Gateway 컴포넌트 상세 기술 요구사항 및 구현 가이드


# API Gateway: 고성능 BFF (Backend for Frontend)

## 🎯 컴포넌트 역할
EKS 환경에서 동작하는 고성능 BFF로서 외부 클라이언트의 단일 진입점 역할을 수행

## 📋 컨텍스트
- **시스템 목표**: 30k RPS 유입 트래픽 안정 처리
- **주요 책임**:
  - JWT 검증 (OIDC/Cognito 연동)
  - 요청 표준화 및 집계
  - Idempotency-Key 검증 및 관리
  - 백엔드 서비스로의 REST 포워딩
  - 선택적 캐싱 및 요청 합성

## 🛠️ 기술 스택

### Core Framework
- **언어**: Go 1.22+
- **런타임**: Linux ARM64 (Graviton)
- **웹 프레임워크**: Fiber v2
- **모듈 시스템**: Go Modules

### 인증 & 보안
- **JWT 라이브러리**: lestrrat-go/jwx v2
- **JWK 캐시**: Redis 기반 (10분 TTL)
- **시큐리티 헤더**: 자동 추가 (CSP, HSTS, X-Frame-Options)

### 데이터 저장소
- **캐시**: Redis (ElastiCache)
  - 레이트 리미팅 카운터
  - Idempotency 키 저장
  - JWK 캐시
- **메트릭**: Prometheus 클라이언트
- **트레이싱**: OpenTelemetry OTLP

### HTTP 클라이언트
- **표준 라이브러리**: net/http
- **커스텀 트랜스포트**:
  - 타임아웃 설정 (600ms 백엔드 호출)
  - 커넥션 풀 (MaxIdleConns: 2000)
  - 재시도 로직 (멱등 요청만)

## 🏗️ 아키텍처 설계

### 프로젝트 구조
```
cmd/gateway/main.go                    # 애플리케이션 진입점
internal/
├── config/config.go                  # 환경설정 관리
├── logging/logger.go                 # 구조화 로깅
├── metrics/metrics.go                # Prometheus 메트릭
├── middleware/                       # 미들웨어 컴포넌트
│   ├── auth.go                      # JWT 인증
│   ├── idempotency.go              # 멱등성 관리
│   ├── ratelimit.go               # 레이트 리미팅
│   ├── tracing.go                 # 분산 추적
│   └── manager.go                 # 미들웨어 관리자
├── clients/                        # 백엔드 클라이언트
│   ├── reservation.go             # 예약 API 클라이언트
│   └── payment.go                # 결제 API 클라이언트
└── routes/                         # API 핸들러
    ├── queue.go                   # 대기열 엔드포인트
    ├── reservation.go            # 예약 엔드포인트
    ├── payment.go               # 결제 엔드포인트
    └── routes.go                # 라우트 설정
```

### 설정 관리
- **환경변수 기반**: 12-factor 앱 준수
- **필수 변수 검증**: 부팅 시점 유효성 검사
- **기본값 제공**: 개발 편의성 고려

## 🔌 API 엔드포인트

### 공개 엔드포인트
```yaml
POST /api/v1/queue/join        # 대기열 참여 (익명 허용)
GET  /api/v1/queue/status      # 대기열 상태 조회
POST /api/v1/reservations      # 예약 생성 (멱등성 필수)
POST /api/v1/reservations/{id}/cancel  # 예약 취소
POST /api/v1/reservations/{id}/confirm # 예약 확정
POST /api/v1/payment/intent    # 결제 인텐트 생성
GET  /healthz                  # 헬스 체크
GET  /readyz                   # readiness 체크
GET  /version                  # 버전 정보
GET  /metrics                  # Prometheus 메트릭
```

### 인증 요구사항
- **Bearer 토큰**: `Authorization: Bearer <JWT>`
- **JWK 검증**: JWKS 엔드포인트 기반 동적 검증
- **클레임 검증**: aud, iss, exp 필수 검증
- **익명 엔드포인트**: 화이트리스트 기반 예외 처리

### 멱등성 관리
- **헤더 요구**: `Idempotency-Key: <uuid-v4>`
- **저장소**: Redis (TTL: 5분)
- **충돌 처리**: 동일 키 + 다른 바디 = 409 에러
- **응답 캐시**: 성공 응답 임시 저장

### 레이트 리미팅
- **알고리즘**: 토큰 버킷 (Redis Lua 기반)
- **제한**: IP/사용자별 50 RPS, 버스트 100
- **응답**: 429 + Retry-After 헤더
- **예외**: 헬스체크 엔드포인트

## 🔄 백엔드 연동

### Reservation API (Spring Boot)
```yaml
base_url: http://reservation-api.tickets-api.svc.cluster.local:8010
timeout: 600ms
mapping:
  /api/v1/reservations/* → /v1/reservations/*
headers:
  - x-request-id
  - traceparent
  - authorization
```

### Payment API (Go)
```yaml
base_url: http://payment-sim-api.tickets-api.svc.cluster.local:8030
timeout: 400ms
mapping:
  /api/v1/payment/* → /v1/sim/*
```

## 📊 관측성 (Observability)

### 메트릭 수집
```yaml
http_server_requests_total{method, route, status_code}
http_server_requests_duration_seconds_bucket{method, route, status_code}
backend_call_duration_seconds{service, method, status_code}
ratelimit_dropped_total{}
idempotency_hits_total{type}  # "hit" or "miss"
```

### 구조화 로깅
```json
{
  "ts": "2024-01-01T12:00:00Z",
  "level": "info",
  "msg": "request_completed",
  "http": {
    "method": "POST",
    "route": "/api/v1/reservations",
    "status": 201
  },
  "latency_ms": 45.2,
  "trace_id": "abc123...",
  "user_id": "user456..."
}
```

### 트레이싱
- **샘플링**: 기본 10%, 예약 경로는 100%
- **컨텍스트 전파**: W3C Trace Context
- **백엔드 추적**: 모든 아웃바운드 요청에 헤더 주입

## ⚡ 성능 요구사항

### 목표 지표
- **P95 지연시간**: < 50ms (백엔드 제외)
- **에러율**: < 0.5%
- **429 비율**: ≤ 1% (Admission 컨트롤)

### 최적화 포인트
- **GOMAXPROCS**: CPU 코어 수에 맞춤
- **HTTP/1.1 우선**: Envoy/ALB 호환성
- **커넥션 풀**: MaxIdleConns=2000, IdlePerHost=1000
- **메모리 관리**: ReadOnlyRootFilesystem

### HPA 고려사항
- **CPU 기반**: 60% utilization
- **메모리 기반**: 70% utilization
- **커스텀 메트릭**: RPS 기반 스케일링

## 🚀 배포 요구사항

### Kubernetes 리소스
```yaml
requests:
  cpu: 200m
  memory: 256Mi
limits:
  cpu: 1000m
  memory: 512Mi
```

### 보안 컨텍스트
```yaml
runAsNonRoot: true
runAsUser: 10001
runAsGroup: 10001
readOnlyRootFilesystem: true
capabilities:
  drop: [ALL]
```

### 네트워크 정책
- **인그레스**: Gateway API 컨트롤러만 허용
- **이그레스**: 백엔드 서비스 + OTLP 콜렉터만 허용

## 🧪 테스트 요구사항

### 유닛 테스트
- **커버리지**: 80% 이상 목표
- **핸들러별**: 미들웨어, 클라이언트, 라우트별 테스트
- **모킹**: 백엔드 서비스, Redis, JWKS 모킹

### 통합 테스트
- **API 계약**: OpenAPI 스펙 기반 검증
- **부하 테스트**: k6 기반 시나리오 테스트
- **E2E 테스트**: 전체 플로우 검증

### 성능 테스트
```javascript
// k6 스크립트 구조
export const options = {
  scenarios: {
    load_test: {
      stages: [
        { duration: '1m', target: 500 },
        { duration: '3m', target: 2000 },
        { duration: '5m', target: 5000 },
      ]
    }
  },
  thresholds: {
    'http_req_duration{status:200}': ['p(95)<500'],
    'http_req_failed': ['rate<0.05'],
  }
}
```

## 📋 구현 체크리스트

### 필수 기능
- [ ] JWT 인증 미들웨어
- [ ] Idempotency 키 검증
- [ ] Redis 기반 레이트 리미팅
- [ ] 백엔드 API 클라이언트
- [ ] 구조화된 로깅
- [ ] Prometheus 메트릭
- [ ] OpenTelemetry 트레이싱
- [ ] 헬스체크 엔드포인트

### 성능 최적화
- [ ] HTTP 클라이언트 튜닝
- [ ] 메모리 누수 방지
- [ ] Graceful shutdown
- [ ] 컨텍스트 타임아웃

### 보안 강화
- [ ] 보안 헤더 추가
- [ ] 입력 검증 강화
- [ ] 에러 정보 노출 제한
- [ ] OWASP Top 10 대응

---

*API Gateway 컴포넌트의 상세한 기술 요구사항 및 구현 가이드입니다.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/traffic-tacos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
