## choir-signup-backend

> Handles two event types:

# Some Voices Choir Signup Backend - Claude Context

## Project Overview
Next.js API backend for the Some Voices choir membership signup flow. Handles Airtable record creation, Stripe checkout/subscription creation, and webhook processing. The frontend is an HTML form embedded on the Squarespace site (somevoices.co.uk).

**Backend URL:** https://choir-signup-backend-atuj.vercel.app
**Website:** https://somevoices.co.uk
**Deployment:** Vercel (auto-deploys on push to main)

---

## Tech Stack
- **Framework:** Next.js (Pages Router, API routes only)
- **Database:** Airtable (via `airtable` npm package)
- **Payments:** Stripe (`stripe` npm package)
- **Frontend:** Static HTML form (`index.html`) embedded in Squarespace via code injection

---

## Signup Flow (Active)

The current active flow uses **subscription-first checkout** via `create-subscription-first.js`:

```
1. User enters email on Squarespace form
   → POST /api/lookup-email (checks Members table for existing member)

2. User fills form (name, choir, voice part, billing date, discount code)
   → POST /api/create-airtable-record-step1 (creates Signup Queue record)

3. Frontend calls POST /api/create-subscription-first
   → Reads Signup Queue record for pro-rata amount, price ID, currency
   → Creates/finds Stripe customer
   → Creates Stripe Checkout session (subscription mode) with:
     - Recurring subscription line item (price ID from Airtable)
     - One-time initial payment line item (pro-rata amount from Airtable)
     - trial_end set to billing anchor date (defers first subscription charge)
     - Discount/coupon if provided

4. User completes payment on Stripe Checkout
   → Redirected to somevoices.co.uk/success
```

### Legacy Flow (create-initial-checkout → create-success-subscription)
An older two-step flow exists but is **not called by the current frontend**:
- `create-initial-checkout.js` — payment-mode checkout for pro-rata only
- `create-success-subscription.js` — called on success URL redirect, creates subscription separately
- Both files still contain the `getBillingAnchorTimestamp` function and should be kept in sync

---

## Critical: Billing Anchor & Trial Period Logic

### How It Works

Two independent questions are answered at checkout:

**Question 1: How much does the customer pay today?** (initial payment — calculated by Airtable)

```
{week 1 date}                          ← When does this term start?
        ↓
{Billing Anchor Rehearsals Left         ← Is this a new-term signup?
 Multiplier}                              If yes → 0 (no pro-rata)
        ↓                                If no → how many rehearsals
{Rehearsals to 1st} or                    remain before the anchor?
{Rehearsals to 15th}                      0=0, 1=0.25, 2=0.5, 3=0.75, 4+=1
        ↓
{Total Cost Initial Invoice}           ← Monthly price × multiplier
                                          + 100 pence (£1 activation)
                                          (minus discount if applicable)
```

**Question 2: When does the first subscription invoice land?** (trial_end — calculated by Airtable + code)

```
{Next Rehearsal Date (from Choir)}      ← Raw lookup: date or "Not yet booked"
        ↓
{Next Rehearsal Date}                   ← Parsed to YYYY-MM-DD (or "" if invalid)
        ↓
{Next Billing Date (Before Push)}       ← Next 1st or 15th based on billing anchor
        ↓
{Days Until Billing}                    ← DATETIME_DIFF to TODAY() in days
        ↓
{Will Push to Next Month}              ← < 3 days? → "Yes" (too close for Stripe)
        ↓
{Skip Next Month}                      ← Also "Yes" if date is blank/out of range
        ↓
getBillingAnchorTimestamp() [CODE]      ← Calculates the trial_end timestamp
                                          If Skip Next Month = Yes → +1 month
                                          If still < 48hrs → +1 more month
```

**The golden rule:** Both paths must agree. If the code pushes the first invoice further out, Airtable must charge enough in the initial payment to cover that period. The `< 3` day threshold in Airtable and the 48-hour check in code are aligned to ensure this.

### The 48-Hour Edge Case (Fixed April 2026)
**Stripe requires `trial_end` to be at least 48 hours in the future.**

When a customer signs up close to their billing anchor (e.g. April 30th with anchor = 1st), the trial would be < 48 hours and Stripe would reject the API call.

