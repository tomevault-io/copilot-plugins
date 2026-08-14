## rio-casa

> Full-featured resort website for **Rio Casa**, Mahabaleshwar, Maharashtra.

# Rio Casa Resort Website — CLAUDE.md

## Project Overview
Full-featured resort website for **Rio Casa**, Mahabaleshwar, Maharashtra.
Goals: direct bookings, brand presence, event/package promotion.

## Tech Stack
- **Next.js 14** — App Router, TypeScript, `app/` directory
- **Tailwind CSS v3** — design tokens in `tailwind.config.js`
- **next-intl** — string store only. **English (`en`) is the only locale.**
- **Prisma 7 + PostgreSQL** (Neon) — bookings, rooms, packages, blog, testimonials
- **Razorpay** — payments (cards, UPI, net banking)
- **Resend** — transactional booking confirmation emails
- **Framer Motion** — page/section animations
- **React Hook Form + Zod** — all form validation

## Directory Structure
```
app/                  Next.js App Router pages
  layout.tsx          Root layout with metadata
  page.tsx            Home page
  globals.css         Tailwind + Google Fonts import
  api/                API routes only (no UI)
    booking/          POST create-booking, GET availability
    payment/          POST verify Razorpay signature
    contact/          POST contact form
components/
  layout/             Navbar, Footer, WhatsAppButton
  sections/           Hero, RoomGrid, PackageCards, Testimonials, etc.
  booking/            BookingWizard, DatePicker, RoomSelector, PaymentStep
  ui/                 Shared primitives (Button, Card, Input, Badge)
lib/
  prisma.ts           Prisma client singleton
  razorpay.ts         Razorpay SDK wrapper
  i18n.ts             (handled by next-intl root i18n.ts)
messages/
  en.json             All UI strings — NEVER hardcode text
prisma/
  schema.prisma       DB schema (Room, Booking, Package, Testimonial, BlogPost)
```

## Strict Rules
1. **No hardcoded UI text** — every visible string must use `useTranslations()` from next-intl
2. **Tailwind only** — no inline styles, no CSS modules (except globals.css for @layer)
3. **Server Components by default** — only add `"use client"` when you need interactivity or hooks
4. **Zod schemas** — validate all API request bodies at the route handler level
5. **Prisma via singleton** — app code always imports `prisma` from `@/lib/prisma`.
   Standalone scripts in `prisma/` use `makeScriptClient()` from
   `prisma/script-client.ts`. Never construct `new PrismaClient()` directly —
   under Prisma 7 it does not even compile without a driver adapter.

## Design Tokens (Tailwind)
```
primary          #4A6741  (forest green)
accent           #8B6914  (golden brown)
earth-bg         #F5F0E8  (warm cream background)
earth-text       #2C2416  (dark earth text)
earth-white      #FDFAF5  (off-white)
font-serif       Cormorant Garamond
font-sans        DM Sans
font-deva        Noto Sans Devanagari (Hindi/Marathi)
```

## Component Classes (globals.css)
```
.btn-primary          Green filled button (public site)
.btn-outline          Green outline button (public site)
.btn-admin            Admin action button — appearance only; keep your own
                      layout utilities (flex-1, w-full, px-4 py-2) alongside it
.section-heading      Serif h2/h3 style
.section-subheading   Italic serif subtitle
.container-resort     max-w-7xl centered with padding
```
Use the design tokens above, never raw hex (`bg-primary`, not `bg-[#4A6741]`).
Exceptions are third-party brand colours (WhatsApp green, Razorpay theme).

## Shared UI (`components/ui/`)
- `Toast` + `useToast` — transient admin confirmations. One implementation;
  don't hand-roll toast state, timers, or markup in a panel.

## Shared Data (`lib/labels.ts`)
`ROLE_LABEL`, `ROOM_TYPE_LABEL`, `ROOM_TYPE_FILTER_LABEL` — display names for
domain enums. Import them; defining a local copy in a panel is how
`ROOM_TYPE_LABEL` previously drifted into two incompatible versions.

## API Conventions
- All API routes live in `app/api/**`
- Validate body with Zod before any DB call
- Razorpay webhook: verify `x-razorpay-signature` header before updating booking status

### Auth — `lib/api-auth.ts`
Never re-implement the cookie/JWT/role dance in a handler. Two lines:
```ts
const auth = await requireRole(req, "manager");  // or requireAuth(req)
if (!auth.ok) return auth.response;              // 401 or 403, already shaped
// auth.staff is AdminPayload from here
```
Server components use `cookies()` from `next/headers`, so they still call
`verifyAdminToken` directly — `requireRole` is for route handlers only.

