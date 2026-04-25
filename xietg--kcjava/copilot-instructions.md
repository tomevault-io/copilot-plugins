## kcjava

> This file is the **single place** for product intent, repo facts, and **your expectations**. Keep it short and current; link to GitHub issues or docs for long specs.

# Claude.md — project ground truth

This file is the **single place** for product intent, repo facts, and **your expectations**. Keep it short and current; link to GitHub issues or docs for long specs.

**How assistants should use it:** treat this as authoritative for goals, workflow, and conventions; **verify implementation in code** when changing behavior. If the codebase lags this document, the doc describes **target product** — call out gaps in issues rather than silently shrinking scope.

---

## 1. Project scope

### 1.1 Product (target)

Web application to **collect vendor questionnaire responses**, **score** answers, and **rank** vendors within a **market segment**. Questionnaires are delivered as Excel workbooks that **always** use two fixed sheet names for machine-readable question definitions — see **§1.4**.

**Business capabilities (product backlog — not all may be implemented yet):**

1. Create a new market segment (Leadership Compass / **LC**).
2. Attach a questionnaire to a market segment.
3. Create a new questionnaire.
4. Import an existing questionnaire from an **XLSX** file.
5. Edit **technical** questions for a questionnaire.
6. Update **standard** questions for **new** questionnaires without changing historical questionnaires.
7. Download a questionnaire as **XLSX** for distribution to vendors.
8. Import a vendor-filled **XLSX** as a submission.
9. Define **scoring categories** per market segment.
10. Score each question **0–1** in **zero or more** categories.
11. Compute **overall** scores per category.
12. Build **charts** for overall results.

**Channels:** a **public REST API** and a **web frontend**.

**Users:** multiple roles are expected; **role model is not final yet** (evolve with issues, keep API extensible).

**Integrations (planned / future):** e.g. **Salesforce** (vendor master data), **enterprise IDP** (identity). Today the app uses **local JWT + DB users** unless/until IDP is wired.

**Non-functional:**

- **Robust** to invalid or malformed input files (clear errors, no silent corruption).
- **Scale:** on the order of **~20 concurrent users** for the MVP trajectory; architecture should not block **SaaS on Azure** later.
- **UX principle:** as **simple** and **reliable** as possible for operators and vendors.

### 1.2 Delivery governance

- **GitHub Issues** are the **backlog source of truth**: all substantive work should be **tracked**, **prioritized**, then executed.
- Prefer **one issue per deliverable** (or a clear parent/child breakdown) so history and scope stay auditable.

### 1.3 Current codebase vs product vision

- This repo is a **Spring Boot backend** (MVP port) with **Postgres**, **JWT auth**, **XLSX** via Apache POI, and **OpenAPI/Swagger**.
- **Frontend** exists under `frontend/` (see repo map). Features in §1.1 items **9–12** (scoring model, aggregates, charts) may be **partial or future** — implement per prioritized issues; do not assume they already exist.

### 1.4 Questionnaire XLSX — sheet names and layout (canonical)

**Fixed tab names (non-negotiable for parsers and template generation):**

- **Standard questions (shared across market segments):** sheet name **`4. Standard Questions`**
- **Technical questions (segment-specific):** sheet name **`5. Technical Questions`**

Code that imports or exports the **question definition** must target **only** these two sheets by **exact name**. Any other sheets in the same workbook are **out of scope** for question-definition I/O unless explicitly specified in a future issue.

**Typical full workbook shape:** a complete vendor-facing file may include **additional human-facing sheets** (e.g. introduction, process, market segment context, contact). Those sheets may contain **images and prose**; the backend must **not** rely on them for structured question data. Parsing logic should **ignore** everything outside the two canonical tabs above when reading or validating **question rows**.

**Row semantics (both tabs, aligned with `ExcelParser`):** rows mix **section headers (“categories”)** and **questions**. Implementations should follow the same rules as today’s parser:

- **Standard tab (`4. Standard Questions`):** a **category** row has **empty** column A, **category title** in column B, and **empty** C and D. A **question** row has **question index** in A (e.g. `1)`), **wording** in B, **type** in C, **help text** in D. **Sub-questions** reuse the parent index from A and use **empty** A with B/C/D filled; the parser synthesizes sub-indices (`a)`, `b)`, …).
- **Technical tab (`5. Technical Questions`):** a **category** row has **category title** in column A and **empty** B, C, D. **Question** and **sub-question** rows follow the same four-column pattern as on the standard tab (index / text / type / help).

