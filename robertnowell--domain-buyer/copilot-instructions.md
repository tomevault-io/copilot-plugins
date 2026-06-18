## domain-buyer

> Purchase a domain name via the Namecheap API with a hard spending cap and explicit human confirmation. Use when the user says "buy <domain>", "register <domain>", "purchase <domain>", asks Claude to acquire a domain, or wants to check availability and price before buying. Drives the full flow inline in the conversation — Claude asks questions, runs the API calls, confirms with the user before any spend.


# Domain Buyer

In-Claude domain registration. Claude orchestrates setup, availability checks, and purchase via the Python wrapper at `~/.claude/skills/domain-buyer/domain_buyer.py`. No standalone CLI wizard — every prompt happens in chat.

## Architecture

- `domain_buyer.py status` — JSON state probe (config exists? sandbox? cap? current public IP?)
- `domain_buyer.py check <domain> --json` — availability + price + over_cap flag, JSON
- `domain_buyer.py buy <domain> --confirmed --json` — registration, JSON result. `--confirmed` tells the script that Claude has already confirmed with the user; without it, the script falls back to a TTY prompt for direct-CLI fallback use.
- `domain_buyer.py log --json` — purchase history, JSON

Config lives at `~/.config/domain-buyer/config.json` (chmod 600). Claude reads and writes this file directly with the Read/Write tools. The Python script never prompts.

## Safety invariants (non-negotiable)

1. **One domain per `buy` call.** Never batch. If the user wants three domains, do three independent confirmations.
2. **Per-purchase cap.** Default $50 USD, lives in config. Enforced via the over-cap second-confirmation flow, not via a hard refusal.
3. **Type-the-domain confirmation.** Before invoking `buy`, Claude must ask the user to type the exact domain name in chat. Anything other than an exact match aborts. If over cap, additionally require the user to type `override` after the domain confirmation.
4. **Re-check immediately before buying.** Run `check` again right before `buy` so the price you confirmed against is the price that gets charged. No stale data.
5. **Never invent prices or API responses.** Every dollar figure shown to the user must come from the script's JSON output.

## Pre-flight (always run first)

When invoked for any reason, start with:

```bash
python3 ~/.claude/skills/domain-buyer/domain_buyer.py status
```

Parse the JSON. Branch:
- `config_exists: false` → run the **Setup flow** below.
- `config_exists: true` → run the **Check / Buy / Log flow** the user asked for.

The `status` output also includes `current_public_ip` — useful when the user's IP changed since setup.

## Setup flow (run when no config)

### Step 1 — Tell the user what's coming

In chat, explain in 2-3 sentences: we need API access from Namecheap (with the IP-whitelist prerequisite), registrant contact info (ICANN requirement), and a spending cap. Note that the API key will appear in chat transcript — they should rotate it after setup if that's a concern.

### Step 2 — Sandbox vs production + cap (AskUserQuestion)

Ask via `AskUserQuestion`:
- Q1 "Environment for first setup": `Sandbox (test, free, separate account at sandbox.namecheap.com)` / `Production (real domains, real money)`. Default to Sandbox if they're new to the API.
- Q2 "Per-purchase cap in USD": offer `$50 (default)` / `$25 (cautious)` / `$100 (room for premium .ai/.io)`. Other lets them type a number.

### Step 3 — Credentials (chat free-text)

Tell the user where to get the key: `namecheap.com → Profile → Tools → Namecheap API Access` (or sandbox.namecheap.com for sandbox). Ask them to reply with two values in any reasonable format:
- Namecheap username (also their API user)
- API key

Parse their reply. If ambiguous, ask for the missing piece.

### Step 4 — IP whitelist reminder

From `status`, you already have `current_public_ip`. Tell the user the exact IP and the exact URL to whitelist it:
- Prod: https://ap.www.namecheap.com/settings/tools/apiaccess/
- Sandbox: https://ap.www.sandbox.namecheap.com/settings/tools/apiaccess/

Wait for the user to confirm they've added it before proceeding. (10-min propagation per Namecheap; usually instant.)

### Step 5 — Registrant contact (pull from Namecheap, do NOT re-ask)

Namecheap already stores contact profiles on the user's account. Never make the user re-type information the registrar has on file. Pull it via the API:

```bash
# Lists address profiles (returns AddressId + nickname per profile)
python3 ~/.claude/skills/domain-buyer/domain_buyer.py addresses --json

# If multiple profiles, show the list with nicknames and let the user pick
# If one profile, use it directly after a one-line confirmation
# If zero profiles, fall back to asking the user (see fallback below)
```

Show the retrieved contact info in chat and confirm with the user: *"I'll register domains under this contact — looks right?"* Single AskUserQuestion with options `Yes, use this` / `Use a different saved address` / `Enter manually`.

**Fallback** (no saved address on Namecheap, or user wants to override): ask for one paste block — First name, Last name, Street, City, State/Province, Postal code, Country (e.g. US), Phone (`+1.5551234567` format), Email. Parse, ask targeted follow-ups for missing fields.

### Step 6 — Write the config

Use the Write tool to create `~/.config/domain-buyer/config.json`:

```json
{
  "api_user": "<username>",
  "api_key": "<key>",
  "username": "<username>",
  "client_ip": "<current_public_ip>",
  "sandbox": true|false,
  "cap_usd": 50.0,
  "contact": {
    "FirstName": "...", "LastName": "...",
    "Address1": "...", "City": "...",
    "StateProvince": "...", "PostalCode": "...",
    "Country": "US", "Phone": "+1.5551234567",
    "EmailAddress": "..."
  }
}
```