### Responses — `lib/api-response.ts`
Always return via a helper. Never hand-write `NextResponse.json({ success: ... })`
— that is how the payload key drifted to `promos` / `plan` / `booking` / `kpi`,
forcing every client to know a different key per endpoint.

| Helper | Shape |
|---|---|
| `ok(payload, status?)` | `{ success: true, data: <payload> }` |
| `okMessage(text, status?)` | `{ success: true, message: string }` — ack, no data |
| `okEmpty(status?)` | `{ success: true }` — deletes and similar |
| `fail(text, status?)` | `{ success: false, error: string }` |
| `failValidation(zodError)` | `fail()` with the first issue message |

**`error` is always a string.** Clients render it directly
(`showToast(data.error ?? "…")`), so returning a Zod error object shows
`[object Object]` to staff. Use `failValidation(parsed.error)`, never
`parsed.error.flatten()`.

Clients therefore always read `data.data` for payloads and `data.message`
for acknowledgements.

## Laundry (linen sent to the laundryman)
`/admin/housekeeping?tab=laundry` — models `LinenItem`, `LaundryBatch`,
`LaundryBatchItem`.

A batch goes out with a count per item type and comes back days later; the
difference is what went missing. Two rules the code depends on:
- **Quantities and rates are snapshotted per line** at dispatch, so editing
  the catalogue later never rewrites what a past batch cost.
- **A batch is only `returned` when every piece is accounted for.** Returned
  + damaged must equal sent; anything short keeps the batch `partial` so the
  missing pieces stay visible in the outstanding list. The API rejects a
  return larger than the dispatch — otherwise the outstanding count goes
  negative and the system invents linen.

Seed the catalogue with `npx tsx prisma/seed-linen.ts` (idempotent; upserts
by name and preserves rates edited in the admin panel).

## Seed scripts
```bash
npm run seed:admin                    # staff logins (see run-app skill)
npx tsx prisma/seed-rooms.ts          # ⚠️ destructive — wipes bookings
npx tsx prisma/normalize-rooms.ts     # reshape rooms, keeping bookings (dry run)
npx tsx prisma/seed-linen.ts          # linen catalogue
npx tsx prisma/seed-bookings.ts       # bookings around today
npx tsx prisma/repair-data.ts         # report drifted derived state (--apply to fix)
```

## Scheduled Jobs (`vercel.json` → `crons`)
| Path | Schedule (UTC) | Purpose |
|---|---|---|
| `/api/cron/night-audit` | `15 0 * * *` (05:45 IST) | Mark no-shows, flag arrivals/departures |
| `/api/cron/detect-conflicts` | `45 0 * * *` (06:15 IST) | Double-booking safety net |

Constraints worth knowing before editing the schedule:
- **Vercel schedules in UTC**, not IST. `runNightAudit()` derives "yesterday"
  from the server clock, so it must run shortly *after* UTC midnight. Moving it
  to midnight IST (18:30 UTC) would audit a day that is still in progress.
- **Sub-daily schedules require a Pro plan** and fail at deploy time on Hobby.
  Hourly conflict detection would be better; on Pro use `"0 * * * *"`.
- **`/api/cron/pull-ota` is deliberately unscheduled** — it targets eZee
  Centrix, which this property does not use. See `CHANNEL-MANAGER-PLAN.md`.

Auth: Vercel sends `Authorization: Bearer $CRON_SECRET` when `CRON_SECRET` is
set as a project env var. Guard every cron route with `denyIfNotCron(req)` from
`lib/cron-auth.ts` — never compare against `process.env.CRON_SECRET` inline,
which renders `"Bearer undefined"` and fails *open* when the var is missing.
**`CRON_SECRET` must be set in the Vercel project**, or every cron returns 503.

## Dates — `@db.Date` columns (`lib/dates.ts`)
`checkIn`, `checkOut`, `blockDate`, `validFrom/To`, `invoiceDate`, `Expense.date`,
`Shift.date` and friends are Postgres **DATE** columns. They hold a calendar
day, not an instant, and Postgres compares them by casting whatever bound you
pass down to a date.

**Never build a bound with local time.** `new Date(y, m, d)` and
`new Date("2026-12-20T00:00:00")` are local midnight — in IST that is
`…T18:30:00Z` on the *previous* day, which Postgres truncates straight back to
that previous day. This shipped three separate bugs: blocking 20–21 Dec stored
19–20 (so the closed day stayed bookable), the dashboard's "today" window
selected yesterday's arrivals and departures, and a report "from 1 Sep" quietly
started on 31 Aug.