**Question `type` values** are stored as strings and must stay consistent with the domain (see DB comments and existing data — e.g. text, numeric, percentage, boolean, comment, choices). Invalid or unknown types should surface as **clear parse or validation errors**, not silent coercion.

**Import order:** each stored question has a **`sheet_row_order`** (integer) set when definitions are imported from the canonical tabs. Template download (`GET /lcs/{id}/xlsx`) emits rows in that order so **categories and sections match the uploaded file**. Re-import or replacing questions requires an empty questionnaire (or a new questionnaire) so existing submissions are not invalidated.

**Confidential format reference (local only):** an example workbook that illustrates the real layout lives **outside this repository** (maintainer path such as `/home/gte/dev/tmp/example.xlsx`). It may contain **sensitive or proprietary content**.

- **Do not** commit it, **do not** copy it into the repo, **do not** attach it to GitHub issues or PRs, and **do not** paste **cell text or images** from it into **tracked files** (including this `Claude.md`).
- It is acceptable to **read it on a local machine** to understand structure (tab names, row patterns, column roles) and to implement or harden parsers — then describe behavior **abstractly** here and in code comments.

---

## 2. Repo map

| Area | Path | Notes |
|------|------|-------|
| Spring Boot API | `backend-spring/` | Main service; `mvn` project root for backend |
| Frontend | `frontend/` | Web UI (see its `package.json` / README if present) |
| Database schema & seeds | `infra/postgres/` | e.g. `001_schema.sql`, `002_seed.sql`, migrations |
| Local orchestration | `docker-compose.yml`, `Makefile` | Profiles: `dev`, `prod` |
| Env template | `.env.example` | Copy to `.env` (gitignored) |
| CI | `.github/workflows/backend-ci.yml` | Backend `mvn clean verify` on push/PR |
| Human-oriented quick start | `README.md` | Commands and security notes |

---

## 3. Commands

Run from **repository root** unless noted.

| Intent | Command |
|--------|---------|
| Backend compile (skip tests) | `cd backend-spring && mvn -q -DskipTests package` |
| Backend tests only | `cd backend-spring && mvn test` |
| **Same as CI (preferred before push)** | `cd backend-spring && mvn -B clean verify` |
| Backend locally with `.env` | `make run-backend-dev` |
| Infra only (Postgres, Metabase, …) | `make start PROFILE=dev` |
| Full stack in containers | `make start PROFILE=prod` |
| Stop / restart | `make stop PROFILE=dev`, `make restart PROFILE=dev` |
| Reset DB (destructive) | `make reset-db PROFILE=dev` |
| Clear local upload files | `make clean-uploads` |

---

## 4. Environment and secrets

- **Never commit** `.env`, credentials, JWT secrets, or customer data.
- **Never commit or leak** real questionnaire workbooks (e.g. local `example.xlsx` under `~/dev/tmp/`) or their **content** in issues, PRs, or docs — see **§1.4**.
- **Admin user:** `APP_ADMIN_EMAIL` and `APP_ADMIN_PASSWORD` are the **source of truth**; `StartupSeeder` **upserts** the admin user on startup (password hash refreshed from config).
- **Typical variables** (see `.env.example` and `README.md`): `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`, `APP_JWT_SECRET`, `APP_ADMIN_EMAIL`, `APP_ADMIN_PASSWORD`, `APP_CORS_ORIGINS`, `UPLOAD_DIR`, `APP_UPLOAD_CLEANUP_ON_STARTUP`.
- **Docker vs host:** container paths (e.g. `/data/uploads`) may differ from host; `make run-backend-dev` sets **`UPLOAD_DIR=./data/uploads`** for local Maven runs.
- **Production / Azure:** plan for **managed secrets** (e.g. Key Vault), **strong** non-default passwords, and **no** weak secrets — `SecretPolicyValidator` already fails fast outside `dev`/`test` when secrets are missing or weak.

---

## 5. API and security expectations

