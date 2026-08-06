## quantumed-hms-main

> > **Canonical master documents** (load before any work):

# Agent Guidance for QuantuMed HMS

> **Canonical master documents** (load before any work):
>
> - [`docs/agents/WORKSPACE_RULES.md`](docs/agents/WORKSPACE_RULES.md) — **Part 1:** canonical stack, monorepo layout, tenancy, security, RBAC, API contract, frontend routes, i18n, data model, adapter matrix, scope boundary.
> - [`docs/agents/AGENTIC_WORKFLOW.md`](docs/agents/AGENTIC_WORKFLOW.md) — **Part 2:** agent roster, cross-agent protocols, CI/CD pipeline overview, phased (A–E) task checklists, acceptance criteria, timeline.
> - [`docs/agents/Agents.md`](docs/agents/Agents.md) — **Narrative supplement:** system context, deployment & multi-hospital plan, non-functional requirements, deliverables per sprint.
>
> **Single source of truth for the spec:** [`docs/action-plan.md`](docs/action-plan.md) — do not mutate via PRs.
> **ADRs** (committed architecture decisions): [`docs/architecture/adr/`](docs/architecture/adr/)

---

## 1. Project Goal

**QuantuMed Hospital Management System (QuantuMed HMS)** is a robust, multi-tenant web SaaS platform that supports single and multiple hospitals (multi-hospital installations), telemedicine/teleradiology, multi-language UI, full financials, pharmacy and inventory, role-based access, SMS/email automation, POS and invoicing, and third-party integrations (Stripe, local mobile-money, SMS gateways, Jitsi). The product must be production-ready, secure (HIPAA/GDPR-aware where applicable), maintainable, observable, interoperable, and internationalized (Arabic, Amharic, Oromo, Somali, Tigrigna, English).

It consolidates the **operational, clinical, financial, and collaborative** needs of hospitals, clinics, diagnostic centers, and nursing homes into a single unified SaaS platform. AI/ML capabilities are embedded from day one via an adapter boundary, not bolted on later:

- **Rule-based patient triage** (MVP) — deterministic scoring using vitals and presenting complaints.
- **Clinical Decision Support (CDS)** (limited MVP surface) — drug-interaction, allergy, dose-range, and duplicate-therapy advisories surfaced at order entry.
- **Automated Clinical QI/QA** (limited scope, MVP→Beta) — structured quality measure extraction from encounters (e.g. diabetes HbA1c adherence, hypertension control), with explainable rule attribution only — no black-box models in MVP.

### Deployment Targets

| Mode                          | Tooling                            | Use Case                  |
| ----------------------------- | ---------------------------------- | ------------------------- |
| Single-hospital quick install | `infra/docker/docker-compose.yml`  | Local / single facility   |
| Production multi-hospital     | Terraform + Kubernetes Helm charts | Multi-facility production |

Stateless app servers behind a load balancer; state in managed PostgreSQL and object storage. Availability target: < 3% downtime during business hours for MVP. Observability: OpenTelemetry traces, Prometheus metrics, structured Pino JSON logs, health endpoints (`/health/live`, `/health/ready`).

---

## 2. Source of Truth

- [`docs/action-plan.md`](docs/action-plan.md) is the canonical spec. Do not deviate without explicit user approval — propose via PR, do not merge unilaterally. **The action plan is never mutated by PRs.**
- ADRs in [`docs/architecture/adr/`](docs/architecture/adr/) record committed architectural decisions. New decisions get a new ADR; accepted ADRs are **never edited in place**.
- `docs/agents/WORKSPACE_RULES.md` is the authoritative coding rule set derived from the action plan.
- `docs/agents/AGENTIC_WORKFLOW.md` is the authoritative phased workflow and agent task reference.
- When this `AGENTS.md` extends scope beyond the action plan (mobile-money adapters, local SMS providers, extended roles, dynamic domain experts), those extensions are recorded in a tracking ADR and reflected back into Part 1 / Part 2 — the three docs are kept in sync (see §15 Doc Hygiene).

---

## 3. Multi-Agent Orchestration

A core 12-agent team collaborates under the **Orchestrator**, augmented by **dynamic domain-expert agents** spun up on demand (see §3.2).

### 3.1 Core Agent Roster

