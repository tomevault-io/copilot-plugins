## demo-repository

> This file provides context for AI assistants (Claude, Copilot, etc.) working on this repository. Keep it updated as the project evolves.

# CLAUDE.md — Jewel-Trade (B2B Jewel-Mall)

This file provides context for AI assistants (Claude, Copilot, etc.) working on this repository. Keep it updated as the project evolves.

---

## Project Overview

**Jewel-Trade** is a B2B precious-metals export company platform built for **Deco Indco**. It replaces manual Excel-based invoicing and messenger-based order management with a web-based Order Management System (OMS).

- **Type:** B2B e-commerce / OMS
- **Audience:** Overseas buyers and internal admins
- **Languages:** Korean (primary team language), English (single supported UI language)
- **Timeline:** 8 weeks; currently in early development (core features starting weeks 4–7)

### Business Context

| Problem (As-Is) | Solution (To-Be) |
|---|---|
| Orders received via messenger | Web-based order/quote request |
| Manual Excel invoice creation | Auto-generated PDF invoices |
| No order status visibility | Real-time order lifecycle tracking |

---

## Repository Structure

```
demo-repository/
├── demo/                         # Spring Boot backend (main application)
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/
│   │   │   │   ├── com/example/demo/
│   │   │   │   │   └── DemoApplication.java      # Entry point (@SpringBootApplication)
│   │   │   │   └── com/deco/demo/
│   │   │   │       └── SecurityConfig.java        # Spring Security configuration
│   │   │   └── resources/
│   │   │       └── application.properties         # Application settings
│   │   └── test/
│   │       └── java/com/example/demo/
│   │           └── DemoApplicationTests.java      # Context load test
│   ├── build.gradle                               # Gradle build configuration
│   ├── settings.gradle
│   └── gradlew / gradlew.bat                      # Gradle wrapper scripts
├── .github/
│   └── workflows/
│       ├── proof-html.yml                         # HTML validation CI
│       └── auto-assign.yml                        # Auto-assign issues/PRs to ksh322
├── index.html                                     # Static demo/landing page
├── package.json                                   # NPM config (Primer CSS dependency)
├── README.md                                      # Team guide (Korean)
├── PRD.md                                         # Product Requirements Document
├── erd.md                                         # Entity Relationship Diagram & site map
└── CLAUDE.md                                      # This file
```

**Package naming convention:**
- `com.example.demo` — Spring Boot application entry point (default generated)
- `com.deco.demo` — Company-specific configuration classes

---

## Tech Stack

### Backend
| Layer | Technology | Version |
|---|---|---|
| Language | Java | 17 |
| Framework | Spring Boot | 3.5.10 |
| Build tool | Gradle | 8.14.4 |
| Security | Spring Security | (Boot managed) |
| ORM | Spring Data JPA | (Boot managed) |
| Query | QueryDSL | 5.1.0 (jakarta) |
| Cache/Session | Spring Data Redis | (Boot managed) |
| API docs | SpringDoc OpenAPI (Swagger) | 2.8.1 |
| HTTP clients | Spring Cloud OpenFeign | 2025.0.1 |
| Validation | Spring Boot Validation | (Boot managed) |
| Utilities | Lombok | (Boot managed) |
| DB (prod) | PostgreSQL | (runtime) |
| DB (test) | H2 | (runtimeOnly) |

### Frontend (planned)
| Layer | Technology |
|---|---|
| Framework | TypeScript React |
| Rendering | Next.js (possible) |
| CSS | Primer CSS 17.0.1 (current static page) |

### Infrastructure
| Service | Purpose |
|---|---|
| AWS (Free-tier) / Oracle | Hosting |
| Docker | Containerization |
| EKS | Orchestration |
| Railway | Alternative deployment |
| Redis | JWT blacklist + session management |
| PostgreSQL | Primary database |

---

## Build & Run Commands

All commands run from the `demo/` directory.

```bash
# Build the project
./gradlew build

# Run all tests
./gradlew test

# Run the application (dev mode with hot reload)
./gradlew bootRun

# Clean build artifacts and QueryDSL generated sources
./gradlew clean

# Build without running tests
./gradlew build -x test
```

QueryDSL Q-classes are auto-generated to `demo/src/main/generated/` during compilation and cleaned by `./gradlew clean`.

---

## Configuration

### application.properties
Currently minimal. Environment-specific values should be injected at runtime via environment variables or Spring profiles.

Required environment variables (not yet in `.env.example` — create one):
```
SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/jeweltrade
SPRING_DATASOURCE_USERNAME=
SPRING_DATASOURCE_PASSWORD=
SPRING_DATA_REDIS_HOST=
SPRING_DATA_REDIS_PORT=6379
SPRING_MAIL_HOST=
SPRING_MAIL_USERNAME=
SPRING_MAIL_PASSWORD=
JWT_SECRET=
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
```

### Spring Security
File: `demo/src/main/java/com/deco/demo/SecurityConfig.java`

- CSRF is **disabled** (convenience for testing — re-enable before production)
- Swagger endpoints (`/v3/api-docs/**`, `/swagger-ui/**`) are public
- All other endpoints require authentication
- Successful form login redirects to `/swagger-ui/index.html`

---

## MVP Feature Scope

### In Scope
| Feature | Description |
|---|---|
| Company branding | Landing page: Brand, Catalog, How We Work, About Us, Contact |
| Authentication | Email login for Admin + Buyer; Google OAuth2 (if time permits) |
| Order Management | Cart/Quote → Order Request → Invoice → T/T payment confirmation → Shipping → Complete |
| PDF Invoice | Auto-generated on order approval; downloadable |
| Admin panel | User (CRUD), Product (CRUD), Order approval |
| Infrastructure | AWS deployment + domain connection |

