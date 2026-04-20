## biogov

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**bioGov** is a planned Progressive Web Application (PWA) designed to guide Israeli small-to-medium business owners through government bureaucracy related to:
- Business registration (VAT dealer status: exempt vs authorized)
- National Insurance (NI) registration for self-employed
- Business licensing (רישוי עסקים)
- Ongoing compliance (VAT deadlines, withholding tax, renewals)
- Document checklists and deep-linking to official government services

**Current Status**: Documentation and planning phase. No source code exists yet—only technical blueprints and requirements.

**Target Users**: Self-employed individuals (עוסק פטור/עוסק מורשה), micro-LTDs (חברה בע״מ קטנה), service trades requiring business licenses.

---

## Technology Stack (Planned)

### Architecture
- **Monorepo**: Turborepo + PNPM workspaces
  - Shared packages for types, utilities, compliance
  - Multiple apps (web, admin, Strapi)
  - Remote caching for faster CI/CD

### Frontend
- **Framework**: Next.js 14+ (App Router) with React 18+, TypeScript
- **PWA**: Service Workers (Workbox) for offline-first functionality
- **Storage**: IndexedDB for offline checklists, compliance calendar, task state
- **State Management**: TanStack Query (React Query) for server state
- **Validation**: Zod for schema validation
- **Localization**: react-i18next (Hebrew RTL-first, English, Russian future)

### Backend & Data
- **Database & Auth**: Supabase (PostgreSQL + Authentication + Storage + Real-time)
  - PostgreSQL for relational data (users, tasks, compliance calendar, audit logs)
  - Built-in authentication (email/password, social login, MFA)
  - Row-Level Security (RLS) for data privacy
  - Real-time subscriptions for live updates
  - File storage for documents
  - Can be self-hosted in Israel for data residency compliance

- **Cache & Jobs**: Redis (Upstash or self-hosted)
  - Job queues for push notification triggers
  - Cache for government API responses (data.gov.il)
  - Session storage for high-performance auth
  - VAT deadline reminder queue

- **CMS**: Strapi (headless CMS)
  - Hebrew micro-guides with government citations
  - Multi-language content management
  - Versioned knowledge cards
  - Non-developer content updates

### External Integrations
- **Government Services**: Deep-linking only (no impersonation). SSO via MyGov authentication
- **Open Data APIs**:
  - ICA company registry (data.gov.il) for form prefill
  - Business licensing dataset for eligibility checks
- **Calendar**: VAT deadline rules (15th/16th/23rd), holiday adjustments, iCal export

---

## Architecture

### Core Components

1. **Eligibility & Pathfinder Engine**
   - Input: Business activity, revenue, location, employees
   - Output: Dealer type (exempt/authorized), licensing requirements, document checklist
   - Data source: Government classification rules

2. **Compliance Calendar Engine**
   - Israeli-specific VAT deadlines (15th/16th/23rd rules)
   - Holiday adjustment table for postponements
   - Push notification triggers
   - iCal export capability

3. **Document Checklist Generator**
   - Maps to business path (e.g., Form 821 for authorized dealer, exempt dealer flow)
   - Dynamic based on user selections
   - Links to official government sources

4. **Company Data Autofill Module**
   - Queries ICA company registry (open data)
   - Prefills business information (non-sensitive data only)

5. **Deep-Link Router**
   - Authenticates via MyGov SSO
   - Routes users to official government services
   - Monitors links for breakage

6. **Knowledge Card System**
   - Educational cards (VAT 18%, e-invoicing rules, licensing requirements)
   - Inline citations to government pages
   - Versioned content with "last updated" timestamps

### Security Model
- **No government service impersonation** (SSO routing to MyGov only)
- **Pseudonymous user IDs** to minimize PII exposure
- **Encrypted storage** of sensitive tokens
- **Access logging** for audit trails
- **Role-based access control** (future multi-tenant)

---

## Database Schema (Proposed)

### Core Tables

**Users**
- `user_id` (PK), `email`, `hashed_password`, `preferred_language`, `accessibility_flags`, `created_at`, `updated_at`, `consent_metadata`