| #  | Agent | Owns |
| -- | ----- | ---- |
| 1  | **Orchestrator** (lead) | Roadmap, phase gates, release cadence, sprint backlogs, final integration checks, deploy checklists. **Only agent that approves phase promotion.** |
| 2  | **Solution Architect** | All ADRs, data partitioning/backup/scaling, infra topology (Terraform/Helm/K8s), tenancy + encryption key lifecycle. First deliverables: ADR-0001/0002/0003, approved before Phase A work begins. |
| 3  | **Backend Engineer** | All NestJS bounded-context modules, REST endpoints, DTO validation, JWT+refresh+RBAC+MFA, OpenAPI 3.1 spec, backend unit tests. |
| 4  | **Frontend Engineer** | All Next.js role dashboards, appointment calendar popup, prescription combobox, language switcher + RTL, per-tenant branding, PWA, WebSocket real-time, frontend unit tests. |
| 5  | **Database Engineer** | Prisma multi-schema definition, all migrations + indexes, field-level encryption integration, retention rules, catalog/permission/demo seeds. |
| 6  | **DevOps & Infrastructure** | All IaC (`infra/docker/`, `infra/helm/`, `infra/terraform/`, `infra/k8s/`), CI/CD pipelines, secrets management, HPA/autoscaling, observability wiring. |
| 7  | **Security & Compliance** | Threat modelling, OWASP hardening, HIPAA/GDPR checklists, encryption key lifecycle, endpoint permission review, incident response runbook. |
| 8  | **QA & Test Automation** | Full test strategy: unit, integration, contract, E2E (Playwright), performance (k6), security scans; nightly pipeline; test reports. |
| 9  | **Localization & Content** | All `packages/i18n/messages/`, `en.json` as source of truth, RTL validation for `ar`, translation pipeline, locale-aware formatting. |
| 10 | **Integration** (Payments / Telemedicine / SMS & Email) | Stripe + mobile-money adapters, Jitsi adapter, SMS (Twilio + local: Ethio telecom, Safaricom) + email adapters, notification triggers, BullMQ processors. |
| 11 | **Data & Reporting** | Reporting module (P&L, AR aging, commissions), CSV/PDF export, daily reconciliation job, audit trail export. |
| 12 | **Documentation & Handover** | Continuously updates `README.md` and `docs/PROJECT_STATUS.md`, OpenAPI export, all guides + runbooks, architecture diagrams. |

### 3.2 Dynamic Domain-Expert Agents (per-need)

The core roster is supplemented by **specialized domain agents** that the Orchestrator spins up when a phase needs deep clinical/EMR expertise. They own a bounded deliverable, contribute under the same protocols, and wind down when their scope ships.

| Domain Expert | Spun up for | Typical Deliverables |
| ------------- | ----------- | -------------------- |
| **EMR/Clinical Informatics Expert** | Encounter schema, problem list, clinical note templates, ICD-10/CPT coding validation, allergy/medication reconciliation | Encounter domain spec, note-template library, coding validation rules, CDS rule catalog |
| **Pharmacy/Pharmacology Expert** | Drug-interaction rules, dose-range checks, controlled-substance workflows, medicine catalog taxonomy | Drug-interaction rule set, dose-range tables, controlled-substance state maps, catalog seed review |
| **Laboratory / Pathology Expert** | Test catalog, reference ranges, critical-value escalation, sample tracking, anatomic pathology | Reference-range library per panel, critical-value thresholds + escalation matrix, barcode/chain-of-custody rules |
| **Radiology / Imaging Expert** | DICOM conformance, study lifecycle, reporting templates (ACR), radiation dose tracking, modality worklists | DICOM conformance statement inputs, study lifecycle state machine, report templates, MWL model |
| **Interoperability Expert** | FHIR R4 profiling, HL7 v2 message mapping, XDS/XCA, terminology services (SNOMED CT, LOINC, RxNorm) | FHIR IG, HL7 v2 message specs, terminology bindings, conformance test bundles |
| **Clinical AI / ML Expert** | Triage scoring, CDS rule engine, QI/QA measure derivation, model governance + explainability | Triage scoring spec, CDS rule library, QI measure logic, model-card + bias-audit templates |
| **Financial / Revenue Cycle Expert** | Insurance claims, EDI 837/835, charge capture, denial management, AR workflows | Claims data model, EDI mapping, denial-reason taxonomy, AR aging logic |

