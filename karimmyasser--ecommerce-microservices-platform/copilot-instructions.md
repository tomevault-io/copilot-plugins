## ecommerce-microservices-platform

> > This file is loaded automatically at the start of every session. It is the

# CLAUDE.md — Project Guide & Rules for Agents

> This file is loaded automatically at the start of every session. It is the
> **single source of truth for how work is done in this repository**. Read it
> fully before writing any code. The detailed design lives under [`docs/`](docs/README.md).

---

## 1. What this project is

An **e-commerce backend** built as a **Spring Cloud microservices** system for the
Ejada Microservices course final project. It is composed of three business
services plus the cloud infrastructure that ties them together.

| Business service | Responsibility (one line) |
|------------------|---------------------------|
| **wallet-service**    | Users, authentication (JWT), wallet balance, deposits/withdrawals, transactions, order payments. |
| **inventory-service** | Product master data, categories, product variants (size/colour), stock levels. |
| **shop-service**      | Catalog browsing, cart, wishlist, orders (checkout saga), reviews & ratings. |

Cloud infrastructure: **config-server**, **eureka-server** (discovery),
**api-gateway**, **OpenFeign** (inter-service calls), **Resilience4j** (circuit breakers).

The endpoints were derived from two Figma designs (a clothes shop "Modeva" and a
shoes shop "StepUp"); the backend is a **union** that satisfies both. See
[`docs/figma-analysis.md`](docs/figma-analysis.md).

> **Important:** the design intentionally **deviates from `docs/project-overview.md`**
> (the course brief) where the brief describes a naive architecture. The brief is
> input, not gospel; the Figma is the real requirement. Read
> [`docs/architecture/00-design-decisions.md`](docs/architecture/00-design-decisions.md)
> before changing structural decisions, and keep that register updated if you
> make new ones.

---

## 2. The rules (non-negotiable)

These are hard constraints. Do not violate them even if a task seems to ask for it.

### 2.1 Secrets & credentials
- **Never commit credentials, passwords, API keys, JWT secrets, or real DB
  connection strings.** Not in code, not in `application.yml`, not in docs, not
  in commit messages.
- All secrets are supplied at runtime via **environment variables** (or a local,
  git-ignored `application-local.yml`). Config files commit only **placeholders**,
  e.g. `${DB_PASSWORD}`, `${JWT_SECRET}`.
- `.env`, `*-local.*`, `*.local` and secret files are git-ignored. Verify before
  every commit that no secret is staged (`git diff --cached`).

### 2.2 Authorship & attribution
- **No AI/agent attribution anywhere in the repository.** Do **not** add
  `Co-Authored-By` trailers, "Generated with ..." footers, or any agent/model
  name to commits, PRs, code comments, or docs.
- Commit as the repository's configured git user only. Do not change the git
  author identity.
- Write in a neutral, human voice. Never sign work.

### 2.3 Git hygiene
- Never force-push a shared branch. Never rewrite published history.
- Never commit `target/`, build output, IDE folders, or the Maven wrapper jar.
- One logical change per commit; use the message convention in
  [`docs/conventions/git-and-collaboration.md`](docs/conventions/git-and-collaboration.md).
- Do not commit directly to `main` for feature work — branch first.

### 2.4 Scope discipline
- Follow the phased plan in [`docs/implementation-plan/`](docs/implementation-plan/README.md).
  Complete and verify a phase before starting the next.
- Do not introduce new frameworks/dependencies without a note in the relevant
  doc explaining why. Prefer what is already in `pom.xml`.

---

## 3. How to pick up work (for the next agent)

1. Read this file, then [`docs/README.md`](docs/README.md).
2. Open [`docs/implementation-plan/README.md`](docs/implementation-plan/README.md)
   and find the first phase whose checklist is not complete — that is your
   starting point.
3. Read the domain + API doc for the service you are about to build.
4. Implement, test, and tick the checklist boxes in that phase file.

The repo is currently at **Phase 0 (planning complete, scaffold only)**. No
business code has been written yet — only the Spring Initializr scaffold and
these docs exist.

---

## 4. Tech stack (fixed)

- Java 17, Spring Boot 4.1.x, Spring Cloud 2025.1.x, Maven (multi-module monorepo).
- Spring Web MVC, Spring Data JPA, Spring Security, Bean Validation, Lombok.
- MySQL (one schema per service). OpenFeign, Eureka, Config Server, Gateway, Resilience4j.

Full details and versions: [`docs/architecture/01-system-overview.md`](docs/architecture/01-system-overview.md).

---

## 5. Build & run (once services exist)

```bash
# Build everything from the repo root
./mvnw clean install

# Start order matters: config → eureka → business services → gateway
./mvnw -pl config-server spring-boot:run
./mvnw -pl eureka-server spring-boot:run
./mvnw -pl inventory-service spring-boot:run
./mvnw -pl wallet-service   spring-boot:run
./mvnw -pl shop-service     spring-boot:run
./mvnw -pl api-gateway      spring-boot:run
```

Ports and startup order are documented in
[`docs/architecture/02-service-topology.md`](docs/architecture/02-service-topology.md).

---

## 6. Coding conventions (summary)

Controllers thin, services hold logic, repositories are Spring Data interfaces.
DTOs at the boundary (never expose JPA entities in responses). Constructor
injection. Centralised exception handling via `@RestControllerAdvice`. Full
details: [`docs/conventions/coding-standards.md`](docs/conventions/coding-standards.md).

---

## 7. Documentation map

Everything is under [`docs/`](docs/README.md) and split into small, independent
files so each concern can be read on its own. Start at
[`docs/README.md`](docs/README.md).

---
> Source: [KarimmYasser/ecommerce-microservices-platform](https://github.com/KarimmYasser/ecommerce-microservices-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-04 -->
