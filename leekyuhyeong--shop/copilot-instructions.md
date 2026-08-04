## shop

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run Commands

```bash
# Build (tests are skipped by default)
mvn clean package -DskipTests

# Run development server (port 8081)
mvn spring-boot:run -Dspring-boot.run.profiles=dev

# Run production profile (port 8080)
mvn spring-boot:run -Dspring-boot.run.profiles=prod

# Docker deployment (compose v2 — 서버에 v1 docker-compose 바이너리 없음)
docker compose up -d
```

## Technology Stack

- **Java 17** with Spring Boot 3.4.1
- **Database**: MariaDB with Spring Data JPA
- **View**: JSP with JSTL (WAR packaging)
- **Build**: Maven (wrapper included: `mvnw`)

## Architecture

### Layered Structure
```
Controller → Service → Repository → Entity
```

### Package Organization (`com.kh.shop`)
- `controller/admin/` - Admin dashboard endpoints
- `controller/client/` - Customer-facing endpoints
- `controller/common/` - Shared endpoints
- `service/` - Business logic (16 services)
- `repository/` - Spring Data JPA repositories (24)
- `entity/` - JPA entities (24)
- `scheduler/` - Batch schedulers (22)
- `config/` - Configuration beans
- `util/` - Utilities (ProfanityFilter)
- `common/dto/` - PageRequestDTO/PageResponseDTO for pagination

### Views (`src/main/webapp/WEB-INF/views/`)
- `/admin/` - Admin dashboard views
- `/client/` - Customer storefront views
- `/common/` - Shared components

### Static Resources (`src/main/resources/static/`)
- `css/admin/`, `css/client/`, `css/common/` - Stylesheets by area
- `js/admin/`, `js/client/`, `js/common/` - JavaScript by area
- CSS uses custom properties (variables) for theming with dark mode support (`body.dark-mode`)

### Key Entities
User, Product, ProductImage, Category, Order, OrderItem, Cart, Review, Wishlist, Point, Coupon, SocialAccount, DailyStats

## Configuration Profiles

- `application.properties` - Default settings
- `application-dev.properties` - Development (port 8081, local MariaDB)
- `application-prod.properties` - Production (port 8080, Docker)
- `application-secret.properties` - API keys and credentials (gitignored)

## External Integrations

- **Payment**: Portone (KG이니시스)
- **OAuth**: Kakao, Naver, Google

## Batch Scheduling

22 scheduled batch jobs handle background processing:
- Product management (rankings)
- Order processing (status updates, auto-cancellation)
- User management (dormant users)
- Cleanup (carts, temp files, sessions, logs)
- Alerts (low stock, reviews, wishlist price changes)
- Stats aggregation

## Deployment

CI/CD via GitHub Actions (`.github/workflows/deploy.yml`):
1. Maven build with JDK 17
2. Docker image build and push to Docker Hub
3. SSH deploy to server via docker-compose

## Security Features

- **CSRF Protection**: Token-based with `CsrfFilter` (excludes `/login`, `/signup`, webhooks)
- **XSS Prevention**: Input sanitization via `XssSanitizer`, Content Security Policy headers
- **Duplicate Login Prevention**: `SessionRegistry` tracks active sessions per user; new login terminates existing session with notification
- **Security Headers**: Configured in `security-headers.jsp` (X-Frame-Options, X-Content-Type-Options, etc.)

## Notes

- UI is in Korean (한국어)
- Session-based authentication with `loggedInUser` session attribute
- Review system requires purchase with DELIVERED order status before writing

## Commit Rules

- 커밋 메시지에 Co-Authored-By 등 Claude와 함께 했다는 내용을 포함하지 않는다

## Outstanding Security Work

- 진행 중인 보안 / 품질 개선 항목은 레포 루트의 `SECURITY_TODO.md` 에 정리되어 있다. 새 audit 결과는 이 파일에 추가하고, 완료된 항목은 `[x]` 로 체크한다.
- 외부 시크릿(Portone / OAuth 3종 / Gmail) 회전 절차는 `SECRETS_ROTATION.md` 에 정리되어 있다.

## 서버 인프라 (SSOT 참조)

- **서버/배포 인프라 SSOT: `D:\server-infra.md`** (로컬 전용, git 미추적 — 리포·운영서버에 없음)
- 포트·도메인·방화벽·컨테이너 TZ 규칙(`Asia/Seoul` 의무)·배포 반영 매트릭스(푸시 시 서버 자동/수동 반영 범위)·트러블슈팅은 전부 그 문서 참조.
- 리포별 `server-infra-*.md`는 폐지됨(2026-06-06). **인프라(compose/nginx/포트/배포) 변경 시 `D:\server-infra.md`를 함께 최신화할 것.**

---
> Source: [LeeKyuHyeong/shop](https://github.com/LeeKyuHyeong/shop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