**Spin-up protocol:** the Orchestrator creates a focused task brief (scope, entry phase, exit criteria, owning core agent). The expert works on a feature branch, is peer-reviewed by the relevant core agent (e.g. EMR work → Backend Engineer; CDS rules → Clinical AI/ML Expert + Pharmacy/Pharmacology Expert), and the expert's rules/catalogs are merged into the relevant module under version control. Domain experts do **not** approve phase promotion — that authority stays with the Orchestrator.

### 3.3 Cross-Agent Collaboration Protocols

1. **ADR-first:** Solution Architect publishes ADRs; Orchestrator reviews before any implementation. New decisions get new ADRs — never edit accepted ADRs.
2. **OpenAPI-first contract:** Backend publishes the OpenAPI spec before Frontend builds API-dependent UI. Frontend builds against the spec, not ad-hoc.
3. **Shared-types-first:** Database + Backend agree on Zod schemas in `packages/shared-types` before building module services; Frontend consumes the same schemas.
4. **Incremental demos:** Every phase produces independently demoable functionality; Orchestrator runs a demo checkpoint before promotion.
5. **Peer PR review:** Every PR reviewed by at least one relevant peer (e.g. Security reviews new endpoints; Database reviews schema changes; Domain Expert reviews clinical rules).
6. **Daily artifact posts:** Each agent posts incremental artifacts to the shared project tracker.

### 3.4 Per-Task Output Format

Every agent, per task, produces:
- Short summary of decisions (1–3 bullets)
- Ordered implementation steps
- Files/artifacts produced (repo paths)
- Tests added and how to run them
- Acceptance checklist with pass/fail evidence

### 3.5 Multi-Agent Kickoff Instruction

Start by having the **Solution Architect Agent** produce the foundational Architecture Decision Records (ADR-0001 architecture, ADR-0002 tenancy model, ADR-0003 encryption key lifecycle). After the ADRs are approved by the **Orchestrator**, spin up the infrastructure sandbox (**DevOps Agent**): ephemeral staging environment with database, object storage, and CI pipeline skeleton. Parallelize **Backend + Frontend** work off the OpenAPI-first contract. Enforce tests and documentation as part of every PR. Produce incremental demoable features. Spin up domain experts just-in-time when a phase needs deep clinical/EMR/interoperability/AI expertise.

---

## 4. Canonical Technology Stack

> **Rule:** Do not deviate from this stack without a new ADR approved by the Orchestrator. Full version table: [`WORKSPACE_RULES.md`](docs/agents/WORKSPACE_RULES.md) §2.

| Layer                | Technology                             |
| -------------------- | -------------------------------------- |
| Frontend             | Next.js 15, React 19, Tailwind 3.4, Shadcn/Radix, TanStack Table, Zod + RHF, next-intl 3, `@jitsi/react-sdk` |
| Backend              | NestJS 10, Node 20 LTS, REST-only under `/api/v1/` |
| Data                 | PostgreSQL 16, Prisma 5, Redis 7, BullMQ 5 |
| AI/ML                | Rule engine (MVP) → `ClinicalAiPort` adapter (CDS, QI/QA) — **no black-box models in MVP** |
| Interoperability     | `FhirPort` + `DicomPort` + `Hl7v2Port` adapters; FHIR R4 + DICOM gateway land in Beta/v1 |
| Observability        | OpenTelemetry + Prometheus + Pino      |
| Monorepo / Quality   | pnpm 9, Turborepo, ESLint 9 strict, Prettier 3, Husky + lint-staged, Conventional Commits |
| Testing              | Jest (backend), Vitest (frontend), Testcontainers (integration), Pact (contract), Playwright (E2E), k6 (perf), Semgrep/Trivy (security) |

---

## 5. Phased Delivery

### 5.1 Phase Discipline

- A PR belongs to **exactly one phase (A / B / C / D / E)**. Cross-phase bundling is prohibited.
- **Phase A is foundation only:** monorepo scaffolds, cross-cutting concerns (encryption, audit, RBAC, tenant context), identity / tenancy / multi-hospital modules, i18n stubs, demo seed, docker-compose, CI pipeline, and **adapter ports** for AI/interoperability/payments/SMS (interfaces only — no Beta/v1 integrations).
- **Every PR must update both [`README.md`](README.md) (phase-status table) and [`docs/PROJECT_STATUS.md`](docs/PROJECT_STATUS.md)** as part of the same PR. These two files are how a fresh reader (human or agent) figures out what's been built; out of date is worse than missing.
- When a phase moves to `shipped`, link the PR number on both files. If the PR was end-to-end tested, link the test report from `docs/testing/` alongside it in `PROJECT_STATUS.md`.
- Use feature branches → PRs → automated CI → QA acceptance → merge.

