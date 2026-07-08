## openscirank

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Editorial Standards Platform (formerly OpenSciRank) — a platform for technical evaluation and visibility of scientific journals and academic books. Journals are evaluated against weighted criteria, scored, and may earn an "Editorial Standards Seal" (1-year validity). Books are indexed for a fee.

The master business logic document is `business-logic.md` at the project root.

## Development Environment

This project uses **Laravel Sail** (Docker). All commands must run through Sail:

```bash
./vendor/bin/sail up -d                    # Start containers (app on port 5000)
./vendor/bin/sail artisan migrate          # Run migrations
./vendor/bin/sail artisan db:seed          # Seed all (Admin, Categories, Criteria, Products)
./vendor/bin/sail npm run dev              # Vite dev server with HMR
./vendor/bin/sail artisan test             # Run PHPUnit tests
./vendor/bin/sail artisan test --filter=AuthenticationTest  # Single test
./vendor/bin/sail composer test:lint       # Pint linter
```

**Key URLs in development:**
- App: `http://localhost:5000`
- Filament Admin: `http://localhost:5000/admin`
- Mailpit (email testing): `http://localhost:8025`

**Admin credentials:** `admin@editorialstandards.com` / `password` (seeded by `AdminUserSeeder`, role `super_admin`). No hay usuario "editor" de prueba seedeado — registrar uno vía `/register` para probar flujos de editor.

## Architecture

**Stack:** Laravel 12, Livewire 4, Filament v5, Tailwind CSS 4, Stripe, MySQL, Sail

### Status Machine (Journals)

```
draft → [pay $99] → submitted → [admin evaluates] → evaluated / certified / rejected
                                                    ↘ requires_changes_evaluation → [user corrects, FREE] → submitted
       → [free]   → pending_listing → [admin reviews] → listed / rejected
                                                       ↘ requires_changes_listing → [user corrects, FREE] → pending_listing
listed → [pay $99] → submitted (evaluation flow)
evaluated / certified / rejected → [pay $99 reevaluation] → submitted
certified → seal_status: active → expiring_soon (30d) → expired → status reverts to evaluated
```

Resubmisión tras `requires_changes_*` es **gratuita** (business-logic 16.5). Cada nueva evaluación o re-evaluación sí tiene costo asociado.

Books: `draft → [pay $49] → submitted → pending_listing → listed / rejected / requires_changes_listing → [free resubmit] → pending_listing`

### Scoring Algorithm

`Journal::calculateScore()` — weighted sum of CriteriaItems. If ANY core indicator (`is_core=true`) fails, score is capped at 49%. Seal requires ≥75% AND all 5 critical indicators met. Score shown as percentage (0-100%), no letter levels.

### Payment Flow (Stripe)

1. User selects plan + addons in `PaymentCheckout` Livewire component
2. `StripePaymentService::createCheckoutSession()` creates Stripe session
3. Stripe redirects to `CheckoutSuccessController` (sync verification)
4. `StripeWebhookController` handles `checkout.session.completed` (async)
5. Payment record created, journal/book status updated to `submitted`
6. `PaymentConfirmed` notification sent

