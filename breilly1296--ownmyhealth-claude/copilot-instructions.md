## ownmyhealth-claude

> Privacy-first HIPAA-compliant health biomarker tracking platform with insurance document management, AI-powered guidance, provider-patient collaboration, and expense tracking. Focused on secure tracking of health metrics with Codex AI educational insights, insurance cost analysis, and provider data sharing via consent-based access control.

# OwnMyHealth - Project Context

## What This Is
Privacy-first HIPAA-compliant health biomarker tracking platform with insurance document management, AI-powered guidance, provider-patient collaboration, and expense tracking. Focused on secure tracking of health metrics with Codex AI educational insights, insurance cost analysis, and provider data sharing via consent-based access control.

## Tech Stack
- **Frontend**: React 18 + Vite 7.3 + TypeScript + Tailwind CSS
- **Backend**: Node.js + Express 4.18 + TypeScript
- **Database**: PostgreSQL (Cloud SQL) + Prisma ORM
- **Auth**: JWT access tokens + refresh tokens (DB-backed sessions) + CSRF double-submit cookie
- **Encryption**: AES-256-GCM for all PHI (per-user keys via PBKDF2-SHA512)
- **AI**: Anthropic Codex API (biomarker guidance, cost analysis, document extraction)
- **OCR**: Google Document AI (scanned lab reports)
- **Email**: SendGrid (verification, password reset)
- **File Storage**: Google Cloud Storage (lab reports, SBC documents)
- **Testing**: Vitest (frontend and backend)
- **Deployment**: GCP Cloud Run (backend) + GCS bucket (frontend) + Cloud SQL (database)

## Current Features
- **Biomarker Tracking**: Manual entry, history, trends, normal ranges, AI educational guidance
- **DEXA Scan Support**: Upload and track bone density measurements
- **Insurance Management**: SBC document upload (Codex AI extraction), plan comparison, benefit search
- **Expense Tracking**: Projections, actuals, AI-powered cost analysis
- **Health Goals**: Goal tracking with progress notes and history
- **Health Needs**: Track health needs with status, type, and urgency
- **Provider Collaboration**: Consent-based provider-patient data sharing with granular permissions
- **File Management**: Lab report upload (PDF parsing + OCR), GCS storage, signed URL downloads
- **Admin Panel**: User management, audit log viewer, system health stats
- **Demo Mode**: Demo account for development/testing (blocked in production)
- **Audit Logging**: HIPAA-compliant access logging with 7-year retention
- **Email Notifications**: Email verification, password reset via SendGrid

## Removed Features (Jan 2025)
- ~~Health Scoring~~ - 0-100 health scores, risk assessments (dashboard shows "Biomarkers in Range %" instead — a simple in-range ratio, not the removed scoring system)
- ~~CMS Marketplace Integration~~ - healthcare.gov plan search
- ~~Provider Directory~~ - doctor search and recommendations

## Project Structure
```
src/
├── components/
│   ├── analytics/      # Trend charts (TrendChart, BiomarkerChart)
│   ├── auth/           # Login, registration, email verification, password reset
│   ├── biomarkers/     # Biomarker display, entry, modals, AI guidance
│   ├── common/         # Shared UI (Button, Modal, RoleGuard, etc.)
│   ├── dashboard/      # Main dashboard
│   ├── files/          # File management (list, download, delete)
│   ├── insurance/      # Insurance hub, plan management, SBC upload
│   ├── settings/       # Data export, account deletion
│   ├── trends/         # Trend visualizations
│   └── upload/         # File upload components
├── contexts/           # React contexts (Auth, Theme)
├── services/
│   └── api/            # API client modules (13 files)
│       ├── client.ts   # Base HTTP client (axios + interceptors)
│       ├── auth.ts     # Auth endpoints
│       ├── biomarkers.ts
│       ├── insurance.ts
│       ├── expenses.ts
│       ├── healthGoals.ts
│       ├── healthNeeds.ts
│       ├── files.ts
│       ├── upload.ts
│       ├── provider.ts
│       ├── patient.ts
│       ├── admin.ts
│       ├── settings.ts
│       └── index.ts    # Re-exports all modules
├── types/              # TypeScript interfaces
└── data/               # Sample data, nav config

backend/src/
├── controllers/        # Route handlers (10 files)
│   ├── authController.ts
│   ├── biomarkerController.ts
│   ├── expenseController.ts
│   ├── fileController.ts
│   ├── healthGoalsController.ts
│   ├── healthNeedsController.ts
│   ├── insuranceController.ts
│   ├── settingsController.ts
│   └── uploadController.ts
├── middleware/          # Security middleware (8 files)
│   ├── auth.ts         # JWT verification
│   ├── csrf.ts         # CSRF double-submit cookie
│   ├── rateLimiter.ts  # 8 named rate limiters
│   ├── rbac.ts         # Role-based access control
│   ├── demoProtection.ts # Demo account restrictions
│   └── ...
├── routes/             # API route definitions (13 files, 60+ endpoints)
├── services/           # Business logic (18 files)
│   ├── encryption.ts   # PHI encryption (AES-256-GCM)
│   ├── userEncryption.ts # Per-user keys (PBKDF2-SHA512)
│   ├── auditLog.ts     # HIPAA audit trail + cleanup scheduler
│   ├── authService.ts  # Auth logic + session cleanup scheduler
│   ├── database.ts     # Prisma client + RLS context
│   ├── claudeExtraction.ts # Codex AI document extraction
│   ├── sbcExtraction.ts # SBC-specific Codex extraction
│   ├── storageService.ts # Google Cloud Storage
│   ├── emailService.ts # SendGrid transactional email
│   ├── ocrService.ts   # Google Document AI OCR
│   ├── pdfParser.ts    # PDF text extraction
│   └── ...
├── config/             # Environment config (20+ variables)
└── utils/              # Helpers (logger, validation)
```