Phase promotion lifecycle: `Not Started → In Progress → In Review → Shipped`.

### 5.2 Authoritative Phase Timeline

| Phase | Focus                                                               | Target Duration |
| ----- | ------------------------------------------------------------------- | --------------- |
| A     | Foundation: monorepo, cross-cutting concerns, identity, tenancy, CI, all adapter ports | 1–2 weeks       |
| B     | Core logic: all bounded-context modules, mobile-money + local SMS adapters, BullMQ worker, CDS + triage rule engine | 3–4 weeks       |
| C     | Interfaces: all role surfaces incl. extended roles, PWA, RTL, real-time | 2–3 weeks       |
| D     | QA: unit, integration, contract, E2E, performance, security scans, CDS/QI rule validation | 1–2 weeks       |
| E     | Production readiness: docs, Helm, Terraform, CD, observability      | 1–2 weeks       |

**Total MVP target:** 8–13 weeks. Per-phase task checklists and acceptance criteria: [`AGENTIC_WORKFLOW.md`](docs/agents/AGENTIC_WORKFLOW.md) §4.

### 5.3 Coarse Roadmap (MVP → Beta → v1)

The phase timeline above is authoritative for *delivery sequencing*; the view below is a coarse *product* roadmap layer for high-level planning:

- **MVP (Phase A–E):** Core users, appointments, prescription, pharmacy POS, payments (Stripe + local mobile-money adapters), local SMS providers, single-hospital install, basic translations, telehealth link, triage + CDS rule engine, all adapter ports, CI/CD, staging.
- **Beta:** Multi-hospital onboarding automation, full Stripe flows, **FHIR R4 API + DICOM gateway/PACS integration** (concrete providers behind the MVP ports), extended CDS library, deeper financial reports, RBAC fine-tuning + custom per-tenant roles, performance tuning.
- **v1:** High-availability infrastructure, compliance hardening, expanded translations, advanced clinical AI (governed, explainable), extended FHIR IG profiling.

> **Scope rule:** Concrete DICOM/PACS, HL7 v2/FHIR, and advanced clinical-AI integrations ship in **Beta/v1 behind the adapter ports** defined in MVP. The ports + pluggable provider registry exist from Phase A so Beta is integration-only work, not architectural rework.

---

## 6. Mandatory Checks Before Every Commit

```bash
pnpm format:check   # Prettier — enforced by Husky + lint-staged on staged files
pnpm lint           # ESLint strict (@typescript-eslint/strict)
pnpm typecheck      # tsc --noEmit across all packages
pnpm test:unit      # Jest (backend) + Vitest (frontend)
pnpm build          # Full Turborepo monorepo build
```

The CI workflow at `.github/workflows/ci.yml` runs the same set. Husky + lint-staged run `prettier --write` on staged files via `pnpm prepare`.

### CI/CD Pipeline Overview

- **PR pipeline (`.github/workflows/ci.yml`):** Lint → TypeCheck → Unit Tests → Integration Tests → Build → Container Scan (Trivy) → SAST (Semgrep) → PR comment with test summary.
- **Main / CD (`.github/workflows/cd.yml`):** All PR steps → Container Build & Push → DB Migration dry-run → Deploy to Staging → E2E Smoke → Manual Approval Gate → DB Migration (with rollback plan) → Canary to Production → Health Check → Promote or Rollback.
- **Nightly (`.github/workflows/nightly.yml`):** Full E2E suite → k6 performance tests → Security scans (ZAP baseline) → Dependency drift report.

---

## 7. Tenancy Invariants

- **Never query a clinical entity without a tenant context.** The `RequireTenantGuard` and `TenantMiddleware` (in `apps/api/src/common/tenant`) enforce this; do not add `@SkipTenant()` to non-platform endpoints.
- Tenancy model is **decided (hybrid):** default schema-per-tenant (`tenant_<slug>`); premium tier DB-per-tenant; the `platform` schema holds only cross-tenant entities (`hospitals`, `users`, `refresh_tokens`, `audit_log`, `packages`, `subscriptions`). The `isolation_mode` column drives provisioning automatically.
- Tenant context propagates through every async hop via `nestjs-cls` AsyncLocalStorage — no service method accepts `tenantId` as a parameter.
- Email uniqueness is scoped per `(hospital_id, email)`, not globally.
- Per-tenant migrations are applied by `tenant-provisioning.service.ts` at onboarding — never run manually.
- Per-tenant configuration drives **locality** of integrations (e.g. which mobile-money providers are enabled, which SMS gateway, default currency, FHIR endpoint base) — see §10.

