## pharmaceutical-stockist

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Full Stack (Docker)
```bash
docker compose up --build          # Build and start all services
docker compose down                # Stop all services
./scripts/create_admin.sh          # Create admin user after stack is up
```

### Backend (Spring Boot / Maven)
```bash
cd backend
mvn clean install                  # Build
mvn spring-boot:run                # Run locally
mvn test                           # Run all tests
mvn test -Dtest=ClassName          # Run single test class
mvn test -Dtest=ClassName#method   # Run single test method
mvn clean package -DskipTests      # Build JAR without tests
```

### Frontend (React / Vite)
```bash
cd frontend
npm install
npm run dev        # Dev server (proxies /api to localhost:8080)
npm run build      # Production build
npm run lint       # ESLint (strict, --max-warnings 0)
```

### E2E Tests (Playwright)
```bash
cd frontend
npx playwright test                          # All tests (3 browsers)
npx playwright test --project=chromium       # Single browser
npx playwright test tests/login.spec.ts      # Single spec file
```

### API Testing
```bash
./scripts/test-api.sh              # 56+ endpoint tests against running stack
```

## Architecture

This is a full-stack pharmaceutical e-commerce platform with 8 Docker services:

```
Browser → Nginx (port 3000) → serves React frontend static files
                             → proxies /api/* → Spring Boot backend (port 8080)
                                              → PostgreSQL (port 5432)
Monitoring: Prometheus (9090) → Grafana (3001), Alertmanager (9093), Loki (3100)
```

**The backend port 8080 is NOT exposed to the host** — only accessible within the Docker network. Always use `docker exec pharma-backend curl ...` to test backend endpoints directly.

### Backend (`backend/src/main/java/com/pharma/`)

- `controller/` — 10 REST controllers delegating to services
- `service/` — 15 business logic services (all core logic lives here)
- `model/` — 9 JPA entities + enums (`Role`: CUSTOMER/ADMIN, `OrderStatus`)
- `dto/` — Request/Response DTOs; all API responses wrapped in `ApiResponse<T>`
- `security/` — `JwtAuthenticationFilter`, RSA RS256 JWT (30-min expiry, HttpOnly cookies)
- `exception/` — `GlobalExceptionHandler`, `ResourceNotFoundException`, `InsufficientStockException`
- `scheduler/` — Scheduled low-stock alert emails
- `config/` — `SecurityConfig`, `CorsConfig`, `WebConfig`

Key patterns:
- All responses wrapped in `ApiResponse<T>`
- Lombok on all entities (`@Data`, `@NoArgsConstructor`, `@AllArgsConstructor`)
- `@CreationTimestamp` / `@UpdateTimestamp` on entities
- Async email via `@EnableAsync`; scheduled tasks via `@EnableScheduling`
- Dev OTP override: hardcoded `123456` in `application-dev.properties`
- Tests use H2 in-memory DB; JaCoCo coverage auto-generated on `mvn test`

### Frontend (`frontend/src/`)

- `pages/` — 14 customer pages + 5 admin pages (route-level components)
- `components/` — Reusable UI; includes `ProtectedRoute` and `AdminRoute` guards
- `services/` — 9 Axios-based API client modules
- `stores/` — Zustand stores for cart and auth state
- `context/` — `ToastProvider` for notifications
- `types/` — TypeScript interfaces for all API shapes
- `hooks/` — Custom React hooks

Key patterns:
- Axios interceptors inject JWT on every request
- Strict TypeScript: `noUnusedLocals`, `noUnusedParameters` enforced
- Tailwind CSS only — no custom CSS files
- React Hook Form + Zod for form validation

### Database

9 entities: `User`, `Address`, `Cart`, `CartItem`, `Order`, `OrderItem`, `Product`, `Category`, `SubCategory`. Schema is auto-managed by Hibernate DDL (`spring.jpa.hibernate.ddl-auto`).

Admin role is set directly in the DB: `UPDATE users SET role = 'ADMIN' WHERE email = '...'` — there is no admin promotion API endpoint.

### Monitoring

- Prometheus scrapes `/actuator/prometheus` every 15s
- Alert rules defined in `monitoring/alert-rules.yml`
- Alertmanager config in `monitoring/alertmanager.yml` (Gmail SMTP — requires setup)
- Grafana datasources auto-provisioned from `monitoring/grafana-datasources.yml`

### Scripts

- `scripts/build_and_deploy.sh` — Full build + deploy orchestrator
- `scripts/create_admin.sh` — Creates admin user; all curl calls run inside `pharma-backend` container
- `scripts/test-api.sh` — Integration test script for all API endpoints
- `scripts/verify_*.sh` — Post-deployment verification

---
> Source: [dave66784/pharmaceutical-stockist](https://github.com/dave66784/pharmaceutical-stockist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