## Critical Rules

### Security (Non-Negotiable)
1. **NEVER use localStorage/sessionStorage** for sensitive data - memory only
2. **All PHI must be encrypted** with AES-256-GCM before database storage
3. **Every PHI access must be audit logged** - 7-year retention required
4. **Validate all input** at API boundaries - never trust user data
5. **No secrets in code** - use environment variables
6. **Sanitize error messages** - never leak internal details to users

### PHI Encryption
PHI fields are defined in `backend/src/services/encryption.ts` (PHI_FIELDS constant).
Must match Prisma schema exactly. Current encrypted fields:
- User: name, DOB, phone, address
- Biomarker/History: values, notes, unit
- Insurance: member ID, group ID, plan name, provider name, benefits
- Health Goals/Progress: descriptions, notes, target values
- Health Needs: description
- Provider-Patient: relationship notes
- Expenses: descriptions, amounts, provider names, notes (all monetary fields stored as `*Encrypted` String columns with AES-256-GCM ciphertext, not Decimal — see migration `20260206_fix_expense_encryption_types`)
- AI Responses: guidance content, analysis results
- Audit Log: previous/new values (encrypted PHI snapshots)

### Row-Level Security (RLS)
Database-level access control ensures users can only access their own data.

**How it works:**
1. Application sets `app.current_user_id` session variable before queries
2. PostgreSQL RLS policies check this variable against `user_id` in tables
3. System operations use `app.is_admin = true` to bypass RLS

**Usage in code:**
```typescript
import { withRLSContext, withRLSTransaction } from './services/database.js';

// ✅ Correct — queries go through `tx`, which carries the SET LOCAL
const biomarkers = await withRLSContext(userId, async (tx) => {
  return tx.biomarker.findMany();
});

// Transaction with RLS (atomic multi-statement)
await withRLSTransaction(userId, async (tx) => {
  await tx.biomarker.create({ data: {...} });
  await tx.auditLog.create({ data: {...} });
});

// System operation (admin context — RLS policies see `is_admin_session() = true`)
await withRLSContext(null, async (tx) => {
  return tx.user.findMany();
});

// ❌ WRONG — prisma.* inside the callback runs on a different connection
// that does NOT carry the SET LOCAL, so RLS evaluates against NULL.
const biomarkers = await withRLSContext(userId, async () => {
  return prisma.biomarker.findMany();
});
```

**Migration:** `backend/prisma/migrations/20260107_add_rls_policies/`

### Product Guidelines
1. **No medical advice** - always include disclaimers on AI-generated content
2. **User owns their data** - export and deletion capabilities required
3. **Consent-first sharing** - provider access only via explicit patient consent
4. **AI is educational** - Codex responses are informational, not diagnostic