---

## 8. Security Invariants

- PHI fields are marked with `@Phi()` from `common/encryption/phi.decorator.ts` and stored encrypted via `FieldEncryptionService` (AES-256-GCM, per-tenant DEK, KEK in AWS KMS).
- Every state-changing operation must write an `AuditLogService.write(...)` entry. Audit writes are the one place where the hash chain is computed — **do not insert directly into `audit_log`**. Audit entries are append-only (no UPDATE/DELETE).
- All endpoints are protected by `AuthGuard` and `RbacGuard` by default. Use `@Public()` and `@RequirePermission(resource, action)` deliberately.
- MFA is mandatory for `Admin`, `SuperAdmin`, `Doctor`, `Pharmacist`, `LabManager`, `Accountant`, `Radiologist`, `Radiographer`, `ImagingTechnologist`, `DepartmentHead`, `QADirector` — any role that accesses PHI.
- JWT access tokens expire in 15 minutes. Refresh tokens rotate on every use.
- Password hashing: Argon2id. No bcrypt for new code.
- Audit log payloads with PHI are **redacted at write time** — the audit log must not itself become a PHI store.
- Helmet middleware applied globally (CSP, X-Frame-Options, Referrer-Policy, Permissions-Policy, HSTS).
- Rate limiting via `@nestjs/throttler`: global 100 req/min/IP; `/auth/login` 5/min/email; `/notifications/bulk` 1/min/tenant; payment-webhook endpoints idempotent and ACK-first.
- **Clinical AI safety:** all CDS/QI advisories are advisory-only with explainable rule attribution and source citation; the human clinician's order stands. No autonomous clinical decisions in MVP. Model governance (model cards, bias audit, change-control) is mandatory before any non-rule-based model is enabled (Beta+).

---

## 8.1 AI/ML Adapter Boundary

> Fulfills the "AI/ML from day one" product goal while keeping the MVP scope to rule-based + explainable methods only.

| Concern | Interface | MVP Scope | Beta/v1 |
| ------- | --------- | --------- | ------- |
| **Triage scoring** | `TriagePort` (rule engine) | Deterministic ESI-style scoring from vitals + complaints; advisory nurse-facing output | Tunable scoring; regional preset packs |
| **Clinical Decision Support** | `CdsPort` | Drug-interaction, allergy, dose-range, duplicate-therapy rules surfaced at order entry (advisory) | Expandable CDS Hooks-style rule library |
| **Clinical QI/QA** | `QualityMeasurePort` | Rule-based measure extraction from encounters (diabetes HbA1c adherence, hypertension control, etc.) with measure definitions versioned in repo | Trending, benchmarking, automated measure updates |
| **External AI models** | `ClinicalAiPort` | **Port only — no provider wired** | Governed, explainable models via adapter (requires bias audit + model card) |

**Rules of engagement:**
- All AI advisories write to the audit log with the rule/model id, input snapshot (PHI-redacted), and recommendation.
- CDS/QI logic lives in versioned, peer-reviewed rule files (co-owned by Backend + Clinical AI/ML Expert) — not inline service code.
- A `cds.advisory` WebSocket event pushes advisories to the ordering clinician's UI in real time.
- CDS overrides are append-only: clinician can dismiss an advisory, but the override + reason is recorded and surfaced to the `QADirector` for quality review.

---

## 8.2 Interoperability Adapter Boundary

> Fulfills the "interoperability including DICOM gateway + PACS, HL7 v2/FHIR" goal. MVP defines the ports; concrete integration ships in Beta/v1.

| Concern | Interface | MVP Scope | Beta/v1 |
| ------- | --------- | --------- | ------- |
| **FHIR R4 API** | `FhirPort` | Port + provider registry; FHIR resource ↔ internal model mapping design (ADR) | Full FHIR R4 RESTful server, IG profiling, `$everything`, SMART-on-FHIR |
| **HL7 v2 messaging** | `Hl7v2Port` | Port + message-type catalog design (ADT, ORM, ORU) | MLLP listeners, message routing, ACK handling |
| **DICOM gateway / PACS** | `DicomPort` | Port + study lifecycle state machine design | DICOMweb (QIDO/WADO/STOW), C-FIND/C-MOVE against PACS, modality worklist |
| **Terminology services** | `TerminologyPort` | Binding to ICD-10, CPT/HCPCS (already required) | SNOMED CT, LOINC, RxNorm via terminology service adapter |