**Code fix** (`getBillingAnchorTimestamp` in both `create-subscription-first.js` and `create-success-subscription.js`):
```javascript
// After computing billingDate...
const minTrialEnd = new Date(now.getTime() + 48 * 60 * 60 * 1000);
if (billingDate < minTrialEnd) {
  billingDate = new Date(billingDate.getFullYear(), billingDate.getMonth() + 1, dayOfMonth);
}
```

**Airtable alignment** — the `Will Push to Next Month` formula must use `< 3` (not `< 2`):
```
IF({Days Until Billing} < 3, "Yes", "No")
```
This is because `Days Until Billing` uses `DATETIME_DIFF(..., TODAY(), 'days')` where `TODAY()` is midnight — 2 calendar days could be as few as 24 actual hours depending on checkout time.

**Both systems must agree:** if the code bumps the anchor to next month, Airtable's pro-rata must also cover that extra period. The `< 3` threshold ensures Airtable always charges the extra month whenever the code might bump.

### Key Airtable Fields (Signup Queue table)
| Field | Purpose |
|-------|---------|
| `Billing Anchor` | "1" or "15" — user's chosen billing date |
| `Skip Next Month` | "Yes"/"No" — set by `Will Push to Next Month` formula |
| `Days Until Billing` | `DATETIME_DIFF({Next Billing Date (Before Push)}, TODAY(), 'days')` |
| `Will Push to Next Month` | `IF({Days Until Billing} < 3, "Yes", "No")` |
| `Billing Anchor Rehearsals Left Multiplier` | 0.25/0.5/0.75/1 based on rehearsals remaining |
| `Total Cost Initial Invoice` | Pro-rata amount in pence (includes +100 for £1 activation fee) |
| `Stripe PRICE_ID` | Recurring subscription price ID (lookup from Choir) |
| `Stripe Customer ID` | Stripe customer ID (created or found during checkout) |
| `Initial Payment Description` | Description shown on Stripe invoice |

---

## API Routes

### Active (used by current frontend)
| Route | Method | Purpose |
|-------|--------|---------|
| `/api/lookup-email` | POST | Check if email exists in Members table, return existing details |
| `/api/get-choirs` | GET | Fetch choirs from `Choirs MASTER` table (view: `Choir CURRENT (SqSp Signup)`) |
| `/api/get-voice-parts` | GET | Fetch voice parts from `Voice Parts` table |
| `/api/check-discount-code` | POST | Validate discount code exists in `Discount Codes` table |
| `/api/create-airtable-record-step1` | POST | Create record in `Signup Queue` table (links discount code by record ID) |
| `/api/create-subscription-first` | POST | Create Stripe Checkout session (subscription mode with initial payment) |
| `/api/stripe-webhook` | POST | Handle `checkout.session.completed` and `invoice.created` events |

### Supporting / Legacy
| Route | Method | Purpose |
|-------|--------|---------|
| `/api/create-initial-checkout` | POST | Legacy: payment-mode checkout for pro-rata only |
| `/api/create-success-subscription` | GET | Legacy: creates subscription after initial payment success |
| `/api/initial-payment` | GET/POST | Legacy: charge initial payment after subscription creation |
| `/api/create-initial-invoice` | POST | Legacy: create invoice for initial payment |
| `/api/get-signup-record` | GET | Fetch Signup Queue record (used for polling readiness) |
| `/api/get-discount-id` | POST | Get Airtable record ID for a discount code |
| `/api/start-signup` | POST | Legacy: combined record creation + checkout |
| `/api/create-checkout-session` | POST | Legacy: unified checkout session |
| `/api/airtable-sqsp` | GET | Airtable proxy for Squarespace (uses separate API key `AIRTABLE_API_KEY_SQSP`) |

---

## Stripe Webhook (`stripe-webhook.js`)

Handles two event types:

### `checkout.session.completed` (payment mode only)
1. Updates Signup Queue with Stripe Customer ID and payment status
2. Creates subscription with `billing_cycle_anchor`
3. Updates/creates Customer Record in Airtable
4. Updates/creates Members record (links choir)
5. Writes webhook report to Signup Queue record

### `invoice.created`
1. Retrieves customer details from Stripe (name, phone, address)
2. Updates/creates Customer Record with contact details
3. Creates record in `Stripe Invoices` table
4. Updates/creates Members record

---

## Environment Variables
```
STRIPE_SECRET_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
AIRTABLE_API_KEY=pat...           # Main API key for signup tables
AIRTABLE_API_KEY_SQSP=pat...      # Separate key for Squarespace proxy
AIRTABLE_BASE_ID=app...           # Main base ID
APP_URL=https://choir-signup-backend-atuj.vercel.app
```

