## dental-saas

> **Role:** Implementer & Documentation Maintainer (SaaS Track)

# Gemini - SaaS Stabilization & Tenant Hardening Implementer Guide

**Role:** Implementer & Documentation Maintainer (SaaS Track)
**Mission:** Execute the **SaaS Program** with surgical precision: tenant isolation, control-plane separation, API semantics, and regression-free guardrails.
**Status:** Active
**Last Updated:** 2026-01-28

---

## A. Role Definition

I am the **Implementer**. My job is to implement only what the Planner/Reviewer (Codex) assigns, and to maintain an audit-grade trail of what changed and how it was verified.

### Current Program of Record (Single Source of Truth)
1) **SaaS Program:** `docs/SaaS/SAAS_PROGRAM.md` (governs scope, priority, DoD for SaaS)
2) **Tenant Context Contract:** `docs/SaaS/TENANT_CONTEXT_CONTRACT.md`
3) **API Contract:** `docs/API_CONTRACT.md` (authoritative behavior for `/api/v1/*` once implemented)
4) **Guardrails:** `docs/SaaS/guardrails/*` and the repo guardrail scripts (if present)

---

## B. Non-Negotiable Rules (Failing any rule = STOP)

1) **No Scope Expansion (Hard Rule):** Implement ONLY assigned Task IDs.
2) **Fail-Closed Always:** Auth, tenant context, RBAC must deny by default.
3) **Sacred Separation:** `global_mode=true` (Control Plane) must never access tenant web routes (Airlock doctrine).
4) **API Determinism:** `/api/*` never redirects; returns JSON/Error codes only.
5) **Minimum Effective Change:** No refactors unless required by DoD. Small, testable patches.
6) **Zero-Trust UI:** All ID-based operations must be scoped by `active_clinic_id`.

---

## C. Application Map (Source of Truth)

This is the canonical reference. Update it on every iteration so future implementers can skip past log replays and immediately understand the frontend, backend, database, connectors, guardrails, and environment.

### 1. Frontend & Layouts (`app/Views`)
- **Control Plane UI:** `app/Views/control_plane/` hosts dashboards, clinics registry, plans, plan requests, invoices, activity log, operations, settings, and search views. Each extends `layouts/main_control_plane.php` (dark sidebar, glass cards, DataTables+AJAX). Use `control_plane/clinics.php` as the template for any new registry.
- **Tenant UI:** `app/Views/dashboard`, `clinic`, `patient`, `appointment`, `finance`, `inventory`, `examination`, `treatment`, `prescription`, `reports`, etc., follow the glassmorphic stats header + DataTables pattern with action dropdowns. Shared pieces live in `app/Views/components/` (e.g., `smtp_form.php`).
- **RBAC & Users:** Tenant user management screens under `app/Views/user_management/`. Role management (system + clinic) lives in `app/Views/roles/` using permission groupings (`Permissions::getGroupedByModule()`), action descriptions, and glassy selectors.
- **Auth & Selection:** `layouts/main_auth.php` powers login/forgot/reset flows in `app/Views/auth/*`. Clinic selection uses `app/Views/clinic/select.php`, which respects guardrail instructions.

### 2. Control Plane vs. Tenant Plane (Routes & Filters)
- **Control Plane Routes:** Declared in the `controlplane` group in `app/Config/Routes.php` (protected by the `controlplane` filter). Key controllers: `ControlPlane\Entry`, `Dashboard`, `Clinics`, `Plans`, `PlanRequests`, `Invoices`, `ActivityLog`, `Operations`, `Settings`, `Search`, `Onboarding`. They require `global_mode=true` and fail closed with 404 when the session is invalid.
- **Tenant Plane Routes:** Protected by `auth`, `tenant`, `subscription`, and `permission` filters. They require `active_clinic_id` and run without `global_mode`. Examples: `/patient`, `/appointment`, `/finance`, `/inventory`, `/reports`, `/notifications`.
- **Session Keys:** `global_mode`, `active_clinic_id`, `is_impersonating`, and `impersonated_clinic_id` ensure airlock separation; missing values trigger the tenant-aware 404 logic.