**Rules of engagement:**
- Internal domain models are the source of truth; FHIR/HL7 are *projections* via mappers (never the persistence model).
- All interoperability adapters are tenant-scoped and audit-logged.
- A conformance/capability statement is auto-generated from the active adapter set.

---

## 9. API Contract Invariants

- **OpenAPI-first.** `@nestjs/swagger` decorators are the single contract; the spec is exported to `docs/api/openapi.yaml`. Frontend agents build against the spec, not ad-hoc endpoint knowledge.
- All routes versioned under `/api/v1/`. Swagger UI served at `/api/docs` in non-production environments. The API surface is **REST only** (no GraphQL in MVP).
- All responses use the standard envelope:
  ```
  Success: { data: <payload>, meta?: { pagination } }
  Error:   { error: { code, message, details? }, request_id }
  ```
- **Cursor-based pagination only** on `(created_at, id)` composite indexes. Never offset pagination on unbounded tables.
- Stripe / mobile-money webhooks validate provider-specific signature/secret headers (not Bearer). Write payload to BullMQ immediately, ACK the HTTP request, process asynchronously with idempotency key.
- All request bodies validated via Zod schemas in `packages/shared-types` — shared between `apps/api` and `apps/web`.

> Critical API routes are enumerated verbatim in [`WORKSPACE_RULES.md`](docs/agents/WORKSPACE_RULES.md) §7.1.

---

## 10. Adapter Pattern Invariants (Payments, SMS, Email, AI, Interop)

Code must **never** couple directly to a single external provider. Use the port interfaces. The active adapter per tenant is resolved at runtime by a **provider registry** driven by tenant config (so one tenant can enable Telebirr + Ethio telecom while another enables Stripe + Twilio).

### 10.1 Payments

| Integration | Interface | MVP Adapters | Beta/v1 |
| ----------- | --------- | ------------ | ------- |
| Card / international | `PaymentGatewayPort` | **Stripe** (default) | Stripe recurring, Connect |
| Mobile money — Ethiopia | `MobileMoneyPort` | **CBE birr**, **Telebirr**, **e-Birr** | Direct merchant onboarding flows |
| Mobile money — regional | `MobileMoneyPort` | **M-Pesa** (Kenya/Tanzania), **Apollo** | Additional regional providers |

**Payment invariants:**
- Every provider implements `PaymentGatewayPort` (card) or `MobileMoneyPort` (USSD/STK push/checkout) with a shared `PaymentIntent` → `PaymentResult` shape.
- Idempotency key on every payment mutation; webhook handlers are ACK-first, BullMQ-processed.
- Refund amount cannot exceed original payment minus prior refunds (service-layer enforced).
- Reconciliation: each provider has a daily reconciliation BullMQ job writing to a normalized ledger; mismatches raise a `reconciliation.exception` alert.
- Provider enablement + credentials are per-tenant config, secrets in AWS Secrets Manager / Vault — never in code or DB plaintext.

### 10.2 SMS

| Integration | Interface | MVP Adapters | Beta/v1 |
| ----------- | --------- | ------------ | ------- |
| International SMS | `SmsPort` | **Twilio** (default) | — |
| Local SMS — Ethiopia | `SmsPort` | **Ethio telecom** SMS gateway | Bulk/gateway tuning |
| Local SMS — Kenya/EA | `SmsPort` | **Safaricom** (bulk SMS API) | Additional regional carriers |

**SMS invariants:**
- All providers share a `send(to, template, vars)` contract with delivery-receipt webhooks.
- Opt-out / consent logging is mandatory; the notifications module suppresses sends to opted-out recipients.
- Rate limiting: `/notifications/bulk` 1/min/tenant; per-provider throughput throttling via BullMQ.
- Templates use ICU MessageFormat placeholders; locale resolved from recipient preference → tenant default → `en`.

### 10.3 Email, AI, Interop, KMS

