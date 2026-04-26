## discordcash

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

DiscordCash — a Zcash tip bot for Discord with Cashu blind token privacy. Users react with 💰 to tip. The server signs tokens it cannot read. Tip amounts are hidden; only tip counts are tracked.

## Running Locally

```bash
# Install deps
pip install -r requirements.txt

# Set up .env (copy from .env.example)
# Required: DISCORD_TOKEN
# Set MOCK_MODE=true to test without a Zcash node

# Run
cd zcash-tip-bot
python bot.py
```

The bot starts a web server on port 3000 alongside the Discord connection. Landing pages at `localhost:3000`.

Mock mode enables `/mockdeposit` to grant fake ZEC for testing.

## Building & Deploying

```bash
# Docker build
docker build -t discordcash .

# Push triggers GitHub Actions → builds linux/amd64 image → pushes to ghcr.io/jayscentco/discordcash:latest

# Phala Cloud deployment (TEE)
phala deploy -c docker-compose.yml -n discordcash \
  -e DISCORD_TOKEN=... -e MOCK_MODE=true \
  -e DSTACK_DOCKER_USERNAME=jayscentco \
  -e DSTACK_DOCKER_PASSWORD=$(gh auth token) \
  -e DSTACK_DOCKER_REGISTRY=ghcr.io
```

## Architecture

### Data Flow: Reaction Tip
```
💰 reaction → on_raw_reaction_add()
  → wallet.load_proofs(tipper_id)         # load blind tokens from DB
  → wallet.prepare_send(proofs, amount)   # select tokens, create blinded change outputs
  → mint.swap_tokens(inputs, outputs)     # validate, mark spent, sign new outputs
  → wallet.receive(send_proofs)           # recipient re-blinds (prevents double-spend)
  → save new proofs for both users        # update user_tokens table
  → record_tip_count(from, to)            # count only, no amount
```

### Cashu Blind Signatures (BDHKE on secp256k1)

The core privacy mechanism. Three steps:

1. **Alice blinds**: `B_ = hash_to_curve(secret) + r*G` — mint can't see the secret
2. **Bob signs blind**: `C_ = k * B_` — mint signs what it can't read
3. **Alice unblinds**: `C = C_ - r*K` — final token the mint never saw

Key types: `Proof(amount, secret, C, keyset_id)`, `BlindedMessage(amount, B_, keyset_id)`, `BlindSignature(amount, C_, keyset_id)`

Denominations are powers of 2: `[1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024]` zats. 1 zat = 0.0001 ZEC.

### Module Responsibilities

- **bot.py**: Discord events, slash commands, background tasks. Orchestrates wallet and mint operations. All tipping logic lives here.
- **cashu_mint.py**: `Mint` (server-side signing, spent secret tracking), `Wallet` (blinding/unblinding, proof storage), and BDHKE crypto primitives. ~700 lines of cryptographic core.
- **database.py**: Users table, tip_counts table. No amounts stored anywhere.
- **web.py**: aiohttp server serving landing pages and `/api/leaderboard` JSON endpoint.
- **zcash_client.py**: Wraps zcashd JSON-RPC. Swaps to `MockZcashClient` when `MOCK_MODE=true`.
- **zcash_mock.py**: Fake Zcash client returning mock addresses and instant "success" for all operations.

### Database Tables

- `users`: discord_id, deposit_address, zaddress, default_tip
- `tip_counts`: from_user, to_user, created_at (no amounts)
- `user_tokens`: discord_id, secret, C, amount, keyset_id (blind token proofs)
- `spent_secrets`: secret (double-spend prevention)
- `mint_keysets`: id, private_keys (JSON), active

### Privacy Model

| What | Visibility |
|---|---|
| Who tipped whom | Visible (Discord reaction) |
| Tip amounts | Hidden (blind signatures) |
| User balances | Bot knows (stores proofs) |
| Deposit→tip→withdrawal trail | Can't be traced (blind sigs break the link) |
| On-chain deposits/withdrawals | Hidden (Zcash shielded pool) |
| /anontip | Fully hidden (wallet-to-wallet ZIP-321) |

### Port Usage

The bot listens on port 3000 for the web server. When deploying to Phala Cloud, port 3000 must be exposed in docker-compose.yml (ports 8080 and 8090 conflict with Phala's internal services).

### Guild-Specific Command Sync

Slash commands sync to guild ID `1429785830389448749` for instant availability. Global sync is disabled (conflicts with Discord's Activity entry point command).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jayscentco) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
