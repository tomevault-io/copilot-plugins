## xenga

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run Commands

```bash
# Facilitator (Express + SQLite) — handles all chain interaction
bun run dev                    # Dev mode with watch (port 3000)
bun run start                  # Production start

# Web (Next.js 15) — pure frontend, calls facilitator API
cd web && bun run dev          # Dev mode (port 3001)
cd web && bun run build        # Production build
cd web && bun run start        # Production start

# Local development (two terminals):
# Terminal 1: bun run dev
# Terminal 2: cd web && NEXT_PUBLIC_FACILITATOR_URL=http://localhost:3000 bun run dev

# Contracts (Foundry)
bun run build:contracts        # forge build (from contracts/)
bun run test:contracts         # forge test (from contracts/)
cd contracts && forge test --match-test test_specificName  # Run single test
cd contracts && forge test -vvvv  # Verbose output with traces

# ABI sync (run after contract changes)
bun run build:contracts && bun run sync-abi

# Docker (facilitator only)
docker build -t xenga-facilitator .
docker run -p 8080:8080 --env-file .env xenga-facilitator
```

## Architecture

On-chain escrow and reputation system on Base Sepolia using USDC (ERC-3009 gasless transfers). The Xenga protocol provides the HTTP integration layer. Contracts use **UUPS upgradeable proxy pattern** (ERC1967Proxy + `initialize()`) for safe on-chain upgrades.

**Split deployment:**
- **`web/`** — Next.js 15 frontend deployed on **Vercel**. Pure client-side: pages, wallet management, EIP-712 signing, Protocol Inspector. Calls the facilitator API via `NEXT_PUBLIC_FACILITATOR_URL`.
- **`src/server/`** — Express facilitator deployed on **Fly.io**. Handles all chain interaction: settlement, event listening, reputation, order management, SQLite DB. Runs with `PRIVATE_KEY` for gas.

```
Vercel (web/)                    Fly.io (src/server/)
┌──────────────────┐             ┌──────────────────────────┐
│ Next.js Frontend │   fetch     │ Express Facilitator      │
│ Pages + Signing  │────────────>│ REST API + Chain + SQLite│
└──────────────────┘   CORS      └──────────────────────────┘
        │                                   │
        │ signTypedData                     │ PRIVATE_KEY (gas)
        ▼                                   ▼
   User's Browser                  EscrowVault (Base Sepolia)
```

**Other layers:**
- **`contracts/`** — Foundry project: EscrowVault (escrow state machine + stats + UUPS upgradeable), SessionEscrow (session micropayments + UUPS upgradeable), MockUSDC (test token). Deployed behind ERC1967Proxy — `initialize()` replaces constructors. Storage gap (`uint256[48] private __gap;`) reserves slots for future upgrades.
- **`src/client/`** — Client SDK: EIP-712 signing, reputation lookup (`getReputation()`), and xenga payment flow (`escrowFetch` with optional `onSellerReputation` callback)

**Shared code** (`src/shared/`): types, constants, EIP-712 domain/types, and auto-generated ABIs (`abi.ts` — never edit manually, use `sync-abi`). The web app imports `@shared/` via webpack alias for types and EIP-712 signing functions (client-safe, no server deps).

### Escrow Lifecycle (On-Chain)

```
None → Active → DeliveryConfirmed → Completed      (buyer releases)
         │             │              AutoReleased   (timeout, facilitator poller triggers)
         │             └────────────→ Disputed ──→ Resolved (arbiter splits %)
         └───────────────────────────→ Refunded   (seller voluntary / arbiter)
```

- **Active**: escrow created, USDC locked, `contentHash` stored on-chain (keccak256 of order terms/metadata). Buyer can release anytime, seller/facilitator can confirm delivery or refund.
- **DeliveryConfirmed**: seller/facilitator confirmed delivery, dispute window starts. Buyer can release or dispute.
- **AutoRelease timing**: From Active state, requires `releaseWindow + disputeWindow`. From DeliveryConfirmed, requires `releaseWindow` from creation AND `disputeWindow` from delivery confirmation.
- **Dispute timing**: From DeliveryConfirmed, within `disputeWindow` of confirmation. From Active, between `releaseWindow - disputeWindow` and `releaseWindow + disputeWindow` from creation.
- **disputeWindow**: Owner-settable global default (default: 3 days, bounds: 1 hour–30 days). Set via `EscrowVault.setDisputeWindow()`. Stored per-escrow at creation — existing escrows keep their original value.
- **Resolved**: arbiter splits funds by buyer percentage (0-100). Fee goes to feeRecipient, split applies to `amount - fee`.
- **Refunded**: buyer gets full deposit back including fee — facilitator absorbs cost.