### 3. Backend Controllers & Flows
- **Control Plane Controllers:** `app/Controllers/ControlPlane/*` handles clinics, onboarding, plans, invoices, plan requests, dashboard, activity log, operations, settings, search, and impersonation. Each enforces `global_mode`.
- **RBAC Management:** `RoleController` orchestrates system and clinic roles, permission syncing (`PermissionSyncService`), stats (permission counts/user counts), and uses `PermissionService`'s `isSuperAdmin` and `canManageRoles`.
- **Tenant User Flow:** `UserManagementController` uses `PlanGuard` for quotas, `ClinicUserModel` for memberships, `PermissionService` for permission grants, and remains inside the tenant plane.
- **Auth & Clinic Selection:** `Auth.php`, `ControlPlane.php`, and `ClinicSelector.php` control logins, control-plane entry/exit, and the clinic select wall, all backed by `IonAuth`.
- **Operation Modules:** `Patient`, `Doctor`, `Appointment`, `Examination`, `Finance`, `Inventory`, `Treatment`, `Prescription`, `Odontogram`, `Reports`, etc., all extend from base controllers and use the `permission` filter to gate access.
- **API Controllers:** `app/Controllers/Api/*` and `app/Controllers/Api/V1/*` provide JSON endpoints protected by `apiv1` filters and permission codes (e.g., `permission:patients:view`).

### 4. Models & Services
- **Tenant Base:** `TenantAwareModel` auto-appends `clinic_id`, throws `PageNotFoundException` when context is missing, and allows controlled use of `withoutTenantScope()`.
- **Key Models:** `ClinicModel`, `ClinicUserModel`, `UserModel`, `RoleModel`, `PermissionModel`, `UserRoleModel`, `PlanModel`, `PlanUsageModel`, `ClinicSubscriptionModel`, `FinanceModel`, `InventoryModel`, `AppointmentModel`, `PatientModel`, `ExaminationModel`, `NotificationModel`, `EmailTemplateModel`, `ControlPlaneInvoiceModel`.
- **RBAC Services:** `PermissionService`, `PermissionSyncService`, `UserRoleModel`, `RolePermissionModel`, `PermissionModel` (with categories/actions from `\App\Config\Permissions`).
- **Utilities & Connectors:** `IonAuth` (identity), `PlanGuard` (quota enforcement), `TenantJob`/`RunTenantJob` (tenant context), `ControlPlaneAuditService`, `StorageService`, `RetentionService`, `OnboardingService`, `NotificationLedger`, `PlanGuard`.

### 5. Database Schema & Tables
- **Identity & RBAC:** `users`, `clinic_users`, `roles`, `role_permissions`, `permissions`, `user_roles`, `permission_sync`.
- **Control Plane Data:** `clinics`, `clinic_subscriptions`, `plans`, `plan_requests`, `plan_usage`, `control_plane_invoices`, `control_plane_invoice_payments`, `activity_log`, `control_plane_audit`, `operations`.
- **Tenant Operations:** `patients`, `examinations`, `appointments`, `finance`, `prescriptions`, `inventory`, `odontograms`, `doctors`, `reports`, `notifications`, `email_templates`.
- **Jobs & Audits:** `job_audit`, `notification_ledger`, `plan_audit`, `clinic_subscription_logs`.
- **Constraints:** Tenant tables enforce `clinic_id` (composite indices). `roles` uses `(clinic_id, slug)` constraints and `clinic_users` enforces `(clinic_id, user_id)` uniqueness.

### 6. Connectors, Guardrails & APIs
- **Identity Connector:** IonAuth via `App\Libraries\IonAuth` handles registration, activation, password resets, and group fallback during RBAC setup.
- **Permission Sync:** `PermissionSyncService` hits `/rbac/sync`, `/rbac/status`, `/rbac/init` routes (fresh from `app/Config/Routes.php`).
- **Guardrails:** Run `scripts/ci/saas_guardrails.sh` (DOM sinks, raw tenant query detection, group-based auth checks). Note `E_ACCESSDENIED` when Bash cannot execute on Windows; document it in each verification doc so future sessions can skip reruns unless the guarded areas change.
- **API Invariants:** `/api/*` never redirects; `/api/v1/*` uses JSON and explicit permission filters.
- **Isolation Filters:** `TenantFilter`, `ControlPlaneFilter`, `SubscriptionFilter`, `PermissionFilter` orchestrate failsafe plane separation.

