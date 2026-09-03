## obraims-mvp-main

> Obraims is an AI-assisted lending MVP built with:

# Obraims Agent Instructions

## Project

Obraims is an AI-assisted lending MVP built with:

- Next.js
- Supabase
- Vercel
- TypeScript

The goal is to build a simple, clean, production-oriented lending workflow, not an overengineered enterprise platform.

## Core Rule

Use the existing **Obraims Simple Core v1** schema as the source of truth.

Do not create a parallel database model.

Do not rename existing tables or columns unless explicitly requested.

Do not duplicate existing concepts.

If a feature can fit into the existing schema, use the existing schema.

If a required field is missing, prefer using `metadata jsonb` first before adding a new column.

Only add a migration if absolutely necessary.

Before adding a table or column, explain why it is necessary.

## Source-of-Truth Tables

Obraims Simple Core v1 uses these tables:

- `customers`
- `loan_applications`
- `documents`
- `consents`
- `decisions`
- `decision_reasons`
- `loan_offers`
- `ai_conversations`
- `audit_events`

## Table Meaning

### `customers`

Represents the borrower or applicant.

Use this for:

- Full name
- Phone
- Email
- Register number
- Customer metadata

Do not create a separate `users` table for borrower profile data.

Auth users should connect through `customers.auth_user_id`.

### `loan_applications`

Represents one loan request from a customer.

Use this for:

- Requested amount
- Requested term
- Loan purpose
- Monthly income
- Employment status
- Application status
- Channel/source

Do not create a separate `applications` table.

### `documents`

Represents uploaded files.

Use this for:

- ID documents
- Salary proof
- Bank statements
- Other supporting files

Do not create separate document tables unless explicitly requested.

### `consents`

Represents customer consent.

Use this for:

- Data processing consent
- Credit assessment consent
- Document collection consent
- Contact permission

### `decisions`

Represents approval, rejection, referral, or counteroffer decision.

Use this for:

- Decision result
- Decision summary
- Approved amount
- Approved term
- Interest rate

Do not store final decision only as a status on `loan_applications`.

### `decision_reasons`

Represents explainable reasons behind a decision.

Use this for:

- Rejection reasons
- Approval reasons
- Manual review reasons

### `loan_offers`

Represents approved terms shown to the customer.

Use this for:

- Offered amount
- Term
- Interest rate
- Monthly payment
- Offer status

Do not create a separate `offers` table.

### `ai_conversations`

Represents chat or AI-assisted intake conversations.

Use this for:

- Conversation transcript
- Extracted intent
- Extracted entities
- Link to customer/application

Do not create a separate `messages` table yet unless explicitly requested.

### `audit_events`

Represents system history and traceability.

Use this for:

- Customer created
- Consent granted
- Application created
- Application submitted
- Decision created
- Offer generated
- Offer accepted
- Admin actions
- AI actions

Do not create separate `logs`, `activity_logs`, or `events` tables.

## Required Audit Events

Important actions must write to `audit_events`.

Required actions include:

- `customer.created`
- `consent.granted`
- `loan_application.created`
- `loan_application.submitted`
- `decision.created`
- `loan_offer.created`
- `loan_offer.accepted`
- `ai_conversation.created`
- `ai_conversation.updated`

## Backend Helper Source

Use existing helper functions in:

```text
src/lib/obraims/simple-core.ts
```

Add new helper functions there when they operate on the Simple Core v1 schema.

Do not scatter duplicate Supabase logic across random components.

## Mock Decision Logic

Mock decision logic should live in:

```text
src/lib/obraims/mock-decision.ts
```

This is for MVP/demo only.

It is not production underwriting.

Do not represent mock scoring as real credit scoring.

## Security Rules

Never expose the Supabase service role key in client components.

Privileged operations must run server-side.

Admin actions must run server-side.

AI agents should not write arbitrary database rows directly.

AI should operate through controlled helper functions or server actions.

## Current MVP Flow

The current core flow is:

1. User applies through `/app/apply`
2. System creates or finds `customer`
3. System creates `consent`
4. System creates `loan_application`
5. System submits the application
6. Admin reviews application in `/app/admin/applications`
7. Admin or mock logic creates `decision`
8. If approved, system creates `loan_offer`
9. User views status and offer in `/app/application/[id]`
10. Important actions write to `audit_events`

## Do Not Build Yet

Do not implement these unless explicitly requested:

- Real KYC integration
- AML checks
- Credit bureau integration
- Bank statement parsing
- Payments
- Loan contracts
- Repayment schedules
- Collections
- Complex risk scoring
- Multi-lender marketplace
- Policy engine

## Product Principle

Obraims should remain simple.

The goal is not to build Palantir.

The goal is to keep a clean lending data backbone that supports:

- AI-assisted loan intake
- Admin review
- Simple decisions
- Offers
- Auditability
- Future expansion

---
> Source: [Obraims/obraims-mvp-main](https://github.com/Obraims/obraims-mvp-main) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
