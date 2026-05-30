## prescryb

> Prescryb is a digital prescription platform for doctors in Bangladesh.

# Prescryb

## What this project is

Prescryb is a digital prescription platform for doctors in Bangladesh.
Doctors write prescriptions in a web app, data flows to a backend, and
pharmaceutical companies access anonymized analytics through a separate
dashboard.

**Wedge:** Replace the current manual prescription capture system (people
photographing prescriptions outside doctor chambers) with a digital
alternative that captures data at the source.

**Endgame:** Become Bangladesh's primary healthcare data company.

## Tech stack

- **Frontend:** Next.js 14 (App Router) + TypeScript (strict) + Tailwind CSS
- **Backend:** Python 3.11+ + FastAPI + SQLAlchemy 2.0 (async)
- **Database:** PostgreSQL 16 (via Docker for local dev)
- **Authentication:** NextAuth.js (development), Clerk-ready for production
- **Charts:** Recharts
- **State management:** Zustand (client), TanStack Query (server)
- **Forms:** React Hook Form + Zod
- **i18n:** next-intl (Bangla + English required)
- **PDFs:** WeasyPrint (server-side)
- **Migrations:** Alembic

## Project structure (to be created)

```
prescryb/
├── frontend/         Next.js app (doctor + pharma + admin UIs)
├── backend/          FastAPI Python server
├── docker-compose.yml  Local Postgres
├── docs/             Architecture notes
├── CLAUDE.md         This file
└── README.md
```

## Application surfaces

One Next.js app with three route groups:

- **(doctor)** — doctor-facing pages. Mobile-first PWA. Must work offline.
- **(pharma)** — pharma dashboard. Desktop-first. Charts and filters.
- **(admin)** — internal admin panel. Manage doctors, drugs, pharma clients.

## Database architecture — CRITICAL

Two schemas in one Postgres instance:

- **operational** — patient names, doctor identities, full prescriptions.
  Only doctor app and admin panel touch this.
- **analytics** — anonymized, aggregated data. Only pharma dashboard
  queries this.

A scheduled job (every 15 min) moves anonymized data from operational
to analytics. **Pharma must NEVER access patient-identifiable data.**
This is non-negotiable.

## Coding conventions

- TypeScript strict mode, no `any` types
- Python type hints required on all function signatures
- All database access through SQLAlchemy ORM, never raw SQL except migrations
- API routes follow REST: `/api/v1/<resource>`
- All timestamps stored as UTC
- Currency as integer (paisa/cents), never float
- User-facing strings via i18n, never hardcoded
- Tests live alongside source files

## Critical rules

- Patient identifiers (name, phone, NID) ONLY in operational schema
- Pharma queries ONLY hit analytics schema
- Anonymization happens in scheduled job, NOT at query time
- NEVER log patient data
- All passwords hashed with bcrypt
- Doctor accounts require admin verification before activation

## MVP scope (build in this order, one at a time)

1. Doctor signup, login, profile
2. Patient creation and search
3. Drug master + drug search autocomplete
4. Write a prescription (the core flow — must be fast)
5. View past prescriptions for a patient
6. Print prescription / generate PDF
7. Anonymization scheduled job
8. Pharma login + dashboard
9. Pharma analytics: top prescribed drugs by therapy area
10. Admin: verify doctors, manage drugs, manage pharma clients

## Out of scope for MVP — DO NOT build

- Payment processing
- Native mobile apps (PWA is enough)
- Telemedicine / video
- Pharmacy POS integration
- Patient-facing app
- ML / predictive analytics
- Languages beyond Bangla + English

## What to do when uncertain

- Ask before big architectural decisions
- Prefer boring, well-documented libraries over trendy ones
- Optimize for readability over cleverness
- If a request conflicts with these rules, flag it before proceeding
- When in doubt, do less

## Definition of done

A feature is done when:
1. Backend endpoint(s) exist with type-safe input/output
2. Frontend page(s) call the endpoint(s)
3. At least one happy-path test exists
4. Works end-to-end in a local browser
5. CLAUDE.md updated if conventions changed

---
> Source: [mahfujars/prescryb](https://github.com/mahfujars/prescryb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
