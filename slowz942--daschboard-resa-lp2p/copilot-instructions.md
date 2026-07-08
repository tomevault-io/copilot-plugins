## daschboard-resa-lp2p

> > Read this before touching any of: `index.html`, `n8n/*.js`, the Tampermonkey

# LP2P — Project Conventions & Gotchas

> Read this before touching any of: `index.html`, `n8n/*.js`, the Tampermonkey
> userscript, the `Sourcing Discord` sheet, or the matching algorithm.

The dashboard is a **single static `index.html`** served via GitHub Pages
(`https://slowz942.github.io/DASCHBOARD-RESA-LP2P/`) from `main`. There is
no build step. Edit and push to `main` to deploy. Pages rebuilds in ~30s.

The repo also contains:
- `userscript/discord-wts-scraper.user.js` — Tampermonkey userscript that
  scrapes WTS posts from Discord
- `n8n/*.js` — Code-node bodies for two n8n flows
- `n8n/PHASE-4-SETUP.md` — step-by-step setup of the Telegram flow
- `n8n/SETUP.md` — step-by-step setup of the original WTS-parsing flow
- `google-apps-script/Code.gs` — Apps Script web app that returns inventory
  rows + cell colors as JSON
- `SOURCING.md` — auto-sourcing setup walkthrough

---

## The full pipeline

```
Tally form (client request)              Discord #wts (community sellers)
       │                                            │
       │ Gmail trigger                              │ Tampermonkey userscript
       ▼                                            │ (read-only, no Discord API)
[n8n flow #1: Tally→CRM→Telegram notif]             │ POST to webhook
       │                                            ▼
       ├─ format message (parse Tally email)     [n8n flow #3: Discord→Sheet]
       ├─ Find matches (NEW Code node)              │
       │  ▸ reads Inventory + Sourcing Discord     │ Webhook trigger
       │  ▸ runs the same matcher as the dashboard │ Code: Anthropic claude-3-5-haiku
       │  ▸ POSTs Telegram notification with        │     parses raw WTS message into
       │    inline keyboard of N proposable         │     structured rows
       │    options (calls api.telegram.org         │ Google Sheets append/update
       │    directly, NOT the n8n Telegram node)    │     to "Sourcing Discord" sheet
       └─ append to "CRM clients lp2p" sheet               │
                                                            ▼
[n8n flow #2: Telegram callback handler]    [Google Sheet "Sourcing Discord"]
       │                                            ▲
       ├─ Telegram Trigger (callback_query only)    │
       ├─ branches in PARALLEL:                     │
       │  ▸ answerCallbackQuery (fires            [Dashboard reads via gviz CSV]
       │    immediately — 15s deadline)
       │  ▸ Resolve match (Code) → fetches sheets,
       │    re-runs matcher, picks chosen by index,
       │    builds proposal text → sendMessage
```

### Sheet IDs (already filled in everywhere)
- **Inventory (your stock):** `1jSQNoni7qW6ShnRw3hi_g_fF90qn5YZ3koRaL1gYTLE`
- **Sourcing Discord:** `10QzZ14S4fA5zuM-UyROwmsIKLcwPata6cVLN23bukW8`
- **CRM clients lp2p (Tally writes here):** see n8n config

### Apps Script for inventory colors
URL: `https://script.google.com/macros/s/AKfycbwKCiudNgJU4RtPk-tCv5A33IX3TVtIEJAU_LwbmdhpXHPbWRqYoLbYDUWzkR12zkQ8Hw/exec`

Stored in the user's localStorage on the dashboard (key `lp2p_apps_script_url`).
Hardcoded in n8n Code nodes (constant `APPS_SCRIPT_URL`).