**BusinessProfile**
- `profile_id` (PK), `user_id` (FK), `business_type`, `activity_code`, `municipality_id`, `projected_revenue`, `date_started`, `license_required_flag`

**Tasks**
- `task_id` (PK), `profile_id` (FK), `task_type` (enum: vat_open, ni_register, license_check), `status`, `due_date`, `link_url`, `required_documents` (JSON)

**ComplianceCalendarEntries**
- `entry_id` (PK), `profile_id` (FK), `entry_type` (vat_monthly, withholding_q), `due_date`, `reminder_sent_flag`, `completed_flag`

**DatasetCache**
- `cache_id` (PK), `dataset_name` (ICA_companies, business_licensing_catalog), `version`, `timestamp_fetched`, `data`

**AuditLogs**
- `log_id` (PK), `user_id` (FK), `action_type`, `timestamp`, `metadata` (JSON)

**Preferences**
- `pref_id` (PK), `user_id` (FK), `push_notification_opt_in`, `other_settings`

---

## Regulatory & Compliance Requirements

### Israeli Privacy Law
- **Amendment 13** to Protection of Privacy Law (effective Aug 14, 2025)
  - Expanded definitions of "personal data" and "highly sensitive data"
  - Mandatory Privacy Protection Officer (PPO) appointment under certain thresholds
  - Stronger enforcement by Privacy Protection Authority (PPA)
  - **Action Required**: Classify data, draft DPIA (Data Protection Impact Assessment), define retention schedules

### Data Security Regulations
- Maintain "database definition document" describing personal data, purposes, access controls
- Transfer data abroad only to countries with equivalent protection
- Encryption at rest, role-based access, incident response plan
- **Minimize PII storage** to reduce regulatory burden

### Accessibility (IS-5568)
- Comply with Israeli Web Accessibility Standard (WCAG 2.0 AA equivalent)
- RTL support for Hebrew, keyboard navigation, focus states, contrast
- Screen reader compatibility
- **Publish accessibility statement** on site
- Non-compliance can lead to lawsuits and fines

### Data Minimization Strategy
- **Store**: Email, business type, activity code, municipality, task state, calendar entries, cached public datasets
- **Avoid**: Scanned documents (IDs, bank statements), sensitive financial data, full ID numbers (unless absolutely necessary and encrypted)
- **Rationale**: "Especially sensitive personal data" triggers heavy compliance burden

---

## Development Workflow (Monorepo with Turborepo)

### Setup Commands
```bash
# Install PNPM globally (if not already installed)
npm install -g pnpm

# Install all dependencies (entire monorepo)
pnpm install

# Generate Supabase types
pnpm run supabase:generate-types
```

### Development Commands
```bash
# Run all apps in development mode (parallel)
pnpm dev

# Run specific app only
pnpm dev --filter=web         # Main PWA
pnpm dev --filter=admin       # Admin dashboard
pnpm dev --filter=strapi      # Strapi CMS

# Run app with dependencies
pnpm dev --filter=web...      # Web + all dependencies
```

### Build Commands
```bash
# Build all apps and packages
pnpm build

# Build specific app
pnpm build --filter=web

# Build with remote caching (team/CI)
pnpm build --cache-dir=".turbo"
```

### Testing Commands
```bash
# Run all tests
pnpm test

# Run tests for specific package
pnpm test --filter=@biogov/utils
pnpm test --filter=@biogov/israeli-compliance

# Run tests with coverage
pnpm test:coverage
```

### Linting & Formatting
```bash
# Lint all packages
pnpm lint

# Lint specific package
pnpm lint --filter=web

# Format all code
pnpm format

# Type-check entire monorepo
pnpm typecheck
```

### Package Management
```bash
# Add dependency to specific package
pnpm add <package> --filter=web
pnpm add <package> --filter=@biogov/utils

# Add dependency to all packages
pnpm add <package> -w

# Update dependencies
pnpm update --recursive
```

