## auto-email-campaign

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Automated email campaign system for **AniOne** (anione.me) that sends marketing emails via the **Resend API** using **Broadcasts** to managed **Segments**. Deployed on **Railway** as a daily cron job.

## Architecture

- **`deploy_new_year.py`** — Active campaign dispatcher (runs daily via Railway cron). Uses a date-keyed `CAMPAIGN_MAP` to determine which local template folder and which Resend Segment to send for a given day. Reads the template's HTML/text/Subject from disk and sends it as a Resend **Broadcast** (create + send in one API call). The schedule is a two-wave + paid-finale design across **five segments (A–E)** — see **Monthly Campaign Structure** below.
- **`sync_contacts.py`** — Pushes the `email/` CSV lists into the five Resend Segments (`RESEND_SEGMENT_A`…`E`), creating each unique contact **once with the full set of segments it belongs to** (so a paid user in both Segment A and E is one contact, billed once). CSVs remain the source of truth; run once per month after updating lists. `--create` creates all five segments and prints IDs; `--fresh` deletes all contacts first — **required whenever segment membership changes**, since Resend only sets a contact's segments on creation. (No bulk-import API — contacts are created one-by-one, throttled under 5 req/s.)
- **`send_daily.py`** — Legacy multi-phase campaign script (Christmas campaign). Sends to VIP list first, waits 90 minutes for IP warmup, then sends to cold list. Run manually with `--day N` flag.
- **`scrub_lists.py`** — Interactive email list cleaner. Validates emails (syntax + MX records via `email_validator`), filters bot patterns (+ aliases, excessive dots). Reads from and writes `CLEANED_` prefixed files to `email/` folder.
- **`remove_duplicates.py`** — Deduplicates CSV email lists with Gmail normalization (dot/plus-alias handling). Hardcoded `INPUT_FILE`/`OUTPUT_FILE` paths must be edited per use.
- **`test.py`** — Scratch file for formatting raw email lists; not a test suite.
- **`email/`** — CSV files containing user email lists segmented by signup date and payment status. Column header is `email` (lowercase) in newer files.

## Monthly Campaign Structure & Contact CSVs

Each month's campaign runs ~26 days in three phases on the daily cron (`CAMPAIGN_MAP`), with **all contacts pre-loaded into five Resend segments A–E at once** — no rotation/clearing (fits the **$80 / 10,000-contact** marketing tier):

| Phase | Dates of month | Sends to (alternating) | Audience |
|---|---|---|---|
| **Wave 1** | 1–12 | Segment A → B | free list 1 + 2, **plus paid riding along in Segment A** |
| **Wave 2** | 13–24 | Segment C → D | free list 3 + 4 |
| **Paid token drops** | 13, 17, 21 | Segment E | paid only — 3 "care package" gift emails (20 image tokens each) filling the paid gap |
| **Paid finale** | 25–26 | Segment E | paid only (2 dedicated finale templates) |

The 6 wave templates (`day-1,3,5,7,9,10`) rotate A→B in Wave 1, then again C→D in Wave 2; the finale uses `finale-1` (gift) + `finale-2` (sale) sent to Segment E. The 3 paid drops use `drop-1/2/3` (codes `DROP1/2/3`). Segment IDs are env vars `RESEND_SEGMENT_A`…`RESEND_SEGMENT_E`.

**Multiple sends per day:** a `CAMPAIGN_MAP` date may map to a **single send dict OR a list of send dicts** — e.g. Jul 13/17/21 each fire a Wave 2 free send *and* a paid token-drop. `main()` dispatches each send for the day.

**CSVs to prepare each month: 5 (4 free + 1 paid).** Wired in `deploy_new_year.py`:

| CSV (in `email/`) | `GROUP_*_FILES` → segment | Phase |
|---|---|---|
| free list 1 | `GROUP_A_FILES` (with paid) → A | Wave 1 |
| free list 2 | `GROUP_B_FILES` → B | Wave 1 |
| free list 3 | `GROUP_C_FILES` → C | Wave 2 |
| free list 4 | `GROUP_D_FILES` → D | Wave 2 |
| **paid** | `GROUP_A_FILES` **and** `GROUP_E_FILES` → A + E | Wave 1 + finale |

The paid list is one file referenced in two segments; `sync_contacts.py` creates each contact **once with the full set of its segments**, so paid is billed once.

**Sizing rule (hold-all-at-once):** the only limit is total unique contacts ≤ the plan's contact cap:
`free1 + free2 + free3 + free4 + paid ≤ CAP` (paid counted once). Equal free lists keep daily volume even, but exact sizes can vary as long as the sum fits.