- **Errors:** single contract — **Problem Detail** JSON with top-level `type`, `title`, `status`, `detail`, `code`, `params` (`ErrorContract`, `GlobalExceptionHandler`). Do not introduce parallel error shapes.
- **Auth:** `Authorization: Bearer <access_token>` after `POST /auth/login`. **`POST /auth/register` does not return a token**; clients must log in to obtain a JWT.
- **Documentation:** keep **OpenAPI** annotations on controllers accurate when behavior changes (Swagger UI + `/v3/api-docs`).
- **Authorization (current backend)** — align with `README.md` **Authorization matrix**:
  - **Admin only:** `POST /lcs`, `POST /lcs/{lcId}/questionnaire`, `PUT /lcs/{lcId}/questionnaire`, `GET /health/admin`
  - **Authenticated (any role with a valid token):** `GET /lcs`, `GET /lcs/{lcId}/xlsx`, `GET /questionnaires`, `POST /lcs/{lcId}/submissions`, `GET /submissions`, `GET /question`
- **Future:** IDP integration and finer roles should **extend** this model; track breaking changes in issues.

---

## 6. Code and architecture preferences

- **Documentation (plain English):** code must be **clearly documented** so a **junior developer** can maintain it. Use **English** only for comments and docstrings. Prefer **short class- or method-level Javadoc** where behavior is not obvious from names alone (e.g. security rules, Excel row semantics, transaction boundaries). Explain **why** for non-trivial logic; skip noise on trivial getters/setters or self-explanatory one-liners. For `frontend/`, use concise comments or JSDoc where the API or state flow is non-obvious.
- **Layers:** **Controller → Service → Repository**; controllers stay thin (HTTP mapping, validation entry, status codes). Business rules live in services.
- **Persistence:** prefer explicit schema/migrations under `infra/postgres/`; keep JPA entities and DB types aligned (e.g. UUID FKs).
- **File ingestion:** validate **type, size, and structure** early; return **actionable** problem details — never log raw file contents or secrets.
- **Logging:** no **PII** or passwords in logs; avoid duplicate or ad-hoc exception handlers.
- **Stack:** **Java 21**, **Spring Boot 4.0.x** (see `backend-spring/pom.xml`).

---

## 7. Testing expectations

- **CI runs** `mvn -B clean verify` in `backend-spring` (unit + integration phases, JaCoCo as configured).
- **Before opening a PR:** run the same locally or at minimum `mvn test` after substantive changes.
- **Unit tests:** services, pure logic, security helpers — mock collaborators.
- **Slice tests:** `@WebMvcTest` for controller contracts with mocked services.
- **Integration tests (`*IT`):** Spring Boot + **Testcontainers Postgres** where full stack behavior matters; keep tests **deterministic** and **independent**.

---

## 8. Git workflow and commit style

- **Default branch:** `main`; **CI** runs on pushes and **pull requests** touching `backend-spring/` (see workflow).
- **Branches:** feature/fix branches + **PR to `main`** (match team branch protection).
- **Commits:** imperative subject line (`Add scoring endpoint`, not `Added…`); optional body for **why**; reference **`#issue`** when applicable.
- **PRs:** describe behavior change, link issue, note **migrations/env** updates, and confirm **`mvn clean verify`** (or CI green).

---

## 9. Definition of done

- [ ] `mvn -B clean verify` passes (or CI green).
- [ ] **OpenAPI** / `README.md` updated if API, auth, or env contract changed.
- [ ] **DB migrations** or `infra/postgres/` updates if schema changed.
- [ ] No secrets, tokens, or unnecessary PII in code or logs.
- [ ] **Documentation** updated per **§6** when adding or changing non-obvious behavior (Javadoc / comments in plain English for junior maintainers).
- [ ] Related **GitHub issue** updated (resolution + commit/PR reference) and **closed** when work is complete (per team process).

---

## 10. Links and contacts

- **Issue tracker:** this repository’s **GitHub Issues** (backlog source of truth).
- **Upstream / legacy context:** original Python backend referenced in `README.md` (`xieTG/kc`).
- **Design / product docs:** <!-- add URLs when available -->
- **Escalation:** <!-- team / channel -->

---

## 11. Appendix — quick reference (current repo)

- Swagger/OpenAPI: `/swagger-ui/**`, `/v3/api-docs/**`; README also mentions `http://localhost:8000/docs` where configured.
- JWT: HS256; access token TTL documented in `README.md` / config.
- XLSX: Apache POI for template and parsing paths; question-definition tabs are **`4. Standard Questions`** and **`5. Technical Questions`** (§1.4).
- `application-dev.yml`: may set `app.uploadCleanupOnStartup=true`; override with `APP_UPLOAD_CLEANUP_ON_STARTUP=false` if needed.
- Non-`dev`/`test` profiles: **`SecretPolicyValidator`** enforces non-weak secrets at startup.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/xieTG) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