### Out of Scope
- PG payment gateway (T/T bank transfer used instead)
- Real-time inventory / ERP integration
- Multi-language support

### Order Lifecycle
```
접수 (Received)
  → 인보이스 발행 (Invoice Issued)
  → T/T 입금 확인 (T/T Payment Confirmed — receipt upload by buyer)
  → 배송 중 (Shipping)
  → 완료 (Complete)
```

### Access Control
- Buyers must be granted purchase permission by an admin after registration
- Admin role: full CRUD + order approval + invoice generation
- Buyer role: browse catalog, manage cart, submit orders, upload payment receipts, track order status

---

## API Documentation

Swagger UI is available at `/swagger-ui.html` (or `/swagger-ui/index.html`) when the application is running. OpenAPI spec at `/v3/api-docs`.

By default, Swagger UI and the OpenAPI docs are publicly accessible without authentication because `/swagger-ui/**`, `/swagger-ui.html`, and `/v3/api-docs/**` are permitted in `SecurityConfig`. If you change security settings, update this section accordingly.

---

## Testing

### Framework
- JUnit 5 (Jupiter) via `useJUnitPlatform()`
- Spring Boot Test (`@SpringBootTest`)
- Spring Security Test
- H2 in-memory database for test isolation

### Running Tests
```bash
./gradlew test
```

### Current Coverage
- `DemoApplicationTests.contextLoads()` — verifies Spring context initializes without error

New tests should mirror the `src/main/java` package structure under `src/test/java`.

---

## CI/CD (GitHub Actions)

### Workflows
| Workflow | File | Trigger | Purpose |
|---|---|---|---|
| Proof HTML | `.github/workflows/proof-html.yml` | Every push + manual dispatch | Validates HTML syntax in repo |
| Auto Assign | `.github/workflows/auto-assign.yml` | Issue/PR opened | Auto-assigns to `ksh322` |

---

## Git Workflow & Conventions

### Branch Strategy
- **Never** commit directly to `main`/`master`
- All work happens on **personal feature branches**
- Merge into `main` via **Pull Request only**, after code review

### Commit Message Format
```
<type>: <description>

Types:
  feat    — new feature
  fix     — bug fix
  style   — formatting only (no logic change)
  docs    — documentation updates
  chore   — build scripts, config, dependency updates
```

**Examples:**
```
feat: 사용자 로그인 기능 추가
fix: 메인 페이지 이미지 깨짐 현상 수정
docs: README.md 내용 업데이트
chore: .gitignore 파일 업데이트
```

### Code Review
PRs must pass review by a mentor or team lead before merging to `main`. Copilot / AI code review is also encouraged.

---

## Team

| Name | Role | Responsibilities |
|---|---|---|
| 17상호 | Planning / Infra / QA | Business logic, team coordination, infra, README |
| 17진욱 | Frontend | Figma wireframes, React client, API integration, FE QA |
| 17상우 | Backend / Infra | ERD, DB, API, Swagger, CI/CD, BE QA |
| 준영 (planned) | Chatbot / Data Viz | Chatbot, data visualization, CLAUDE.md |

**Mentors:** BE (AUTOEVER), FE (ex-Toss), Infra (AWS Korea)

---

## Key Conventions for AI Assistants

1. **Java style:** Use Lombok annotations (`@Getter`, `@Builder`, etc.) to reduce boilerplate. Follow existing package separation (`com.example.demo` vs `com.deco.demo`).

2. **Security:** Do not remove or weaken Spring Security configuration. CSRF is currently disabled for development — flag this if touching security code.

3. **QueryDSL:** Q-classes live in `src/main/generated/` and are gitignored. Do not commit generated Q-classes. Reference them as `QEntity` in queries.

4. **Database:** Use JPA + QueryDSL for queries. Prefer Spring Data repositories for simple CRUD; use QueryDSL for complex dynamic queries.

5. **API design:** All APIs should be documented via Swagger annotations. Follow RESTful conventions.

6. **Validation:** Use `@Valid` + Bean Validation annotations (`@NotNull`, `@Size`, etc.) on request DTOs.

7. **Error handling:** Implement `@ControllerAdvice` / `@RestControllerAdvice` for global exception handling.

8. **Redis:** Used for JWT blacklisting on logout and session management. Do not store sensitive data in Redis without TTL.

9. **Documentation:** Update this file when adding major features, changing architecture, or establishing new conventions. Key areas to document: business logic, API endpoints, concurrency controls, session expiry behavior.

10. **Language:** Code and comments may be in Korean or English. Prefer English for code identifiers; Korean is acceptable in comments and documentation.

---

## Site Map (External-facing pages)

| Page | Key Content |
|---|---|
| Home | Company USP, manufacturing strengths, key markets, CTA |
| Products | Category catalog, product detail, image gallery, quote CTA |
| How We Work | Trade procedure, MOQ, production process, lead times |
| Gold & Silver Rate | Live metal rates, exchange rates |
| Request Quote | Product selection, file upload, quantity input |
| Wholesale / Partner | Wholesale info, registration guide, approval process |
| About Us | Company history, export markets, quality control |
| Contact | Email, KakaoTalk/WhatsApp, map |

---

*Last updated: 2026-02-25*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Deco-Project-26-01) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