Use the helpers instead — they answer "which day is it?" in the *property's*
timezone (the dev box runs IST, Vercel runs UTC) and always return UTC midnight:

| Helper | Use |
|---|---|
| `today()` | today at the property, as a DATE value |
| `dateOnly("2026-12-20")` | parse a `YYYY-MM-DD` input |
| `addDays(day, n)` | shift by whole days |
| `startOfMonth` / `addMonths` | month buckets |
| `daysBetween(a, b)` | whole days between two days |
| `toDayString(day)` | back to `YYYY-MM-DD` |

Ranges are half-open: `{ gte: start, lt: end }`. An inclusive `lte` on the last
day matches that whole day and pulls in one extra.

## Derived state that used to drift
Two things are computed from bookings rather than maintained incrementally,
because incremental updates silently fell out of sync:

- **`roomStatus.currentBookingId`** — only check-out used to clear it, so
  no-shows and cancellations left rooms pointing at dead bookings, showing that
  guest's name on the board forever. Call `releaseRoomsHolding(bookingIds)`
  whenever bookings end any way other than check-out.
- **`guest.totalStays` / `totalRevenue`** — kept only so the guest list can sort
  on them. Call `recalcGuestTotals(db, guestId)` after anything that changes a
  booking's status or amount; never `increment`/`decrement`. Cancelled and
  no-show bookings are excluded.

`npx tsx prisma/repair-data.ts` reports both kinds of drift; `--apply` fixes it.
Idempotent and safe to re-run.

## Booking Flow
1. User picks dates + guests → `/api/booking/availability?roomId=&checkIn=&checkOut=`
2. User selects room → clicks "Book Now"
3. Fill guest details form (React Hook Form + Zod) — the step-3 "Continue"
   button must `trigger()` validation before advancing. Errors render inside
   the step-3 markup, so advancing with invalid input unmounts them and step 4
   then refuses to submit with nothing shown to the guest.
4. POST `/api/booking/create` → creates Prisma Booking
   (`status: "confirmed"`, `paymentStatus: "pending"`) + Razorpay order
5. Razorpay checkout opens in browser (razorpay.js)
6. On success → POST `/api/payment/verify` → verify signature → update status to "paid"
7. Redirect to `/booking/confirmation?id=...` + Resend email

**The booking is committed before the Razorpay order exists.** `createBooking()`
commits the booking row, then updates the guest's `totalStays`/`totalRevenue`
and writes an audit row just after the commit — outside the transaction, so
neither can extend the room lock or fail the booking. `createOrder()` only runs
afterwards. If it throws, the
route must void the booking (`cancelled` + `paymentStatus: "failed"`) and
decrement the guest stats — otherwise a `confirmed` row with no order holds the
room on the calendar for a guest who only ever saw an error. Availability
queries skip `cancelled`/`failed`, so voiding is what frees the room.

Clients must not assume an error response has a JSON body: an unhandled route
error returns an empty 500, and a bare `res.json()` shows the guest
`"Unexpected end of JSON input"`. Parse with `.catch(() => null)` and fall back
to your own message.

## Pricing — `quoteStay` / `applyGst` (`lib/booking-service.ts`)
**Every booking path prices through these two.** There were two implementations:
the walk-in route priced off `room.baseRate` with no rate plan, no weekend
markup and no extra bed, while the website used the rate plan for all three.
They agreed only because no rate plan existed — the first one a manager created
from `/admin/setup` would have made walk-ins quietly cheaper than the same room
booked online, with no error anywhere.

Split in two because the promo claim sits between the halves: the discount a
code buys depends on the subtotal, so `quoteStay` → `claimPromo` → `applyGst`.

- **The no-rate-plan fallback is `room.pricePerNight`, not `room.baseRate`.**
  The public site displays `pricePerNight`; pricing off the other column could
  charge a guest more than the page quoted them. `baseRate` is still stored and
  editable in the admin panel but no longer feeds pricing.
- **Without a rate plan an extra bed is free**, on both paths — `extraBedRate`
  lives only on the rate plan. That is inherited behaviour, not a decision. The
  walk-in form collects `extraBed` and bills ₹0 for it today.
- **The GST slab follows the discounted amount**, so a promo can move a stay
  from 18% to 12%.

