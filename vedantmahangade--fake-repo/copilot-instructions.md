## fake-repo

> **ETHGlobal New York 2026** · anti-scalp NFT tickets on Hedera

# Hedera World Cup Ticket — Hackathon Onboarding

**ETHGlobal New York 2026** · anti-scalp NFT tickets on Hedera

---

## 1. What this is

Each World Cup ticket is an NFT on Hedera. When the organizer creates a collection they bake in a **10% royalty fee**: every time the NFT moves in an atomic swap (NFT transferred in exchange for HBAR in the same transaction), the organizer gets 10% of the sale price automatically and without any app-level code. Authenticity is cryptographically guaranteed — no counterfeits.

**Mint-on-buy:** tickets are not pre-minted. The organizer sets `maxSupply` and **face value** only. When a purchaser buys, the backend mints one serial and transfers it in the same flow.

**World ID gates:**
- **Onboarding / login** — one nullifier → one Hedera account (Sybil protection at signup).
- **Secondary purchase** — buyer must verify World ID; nullifier stored so one human = one secondary purchase (configurable cap).

**What this is NOT:** there is no hard on-chain price cap on resales. The royalty and the World ID cap together create strong disincentives for scalping, but a determined buyer can still overpay once. That framing matters for the pitch.

---

## 2. Prize targets

| Prize | Track | Why we qualify |
|---|---|---|
| Hedera Tokenization | $3,000 — anchor | Custom fees / royalties, NFT mint, on-chain transfer |
| World ID Track B | $2,500 | Onboarding + login + resale require World ID; nullifier enforces one wallet + secondary cap |
| ENS pool | Small bolt-on | ENS subdomain as a human-readable ticket address |
| Hedera No-Solidity | $3,000 — fallback | HCS used as an audit log (second native Hedera service, zero Solidity) |

---

## 3. Tech stack

- **Runtime:** Node.js, ES modules (`"type": "module"` in package.json)
- **App:** Next.js 16 (App Router) — single server for UI + API routes
- **Hedera SDK:** `@hashgraph/sdk` v2.81+
- **Network:** Hedera testnet (custodial accounts, operator pays all fees)
- **Payment:** HBAR for the core sale; USDC planned later
- **Proof of human:** `@worldcoin/idkit` v4 — IDKit widget + backend verify via `/api/v4/verify/{rp_id}`
- **Database:** SQLite (`better-sqlite3`) — users, tokens, tickets, ownership in `data/users.db`

> **SDK name trap:** The Hedera Playground and older snippets use `@hiero-ledger/sdk`. This repo uses `@hashgraph/sdk`. The API is identical; only the import changes. Don't mix them.

---

## 4. Architecture & conventions

```
app/
  page.jsx                              ← Marketplace (mint-on-buy at face value)
  login/page.jsx                        ← Returning user: World ID → existing account
  onboard/page.jsx                      ← New user: World ID → create account + role
  wallet/page.jsx                       ← My tickets, resale UI
  organizer/page.jsx                    ← Create collections (maxSupply + faceValueHbar)
  components/Nav.jsx                    ← Global nav, log out
  lib/storage.js                        ← localStorage session (ticket_account_id)
  api/
    login/route.js                      ← verify proof → lookup existing user
    verify-and-onboard/route.js         ← verify proof → createUserAccount(60)
    world-id/sign/route.js              ← RP signature for IDKit v4
    marketplace/route.js                ← collections with faceValueHbar, remaining
    tokens/route.js                     ← POST create collection (organizer)
    tokens/[tokenId]/buy/route.js       ← mint-on-buy primary purchase
    tickets/[tokenId]/[serial]/resell/  ← atomic resale + World ID on buyer
    wallet/[accountId]/route.js         ← tickets + ownership history

src/
  client.js                             ← shared Hedera client factory
  state.js                              ← loadState() / saveState() for state.json
  lib/auth.js                           ← requireUser(), requireRole()
  db/
    schema.sql                          ← users, tokens, tickets, ownership
    users.js, tokens.js, tickets.js, db.js
  world/
    verifyProof.js                      ← verifyWorldIdProof(), extractNullifier()
  hedera/
    createToken.js                      ← createTicketToken() — NFT collection + royalty
    createAccount.js                    ← createUserAccount(initialHbar) — custodial wallet
    mintTicket.js                       ← mintTickets(tokenId, supplyKey, pointers)
    primaryPurchase.js                  ← mint → primarySale → DB (sold_primary)
    transferTicket.js                   ← primarySale(), atomicResale()
    compliance.js                       ← freezeHolder/unfreezeHolder/pauseToken/unpauseToken

scripts/
  01-check-balance.js                   ← prove .env + network; print operator HBAR
  02-create-token.js                    ← [maxSupply] [faceValueHbar] [name] [symbol]; requires operator seeded in DB (run app once first)
  03-create-account.js                  ← dev bypass: create buyer without World ID; writes buyer → state.json
  04-mint.js                            ← DEPRECATED (superseded by mint-on-buy in 05)
  05-primary-sale.js                    ← CLI mint-on-buy; requires tokenId + buyer in state.json
  06-resale.js                          ← CLI atomic resale; args: [serial] [priceHbar] [sellerAccountId] [buyerAccountId]
  07-scan-gate.js                       ← gate entry: freeze holder on-chain + mark ticket used; arg: [serial]
  08-pause-match.js                     ← pause/unpause whole token (cancelled match); arg: [unpause]
  09-unfreeze.js                        ← reset helper: unfreeze holder + status→owned; arg: [serial]
  promote-organizer.js                  ← set role=organizer for any account_id in DB
  reset-db.js                           ← delete data/users.db; restart npm run dev to re-seed operator
```

