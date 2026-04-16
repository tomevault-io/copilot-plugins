## finance-agent

> This is a headless personal finance and wealth management application built for a single household (multi-tenant via `Family` boundary). It features an AI-assisted document ingestion pipeline for bank statements and enforces strict double-entry accounting principles.

# Project Overview
This is a headless personal finance and wealth management application built for a single household (multi-tenant via `Family` boundary). It features an AI-assisted document ingestion pipeline for bank statements and enforces strict double-entry accounting principles.

# Project Structure
```
finance_agent/
├── manage.py
├── requirements.txt
├── finance_backend/          # Django project config
│   ├── api.py                # Root API router
│   ├── urls.py
│   ├── celery.py
│   ├── settings/
│   │   ├── base.py
│   │   ├── local.py
│   │   └── remote.py
│   └── utils/
│       └── time_value.py
├── users/                    # Auth, Family, FamilyMember models
├── accounting/               # Chart of Accounts, JournalEntry, TransactionLine (django-mptt)
├── banking/                  # BankStatementImport, StagedTransaction, FinancialProduct
│   ├── extraction.py         # PDF parsing logic (tabula-py, pandas)
│   ├── services.py           # Ingestion pipeline orchestration
│   ├── mappers.py
│   └── tasks.py              # Celery async tasks
├── categorization/           # Merchant, TransactionMappingRule, auto-categorization
│   └── services.py
├── ai_core/                  # AI extraction strategies
│   └── extractors/
│       ├── factory.py
│       ├── strategies.py
│       └── base.py
└── frontend/                 # React 18 + TypeScript + Vite
    └── src/
        ├── pages/            # Dashboard, FamilyManager, MerchantManager, AccountDetail, etc.
        ├── components/       # Reusable UI components (Shadcn UI patterns)
        ├── context/          # AuthContext
        └── api/              # client.ts — API calls to backend
```

# Tech Stack
## Backend
- Python 3.x, Django 4.2
- **API:** Django Ninja (strictly preferred — never use DRF)
- **Auth:** ninja-jwt (JWT tokens)
- **Hierarchical data:** django-mptt (Chart of Accounts)
- **PDF parsing:** tabula-py, pandas
- **Async tasks:** Celery
- **DB:** PostgreSQL

## Frontend
- React 18, TypeScript, Vite (runs on `localhost:5173`)
- Styling: Tailwind CSS, CSS Modules
- UI Components: Shadcn UI (lucide-react, clsx, tailwind-merge)

# Dev Commands
## Backend
```bash
python manage.py runserver           # Start dev server
python manage.py test                # Run tests
python manage.py verify_ledger_integrity   # Validate accounting integrity
python manage.py rebuild_accounting_master # Rebuild chart of accounts
```

## Frontend
```bash
cd frontend
npm run dev      # Starts on localhost:5173
npm run build
```

# Coding Conventions

## Backend — Hard Rules (Never Violate)
1. **API endpoints** must use Django Ninja (`@api.get`, `@api.post`) with Pydantic schemas. Never use DRF or standard Django views for API work.
2. **Multi-tenancy** — all queries must be scoped to the `Family` model. Never leak data across families.
3. **Double-entry accounting** — all `JournalEntry` mutations must use `transaction.atomic()`. The sum of all `TransactionLine` amounts must always equal zero.
4. **Financial amounts** — always use `Decimal` from the `decimal` module. Never use floats.
5. **Business logic** belongs in service layers (`services.py`) or model methods, never in API endpoints.

## Frontend — Hard Rules
1. Use functional components and React Hooks exclusively — no class components.
2. All props, state, and API responses must have strict TypeScript interfaces/types.
3. Use Tailwind CSS utility classes for styling; default to Shadcn UI patterns for new UI elements.

# Domain Knowledge
- **Ingestion Pipeline:** PDFs → `BankStatementImport` → parsed into `StagedTransaction` (status: `PENDING_REVIEW`) → manually reconciled into `JournalEntry`
- **Categorization:** `StagedTransaction` records are matched to `Merchant` via `TransactionMappingRule`. Auto-categorization runs through `categorization/services.py`.
- **Demographics:** The system tracks Canadian statutory milestones (RESP deadlines, TFSA eligibility, CEGEP start dates, QPP eligibility) based on `FamilyMember` age and role.
- **AI Extraction:** `ai_core/extractors/` uses a strategy pattern (`factory.py` → `strategies.py`) for bank-specific PDF extraction logic.

# Maintenance Note
When you add new Django apps, models, services, or frontend pages, update the Project Structure section above to reflect the change.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LPDavidbdeb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
