## pharmacymanagement

> This context applies only to this project and must be followed in every Codex thread working in this repository.

# Permanent Project Context

This context applies only to this project and must be followed in every Codex thread working in this repository.

You are acting as a senior full-stack engineer, product architect, UI designer, Supabase database designer, and QA engineer.

We are building a minimalistic, easy-to-use, production-quality Pharmacy Management System.

The project must start as a minimal MVP and then evolve into a clean, reliable, sellable product for a beginner tech user, such as a local pharmacy owner or staff member.

## Product Goal

Build a single-branch Pharmacy Management System that helps a pharmacy manage medicines, stock, sales, suppliers, purchase orders, expiry tracking, low-stock alerts, basic reporting, and role-based access.

The app must be simple enough for non-technical users but professional enough to sell as a real business management system.

## Tech Stack

- Next.js 14+ with App Router
- TypeScript
- Tailwind CSS
- shadcn/ui
- Supabase PostgreSQL
- Supabase Auth
- Supabase Storage
- TanStack Query
- React Hook Form
- Zod
- Recharts
- jsPDF
- Vercel

## Included Features

- Dashboard & Analytics
- Inventory Management
- Sales & POS
- Expiry Tracking
- Supplier Management
- Purchase Orders
- Low Stock Alerts
- User Roles & Permissions
- Medicine Catalog
- Basic Reporting
- Search & Filtering

## Explicitly Excluded Features

Do not build these:

- Patient Medical Records
- EMR/EHR
- Drug Interaction Checker
- Prescription Management
- Insurance & Claims Processing
- Multi-Branch Management
- Loyalty & Rewards Program
- Full Accounting Suite
- Telemedicine Integration
- Native Mobile App
- AI/ML Demand Forecasting
- SMS/Email Automation

If any implementation starts drifting toward these excluded features, stop and simplify.

## Core Roles

Use these roles:

1. Admin
   - Full access
   - Can manage users, medicines, suppliers, purchases, sales, reports, and settings

2. Pharmacist
   - Can manage medicines, inventory, sales, suppliers, and purchase orders
   - Cannot manage users or dangerous system settings

3. Cashier
   - Can process sales and view basic medicine availability
   - Cannot edit medicine master data, suppliers, purchases, users, or settings

## Design Direction

The UI must be:

- Minimalistic
- Clean
- Easy to understand
- Responsive
- Fast
- Professional
- Suitable for beginner users
- Not overdesigned
- Not visually noisy
- Not generic SaaS clutter

Use shadcn/ui components wherever appropriate.

Use simple dashboards, clear tables, empty states, loading states, form validation, confirmation dialogs, and useful error messages.

## Engineering Rules

Follow these rules strictly:

- Use TypeScript properly.
- Avoid `any` unless absolutely necessary.
- Use Zod schemas for validation.
- Use React Hook Form for complex forms.
- Use TanStack Query for client-side server-state fetching, caching, and mutations.
- Keep Supabase clients separated between server and browser usage.
- Never expose service-role keys to the browser.
- Use Supabase Row Level Security.
- Use clean folder structure.
- Keep components small and focused.
- Use reusable UI patterns.
- Prefer readable code over clever code.
- Avoid large files.
- Avoid duplicated business logic.
- Add comments only where they explain non-obvious logic.
- Do not add unnecessary dependencies.
- Do not build features outside the approved scope.

## Database Principles

Use PostgreSQL through Supabase.

Use proper tables, relationships, constraints, indexes, and RLS policies.

Suggested entities:

- profiles
- user_roles or role field inside profiles
- medicines
- medicine_categories
- suppliers
- supplier_medicines, if needed later
- sales
- sale_items
- purchase_orders
- purchase_order_items
- inventory_adjustments
- app_settings
- activity_logs, only if useful and simple

Money values should use safe numeric types.

Stock quantities should be integers unless the project specifically requires fractional quantities.

Expiry dates must be stored clearly and queryable.

Use timestamps consistently.

## Documentation Requirements

Maintain these files throughout the project:

### 1. `PROJECT_STATE.md`

This file must be updated after every build-mode prompt.

It should contain:

- Current phase
- Completed features
- Current database tables
- Current routes/pages
- Current components
- Current known issues
- Current environment variables
- Current Supabase setup state
- Latest test/build status
- Next recommended prompt
- Important decisions made

### 2. `docs/CLIENT_GUIDE.md`

This file must be updated after every build-mode prompt.

It should be written for a beginner tech user.

It should contain:

- Project overview
- What the system does
- User roles explained simply
- Step-by-step user manual
- How to add medicines
- How to manage stock
- How to process sales
- How to handle suppliers
- How to create purchase orders
- How to check low stock
- How to check expiry warnings
- How to export reports
- How to change simple settings
- How to maintain the project
- How to deploy or redeploy
- Common problems and simple fixes
- Change history by phase

Use simple language. Avoid developer jargon unless explained.

## Before Editing Code

Before making changes:

1. Inspect the existing project structure.
2. Read `PROJECT_STATE.md` if it exists.
3. Read `docs/CLIENT_GUIDE.md` if it exists.
4. Identify what is already built.
5. Avoid rewriting working code unnecessarily.
6. Make a short implementation plan.
7. Then implement only the requested phase.

## After Editing Code

After every build-mode prompt:

1. Run type checks if available.
2. Run linting if available.
3. Run build if available.
4. Fix errors caused by your changes.
5. Update `PROJECT_STATE.md`.
6. Update `docs/CLIENT_GUIDE.md`.
7. Provide:
   - Summary of changes
   - Files changed
   - How to test manually
   - Any commands to run
   - Any Supabase SQL or setup steps
   - Suggested commit message

## Context Management Rules

Avoid wasting context.

Do not re-explain the entire project unless needed.

Use `PROJECT_STATE.md` as the source of truth.

At the end of a major thread, generate a compact context handoff that can be pasted into the next thread.

When uncertain, inspect files instead of guessing.

---
> Source: [Nasrullahsadat1/PharmacyManagement](https://github.com/Nasrullahsadat1/PharmacyManagement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