### Telegram bot
- Bot token: stored in n8n Code node (constant `TELEGRAM_BOT_TOKEN` at line ~27 of `find-matches-and-notify.js`). The user pastes their token; **never push a real token to git**. The repo file holds a placeholder.
- Chat ID: `5135913166` (operator's chat with the bot)

---

## Critical conventions (BREAK ONE → BUG)

### 1. Per-place pricing is the source of truth
Inventory's `Achat`/`Revente` columns store **TOTALS** for the whole listing
(verified against the Benef column on actual rows like Roland Garros duo
cat 2 = 490/1400/910 → Benef = 1400 - 490, so totals).

Discord listings carry `price_per_unit` (per-place) from the LLM.

The matcher requires uniform per-place semantics. Do NOT mix totals and
per-place in `prixAchat`. Always run inventory rows through
**`inventoryTotalsToPerPlace(it)`** right after `parseInvRow()`. This
divides `prixAchat` and `prixVente` by `qty` (when qty > 1).

This conversion lives in three files — keep them in sync:
- `index.html` syncInventory (Apps Script + CSV branches)
- `n8n/find-matches-and-notify.js` fetchInventoryViaAppsScript +
  fetchInventoryViaCSV
- `n8n/callback-handler.js` fetchInventory

### 2. Demand totals use BUYER's number of places, not seller's qty
A seller might have a Trio (qty=3) but the buyer wants 2. The proposal
total is `pricePerPlace × buyerPlaces`, not `pricePerPlace × sellerQty`.
Both the dashboard and Telegram use:
```js
const buyerPlaces = parseInt((demand.places||'1').toString().replace(/[^\d]/g,''))||1;
```

### 3. `normalizeArtist` is the canonical entry point
Always pipe artist values through `normalizeArtist()` before storing on a
demand or matching against inventory. The function:
- Strips diacritics (`Céline` → `CELINE`)
- Maps aliases (`CELINE` → `CELINE DION`, `BRUNO` → `BRUNO MARS`, `AYA` →
  `AYA NAKAMURA`, etc.)

Three places have a copy. Keep them in sync. New artists go in the
`ALIASES` literal in all three files.

The dashboard's `syncTelegram()` had a bug where it ran its own
artist-detection regex but never piped the result through
`normalizeArtist()`. Make sure that line is preserved:
```js
artist = normalizeArtist(artist);
```

### 4. `normalizeCat` + strict cat matching
- Maps `Catégorie 1` / `Categorie 1` / `Cat. 1` → `CAT 1`
- Maps `n'importe` / `pas de preference` / `tout` / `libre` / etc. → `NC`
- `findMatches` HARD-FILTERS by category when the demand specifies one.
  Wrong-cat options never appear (don't go back to "score -3 and keep").

### 5. `looseCatMatch` is precise, not fuzzy
- `FOSSE` ↔ `FOSSE OR` ↔ `FOSSES early` → match (all contain FOSSE)
- `CAT 1` ↔ `CAT 1` only (NOT CAT 2, NOT CAT OR)
- `CAT OR` ↔ `CAT OR`

### 6. `dedupeMatches` returns a NEW array
Properties assigned to the input array are LOST. If you set
`out._demandMeta = m;` before `return dedupeMatches(out);`, the renderer
will not see it. Always assign properties AFTER dedupe:
```js
const deduped = dedupeMatches(out);
deduped._demandMeta = m;
return deduped;
```

### 7. Telegram body — title emoji ≠ category emoji
The notification builder uses **`📨`** for the title (`📨 Nouvelle demande`)
and **`🎫`** for the category line (`🎫 FOSSE`). Don't unify them — the
callback handler's regex parser keys off these emojis to reconstruct the
demand from the message text. Same emoji on both lines = wrong cat parsed.

### 8. Telegram callback_query has a ~15s answer deadline
The callback workflow MUST run `answerCallbackQuery` in PARALLEL with the
slow work (sheet refetch + rematch). Do not put it after `Resolve match`
in series — sheet fetches are 5-10s and you'll exceed the timeout, getting
"query is too old or query ID is invalid".

Topology:
```
Trigger ─┬─ answerCallbackQuery     (fires ~immediately)
         └─ Resolve match → sendMessage   (slow, but no longer blocks)
```

### 9. Same-date matches are surfaced first via stable sort
After scoring, the matcher sets `dateMatch: true|false` per item, then
sorts: `dateMatch` DESC first, `score` DESC second. The "Autres dates
disponibles" divider is inserted at the boundary in both Telegram body
and dashboard panel. Color carries the meaning:
- Green left stripe = same date, `✓ même date` chip
- Amber left stripe + tint + chip = other date

### 10. Userscript needs `@connect *` (or specific) directives
Tampermonkey blocks cross-origin POSTs by default. The metadata block
must list every host the script POSTs to. We have `@connect *` as a
catch-all so swapping webhooks doesn't require re-installing.

### 11. n8n Webhook node wraps body under `data.body`
n8n's webhook output is `{ params, query, body, webhookUrl, executionMode }`.
The Code nodes look at `data.body?.messages || data.messages` so the same
code works whether the input is the wrapped webhook output or directly
piped mock data.

### 12. n8n Code node `helpers.httpRequest` error shape varies
Across n8n versions, error bodies live at different paths:
`err.response.body`, `err.response.data`, `err.cause.response.body`, etc.
The error handlers walk all of them. Don't simplify back to
`err.message` — you'll lose the actual Anthropic error JSON which is
critical for debugging (e.g. "credit balance too low" → 400 with no
visible message otherwise).

### 13. Anthropic model name
Use `claude-3-5-haiku-20241022` (fully versioned, universally available).
The `MODEL` constant is at the top of the n8n Code node — easy to swap
to `claude-3-5-haiku-latest` or a newer haiku ID if needed.

### 14. WTS-only filter in the userscript
The DOM scraper only forwards messages matching `/wts|vds|vends|sell|sale/i`.
Toggle off via the Tampermonkey menu if debugging.

### 15. Userscript dedupe via Tampermonkey storage
Seen Discord message IDs are persisted across reloads. Reset via the
"LP2P · Clear seen IDs" or "LP2P · Re-send last visible" menu commands.

---

## Permissions / settings

`.claude/settings.local.json` allows: `Bash(git *)`, `Bash(gh *)`,
`Bash(cd *)`, `Bash(GIT_TERMINAL_PROMPT=* git *)`, `Bash(node *)`,
`Bash(npm *)`, `Bash(npx *)`, `Bash(curl *)`, `Read`, `Edit`, `Write`,
`Glob`, `Grep`. So agent CAN push directly to `main` if the user asks.

PR workflow: branches push freely; `git push` to `main` works because of
the rule above. Pages rebuilds automatically.

---

## Outstanding / next-up work

### Phase 5 — auto-fire IG DM via ManyChat
The callback handler currently sends the operator a copyable proposal
text in Telegram. To fully automate:

1. Get a ManyChat API key (https://app.manychat.com/settings/api/access).
2. In the callback workflow, swap the final `sendMessage` (operator)
   for an HTTP Request to:
   ```
   POST https://api.manychat.com/fb/sending/sendContent
   Authorization: Bearer <MANYCHAT_API_KEY>
   ```
   with body:
   ```json
   {
     "subscriber_id": "<looked up by IG handle>",
     "data": {
       "version": "v2",
       "content": {
         "messages": [{ "type": "text", "text": "{{ $json.proposal_text }}" }]
       }
     }
   }
   ```
3. ManyChat subscriber lookup is needed because IG handles aren't direct
   subscriber IDs — pre-step: `GET /fb/subscriber/findByName` or
   `findByCustomField` keyed off the demand's `instagram` field.
4. Add a confirmation step (e.g. "DM sent to @karim_flb") back to the
   operator chat.

The `internal` field on `buildProposalMessage()` already exposes
`client`, `instagram`, `proposal_text`, `proposed_price_per_place`,
`proposed_price_total` etc. — everything the ManyChat call needs.

### Other ideas that came up but didn't ship
- Status column color-coding in Sourcing Discord sheet (red=available,
  green=taken/sold) — like the inventory sheet. Today the dashboard reads
  the status text only.
- Apps Script for the Sourcing Discord sheet to enable color-based
  status (mirrors `google-apps-script/Code.gs`).
- Bulk select / mark-sold actions in the dashboard.
- Date-range filter and persistence in the URL.
- Per-artist analytics (incoming demand timeline).
- Group-by-event view for batched events (e.g. JUL 16 Mai bulk).

---
> Source: [Slowz942/DASCHBOARD-RESA-LP2P](https://github.com/Slowz942/DASCHBOARD-RESA-LP2P) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