### Supabase Commands
```bash
# Start local Supabase (Docker required)
pnpm supabase:start

# Stop local Supabase
pnpm supabase:stop

# Generate TypeScript types from Supabase schema
pnpm supabase:generate-types

# Run migrations
pnpm supabase:migrate

# Reset database
pnpm supabase:reset
```

### Redis Commands
```bash
# Start local Redis (Docker required)
docker run -d -p 6379:6379 redis:alpine

# Or use docker-compose
docker-compose up redis
```

### Testing Strategy (Planned)
1. **Functional Testing**
   - VAT pathfinder logic verification
   - NI registration flow validation
   - Licensing checker accuracy
   - Calendar date correctness (Israeli-specific deadline rules)

2. **Accessibility Testing**
   - IS-5568 AA compliance (WCAG 2.0 AA)
   - Screen reader support (NVDA, JAWS)
   - Keyboard navigation
   - RTL layout testing
   - Focus states and contrast verification

3. **Compliance Testing**
   - Privacy & security requirements verification
   - DPIA validation
   - Audit logging verification

4. **Content Testing**
   - Hebrew language proofing
   - Citation accuracy to government sources
   - Link validity monitoring

---

## Key Government Service Deep Links

### Identity & Authentication
- **MyGov National Login**: https://my.gov.il/

### Tax Authority (רשות המסים)
- **VAT Authorized Dealer (Form 821)**: https://www.gov.il/en/service/vat-821
- **Exempt Dealer Application**: https://www.gov.il/he/service/request-open-exempt-dealer-via-internet
- **Verify Dealer Status**: https://www.gov.il/he/service/vat-apply-online
- **VAT Deadlines (15/16/23 rules)**: https://www.gov.il/he/pages/pa151025-2

### National Insurance (ביטוח לאומי)
- **Self-Employed Registration**: https://www.btl.gov.il/Insurance/National%20Insurance/type_list/Self_Employed/Pages/default.aspx
- **Single Digital Process (2024+)**: https://www.btl.gov.il/About/news/Pages/ArchiveFolder/2025/PtictTik.aspx
- **NI Personal Area**: https://ps.btl.gov.il/

### Business Licensing
- **Licensing Hub**: https://www.gov.il/he/departments/topics/business_licensure/govil-landing-page
- **Apply for License**: https://www.gov.il/he/service/application-for-new-business-license
- **Open Data (Activities Requiring License)**: https://data.gov.il/dataset/business-licensing-br7

### Corporations Authority
- **Company Extracts**: https://www.gov.il/he/service/company_extract
- **ICA Company Registry Dataset**: https://data.gov.il/dataset/ica_companies

---

## Critical Development Considerations

### VAT Compliance Calendar Rules
- **15th Rule**: Most VAT dealers report by 15th of following month
- **16th Rule**: Some categories get extension to 16th
- **23rd Rule**: Special categories (large businesses) report by 23rd
- **Holiday Adjustments**: Build override table for Shabbat/holiday postponements
- **2025 Update**: VAT rate is 18% (changed Jan 1, 2025)

### E-Invoicing (Future Phase 2)
- **Reference Number**: Required from 2025
- **Allocation Number**: Phased thresholds (15k→10k→5k revenue, 2026-2028)
- Provide educational cards and checklists, but **do not submit on user's behalf**

### Offline-First Strategy
- Cache critical content (guides, checklists, saved tasks) via Service Worker
- Store task state in IndexedDB for offline access
- Sync queue for reminders when device comes online
- **Include "last updated" timestamps** to alert users to stale offline data
- Force online check for critical flows (e.g., final submission links)

### Content Versioning
- Government pages change frequently
- Use headless CMS (Strapi) with versioned content
- Inline citations to official sources
- Implement link-monitoring system to detect broken deep links
- Display "Last verified: [date]" on all knowledge cards

### Hebrew RTL & Localization
- Hebrew is primary language (RTL-first design)
- Test form alignment, number formatting, date formats
- Avoid text truncation issues in RTL
- Support accessibility in all languages

---

## Delivery Plan (6 Sprints)