**Example at the 10,000 cap with ~600 paid** → 4 free lists ≈ **2,350 each** (~9,400) + paid ~600:
- Wave 1: Segment **A ≈ 2,950** (free + paid), Segment **B ≈ 2,350**
- Wave 2: Segment **C ≈ 2,350**, Segment **D ≈ 2,350**
- Finale: Segment **E ≈ 600** (paid)

This reaches ~10,000 people **once** across the month (each in their wave) + the paid finale. To reach MORE than the cap in one month, clear-and-reload between waves (rotation) — same 5 CSVs loaded in phases, never exceeding the cap at once.

> **When asked "what CSVs do I need this month?" → answer: 5 (four free + one paid), sized so `free×4 + paid ≤ contact-cap`, free lists roughly equal. State the count per list and the per-send totals (Wave 1 A = free+paid, etc.).**

## Key Configuration

- **Environment** (in `.env`, loaded via `python-dotenv`): `RESEND_API_KEY`, plus `RESEND_SEGMENT_A` … `RESEND_SEGMENT_E` (five segment IDs). Optional: `SENDER_EMAIL`, `SENDER_NAME`.
- **Sender**: `AniOne <contact@mail.anione.me>` — the `mail.anione.me` domain must be verified in Resend (DKIM/SPF/MX) before sends succeed.
- **Sending model**: Resend Broadcasts → Segments (no Postmark message streams). Unsubscribe is handled automatically by Resend via the `{{{RESEND_UNSUBSCRIBE_URL}}}` placeholder in the template footer.
- **Rate limit**: 5 requests/sec per team (relevant to `sync_contacts.py`, which throttles itself). Broadcasts are a single call per send.
- **Railway cron**: Runs `deploy_new_year.py` daily at 03:00 UTC (`railway.json`)

## Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run the active daily campaign (normally triggered by Railway cron)
python deploy_new_year.py

# First-time setup: create all five Resend segments (A–E) and print their IDs
python sync_contacts.py --create

# Sync the CSV lists into the Resend segments (run monthly after updating lists)
python sync_contacts.py

# Sync but clear ALL contacts first — required when segment membership changes
python sync_contacts.py --fresh

# Clean email lists (interactive file selector)
python scrub_lists.py

# Deduplicate emails (edit INPUT_FILE/OUTPUT_FILE in script first)
python remove_duplicates.py
```

## Previewing & Broadcast Testing (`preview.py`)

`preview.py` sends a TEST copy of a campaign template to a single inbox so you can check it before the real campaign runs. **Two modes:**

```bash
# Transactional preview (default): sends via Resend's /emails endpoint to one inbox.
# Fast render check (images, dark mode, fonts). Does NOT touch segments. The
# {{{RESEND_UNSUBSCRIBE_URL}}} footer shows as LITERAL text (only resolves in a real Broadcast).
python preview.py you@example.com                 # all day-*/finale-* templates
python preview.py you@example.com day-1 day-7     # specific ones

# Broadcast test (--broadcast): exercises the REAL production send path end-to-end,
# reusing deploy_new_year.send_broadcast() verbatim, then polls each broadcast to its
# true terminal status (sent/failed). NO email arg — it sends to whoever is in the
# "AniOne BROADCAST TEST" segment (manage that segment's membership in the Resend dashboard).
python preview.py --broadcast            # all templates to the test segment
python preview.py day-1 --broadcast      # one template to the test segment
```

**How `--broadcast` works (and why it's built this way) — important:**
- A Broadcast targets a **segment**, not an address, so the tool sends to a **persistent test segment named `AniOne BROADCAST TEST`**, found-or-created by name and **REUSED every run — never deleted.** You manage its membership (your test recipients) in the Resend dashboard; `--broadcast` takes **no email argument** and sends to whatever contacts are in it.
- This persistence is deliberate. **Resend sends broadcasts asynchronously (~30s).** An earlier version created a throwaway segment and deleted it immediately after sending — by the time Resend processed the send the segment was gone, so the broadcast reported `failed` even though the send path was fine. Keeping one persistent segment avoids that race. (The real daily cron is never affected — segments A–E are permanent.)
- A `2xx` from the create call means **accepted, not delivered.** The tool polls `GET /broadcasts/{id}` until `status` is `sent`/`failed`, so trust that (or Resend's broadcast history), not the create response.
- This segment is also the natural place to **test managed unsubscribe**: send a broadcast to it, click the unsubscribe link in the received email, and the contact's `unsubscribed` flag flips to `true` — which fires the Resend `contact.updated` webhook (→ `https://anione.me/api/webhooks/resend`).
- `--broadcast` only covers the dirs in `TEMPLATE_DIRS` (`day-*` + `finale-*`); add the `drop-*` dirs there to test those too.