---

## Airtable Tables Referenced
| Table | Base | Used By |
|-------|------|---------|
| `Signup Queue` | Main | Record creation, checkout, webhook |
| `Members` | Main | Email lookup, member creation/update |
| `Customer Record` | Main | Stripe customer tracking |
| `Stripe Invoices` | Main | Invoice logging |
| `Choirs MASTER` | Main | Choir list for signup form |
| `Voice Parts` | Main | Voice part options |
| `Discount Codes` | Main | Discount validation |

---

## Frontend (`index.html`)

Static HTML form embedded in Squarespace. Key features:
- Email check step (pre-fills returning member details)
- Searchable choir dropdown
- EUR/GBP currency support (shows iDEAL info for EUR)
- Discount code validation
- Terms & Conditions scroll-to-agree
- Loading overlay during API calls
- Calls `create-subscription-first` flow (not the legacy flow)

**Note:** The frontend in this repo (`index.html`) may differ from what's actually embedded in Squarespace. The Squarespace version includes the searchable choir dropdown; the repo version has the basic `<select>`.

---

## Multi-Currency Support
- **GBP:** Card payments only
- **EUR:** Card + iDEAL on Stripe Checkout; recurring payments forced to card
- Currency determined by choir selection (from `Stripe 'default_price_data[currency]'` field)

---

## Git Workflow
- **Main Branch:** `main`
- **Deployment:** Automatic via Vercel on push to main

---

## Never Do This
- Change `Will Push to Next Month` threshold without also checking the 48-hour code logic in `getBillingAnchorTimestamp`
- Change `getBillingAnchorTimestamp` without checking Airtable's pro-rata formula alignment
- Modify `stripe-webhook.js` body parser config (must be `bodyParser: false` for signature verification)
- Commit environment variables or API keys
- Delete legacy API routes without checking if other systems still call them

---

## Session Log: April 1, 2026

### What We Fixed

**Stripe 48-Hour Trial Minimum Edge Case:**
- Customers signing up within 48 hours of their billing anchor (e.g. April 30th with anchor = 1st) were getting Stripe errors because `trial_end` was too close
- Added 48-hour safety check to `getBillingAnchorTimestamp()` in both `create-subscription-first.js` and `create-success-subscription.js`
- When computed anchor is < 48 hours away, bumps to next month's anchor date
- Aligned Airtable `Will Push to Next Month` formula from `< 2` to `< 3` days to match (because `Days Until Billing` uses `TODAY()` which is midnight, so 2 calendar days could be < 48 actual hours)

### Key Decision
| Decision | Reason |
|----------|--------|
| Fix in code, not Airtable | The 48-hour limit is a Stripe API constraint checked at checkout time; Airtable doesn't know what time the customer checks out |
| `< 3` in Airtable (not `< 2`) | `DATETIME_DIFF(..., TODAY(), 'days')` rounds to whole days from midnight; 2 days could be only 24 hours if checkout is late evening |
| Both systems must agree | Code bumps anchor → Airtable must charge extra month in pro-rata to prevent free month |

### Files Modified
- `pages/api/create-subscription-first.js` — Added 48-hour check to `getBillingAnchorTimestamp`
- `pages/api/create-success-subscription.js` — Same fix (duplicate function)

### Airtable Change (Manual)
- `Will Push to Next Month` formula: `IF({Days Until Billing} < 3, "Yes", "No")`

---

## Session Log: April 3, 2026

### What We Fixed

**500 Errors on Certain Choirs — Airtable `#ERROR!` Cascading to Stripe:**
- Some choirs caused `create-subscription-first` to return 500: `Invalid string: {:error=>"#ERROR!"}`
- Root cause: Airtable formula fields returned `#ERROR!` when `{Next Rehearsal Date (from Choir)}` was "Not yet booked" or blank, and that error string was passed directly to Stripe
- Fixed multiple formula fields to handle blank/invalid dates gracefully:

**1. `{Next Rehearsal Date}` formula:**
- Was: `DATETIME_FORMAT(DATETIME_PARSE(...), "YYYY-MM-DD")` — errored on "Not yet booked"
- Fix: Wrapped in `IF()` that checks for empty and "Not yet booked", returns `""` instead