**Sprint 0 — Governance & Compliance**
1. Draft Privacy Notice (Amendment 13 compliant)
2. Define DPIA and data classification
3. Accessibility plan (IS-5568)
4. Define retention schedules

**Sprint 1 — MVP Skeleton**
1. Next.js PWA scaffolding (Hebrew RTL, Service Worker, IndexedDB)
2. Strapi CMS setup for knowledge cards

**Sprint 2 — Eligibility & Checklists**
1. Encode VAT dealer decision logic (exempt vs authorized)
2. Build NI registration guidance
3. Document checklist generator for Form 821

**Sprint 3 — Business Licensing**
1. Activity → license eligibility checker
2. Ministry approval map
3. Deep-link to municipal application portals

**Sprint 4 — Compliance Calendar**
1. Implement Israeli VAT deadline rules (15/16/23)
2. Holiday adjustment table
3. Push notifications and iCal export

**Sprint 5 — Company Data Autofill (Optional)**
1. ICA company registry lookup
2. Form prefill with non-sensitive data

**Sprint 6 — Quality & Release**
1. IS-5568 AA accessibility audit
2. Hebrew content proofing
3. Privacy & security review
4. Link validity testing

---

## Risk Register (High Priority)

| Risk | Impact | Mitigation |
|------|--------|------------|
| **Amendment 13 non-compliance** (holding PII without proper documentation) | High | Classify data, draft DPIA, track purpose/retention, appoint PPO if needed |
| **IS-5568 accessibility non-compliance** | Medium-High | Build accessibility from day one, test WCAG AA, publish statement |
| **Broken deep-links or stale guidance** | Medium | Monitor links, version content, display "last verified" dates |
| **Incorrect compliance calendar logic** | Medium | Encode Israeli deadline rules, holiday adjustments, user testing |
| **Offline data staleness** | Medium | Include timestamps, force online check for critical flows |
| **Storing sensitive documents (bank, IDs)** | High | Avoid in MVP; if needed, encrypt, restrict access, document in DPIA |

---

## Important Notes for Future Developers

### What NOT to Do
- **Do not impersonate government services** — only deep-link via MyGov SSO
- **Do not store sensitive financial data** (bank statements, full ID numbers) unless absolutely necessary
- **Do not rely on generic calendar logic** — Israeli VAT deadlines follow specific 15/16/23 rules with holiday adjustments
- **Do not ignore accessibility** — IS-5568 compliance is legally required and auditable
- **Do not cache government content indefinitely** — implement refresh cycles and version tracking

### What TO Do
- **Always cite official government sources** with versioned URLs
- **Always test RTL layouts** in Hebrew before deployment
- **Always encrypt tokens** using WebCrypto for client-side storage
- **Always log access** for audit trail (required by Data Security Regulations)
- **Always validate links** to government services periodically (they change frequently)
- **Always include "last updated" timestamps** on knowledge cards and cached content
- **Always minimize PII storage** to reduce regulatory burden
- **Always provide disclaimers** that the app guides but does not replace legal/tax advisors

---

## Documentation Files

- **`docs/softwareAnalyse`** — Complete product specification (259 lines): business requirements, user stories, functional scope, 10 canonical government sources with deep links, 6-step delivery plan, QA checklist, e-invoicing phase 2 planning
- **`docs/technicalTips.md`** — Server-side architecture requirements, database schema (8 core tables), client-side offline-first strategy, regulatory compliance requirements, data retention policies, risk analysis for PII storage
- **`docs/notToGorget.md`** — Risk register with 6 high-priority items, regulatory compliance risks (Amendment 13, IS-5568), product/UX pitfalls (offline stale data, RTL issues), business/operational risks, monetization strategy, technical debt warnings

---

## Contact & Support

This project adheres to Israeli regulatory requirements and is designed to help SMBs navigate government bureaucracy. All development must prioritize:
1. **Accessibility** (IS-5568 compliance)
2. **Privacy** (Amendment 13 compliance)
3. **Data minimization** (reduce regulatory burden)
4. **Content accuracy** (versioned government citations)
5. **User trust** (no impersonation, clear disclaimers)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MichaelMishaev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