`rateOverride` is the front desk negotiating a nightly rate. It replaces the
whole tariff: no rate plan, no weekend markup, no extra bed on top — the desk
quoted a number and that is what the guest pays. Overrides are recorded in the
audit log (`rateOverridden`, `nightlyRate`) because they are the one figure a
manager may need to question later. Any `frontdesk` user can set one; there is
no approval step.

## Strings (`messages/en.json`)
**This site is English-only. Do not add Hindi, Marathi, or any other locale.**
`middleware.ts` registers `locales: ["en"]` and `i18n.ts` always loads
`en.json`; `/hi` and `/mr` 404 by design. next-intl is kept purely as the
string store so copy lives in one file instead of being scattered in JSX.

- `useTranslations('namespace')` in client components
- `getTranslations('namespace')` in server components / metadata
- Keys live in `messages/en.json`, grouped by namespace

**Mind the namespace.** A key read from the wrong one renders the raw key path
to the visitor — next-intl does not fall back. `perNight` lives under `rooms`,
and reading it as `booking.perNight` is how the booking wizard once showed
guests "₹5,500 booking.perNight". Missing keys log
`IntlError: MISSING_MESSAGE` in the console; that error is worth grepping for
after touching copy.

## Running Locally
```bash
npm run dev          # Start dev server on :3000
npx prisma studio    # Open DB GUI
npx prisma migrate dev --name <name>   # Create + apply a migration (local only)
npx prisma migrate deploy              # Apply pending migrations (shared/prod)
npx prisma migrate status              # What's applied where
```

### Prisma 7
The client is **Rust-free** — generated TypeScript in `lib/generated/prisma`
(gitignored, built by `prisma generate`, already wired into `npm run build`).
That took the deployed client from ~47 MB to ~1.9 MB with no native binary,
which is the whole reason for being on 7.

Consequences worth knowing before editing anything database-shaped:

- **A driver adapter is mandatory.** `lib/prisma.ts` uses `@prisma/adapter-pg`.
  There is no built-in connection layer any more.
- **Import from `@/lib/generated/prisma/client`,** not `@prisma/client` — the
  client is no longer generated into `node_modules`.
- **`datasource.url` is gone from `schema.prisma`.** The CLI reads it from
  `prisma.config.ts`; the runtime gets it via the adapter.
- **`.env` is not auto-loaded.** `prisma.config.ts` and `prisma/script-client.ts`
  both `import "dotenv/config"` for this reason.
- **Client middleware (`$use`) was removed.** Use client extensions.

### Booking concurrency
`createBooking` holds a SERIALIZABLE transaction with `SELECT … FOR UPDATE` on
the room row, so simultaneous bookings for one room serialise. Three things keep
that from surfacing as noise:

- **Pool size** (`DATABASE_POOL_MAX`, default 20). node-postgres defaults to 10;
  waiters then could not open a transaction at all and failed with `P2028`.
- **`withSerializableRetry`** retries `P2034`/`P2028` with jittered backoff.
  Postgres SERIALIZABLE *expects* clients to retry — without it, losing requests
  reached guests as "Something went wrong."
- **`maxWait: 15s` / `timeout: 20s`** on the transaction, because a waiter
  legitimately needs longer than one transaction's worth of time.

`ROOM_NOT_AVAILABLE` and `BLOCKED_DATE` are deterministic and never retried.
A serialization failure raised by a **raw** query is not `P2034` — Prisma
reports it as `P2010` with SQLSTATE `40001` in `meta.code`. Both the room lock
and the availability re-check are raw, so `isTransientTxError` matches on the
SQLSTATE as well; matching only on `P2034` means every loser of a race reaches
the guest as "Something went wrong."

#### Keep the critical section short
Bookings for one room run strictly one at a time, so the Nth guest in the queue
waits for everyone ahead of them. **The length of the critical section, not the
length of the request, is what sets the tail latency.** Twelve concurrent
bookings for one room took ~22s when the transaction did all of its work under
the lock; the same test now runs ~12s with every loser cleanly rejected.

**Every path that writes a booking goes through `guardRoomAvailability(tx, …)`**
— it takes the `FOR UPDATE` and re-checks conflicts *and* blocked dates in one
round trip. It is a shared function because the admin walk-in route used to
hand-roll its own check: no lock, no blocked-date test, and a conflict predicate
that disagreed with this one about failed payments, so a room the calendar
showed as free could not be booked at the front desk. New booking routes call
it; they do not write their own availability query.

Only three things run with the room locked: the `FOR UPDATE`, the availability
re-check, and the insert. When adding to a booking path, put the new work in
whichever of these is true:

| Where | For work that… | Examples |
|---|---|---|
| Before the transaction | cannot be invalidated by a competing booking | rate plan, pricing, GST, promo claim, booking number |
| Inside the tx, before `FOR UPDATE` | needs the isolation but not the room | the guest lookup/create |
| Under the lock | decides whether the room is free | the re-check, the insert |
| After the commit | is bookkeeping | guest totals, audit log |

Post-commit bookkeeping is deliberately non-fatal: a booking that exists must
not be lost because an audit row failed. That is why guest totals are repairable
(`prisma/repair-data.ts`) rather than transactional.

Do **not** fan the pre-transaction reads out with `Promise.all` — two pool
connections per in-flight request is how contention turned into `P2028`.

Verify with `npx tsx prisma/verify-booking-race.ts [n]` — fires n simultaneous
bookings at one room, asserts exactly one wins, and reports the latency spread.
Run it after touching the transaction, the isolation level, the pool, or the
Prisma version. Unit tests mock the database and cannot catch any of this.
Pipe it to `tail`, never `head`: SIGPIPE kills the script before its cleanup and
leaves a booking behind that fails every later run.

#### Booking numbers — `nextBookingNumber(day)`
`BK-YYYYMMDD-NNN`, allocated by a one-row upsert on `booking_counters`. Both the
website and the admin walk-in route go through it, so the two cannot collide.

It replaced a `COUNT(*)` that was broken and slow at once: the prefix came from
the check-in day while the count window came from `created_at`, so **every
advance booking for the same date computed `-001`** and the second one died on
the unique index — reported to the guest as "this room was just booked". A COUNT
over a predicate also takes a predicate lock, letting bookings for unrelated
rooms abort each other.

Allocation sits outside the transaction, so a failed booking burns a number.
Gaps are expected; duplicates are not.

#### Promo codes
`claimPromo` reserves a use with a single guarded `UPDATE … WHERE (usage_limit
IS NULL OR used_count < usage_limit)`, so the cap is enforced by the statement
itself and needs no surrounding transaction. Claiming happens *before* the
booking transaction — a shared counter row touched under SERIALIZABLE is
contention between bookings that have nothing to do with each other. Because
the claim precedes the availability decision, `createBooking` must hand it back
via `releasePromoClaim` on any failure, or losing a race burns a redemption.

### Migrations
The schema is under migration control as of `0_init`, which was **baselined**
from the existing database — it was generated with `migrate diff` and marked
applied, not replayed. Two migrations exist:

| Migration | Contents |
|---|---|
| `0_init` | Every table, index and FK in `schema.prisma` |
| `1_double_booking_guard` | `btree_gist`, the `no_overlapping_bookings` exclusion constraint, and the five hand-written indexes |
| `2_booking_counter` | `booking_counters`, backfilled from the highest `BK-` suffix already issued per date |

`1_double_booking_guard` holds objects Prisma cannot express in the schema. It
was previously a loose `prisma/add_exclusion_constraint.sql` that had to be run
by hand with `psql` — and **it never was**, so the live database ran without
Layer 1 while the code claimed it was protected. Anything that cannot live in
`schema.prisma` belongs in a migration, never in a file someone is expected to
remember.

**`DATABASE_URL` points at Neon.** Two consequences:
- Never run `migrate dev` or `migrate reset` against it — both can drop data.
  Use `migrate deploy`.
- Neon times out Prisma's advisory lock. Prefix with
  `PRISMA_SCHEMA_DISABLE_ADVISORY_LOCK=1` and use the **direct** endpoint
  (hostname without `-pooler`) for DDL.

The exclusion constraint rejects a second overlapping booking outright, so
`createBooking` can hit a raw constraint violation on `no_overlapping_bookings`
under a race — that is Layer 1 doing its job, not a bug.

### Browser testing
`node scripts/shot.mjs <path>` screenshots a page (admin login handled) — the
fast check that something still renders. For interactive testing, console
errors, or network inspection, the `chrome-devtools` MCP server in `.mcp.json`
drives a real Chrome. See the `run-app` and `test-in-chrome` skills.

## Environment Variables
See `.env` — copy to `.env.local` and fill in real values:
- `DATABASE_URL` — PostgreSQL connection string
- `RAZORPAY_KEY_ID` / `RAZORPAY_KEY_SECRET` — from Razorpay dashboard
- `RESEND_API_KEY` — from resend.com
- `NEXT_PUBLIC_WHATSAPP_NUMBER` — resort WhatsApp number (with country code, no +)

---
> Source: [search4mahesh/rio-casa](https://github.com/search4mahesh/rio-casa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