**2. `{Skip Next Month}` formula:**
- Was: Used `DATETIME_FORMAT({Next Rehearsal Date}, ...)` directly — errored when date was blank
- Fix: Airtable evaluates both branches of `OR()` even if first is true, so moved the blank check into a separate outer `IF()` — date functions only run when field has a value

**3. `{Billing Anchor Rehearsals Left Multiplier}` — New Term Pro-Rata Fix:**
- **Problem:** Members signing up at the start of a new term (Jan/May/Sep) were double-charged — pro-rata for rehearsals before the anchor PLUS the full monthly payment on the anchor date
- Example: Sign up Jan 2nd, anchor = 15th → paid £7.25 pro-rata + £29 on Jan 15th = £36.25 (should be max £29)
- **Additional complexity:** Summer term rehearsals sometimes start in late April, but first full payment should be May
- **Fix:** Added `{week 1 date}` lookup check — if today is before the first billing anchor on/after the term's first rehearsal, multiplier = 0 (just £1 activation fee, first full payment on anchor)
- Logic: parse `{week 1 date}`, find the 1st/15th of that month (or next month if week 1 falls after the anchor day), compare against TODAY()

### Key Decisions

| Decision | Reason |
|----------|--------|
| Nested `IF()` instead of `OR()` for blank checks | Airtable evaluates all branches of `OR()` — `DATETIME_FORMAT` on blank still errors even if first condition is true |
| `{week 1 date}` for term boundary detection | Available as lookup on Signup Queue; more reliable than hardcoding months since summer term can start in April |
| Multiplier = 0 for new term signups (not "skip" flag) | The 4 monthly payments from the anchor already cover the full term; pro-rata is only for mid-term joins |
| Empty `{week 1 date}` → multiplier 0 | No term data = no rehearsals to charge for; just take £1 activation |

### Issues Encountered / Workarounds

**Airtable `OR()` eagerly evaluates all branches:**
- First attempt at `{Skip Next Month}` fix used `OR({Next Rehearsal Date} = "", NOT(DATETIME_FORMAT(...)))` — still errored because Airtable evaluates the `DATETIME_FORMAT` branch even when the first condition is true
- Fix: Nested `IF()` so date functions are only reached when the field has a value

**`#ERROR!` cascade:**
- One broken formula field (`{Next Rehearsal Date}`) cascaded errors into `{Skip Next Month}`, `{Days Until Billing}`, `{Billing Anchor Rehearsals Left Multiplier}`, and `{Total Cost Initial Invoice}` — all of which feed into the Stripe checkout call
- All downstream formulas now handle blank inputs gracefully

### Airtable Formula Changes (Manual)

**`{Next Rehearsal Date}`:**
```
IF(
  AND(
    {Next Rehearsal Date (from Choir)} != "",
    {Next Rehearsal Date (from Choir)} != "Not yet booked"
  ),
  DATETIME_FORMAT(
    DATETIME_PARSE(ARRAYJOIN({Next Rehearsal Date (from Choir)}, "")),
    "YYYY-MM-DD"
  ),
  ""
)
```

**`{Skip Next Month}`:**
```
IF(
  OR(
    {Next Rehearsal Date} = "",
    {Next Rehearsal Date} = BLANK()
  ),
  "Yes",
  IF(
    NOT(
      OR(
        DATETIME_FORMAT({Next Rehearsal Date}, 'YYYY-MM') = DATETIME_FORMAT(TODAY(), 'YYYY-MM'),
        DATETIME_FORMAT({Next Rehearsal Date}, 'YYYY-MM') = DATETIME_FORMAT(DATEADD(TODAY(), 1, 'month'), 'YYYY-MM')
      )
    ),
    "Yes",
    "No"
  )
)
```

**`{Billing Anchor Rehearsals Left Multiplier}`:**
- Added `{week 1 date}` blank check at top (returns 0)
- Added new-term check: if today < first billing anchor on/after `{week 1 date}`, returns 0
- Otherwise falls through to existing rehearsal-count SWITCH logic

### What's Next / Where We Left Off

- All `#ERROR!` formula cascades fixed — choirs with "Not yet booked" or blank rehearsal dates can now sign up
- New term pro-rata fix deployed — members joining at term start pay £1 activation only, first full payment on anchor
- One choir currently showing "push to next month" — likely correct behaviour due to blank `{Next Rehearsal Date}`, needs verification
- May need to verify `{Rehearsals to 1st}` and `{Rehearsals to 15th}` also handle blank dates gracefully

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SomeVoicesChoir)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/SomeVoicesChoir)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