Products identified by `slug`: `journal-evaluation`, `journal-reevaluation`, `seal-renewal-1y`, `seal-renewal-2y`, `seal-renewal-3y`, `book-listing`, `book-listing-featured-1y`, `action-plan-consulting`, `new-journal-consulting`. Express service is no longer a public SKU — it is a +$50 uplift toggled at checkout for evaluation/re-evaluation flows. Legacy slugs kept inactive for FK integrity: `express-evaluation`, `institutional-pack`. The old `premium-report` slug was renamed to `action-plan-consulting` (roadmap #17, 2026-05-13).

**Consulting products:** Two SKUs, both create `AdminTask` of type `consulting`.
- `action-plan-consulting` (USD 215): add-on to a journal evaluation. `Payment.payable_type=Journal`. 1 session.
- `new-journal-consulting` (USD 1,500): standalone product for editors creating a new journal. `Payment.payable_type=User` (no journal exists yet). 3 sessions package + domain + OJS hosting for 12 months. "Pack Lanzamiento Editorial".

**Consulting scheduling flow (Sprint 3.6 #39, 2026-05-14):**
1. Payment confirmed → `AdminTask` created (status `pending`) + `ConsultingPaymentConfirmed` email to editor.
2. Evaluator action "Proponer fechas" → 1-3 candidate slots → status `proposal_sent` → email to editor with signed accept-URLs.
3. Editor in `/app/consulting` accepts one slot → status `scheduled` + `.ics` attachment + both parties emailed.
4. Cron `consulting:send-reminders` 24h before sends `ConsultingReminder`.
5. Cron `consulting:expire-proposals` daily expires unanswered proposals after 5 business days.
6. Cancellation policy: >48h reagenda libre, 24-48h una sola vez, <24h pierde la sesión. Override por super_admin con motivo. Tracked en `admin_tasks.reschedule_count` (cap 3 rondas).

**Universal messaging (Sprint 3.6 #40):** Polymorphic `conversations` over Journal/Book/AdminTask/null (general). `messages` with `message_attachments` (max 10 MB, types: PDF/DOCX/XLSX/images/TXT/CSV). Auto-join admin when responding. Read state tracked per `conversation_participants.last_read_at`. Email notification per message with 5-min batching cooldown.

### Coupons / Discount system (Sprint 3 #13)

Coupons are stored in the local `coupons` table (`App\Models\Coupon`) **and must also exist in Stripe** with the same ID (the `code` column doubles as the Stripe coupon ID via the `stripe_id` accessor). The local row controls business logic (window, eligibility, activity); Stripe applies the actual discount to the checkout session.

**Why two sources of truth:** Stripe needs the coupon to validate at checkout (otherwise the session creation fails). The app needs its own row to decide when to apply the coupon, track usage, and disable it without touching Stripe. Coupons are NEVER created from code on Stripe — admin creates them manually in the Stripe Dashboard to avoid duplicates in production.

**Active coupons:**

| Code | Discount | Auto-applied when | Eligible products |
|---|---|---|---|
| `RENEW_EARLY_10` | 10% off | Journal `seal_expires_at` is between 30 and 60 days away (inclusive), seal not yet expired | Slug starts with `seal-renewal-` |

**Auto-apply logic** lives in `App\Livewire\PaymentCheckout`:
- `getIsInEarlyRenewalWindowProperty()` — Carbon window check on `seal_expires_at` with `startOfDay()` on both sides.
- `selectedIsRenewalProduct()` — true when current plan slug starts with `seal-renewal-`.
- `getAppliedCouponCodeProperty()` — precedence: manual code (`$manualCouponCode`) > auto-apply > none.
- `getEarlyDiscountDeadlineProperty()` — `seal_expires_at - 30 days`, shown in UI and in `SealExpiringEarlyReminder` email.

**Manual coupons** always win: if the editor pastes any code into the input, the auto-apply is skipped (even if the manual code is invalid — Stripe rejects it during session creation, and the editor sees the error).

**Flow at checkout:**
1. `PaymentCheckout` resolves `appliedCouponCode` (manual > auto > null).
2. Passes it as `couponCode` to `StripePaymentService::createCheckoutSession()`, which adds `discounts: [{ coupon: code }]` to the Stripe session params.
3. Stripe validates the coupon ID exists in its catalog. If not → session creation fails → error flashed to editor.
4. Editor pays the discounted total in Stripe Checkout.
5. Webhook creates the `Payment` record at full pre-discount product price minus discount (Stripe sends `amount_total` net of discount in cents).

**To add a new coupon:**
1. Create it in Stripe Dashboard (Coupons → New). Note the ID.
2. Insert a row in `coupons` table or extend `CouponSeeder` using the same ID as `code`.
3. If it should auto-apply under some condition, extend `PaymentCheckout::getAppliedCouponCodeProperty()` with the new rule (don't add ad-hoc logic in views).

### Notifications (app/Notifications/)

All notifications are **synchronous** (no ShouldQueue) — they send immediately via SMTP. In dev, Mailpit captures all emails at localhost:8025.

Triggered from: `CheckoutSuccessController`, `EvaluateJournal::save()`, `ReviewListing::save()`, `JournalResource` (assign evaluator, notify seal).

### Key Patterns

- **Polymorphic payments:** `Payment.payable_type` → Journal or Book
- **Soft deletes:** Journal and Book models
- **Livewire multi-step wizards:** `SubmissionWizard` (7 steps), `BookSubmissionWizard` (6 steps), auto-save drafts
- **Filament custom pages:** `EvaluateJournal`, `ReviewListing` — not standard CRUD, use `InteractsWithRecord`
- **OAI-PMH harvesting:** `OaiPmhService` with resumption tokens, supports `oai_dc` and `marcxml`

## Key Files

| Area | Location |
|------|----------|
| Models | `app/Models/` — Journal.php is the central model (~80 fillable fields) |
| Livewire | `app/Livewire/` — SubmissionWizard, PaymentCheckout, EditorDashboard, SearchJournals |
| Filament | `app/Filament/Resources/` — JournalResource (with EvaluateJournal, ReviewListing pages) |
| Services | `app/Services/` — StripePaymentService, OaiPmhService |
| Controllers | `app/Http/Controllers/` — CheckoutSuccessController, StripeWebhookController |
| Notifications | `app/Notifications/` — 8 notification classes |
| Routes | `routes/web.php` — all routes; webhook at POST `/stripe/webhook` (CSRF excluded) |
| Seeders | `database/seeders/` — ProductSeeder (7 products), CriteriaItemSeeder (18 indicators in 5 categories) |

## Conventions

- **Language:** UI and code comments in Spanish. Class/method names in English.
- **Branding:** System name is "Editorial Standards Platform" — never expose "Laravel" or "OpenSciRank" in user-facing content.
- **Environment config:** Stripe keys in `.env` (`STRIPE_KEY`, `STRIPE_SECRET`, `STRIPE_WEBHOOK_SECRET`). Mail via Mailpit locally, Amazon SES in production.
- **Confirmation modals:** All destructive or state-changing user actions must have `wire:confirm` or `onclick="return confirm(...)"`.
- **Seal management:** Admin-driven via Filament (individual + bulk actions). No cron — admin manually notifies via "Notificar" button.

## Git Workflow

**Standing instruction from user (2026-05-15):** Por cada tarea del roadmap completada, crear un commit con el título de la tarea.

**Reglas:**

- Cuando una tarea del roadmap (`memory/project_admin_roadmap.md`) queda completamente terminada (código + tests si aplican + verificación end-to-end), Claude DEBE crear un commit antes de pasar a la siguiente tarea, sin esperar a que el usuario lo pida.
- **Formato del título del commit:** `[#N] Título corto de la tarea` donde `N` es el ID del roadmap. Ejemplos:
  - `[#46] Producto Soporte $55 — bajo pedido del admin`
  - `[#47] Comando mail:health-check para auditoría de deliverability`
  - `[#37] Fix: resubmisión no genera admin_task`
- **Cuerpo del commit:** 1-3 líneas resumiendo qué se cambió. Si tocaron varios archivos significativos, listarlos brevemente.
- **Co-autoría:** incluir el trailer `Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>` al final.
- **Una tarea = un commit ideal.** Si la tarea fue extensa y se hicieron commits intermedios durante el trabajo, está bien. Lo importante es que el último commit antes de cerrar la tarea tenga el `[#N]` que la cierra.
- **Si la tarea queda incompleta** (decisión pendiente, código a medio camino, tests fallando): NO commitear con `[#N]` — usar título descriptivo + `WIP` para que quede claro.
- **Si fueron varios cambios cosméticos no atados a una tarea del roadmap** (ej. fixes durante QA, tweaks de UI): commit normal sin `[#N]`, título descriptivo.
- **Pre-commit hooks:** si fallan, fijar el problema y crear un commit nuevo (no `--amend` salvo que el usuario lo pida explícito).
- **NO push automático.** El push lo decide el usuario.

**Excepciones donde NO commitear automáticamente:**
- Tareas exploratorias sin output de código (research, comparativas, planning).
- Cambios solo en archivos de configuración local del usuario (`.claude.json`, `.mcp.json`).
- Edits a archivos fuera del proyecto (memory files de Claude en `C:\Users\...\.claude\`).

---
> Source: [aldorodrigo/OpenSciRank](https://github.com/aldorodrigo/OpenSciRank) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