## Important Patterns

- Campaign schedules are defined as hardcoded date maps in `CAMPAIGN_MAP` — update this dict for each new campaign period (each date → `{template_dir, segment}`).
- Within a wave, sends alternate between two segments on consecutive days to spread volume (A↔B in Wave 1, C↔D in Wave 2); the paid finale sends to Segment E. See **Monthly Campaign Structure & Contact CSVs**.
- CSV files in `email/` remain the source of truth for recipients; `sync_contacts.py` pushes them into the Resend Segments. The CSV email column is `email` (lowercase); the cleaning scripts do case-insensitive header lookup.
- Templates are sent **inline** (read from `templates/{month}/day-N/`) — there is no server-side template upload step. Every template footer must keep the `{{{RESEND_UNSUBSCRIBE_URL}}}` placeholder so Resend can wire up managed unsubscribe.
- The `.env` file is gitignored (not tracked); set the same vars in Railway's environment for the cron deploy.

## Campaign Codes & Vouchers (`create_codes.py`)

Each campaign's templates reference redemption codes/vouchers that must exist in the **animechat-ai** app (anione.me, Next.js + Supabase). `create_codes.py` creates them by POSTing to that app's admin API. **Edit its `CONFIG` block, then run.**

```bash
python create_codes.py            # DRY RUN — print every payload, write nothing
python create_codes.py --create   # actually create (idempotent: skips ones that exist)
python create_codes.py --verify   # read back stored values to confirm
```

**Two kinds — pick by what the email link does:**
| Kind | Email link | Endpoint → table | Has start date? | Key fields |
|---|---|---|---|---|
| **Code** (token grant) | `/Profile?tab=redeem&code=X` | `POST /api/admin/codes` → `RedeemCodes` | ✅ `startsAt` + `expiresAt` | `imageTokens`, `textTokens` (≥1 must be >0) |
| **Voucher** (checkout) | `/Pricing?voucher=X` | `POST /api/admin/vouchers` → `Vouchers` | ❌ `expires_at` only (live on create) | `type` + discount field |

**Mapping templates → what to create** (read each template's body for the exact code string + amount):
- **Gift template** ("N image tokens", `?code=`) → a **code** with `imageTokens: N`.
- **Sale / % off** ("20% off", `?voucher=`) → a **voucher**, `type: "tokenDiscount"`, `percent_off: 20`, `not_applicable_to: "tokens"` (subscriptions only).
- **Token multiplier** ("1.5× tokens on pack purchases", `?voucher=`) → a **voucher**, `type: "tokenMultiplier"`, `multiplier: 1.5`, `not_applicable_to: "packages"` (token purchases only).
- `not_applicable_to`: `"tokens"` = subscriptions only · `"packages"` = token purchases only · `None` = both.

**Scheduling rules (the important ones):**
- **Start a day before the campaign** (codes only — vouchers have no start field, so they go live on creation; harmless since the code isn't public until emailed).
- **Expiry must cover the LAST date the code is sent, not just the first.** The same template (and code) is reused across **both campaign waves** (Wave 2 sends 12 days after Wave 1), so a code emailed on day-7 of Wave 2 can go out as late as ~Jul 24. **Set expiry to the end of the month** (e.g. `Jul 31 23:59`) so the second wave never hits an expired code. (Historical bug: codes were set to expire mid-month, which broke Wave 2.)
- The two finale codes (paid Jul 25–26) start a day before their send and also expire end-of-month.

**Notes:**
- Schedules in CONFIG are **local time (UTC+8)**; the script converts to each endpoint's stored format (codes → UTC ISO; vouchers → naive local). Codes require `startsAt` to be in the future — **run before the earliest start date**.
- The admin endpoints are currently **unauthenticated**, so the script POSTs directly (no key/deploy needed). This is a known security gap — a key-protected webhook is the eventual fix.
- Codes are **case-sensitive** and must match the email URL exactly; voucher codes are auto-uppercased. Both are unique (re-running is safe — existing ones report "already exists").

---
> Source: [lerboi/Auto-Email-Campaign](https://github.com/lerboi/Auto-Email-Campaign) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