**Rule:** `src/hedera/` functions are pure — no `console.log`, no `process.env` reads, they close the Hedera client before returning. Scripts and API routes are the runners.

**`state.json`** — local store for CLI scripts. Holds `tokenId`, token keys, latest `buyer`. Gitignored.

**`data/users.db`** — SQLite for all app state. Gitignored. Contains custodial private keys — treat as secrets even on testnet.

---

## 5. User roles

| Role | How assigned | Can do |
|---|---|---|
| `organizer` | Selected on `/onboard` or promoted via `ADMIN_SECRET` | Create collections, receive primary sale HBAR + royalties |
| `purchaser` | Default on onboard | Buy tickets at face value |
| `reseller` | Auto-promoted after first primary purchase | Resell owned tickets |

Operator account (`OPERATOR_ID`) is auto-seeded as `organizer` on first DB init.

---

## 6. Pricing

| Term | Set by | Stored in | Used for |
|---|---|---|---|
| **Face value** | Organizer at collection creation | `tokens.primary_price_hbar` | Primary marketplace buy |
| **Resale price** | Seller on `/wallet` | `ownership.price_hbar` on secondary transfer | Fan-to-fan resales |

Primary buy API ignores any client-sent price — always uses `token.primary_price_hbar`.

---

## 7. Accounts vs tokens (HBAR)

- **Account** — Hedera wallet (`0.0.xxxxx`). Holds HBAR balance and NFT serials.
- **Token** — NFT collection ID (`0.0.yyyyy`). A definition only; does not hold HBAR.

On onboarding, `createUserAccount(60)` transfers 60 HBAR from the **operator** into the new user's account (starter funds for fees + one ticket purchase). Token creation only pays a small network fee — no 60 HBAR transfer.

---

## 8. Auth & session

Browser session = `localStorage` key `ticket_account_id`. Not server-side sessions.

| Flow | Endpoint | Result |
|---|---|---|
| **Create wallet** | `POST /api/verify-and-onboard` | New Hedera account if nullifier unseen; 409 → use login |
| **Log in** | `POST /api/login` | Restore existing account by nullifier; 404 if never onboarded |
| **Log out** | Nav button | Clears localStorage → `/login` |

World ID ties one human to one account permanently. Log out only clears the browser; the same person logging in again gets the same account.

---

## 9. Sale flows

### Primary (mint-on-buy)

```
Purchaser → POST /api/tokens/{tokenId}/buy
  → primaryPurchase()
    → mintTickets(1 serial)
    → primarySale(NFT + HBAR at face value)
    → DB: status sold_primary, acquisition primary
```

Organizer treasury receives face-value HBAR. No royalty on primary sale (royalty is for resales).

### Secondary (resale)

```
Seller on /wallet → buyer account ID + resale price → Resell
  → buyer scans World ID
  → POST /api/tickets/{tokenId}/{serial}/resell
    → verify proof, check secondary cap on buyer nullifier
    → atomicResale(NFT + HBAR in one TransferTransaction)
    → 10% royalty to organizer on-chain automatically
    → DB: status sold_secondary, acquisition secondary
```

Hackathon UX: seller initiates and enters buyer's account ID manually. No public resale listings page yet.

---

## 10. Key Hedera facts