Then `chmod 600` it via Bash. Confirm to the user that setup is done and suggest a smoke-test check (sandbox: any domain; prod: a domain you don't care about, since `check` is read-only).

### Step 7 — Smoke test + balance check

Run `status` again — it now pulls live balance from Namecheap. If `available_balance: 0`:

> "Heads up: your Namecheap balance is $0. The API spends from prepaid balance, NOT your card on file (Namecheap-specific quirk). Top up at https://ap.www.namecheap.com/Profile/Funds/Add before your first buy. $20-50 is reasonable."

Open that URL automatically via `open`. Don't wait for the user to top up — flag it and move on. Setup is otherwise complete.

If `status` fails with the IP-whitelist error block, walk the user back to step 4 (IP changed since they whitelisted).

## Check flow

```bash
python3 ~/.claude/skills/domain-buyer/domain_buyer.py check <domain> --json
```

`check` already runs the ownership diagnostic when `available: false` — the JSON includes `owned_by_us` and `owned_detail`. Never report a bare "not available" without that context; the script makes it impossible to forget.

Parse the JSON. Present in chat:
- `available: false, owned_by_us: true` → "Not available — you already own it (created X, expires Y). Want to do something with it (renew, transfer, configure DNS), or skip?"
- `available: false, owned_by_us: false` → "Not available — owned by someone else. Want to try variants?"
- `available: true, is_premium: false` → "Available — $X.XX total (breakdown). Within your $50 cap." or note over-cap.
- `available: true, is_premium: true` → highlight premium pricing prominently and warn it's typically 10-1000× standard.

**Principle:** a negative read result is a question, not an answer. If `check` returns "not available," `mine` is the obvious follow-up — that's why `check` runs it automatically.

Don't go further unless the user explicitly says to buy.

## Mine flow (list owned domains)

```bash
python3 ~/.claude/skills/domain-buyer/domain_buyer.py mine --json
```

Returns every domain on the account with creation date, expiry, and auto-renew status. Use to answer "what do I own?" or to verify before a renewal / DNS conversation.

## Buy flow

### Step 1 — Re-check immediately + balance preflight

Run `check <domain> --json` *fresh*. Do not reuse a previous check's price — it may have changed.

Then check balance: the `buy --confirmed` subcommand will refuse upfront with `insufficient_balance` if there aren't enough funds, but Claude should preempt that by surfacing balance vs. total in the confirmation summary (Step 2). Run `balance --json` and include both numbers.

If balance < total_cost: do NOT proceed to the confirmation prompt. Instead surface the shortfall, open the top-up URL via `open "https://ap.www.namecheap.com/Profile/Funds/Add"`, and tell the user to say "retry" once funded.

### Step 2 — Present full summary in chat

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PURCHASE CONFIRMATION  [SANDBOX|PRODUCTION]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Domain:       <domain>
Registrant:   <email from config>
Years:        1 (unless user specified otherwise)
Breakdown:    <from check JSON>
TOTAL:        $X.XX USD
NC balance:   $A.AA USD (sufficient | INSUFFICIENT)
Cap:          $50.00 USD ← exceeded   (only if over)
30-day spend: $Y.YY USD (this brings it to $Z.ZZ)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Type the exact domain name in chat to confirm: <domain>
Anything else aborts.
```

Pull the 30-day spend by parsing `log --json` and summing `price_usd` for non-sandbox entries within the last 30 days (or pull from `status` output — it's there).

### Step 3 — Wait for the user's reply

The user's next message must equal the domain string exactly (case-insensitive is acceptable; whitespace stripped). Anything else → abort and tell them the purchase did not go through.

### Step 4 — Over-cap second confirmation

If `over_cap: true`, after they typed the domain, separately ask: "This is over the $50 cap. Type `override` to proceed, anything else aborts." Wait for that second reply.

### Step 5 — Invoke the buy

```bash
python3 ~/.claude/skills/domain-buyer/domain_buyer.py buy <domain> --confirmed --json
```

Parse the result. The JSON includes `ok`, `domain`, `price_usd`, `transaction_id`, `registered`.

### Step 6 — Report

- Success: "Registered <domain> for $X.XX. Namecheap order ID <txn>. WHOIS privacy is on. Auto-renew is OFF — toggle in the dashboard if you want it on. DNS isn't configured yet."
- Failure: surface the error verbatim. The script's error path already includes the IP-whitelist help block when relevant.

## Log flow

```bash
python3 ~/.claude/skills/domain-buyer/domain_buyer.py log --json
```

Present a clean summary in chat. Filter or summarize based on the user's question ("last month", "everything", etc).

## When NOT to invoke this skill

- DNS management on existing domains — wrong tool, recommend Cloudflare or Porkbun for DNS.
- Domain transfers in/out — not supported.
- Bulk purchases of >5 domains — flag the risk and ask the user to confirm the full list explicitly before doing them one at a time.

## Failure modes Claude should expect

- **IP whitelist miss** — the script's error already prints the current IP and exact URL. Surface it verbatim and wait for the user to whitelist + confirm.
- **Premium pricing** — `is_premium: true` with a price 10-1000× normal. The cap (with override flow) catches this.
- **Restricted TLDs** — `.ai`, `.uk`, `.de`, `.eu` have registrant-residency rules. Namecheap returns specific errors; surface verbatim.
- **Insufficient balance** — Namecheap charges from account balance, not card. User must top up in dashboard.

## What this skill does NOT do

- Does not set DNS records after registration.
- Does not configure auto-renew (defaults OFF at Namecheap; user toggles in dashboard).
- Does not handle transfers.
- Does not work for users in geographies Namecheap can't serve — error surfaces verbatim.

---
> Source: [robertnowell/domain-buyer](https://github.com/robertnowell/domain-buyer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