| Integration | Interface | Default / MVP Adapter |
| ----------- | --------- | --------------------- |
| Email | `EmailPort` | Nodemailer SMTP (dev) / SES (prod) |
| KMS | `KmsPort` | AWS KMS |
| Triage | `TriagePort` | Rule engine (built-in) |
| CDS | `CdsPort` | Rule library (built-in) |
| QI/QA | `QualityMeasurePort` | Rule library (built-in) |
| External clinical AI | `ClinicalAiPort` | Port only — no MVP provider |
| FHIR | `FhirPort` | Port only — Beta/v1 provider |
| HL7 v2 | `Hl7v2Port` | Port only — Beta/v1 provider |
| DICOM / PACS | `DicomPort` | Port only — Beta/v1 provider |
| Terminology | `TerminologyPort` | ICD-10 + CPT/HCPCS (built-in); SNOMED/LOINC/RxNorm Beta+ |

---

## 11. RBAC & Roles

### 11.1 Base Roles (9)

`SuperAdmin`, `Admin`, `Doctor`, `Nurse`, `Receptionist`, `Accountant`, `Pharmacist`, `Laboratorist`, `Patient`

### 11.2 Extended Roles (introduced by feature modules)

`HRAdmin`, `LabTechnician`, `Pathologist`, `LabManager`, `Radiologist`, `Radiographer`, `ImagingTechnologist`, `PharmacyTechnician`, `ReferralCoordinator`, `DonorCoordinator`, `TelemedicineProvider`

### 11.3 New Extended Roles (this expansion)

- **`Radiographer`** — performs imaging studies, owns modality worklist, dose tracking input.
- **`ImagingTechnologist`** (already in §11.2) — distinct from Radiographer where a facility separates MR/CT/nuclear technologists.
- **`DepartmentHead`** — clinical/ops department lead; read across the department, approve schedules/leave, sign-off on department-level reports.
- **`QADirector`** — owns clinical quality measures, reviews CDS overrides, approves QI/QA report distribution, audit-trail access for quality review.

### 11.4 Per-Tenant Role & Permission Customization

- Tenants may **clone a base/extended role** and adjust permissions within an allowed envelope — they cannot escalate to platform-level or SuperAdmin-equivalent privileges.
- Custom roles are stored tenant-scoped (in the `tenant_<slug>` schema) and resolved by `RbacGuard` identically to built-in roles.
- A role-editor surface (Admin-only) lets a tenant add users and define custom roles; all changes are audit-logged.
- The permission matrix is the source of truth in `common/rbac/permissions.matrix.ts`; custom roles are deltas against it, not overrides.

> Role → primary route prefix mapping and the full permission matrix: [`WORKSPACE_RULES.md`](docs/agents/WORKSPACE_RULES.md) §6.

---

## 12. Translations

- `packages/i18n/messages/en.json` is the source of truth. When adding keys, add them in `en.json` first.
- Non-English files (`ar`, `am`, `om`, `so`, `ti`) carry `__meta.status = "needs_professional_review"` until a medical-grade translation pass lands.
- **Never delete a key** — add a new key and migrate consumers.
- Arabic (`ar`) requires `dir="rtl"` on the `<html>` element. The `packages/i18n/src/dir.ts` resolver handles this automatically.
- Use `next-intl` with ICU MessageFormat for pluralization and gender; fall back to English for any missing key. Date, time, currency, and number formatting must be locale-aware.

---

## 13. Data Model Invariants

- `Appointment.scheduled_at` conflicts prevented by partial unique index on `(doctor_id, scheduled_at)` where `status != 'CANCELLED'`.
- `Prescription.signed_at` must be set before the prescription can transition to `DISPENSED`.
- `ConsentRecord` status transitions are append-only — revocation creates a new row, never mutates the prior row.
- Refund amount cannot exceed original payment minus prior refunds (enforced at service layer, not just DB).
- ICD-10 and CPT/HCPCS codes are required fields on encounters and bills.
- MRN is auto-generated per-tenant with a configurable prefix; PHI columns are NOT NULL where business logic requires.
- **CDS overrides** are append-only: clinician can dismiss an advisory, but the override + reason is recorded and surfaced to the QADirector.
- **Quality measures** are derived (read-model) from encounters, never mutated directly; measure definitions are versioned.
- **Custom roles** are stored tenant-scoped as deltas against the canonical permission matrix.

---

## 14. Performance Requirements

- Patient lookup: p95 < **2 seconds**.
- Order entry (with CDS evaluation): p95 < **3 seconds**.
- All list endpoints use server-side pagination (TanStack Table on the frontend).
- Redis caching for hot read paths: doctor availability calendars, medicine catalog, RBAC permission matrices, **active payment/SMS provider config**.
- BullMQ for all async processing: notifications, payment webhooks, billing reconciliation, lab critical alerts, report generation, QI measure recalculation.
- DICOM/FHIR endpoints (Beta+) have their own latency budgets defined in their ADRs.