### 7. Environment & Infrastructure (Local)
- **Docker:** Container `mysql8-dental` (image `mysql:8.0`, exposed port 3307 -> 3306). Use it in verification docs with `docker exec mysql8-dental ...`.
- **Canonical `.env` Data (do not override):**
    * `APP_ENV=development`
    * `APP_URL=http://localhost/dental-saas/`
    * `database.default.hostname=127.0.0.1`
    * `database.default.port=3307`
    * `database.default.database=democa_dental_saas`
    * `database.default.username=root`
    * `database.default.password=root`
    * `encryption.key=9f3a2c1d8b7e6a5c4d3e2f1a0b9c8d7e`
    * `session.cookie_name=ci_session_saas`

### 8. Logging, Verification & Knowledge Flow
- **SaaS Logs:** Append-only logs `docs/SaaS/logs/IMPLEMENTATION_LOG_SaaS.md` and `docs/SaaS/logs/DECISION_LOG_SaaS.md` are the historical record-read the latest; do not edit previous entries.
- **Verification Artifacts:** Each Task ID (e.g., `P5-20a`, `CP-31`) needs `docs/SaaS/verification/<TASK_ID>.md` with commands, expected outputs, guardrail evidence, and any known failures (like guardrail script `E_ACCESSDENIED`).
- **Knowledge Transfer:** This `gemini.md` is now the single source-of-truth map. Update it when you add new subsystems so future sessions can jump in quickly.

---

## D. Recent Accomplishments (Jan 2026)

### 1. Control Plane & Airlock
*   **P5-20a:** Split entry vs. protected control-plane routes in `app/Config/Routes.php` and restored `/controlplane` access (log: `docs/SaaS/logs/IMPLEMENTATION_LOG_SaaS.md`, same day).
*   **Airlock Doctrine:** `TenantFilter`/`ControlPlaneFilter` now fail closed when session context mismatches; coverage ensured by guardrail scripts.

### 2. RBAC, Roles & Users
*   **RoleController:** Manages system + clinic roles, permissions (sync, stats) via `PermissionService`/`RoleModel`.
*   **UserManagementController:** Tenant user flows with `PlanGuard` quotas and `ClinicUserModel` assignments. Works entirely inside the tenant plane; a control-plane counterpart is planned.

### 3. SaaS Guardrails & Docs
*   **Guardrails Documented:** S0-series doc entries describe DOM sink removal, preference persistence, ASCII fixes, and route corrections. Guardrail script path: `scripts/ci/saas_guardrails.sh`.
*   **Logs:** The Implementation/Decision logs embed each milestone; they are now the single catch-up source.

### 4. Automation & Jobs
*   **TenantJob & TenantAwareModel:** Strict context enforcement; jobs require `clinic_id`, otherwise they throw 404.
*   **PlanGuard & Subscription Enforcement:** Quota checks (`users_max`) run per clinic (`SubscriptionFilter`, `PlanGuard`).

---

## E. Environment & Infrastructure (Local)

### Local Path: `C:\xampp\htdocs\dental-saas`

### 1. Docker Container Info
*   **Container Name:** `mysql8-dental`
*   **Image:** `mysql:8.0`
*   **Status:** Running (Port Mapping: `0.0.0.0:3307->3306/tcp`)
*   **Container ID:** `7eb3024aa7d5`

### 2. `.env` Configuration (canonical and shared)
*   **Environment:** `development`
*   **Base URL:** `http://localhost/dental-saas/`
*   **Database Host:** `127.0.0.1`
*   **Database Port:** `3307`
*   **Database Name:** `democa_dental_saas`
*   **DB Credentials:** `root` / `root`
*   **Encryption Key:** `9f3a2c1d8b7e6a5c4d3e2f1a0b9c8d7e`
*   **Session Cookie:** `ci_session_saas`

### 3. Guardrails & Tools
*   **Guardrail Script:** `bash scripts/ci/saas_guardrails.sh` (fails with `E_ACCESSDENIED` on Windows; note that in verification logs so future runs can skip if unchanged).
*   **Additional Commands:** `rg -n "innerHTML|outerHTML|insertAdjacentHTML|\.html\(" app/Views`; `php spark routes | findstr /I "controlplane api/v1 api/search"`.

---

## F. Behavioral Doctrine

- **Precise & Conservative:** Modify only what is necessary.
- **Evidence-Driven:** Verification artifacts are required for every task.
- **Fail-Closed:** Security is non-negotiable. If context is lost, deny access.

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Bassamalsaqqa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Bassamalsaqqa)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
