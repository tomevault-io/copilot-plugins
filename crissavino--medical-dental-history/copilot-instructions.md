## medical-dental-history

> **→ think before coding**

# Dental EHR - Clinical History

## Working Principles

**→ think before coding**
State your assumptions, ask when unsure. Never guess.

**→ simplicity first**
Write the minimum code that solves the problem. No abstractions nobody asked for.

**→ surgical changes**
Don't touch code unrelated to the request. Every changed line must trace back to what was asked.

**→ goal-driven execution**
Turn vague instructions into verifiable success criteria before writing a single line.

## Build & Run Commands

> **PHP version:** This project requires PHP >= 8.2, but the system `php` on this machine is 7.4 (MAMP). Use the Homebrew binary `/usr/local/opt/php@8.3/bin/php` for every `php`/`artisan` command. Example: `/usr/local/opt/php@8.3/bin/php artisan test`. Plain `php artisan ...` will fail with a platform-requirement error.

```bash
# Install dependencies
composer install
npm install

# Database setup (requires MySQL 8 running with 'clinical_history' database)
php artisan migrate --seed

# Development
npm run dev          # Vite dev server (HMR)
php artisan serve    # Laravel dev server

# Production build
npm run build

# Run tests
php artisan test

# Clear caches
php artisan optimize:clear
```

## Architecture Overview

- **Laravel 11 LTS** + **Inertia.js** + **Vue 3** (TypeScript) monorepo
- **MySQL 8** database, **Tailwind CSS 3** styling
- **Laravel Breeze** (Inertia + Vue) for authentication scaffolding
- **Sanctum** session-based auth (built into Breeze)
- **vue-i18n** for multilingual UI (EN/RO/ES — translation files in `resources/js/i18n/`)

## Key Patterns

### Audit Observer
`app/Observers/AuditObserver.php` is registered on Patient, AnamnesisVersion, Encounter, Treatment, and Attachment models. It logs `created`, `updated`, `deleted` events with old/new values to the `audit_logs` table.

### Polymorphic Attachments
`Attachment` model uses `morphTo` relationship (`attachable_type` + `attachable_id`). Supported types: Patient, Encounter, Treatment. Files stored in `storage/app/attachments/`.

### Role-Based Authorization
Simple enum role on `users` table: `admin`, `dentist`, `assistant`, `receptionist`.
- `EnsureRole` middleware for route-level checks (aliased as `role:`)
- Policy classes for model-level authorization
- `User::hasRole()` helper method

### Anamnesis Versioning
Each anamnesis submission creates a new `AnamnesisVersion` record (immutable). Versions are auto-incremented per patient. Form data is stored as JSON.

### Public Intake Wizard
The `/intake` route is publicly accessible (no auth). Patients can fill out their anamnesis form in EN or RO. Creates patient record if needed.

### Encounter Signatures & Lockdown (GDPR)
Each encounter is closed by capturing a **fresh** patient signature **and** dentist signature at the moment of completion (no reusing the patient's stored signature — every encounter is treated as high-risk). On signing, the encounter is set to `completed` and **locked**: it, its treatments, and its attachments become immutable. Enforced with defense-in-depth — both Policy methods (`Encounter::isLocked()`) and FormRequest `authorize()` checks.
- **Sign:** `POST /encounters/{encounter}/sign` → `EncounterController::sign` (wrapped in `DB::transaction` + `lockForUpdate`). Stores both signatures, timestamps, `signed_ip`, and a SHA-256 `signed_hash` over canonical content for tamper evidence. Writes an explicit `signed` audit log row.
- **Corrections:** signed encounters are never edited. `POST /encounters/{encounter}/rectify` creates a NEW encounter with `rectifies_encounter_id` pointing to the original, copying metadata + treatments. Writes a `rectified` audit log row on the original.
- **Dentist signature reuse:** the dentist (authenticated) MAY reuse their own stored `users.signature_data` via a toggle. The patient may not.
- **PDFs for inspection:** `GET /encounters/{encounter}/pdf` (per-encounter) and `GET /patients/{patient}/clinical-history/pdf` (consolidated). The clinical-history PDF is also bundled into the GDPR Export ZIP. Both use `barryvdh/laravel-dompdf` with blade templates in `resources/views/pdf/`.
- **UI:** `resources/js/Pages/Encounters/SignWizard.vue` (3-step modal: review → patient sig → dentist sig), reusable `resources/js/Components/SignaturePad.vue`.

## Database Schema

| Table | Key Columns |
|-------|-------------|
| users | name, email, role (enum), password |
| patients | identifier (unique), first_name, last_name, date_of_birth, gender, phone, email, cnp, soft_deletes |
| anamnesis_versions | patient_id, version, form_data (JSON), consent_given, language |
| encounters | patient_id, provider_id, encounter_date, chief_complaint, clinical_notes, diagnosis, status, patient_signature_data, dentist_signature_data, patient_signed_at, dentist_signed_by, dentist_signed_at, signed_ip, signed_hash, rectifies_encounter_id, soft_deletes |
| treatments | encounter_id, tooth_number (FDI), treatment_code, description, surface, cost, status |
| attachments | attachable_type, attachable_id (polymorphic), uploaded_by, file_name, file_path, mime_type, file_size |
| audit_logs | entity_type, entity_id, user_id, action (created/updated/deleted/signed/rectified), metadata_json, ip_address |

## Directory Structure

```
app/
├── Http/
│   ├── Controllers/    # 8 controllers (Patient, Encounter, Treatment, Attachment, Anamnesis, Intake, AuditLog, GdprExport)
│   ├── Middleware/      # EnsureRole middleware
│   └── Requests/       # form request validators (incl. SignEncounterRequest)
├── Services/           # AnamnesisPdfService, EncounterPdfService, ClinicalHistoryPdfService
├── Models/             # 7 models (User, Patient, AnamnesisVersion, Encounter, Treatment, Attachment, AuditLog)
├── Observers/          # AuditObserver
└── Policies/           # 4 policies (Patient, Encounter, Treatment, Attachment)

resources/js/
├── Pages/              # Inertia pages (Dashboard, Patients/*, Encounters/*, Intake/*, AuditLogs/*)
├── Components/         # Vue components (Layout/*, Patient/*, Encounter/*, Treatment/*, Attachment/*, Anamnesis/*, UI/*)
├── Composables/        # useToothNotation, useResponsive
├── i18n/               # EN/RO translation files
└── types/              # TypeScript interfaces
```

## Roles & Permissions

| Action | Admin | Dentist | Assistant | Receptionist |
|--------|-------|---------|-----------|--------------|
| CRUD Patients | ✓ | ✓ | Read only | ✓ |
| CRUD Encounters | ✓ | ✓ | Create/Edit | Read only |
| CRUD Treatments | ✓ | ✓ | Create/Edit | ✗ |
| CRUD Attachments | ✓ | ✓ | Create/View | ✗ |
| Sign / Rectify Encounter | ✓ | ✓ | ✗ | ✗ |
| Encounter & Clinical-History PDF | ✓ | ✓ | ✗ | ✗ |
| View Audit Logs | ✓ | ✗ | ✗ | ✗ |
| GDPR Export | ✓ | ✓ | ✗ | ✗ |

## Seed Credentials

| Email | Password | Role |
|-------|----------|------|
| admin@clinic.com | password | admin |
| maria@clinic.com | password | dentist |
| andrei@clinic.com | password | dentist |
| elena@clinic.com | password | assistant |
| ana@clinic.com | password | receptionist |

---
> Source: [Crissavino/medical-dental-history](https://github.com/Crissavino/medical-dental-history) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