### Reputation System

On-chain credit scoring for agents/wallets, computed from escrow transaction history. The facilitator tracks both `sellerStats` and `buyerStats` on-chain (in `EscrowVault.sol`), and the server computes weighted reputation scores from this data.

**Architecture:**
- **On-chain**: `buyerStats[address]` and `sellerStats[address]` mappings in EscrowVault track totalEscrows, completedCount, disputedCount, refundedCount, resolvedCount, and amounts. Raw data is permissionless — anyone can read and compute their own scores.
- **Off-chain**: `reputationService.ts` computes weighted scores (0-100) with confidence levels, cached 60s in-memory.
- **Dynamic params**: Service types use `adjustParams()` to shorten/extend release windows based on counterparty reputation (e.g. high-trust pairs get 3-day instead of 7-day marketplace window).
- **Client integration**: Seller reputation is included in payment responses; clients can check via `onSellerReputation` callback before paying.

**Scoring formulas** (each component's max weight sums to 100):
- Seller: completionRate×40 + (1-disputeRate)×25 + (1-refundRate)×15 + resolutionFairness×10 + volumeBonus (0-10, log-scaled)
- Buyer: completionRate×45 + (1-disputeRate)×25 + (1-frivolousDisputeRate)×20 + volumeBonus (0-10, log-scaled)
- Scores are clamped to [0, 100]. `resolutionFairness` defaults to 1.0 (clean record) when seller has no resolved disputes.
- Overall score: escrow-count-weighted average of seller + buyer scores (not simple average).
- Confidence: `"low"` (<3 escrows), `"medium"` (3-9), `"high"` (≥10)

**Key files:**
- `contracts/src/EscrowVault.sol` — `buyerStats` mapping + `getBuyerStats()` view
- `src/server/services/reputationService.ts` — `computeReputation()`, `computeReputationHistory()`
- `src/server/routes/reputation.ts` — `GET /api/reputation/:address`, `GET /api/reputation/:address/history`
- `src/shared/types.ts` — `ReputationScore`, `SellerReputation`, `BuyerReputation`

### Fee System

The facilitator pays all gas fees for on-chain transactions (createEscrowWithAuth, confirmDelivery, resolveDispute, refund). To cover costs and earn margin, a configurable facilitator fee is deducted from the seller's payout at settlement (seller-pays model, like Stripe/PayPal).

**How it works:**
- `feeBps`, `flatFee`, and `feeRecipient` are global contract state, set by owner via `setFeeConfig()`
- Fee is computed at escrow creation: `fee = (amount * feeBps) / 10000 + flatFee`, stored in `Escrow.facilitatorFee`
- The percentage component (`feeBps`) covers facilitator profit/margin; the fixed component (`flatFee`) covers gas costs — similar to Stripe's `2.9% + $0.30` model
- Buyer pays exactly `orderPrice` — no amount inflation
- On release/autoRelease: seller gets `amount - fee`, feeRecipient gets `fee`
- On refund: buyer gets full `amount` back (facilitator absorbs cost)
- On dispute resolution: fee goes to feeRecipient, `buyerPct` split applies to `amount - fee`
- `MAX_FEE_BPS = 1000` (10% cap), `MAX_FLAT_FEE = 50_000_000` (50 USDC cap)
- Safety guards: `if (fee >= amount) revert InvalidFee()` prevents tiny escrows where fee exceeds deposit; `if (fee > 0 && feeRecipient != address(0))` prevents revert if feeRecipient changed to zero while escrows are active
- Stats track gross `amount` (not net) — reputation scoring uses transaction volume

**Config:** `FEE_BPS`, `FEE_FLAT_USDC`, and `FEE_RECIPIENT` env vars (see `.env.example`). Defaults: 0 (no fee).

### Post-Deployment Configuration

All owner-callable setters. Ownership uses `Ownable2Step` — transfer requires a 2-step confirmation (e.g. `transferOwnership(gnosisSafeAddress)` then `acceptOwnership()` from new owner).

| Config | Contract | Setter | Default | Bounds |
|---|---|---|---|---|
| Arbiter address | EscrowVault | `setArbiter(address)` | deployer | — |
| Fee config | EscrowVault | `setFeeConfig(recipient, bps, flat)` | 0 | max 10% + 50 USDC |
| Dispute window | EscrowVault | `setDisputeWindow(uint256)` | 3 days | 1 hour – 30 days |
| Pause / unpause | EscrowVault, SessionEscrow | `pause()` / `unpause()` | unpaused | — |
| Facilitator address | EscrowVault, SessionEscrow | `setFacilitator(address)` | address(0) | — |
| UUPS upgrade | EscrowVault, SessionEscrow | `upgradeTo(address)` / `upgradeToAndCall(address, bytes)` | — | Owner only |

**UUPS Upgrade procedure:**
1. Deploy the new implementation contract (do NOT call `initialize()` on the implementation directly).
2. Call `upgradeTo(newImplementation)` or `upgradeToAndCall(newImplementation, data)` on the **proxy** address. Only the owner can upgrade.
3. The proxy address stays the same — all state is preserved. Use `Upgrade.s.sol` for scripted upgrades.
4. **Safety**: Always verify storage layout compatibility before upgrading. The `__gap` (48 slots) provides room for new state variables without colliding with existing storage. Never reorder or remove existing state variables.

**Design note — why `releaseWindow` is per-escrow but `disputeWindow` is a global default:**
`releaseWindow` is a business timing parameter that must vary by service type (1h for agent-service, 7d for marketplace). It is computed by the server per-escrow from service types + reputation and stored in the escrow struct. `disputeWindow` is a consumer protection parameter — a uniform "cooling off period" — set globally by the owner so it cannot be manipulated by the facilitator on a per-escrow basis.

### Service Types

Service types (`src/server/service-types/`) define escrow parameters per use case:
- **marketplace**: 7-day release window, manual delivery confirmation
- **agent-service**: 1-hour release window, auto-verify delivery

Each service type can implement `adjustParams(params, reputation)` to dynamically adjust escrow parameters (e.g. release window) based on counterparty reputation scores.

### Auto-Release Poller

The facilitator runs a built-in poller (`src/server/services/autoReleasePoller.ts`) that checks escrowed orders every 60 seconds and calls the permissionless `autoRelease()` on EscrowVault when `isReleasable()` returns true. No external automation (Chainlink, cron) needed — the facilitator already pays gas.

### Xenga Integration Layer

HTTP transport for triggering escrow creation. x402-compatible — any x402 agent can pay without Xenga-specific code. The contracts can also be called directly.

1. Client POSTs to a payment-protected endpoint without payment header
2. Middleware returns **402** with `PAYMENT-REQUIRED` header (x402 envelope) and `X-PAYMENT-REQUIRED` header (legacy Xenga format)
3. Client signs ERC-3009 `ReceiveWithAuthorization` via EIP-712 (USDC gasless transfer to EscrowVault)
4. Client retries with `PAYMENT-SIGNATURE` header (x402 format) or `X-PAYMENT` header (legacy)
5. Server normalizes payload (`normalizePaymentPayload` handles both formats), verifies signature off-chain, submits `createEscrowWithAuth` on-chain
6. Returns **200** with `PAYMENT-RESPONSE` header (x402 format) and `X-PAYMENT-RESPONSE` header (legacy)

**x402 compatibility:** The `PAYMENT-REQUIRED` header contains a standard x402 envelope (`{ x402Version: 1, accepts: [{ scheme: "escrow", ... }] }`). Escrow-specific fields (`orderId`, `sellerAddress`, `releaseWindow`, `serviceType`) are in the `extra` object. `extra.primaryType: "ReceiveWithAuthorization"` tells x402 clients which EIP-712 type to sign.

**Key files:**
- `src/server/middleware/paymentCore.ts` — `processEscrowPayment()`, `normalizePaymentPayload()`, `buildPaymentRequiredResponse()`
- `src/server/middleware/escrowPayment.ts` — Express adapter
- `src/shared/types.ts` — `X402PaymentRequirements`, `X402PaymentPayload`, `X402SettlementResponse`
- `src/client/escrowFetch.ts` — `unwrapPaymentRequired()` (handles x402 + legacy)

### Seller Dashboard

Self-service dashboard at `/dashboard` for sellers to manage orders, profile, and API keys.

**Backend:**
- `src/server/routes/sellerApiKeys.ts` — CRUD for self-service API keys (POST create, GET list, DELETE revoke), all behind `sessionAuth()`. Exports `hashApiKey()` utility used by auth middleware.
- `src/server/middleware/auth.ts` — `apiKeyAuth()` checks env var keys first, then falls back to `seller_api_keys` DB table (SHA256 hash lookup). `apiKeyOrSessionAuth()` combined middleware accepts API key or session JWT. `walletAuth()` uses `req.originalUrl` (not `req.path`) for routeId to match client signatures correctly on mounted routers.
- `src/server/db/schema.ts` — `seller_api_keys` table (id, seller_address, key_hash, key_prefix, name, timestamps), `sellers` table (with `payout_address`), `webhooks` table (with `seller_address`), `payment_links` table
- `src/server/routes/sellers.ts` — POST uses upsert (`ON CONFLICT...DO UPDATE`) for register + profile update in one endpoint. Accepts `payoutAddress` for custom USDC payout destination.
- `src/server/routes/orders.ts` — GET `?seller=` with wallet headers verifies signer matches seller param; confirm-delivery and refund use `apiKeyOrSessionAuth` with seller identity check (facilitator pays gas)
- `src/server/routes/webhooks.ts` — CRUD for webhook endpoints, scoped per seller via `apiKeyOrSessionAuth()`. Seller can only see/manage their own webhooks.
- `src/server/routes/paymentLinks.ts` — CRUD for payment links (sessionAuth), plus public details and checkout endpoints (no auth, rate limited). Checkout creates orders from link data, resolving seller payout address.

**Frontend (`web/`):**
- `/dashboard` — Overview: stats row (active, pending, revenue, reputation), activity feed, quick actions
- `/dashboard/orders` — Order table with filter tabs (All/Active/Completed/Disputed), inline expand with escrow details, confirm delivery + refund via facilitator API (gas-free for sellers)
- `/dashboard/settings` — Seller profile registration/update (name, editable payout address)
- `/dashboard/api-keys` — Create, list, revoke API keys with copy-once-on-create pattern
- `/dashboard/webhooks` — Register, list, delete webhook endpoints with event type selection and secret generation
- `/dashboard/payment-links` — Create, list, deactivate payment links with copy URL
- `/pay/[id]` — Public checkout page for payment links (standalone, uses demo wallet)
- Layout: `DashboardSidebar` (responsive with mobile hamburger) + `WalletGate` (connect prompt when disconnected)
- `web/lib/api/wallet-auth.ts` — `authenticatedFetch()` adds session JWT to facilitator API calls

**Auth flow:** Client signs `xenga-auth:{path}:{timestamp}` via `walletClient.signMessage()`. Server verifies via `verifyMessage()` with 5-minute replay window. Addresses normalized to lowercase for storage/comparison.

## Web (`web/`)

Next.js 15 App Router frontend. Deployed on Vercel. Calls the Express facilitator API for all backend operations — no server-side code, no SQLite, no chain interaction.

### Structure

```
web/
  app/
    page.tsx                 # Landing page
    playground/
      page.tsx               # Playground hub (links to demos)
      marketplace/page.tsx   # Interactive marketplace demo
      agent/page.tsx         # Auto-advancing agent service demo
    marketplace/page.tsx     # Redirect → /playground/marketplace
    agent/page.tsx           # Redirect → /playground/agent
    dashboard/
      layout.tsx             # Sidebar + WalletGate wrapper
      page.tsx               # Overview (stats, activity feed)
      orders/page.tsx        # Order management with on-chain actions
      settings/page.tsx      # Seller profile registration
      api-keys/page.tsx      # API key self-service
      webhooks/page.tsx      # Webhook management
      payment-links/page.tsx # Payment link management
    pay/[id]/page.tsx        # Public payment link checkout
    seller/page.tsx          # Redirects to /dashboard
  components/
    landing/                 # Hero, ProtocolFlow, DemoCards, HowItWorks, Footer
    marketplace/             # PaymentFlow, ProductGrid, StepTracker, SellerPanel
    agent/                   # AgentTerminal
    dashboard/               # WalletGate, DashboardSidebar, StatsRow, ActivityFeed, OrderTable, OrderActions, SellerProfile, ApiKeyManager, WebhookManager, PaymentLinkManager
    payment-links/           # PaymentLinkCheckout (public checkout component)
    protocol-inspector/      # InspectorPanel + 4 tab components
    ui/                      # Badge, AddressDisplay, TxLink, UsdcAmount, JsonViewer, ReputationBadge
    layout/                  # Navbar
  lib/
    api/client.ts                 # facilitatorFetch() + facilitatorUrl() — all API calls go through here
    api/wallet-auth.ts            # authenticatedFetch() — adds wallet auth headers for dashboard API calls
    api/payment-flow.ts           # Decomposed xenga client flow with inspector hooks
    wallet/WalletProvider.tsx     # Demo wallet (sessionStorage) + browser wallet (MetaMask)
    protocol-inspector/context.tsx # Inspector event bus + auto-tab-switching
    env/isMockChainClient.ts      # Client-side mock chain detection
    utils.ts                      # cn(), shortenAddress(), formatUsdc()
```

### Key Configuration

- **`next.config.ts`**: webpack alias (`@shared/` → `../src/shared/`), `extensionAlias` (`.js` → `.ts` for shared code), TS loader for `../src/shared`
- **`tsconfig.json`**: path alias `@shared/*` → `../src/shared/*`, `moduleResolution: bundler`
- **Environment**: `NEXT_PUBLIC_FACILITATOR_URL` (required for production, empty for same-origin dev)

### Web Technical Notes

- **API client**: All `fetch()` calls go through `lib/api/client.ts` which prepends `NEXT_PUBLIC_FACILITATOR_URL`. When empty (local dev), paths are relative.
- **`@shared/` imports**: Frontend imports types (`ReputationScore`) and pure functions (`buildReceiveAuthSigningParams`) from `src/shared/` via webpack alias. These have no server dependencies.
- **Wallet**: `WalletProvider` manages ephemeral demo wallets (`generatePrivateKey()` stored in `sessionStorage`) and browser wallets (`window.ethereum`). Both expose viem `WalletClient`.
- **Protocol Inspector**: React context + 4-tab panel showing HTTP traffic, EIP-712 signatures, on-chain transactions, and escrow state machine. Events emitted by `payment-flow.ts` during the xenga flow.
- **CORS**: Required since frontend and facilitator are on different origins. Express facilitator has `CORS_ORIGIN` env var (defaults to `*`).
- **Demo funding**: `POST /api/demo/fund` on the facilitator sends 10 USDC + 0.005 ETH from operator wallet. Rate-limited to 100 USDC/hr per IP+address. Automatically available on testnets (detected via `chainConfig.isTestnet`).

## Key Technical Notes

- **`via_ir = true`** in `foundry.toml` is required — OpenZeppelin contracts cause "stack too deep" without it
- **`@types/express` v5**: `req.params` values are `string | string[]`, cast to `string` when needed
- **Foundry tests**: default `block.timestamp` is 1 (not 0); use explicit absolute timestamps with `vm.warp()` rather than relative offsets from captured `block.timestamp` (via_ir can change evaluation order)
- **ABI source of truth**: Foundry artifacts in `contracts/out/` → run `sync-abi` to regenerate `src/shared/abi.ts`
- **`disputeWindow` vs `releaseWindow`**: `releaseWindow` is per-escrow (set at creation from service type config). `disputeWindow` is a global owner-set default (applies to all new escrows, stored in each escrow struct at creation). Changing it post-deployment does not affect existing escrows.
- **Workspaces**: root `package.json` has `"workspaces": ["packages/*", "web"]`; run `bun install` from root to link
- **UUPS proxy pattern**: EscrowVault and SessionEscrow are deployed behind ERC1967Proxy. Constructors are disabled (`_disableInitializers()` in constructor); use `initialize()` instead. Deploy scripts (`Deploy.s.sol`, `DeployLocal.s.sol`) deploy implementation + proxy. `Upgrade.s.sol` handles upgrades. Never call `initialize()` on the implementation contract directly — only on the proxy.
- **Storage gap**: Both contracts include `uint256[48] private __gap;` to reserve storage slots for future upgrades. When adding new state variables, reduce the gap size accordingly (e.g., adding 2 new `uint256` variables means changing `__gap` from `[48]` to `[46]`).
- **`contentHash`**: `bytes32` field in the Escrow struct, set at creation time. Contains `keccak256` of the order terms/content metadata. Emitted in the `EscrowCreated` event. Provides tamper-proof evidence of agreed terms for dispute resolution. A value of `bytes32(0)` means no content hash was provided.

## Environment

### Facilitator (`src/server/`)
Requires `.env` (see `.env.example`): `PRIVATE_KEY`, `ESCROW_VAULT_ADDRESS`, optionally `BASE_SEPOLIA_RPC`, `PORT`, `CORS_ORIGIN`.

### Web (`web/`)
Requires `NEXT_PUBLIC_FACILITATOR_URL` pointing to the facilitator. For local dev, set to `http://localhost:3000` or leave empty if running on same origin.

### Docker
```bash
docker build -t xenga-facilitator .
docker run -p 8080:8080 --env-file .env xenga-facilitator
```

## Constants

- Chain: Base Sepolia (84532)
- USDC: `0x036CbD53842c5426634e7929541eC2318f3dCF7e` (6 decimals)
- Runtime: Bun

---
> Source: [uhyunpark/xenga](https://github.com/uhyunpark/xenga) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