- **Token keys are permanent.** All keys (admin, supply, freeze, pause, metadata) must be set at creation time.
- **NFT initial supply must be 0.** `setInitialSupply(0)` required for `NonFungibleUnique`.
- **NFT metadata ≤100 bytes.** Short pointer URL only (e.g. `/api/tickets/{tokenId}/{serial}`).
- **Royalty only fires on atomic swap.** NFT + HBAR in the same `TransferTransaction`. Plain NFT transfer triggers fallback fee (5 HBAR).
- **Mirror Node lags ~5 seconds.** Wait before querying EVM address after account creation.
- **Compliance layer:** Freeze is account-level, not per-serial — freezing a holder blocks transfer of *all* their tickets for that token. A frozen holder = entered/used = can't resell (rejected on-chain with `ACCOUNT_FROZEN_FOR_TOKEN`). Pause blocks the whole token (`TOKEN_IS_PAUSED`). Verified end-to-end on testnet.

---

## 11. Environment variables

```env
OPERATOR_ID=0.0.xxxx
OPERATOR_KEY=0x...

WORLD_APP_ID=app_xxx
WORLD_RP_ID=rp_xxx
WORLD_RP_SIGNING_KEY=0x...
WORLD_ACTION=ticket-onboarding
WORLD_ENVIRONMENT=production

NEXT_PUBLIC_WORLD_APP_ID=app_xxx
NEXT_PUBLIC_WORLD_ACTION=ticket-onboarding
NEXT_PUBLIC_WORLD_ENVIRONMENT=production

APP_BASE_URL=http://localhost:3000
ADMIN_SECRET=change-me
SECONDARY_PURCHASE_CAP=1
```

For phone demos use `production`. For local dev use `staging` + World ID Simulator.

---

## 12. Getting started

```bash
cd ~/Documents/Project/fake-repo
npm install
cp .env.example .env
# fill in .env

npm run dev
# → http://localhost:3000
```

**Demo path (two users):**
1. User A → `/onboard` → Organizer → `/organizer` → create collection (face value e.g. 50 HBAR)
2. User B → `/onboard` → Purchaser → `/` → Buy
3. User B → `/wallet` → enter User C account ID + resale price → Resell → User C scans World ID

**CLI alternative** (all scripts require the app to have run at least once so the operator is seeded in the DB):
```bash
node scripts/01-check-balance.js                          # verify .env + connectivity

node scripts/02-create-token.js 100 50 "World Cup Ticket" WCT  # writes state.json + DB

node scripts/03-create-account.js                         # dev-only buyer (skips World ID); writes buyer → state.json

node scripts/05-primary-sale.js                           # reads tokenId + buyer from state.json

node scripts/06-resale.js 1 75 0.0.seller 0.0.buyer       # serial priceHbar sellerAccountId buyerAccountId

node scripts/promote-organizer.js 0.0.xxxx                # elevate any DB account to organizer

node scripts/reset-db.js                                  # wipe DB; restart npm run dev to re-seed
```

---

## 13. Status & next steps

**Done**
- [x] Hedera client, state helpers, SQLite schema (users, tokens, tickets, ownership)
- [x] World ID onboarding + login + logout
- [x] Roles: organizer / purchaser / reseller
- [x] Token creation with 10% royalty + organizer face value
- [x] Mint-on-buy primary sales at face value
- [x] Atomic resale with on-chain royalty + World ID on buyer + secondary cap
- [x] Marketplace, wallet, organizer UI
- [x] CLI scripts 01–03, 05–06, promote-organizer, reset-db

**Up next**
1. Public resale listings marketplace (seller lists, buyer browses — no manual account ID entry)
2. HCS audit log — every ticket event on a Consensus Service topic (No-Solidity prize)
3. Gate scan + ENS — QR at venue, ENS subdomain as ticket identity
4. Tie World ID proof nullifier to buyer account ID on resale (currently only checks secondary cap)

---

## 14. Security

- **Testnet only.** Never use mainnet keys in this codebase.
- **Never commit `.env`.** Operator key, World ID signing key, admin secret stay server-side.
- **`data/users.db` is gitignored.** Custodial private keys for all users. Do not commit.
- **`state.json` is gitignored.** Token supply/admin keys for CLI. Losing it means you cannot mint more for that token.
- **`WORLD_RP_SIGNING_KEY` is server-only.** Never expose via `NEXT_PUBLIC_*`.
- For phone testing, add LAN IP to `allowedDevOrigins` in `next.config.js`.

---
> Source: [vedantMahangade/fake-repo](https://github.com/vedantMahangade/fake-repo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