---

## 15. Repository Status Hygiene & Doc Hygiene

- **Every PR must update both `README.md` (phase-status table) and `docs/PROJECT_STATUS.md`** as part of the same PR.
- When a phase moves to `shipped`, link the PR number on both files. Link the test report from `docs/testing/` in `PROJECT_STATUS.md`.
- The action plan (`docs/action-plan.md`) is the spec and is **never mutated by PRs** — only `README.md` and `docs/PROJECT_STATUS.md` move forward with each merge.
- **Three-doc sync:** `AGENTS.md`, `WORKSPACE_RULES.md`, and `AGENTIC_WORKFLOW.md` must stay mutually consistent. A PR that changes scope, adapters, roles, or the agent roster must update **all three** in the same PR. The Orchestrator blocks PRs that leave the docs out of sync.
- New scope beyond the action plan (mobile-money adapters, local SMS providers, extended roles, dynamic domain experts, AI/interop ports) is recorded in a tracking ADR and back-synced into the masters within the same PR.

---

## 16. Demo Seed (Required)

`scripts/seed-demo-tenant.sh` must seed all of the following for the demo hospital tenant:

| Email                     | Password  |
| ------------------------- | --------- |
| `doctor@demo.com`         | `demo123` |
| `patient@demo.com`        | `demo123` |
| `hr@demo.com`             | `demo123` |
| `lab@demo.com`            | `demo123` |
| `pharmacy@demo.com`       | `demo123` |
| `imaging@demo.com`        | `demo123` |
| `reception@demo.com`      | `demo123` |
| `referral@demo.com`       | `demo123` |
| `accountant@demo.com`     | `demo123` |
| `patientgateway@demo.com` | `demo123` |
| `notifications@demo.com`  | `demo123` |
| `telemed@demo.com`        | `demo123` |
| `donor@demo.com`          | `demo123` |
| `radiographer@demo.com`   | `demo123` |
| `dept-head@demo.com`      | `demo123` |
| `qa-director@demo.com`    | `demo123` |

The seeded super-admin account requires **one-time forced password rotation** on first login. The demo tenant enables Stripe + Telebirr (payments) and Ethio telecom (SMS) to exercise multiple adapters.

---

## 17. Scope Boundary (What NOT to Build in MVP)

**Out of scope (MVP) — these land in Beta/v1 behind the adapter ports:**

- Concrete **DICOM gateway / PACS** integration — port defined in MVP, provider wired in Beta+.
- Concrete **HL7 v2 / FHIR API** server — ports defined in MVP, server + IG in Beta+.
- **Advanced clinical AI beyond rule-based triage / limited CDS / limited QI-QA** — `ClinicalAiPort` defined in MVP, governed models only in Beta+ (requires bias audit + model card).
- **SNOMED CT / LOINC / RxNorm terminology services** beyond ICD-10/CPT — port in MVP, providers Beta+.

**Out of scope (MVP and beyond MVP per product direction):**

- Computer-aided diagnosis (CAD) as autonomous decision-maker (always advisory; clinician's order stands)
- Embedded BI / OLAP dashboards beyond the fixed operational + QI report set
- Specialty workflows: dental, ophthalmology, veterinary, mental health, oncology, dialysis, ICU
- Multi-region active-active deployment or national-scale (1000+ tenant) provisioning
- Native iOS/Android app (MVP ships a web PWA)
- Two-way SMS, IVR, WhatsApp, or mobile push notifications (MVP is one-way transactional SMS/email)
- Custom ML training infrastructure (MLOps pipeline) — inference only via governed adapters
- White-label OEM or per-tenant custom domain SSL certificates
- Chaos engineering suite or continuous performance regression autopilot
- Legacy QuantuMed/HMS data migration execution (the upgrade path is documented; execution is per-engagement)

---

_For the full canonical rule set and phased workflow, always refer to the master documents in [`docs/agents/`](docs/agents/): [`WORKSPACE_RULES.md`](docs/agents/WORKSPACE_RULES.md) (Part 1) and [`AGENTIC_WORKFLOW.md`](docs/agents/AGENTIC_WORKFLOW.md) (Part 2)._

---
> Source: [dawit-devops/quantumed-hms-main-](https://github.com/dawit-devops/quantumed-hms-main-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