## Key Files
| File | Purpose |
|------|---------|
| `backend/prisma/schema.prisma` | Database models (15+), encrypted field definitions |
| `backend/prisma/migrations/20260107_add_rls_policies/` | Row-Level Security policies |
| `backend/src/services/database.ts` | Prisma client, RLS context management |
| `backend/src/services/encryption.ts` | PHI encryption, PHI_FIELDS mapping |
| `backend/src/services/userEncryption.ts` | Per-user key derivation (PBKDF2-SHA512) |
| `backend/src/services/auditLog.ts` | HIPAA audit trail + retention cleanup scheduler |
| `backend/src/services/authService.ts` | Auth logic + session cleanup scheduler |
| `backend/src/services/claudeExtraction.ts` | Codex AI document extraction |
| `backend/src/services/storageService.ts` | Google Cloud Storage file operations |
| `backend/src/services/emailService.ts` | SendGrid transactional email |
| `backend/src/services/ocrService.ts` | Google Document AI OCR |
| `backend/src/middleware/auth.ts` | JWT verification, route protection |
| `backend/src/middleware/csrf.ts` | CSRF double-submit cookie validation |
| `backend/src/middleware/rbac.ts` | Role-based access control (PATIENT/PROVIDER/ADMIN) |
| `backend/src/middleware/rateLimiter.ts` | 8 named rate limiters |
| `backend/src/middleware/demoProtection.ts` | Demo account restrictions |
| `backend/src/config/index.ts` | All environment variables (20+) |
| `src/contexts/AuthContext.tsx` | Frontend auth state management |
| `src/services/api/client.ts` | Base HTTP client with auth headers |
| `.github/workflows/deploy.yml` | CI/CD: Docker build → Cloud Run + GCS |

## Development Commands
```bash
# Frontend
npm run dev          # Start dev server (port 5173)
npm run build        # Production build
npm run test         # Run Vitest tests

# Backend
cd backend
npm run dev          # Start dev server (port 3001)
npm run build        # Compile TypeScript
npm run test         # Run Vitest tests

# Database
npx prisma generate  # Generate Prisma client
npx prisma migrate dev  # Run migrations
npx prisma studio    # Database GUI
```

## Code Review Checklist
- [ ] Auth middleware on all protected routes?
- [ ] RBAC role check for provider/admin routes?
- [ ] PHI encrypted before storage?
- [ ] Audit logs created for PHI access?
- [ ] RLS context set for database queries? (`withRLSContext` or `withRLSTransaction`)
- [ ] Input validated with Zod at API boundaries?
- [ ] Errors handled without leaking data?
- [ ] No console.log with sensitive data?
- [ ] CSRF token required for mutations?
- [ ] Rate limiting applied where needed?
- [ ] Provider access scoped to consented permissions?
- [ ] Demo account blocked from sensitive operations?
- [ ] AI responses include educational disclaimers?

## Environment Variables
```
# Critical Secrets
DATABASE_URL=postgresql://...
JWT_ACCESS_SECRET=<256-bit secret>
JWT_REFRESH_SECRET=<256-bit secret>
PHI_ENCRYPTION_KEY=<64 hex chars>
AUDIT_LOG_SALT=<hex salt, >=64 hex chars>
ANTHROPIC_API_KEY=<api-key>
SENDGRID_API_KEY=<api-key>
GCP_PROJECT_ID=<project-id>

# Configuration
NODE_ENV=development|production
PORT=3001
CORS_ORIGIN=http://localhost:5173
FRONTEND_URL=http://localhost:5173
EMAIL_FROM=<verified-sender>
GCS_BUCKET_NAME=<bucket-name>
GCP_PROCESSOR_ID=<processor-id>
GCP_LOCATION=<location>

# Security
# (CSRF uses a double-submit cookie — there is NO server-side CSRF secret)
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100

# Demo (development only)
DEMO_ACCOUNT_ENABLED=true|false
DEMO_EMAIL=<email>
DEMO_PASSWORD=<password>
```

## Roles & Access Control
| Role | Level | Capabilities |
|------|-------|-------------|
| PATIENT | 1 | Own data CRUD, manage provider consent, AI guidance |
| PROVIDER | 2 | + View authorized patient data (scoped by consent permissions) |
| ADMIN | 3 | + User management, audit log viewer, system health stats |

## Middleware Stack (Request Processing Order)
1. Helmet (security headers)
2. CORS (origin validation)
3. Cookie Parser
4. CSRF Protection (double-submit cookie)
5. Rate Limiting (global + endpoint-specific)
6. Body Parser (JSON, 10MB limit)
7. Routes (API endpoint handlers)
8. Error Handler (centralized)
9. 404 Handler

---
> Source: [breilly1296/OwnMyHealth-Claude](https://github.com/breilly1296/OwnMyHealth-Claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
