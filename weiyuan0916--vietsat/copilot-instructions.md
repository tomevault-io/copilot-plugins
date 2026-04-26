## vietsat

> 🧠 CURSOR PROMPT — SINGLE PLAN VERSION


🧠 CURSOR PROMPT — SINGLE PLAN VERSION

You are a senior full-stack engineer.

Your task is to IMPLEMENT a production-ready system using the constraints below.
This is PHASE 1, with ONLY ONE DEFAULT SERVICE PLAN.

Do NOT introduce multiple plans yet, but the system MUST be extensible for future plans without refactoring core logic.

1. FIXED STACK (DO NOT CHANGE)
Frontend

Framework7 (PWA, SPA)

Served at /app

No Blade

Fetch API + WebSocket

No hardcoded price logic in frontend

Backend

Laravel 12

API-only

Laravel Reverb

MySQL

2. BUSINESS RULE (PHASE 1)

There is ONLY ONE SERVICE PLAN:

Name	Duration	Price
Default Plan	3 months	100,000 VND

Frontend:

Display ONLY ONE PLAN

No plan selector UI

Price and duration are fetched from backend

Backend:

Service plan is stored in database

Created automatically via database seeder

All orders must reference this plan

3. FRONTEND (Framework7) REQUIREMENTS
UI

Display:

Service name

Duration (3 months)

Price (100k)

Input:

Facebook profile link

Button:

“Thanh toán & kích hoạt”

Flow

Load service config from /api/service/default

Render UI dynamically

Submit order request

Show QR + transfer content

Listen realtime payment status via WebSocket

Frontend must:

NOT hardcode price

NOT hardcode duration

NOT calculate expiration

4. BACKEND API REQUIREMENTS
Endpoint: Get default service
GET /api/service/default


Returns:

{
  "id": 1,
  "name": "Default Plan",
  "duration_days": 90,
  "price": 100000
}

Endpoint: Create order
POST /api/orders


Input:

{
  "facebook_profile_link": "https://facebook.com/..."
}


Backend:

Fetch default service

Generate order_code

Set expires_at = now + 5 minutes

Status = pending

Generate QR content

5. DATABASE REQUIREMENTS
services

id

name

duration_days

price

is_active

Seeder:

Auto create ONE default service

Must be idempotent

orders

id

order_code (unique)

service_id

amount

status

expires_at

paid_at

bank_txn_id (unique)

6. REALTIME REQUIREMENTS

Use Laravel Reverb

Private channel per order_code

Events:

payment.pending

payment.success

payment.expired

Frontend subscribes only after order creation

7. SECURITY REQUIREMENTS

No frontend price trust

No frontend duration trust

Order amount comes ONLY from backend

One order → one payment

Order cannot be reused

8. EXTENSIBILITY REQUIREMENT (VERY IMPORTANT)

Even though only ONE plan exists:

Code must support future:

Multiple plans

Different durations

Different limits

DO NOT:

Hardcode plan id in controller

Hardcode price in frontend

Use magic numbers

9. DELIVERABLES

You must generate:

Database migrations

Seeder for default service

API controllers

Reverb events

Frontend API integration example

Clean folder structure

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/weiyuan0916) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
