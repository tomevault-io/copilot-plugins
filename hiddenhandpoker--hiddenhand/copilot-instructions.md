## hiddenhand

> **The card-privacy backend is now Arcium MPC, NOT Inco TEE + MagicBlock VRF.** The

# HiddenHand - Privacy Poker on Solana

## ⚡ CURRENT STATE (2026) — Arcium MPC (READ THIS FIRST)

**The card-privacy backend is now Arcium MPC, NOT Inco TEE + MagicBlock VRF.** The
hackathon-era sections further down describe the retired Inco/VRF design and are kept
only as historical context. Where they conflict with this banner, this banner wins.

**Why the switch:** the old design had a SEV-HIGH flaw — the deck was reconstructable
from the public VRF callback randomness, so "encryption" was theater against a chain
observer. Arcium shuffles inside MPC; randomness never touches the chain; the deck lives
on-chain only as an opaque MXE ciphertext.

**Architecture (the card lifecycle = 6 Arcium MPC circuits in `encrypted-ixs/src/lib.rs`):**
- `shuffle` → shuffles the 52-card deck in-MPC, seals it to the MXE, stored on-chain in
  `DeckState.deck` (`[[u8;32];2]`) + `deck_nonce`. Re-fed unchanged into every later circuit.
- `deal_to_seat` (once per seated player) → seals that seat's 2 hole cards to the player's
  own x25519 key. Emitted via the `HoleDealt` event; only that player can decrypt (client-side
  RescueCipher). Deck positions `2i, 2i+1` for seat `i`; board at 18–22.
- `reveal_flop` / `reveal_turn` / `reveal_river` → re-feed the deck, `.reveal()` the board
  publicly; callback writes `HandState.community_cards` and advances the phase.
- `showdown_reveal` → batched; reveals non-folded hole cards (mask from on-chain fold state)
  into each seat's `revealed_card_1/2`; the existing `showdown` eval then runs on public cards.

Retired: `request_shuffle`/`callback_shuffle` (VRF), `encrypt_hole_cards`, `grant_card_allowance`,
`grant_own_allowance`, `grant_community_allowances`, `reveal_cards`/`reveal_community` (Inco/Ed25519),
`inco_cpi.rs`, `ephemeral-vrf-sdk`. Betting / pot / side-pots / rake / hand-eval are unchanged
public Solana logic (still 48 passing unit tests).

**Version stack:** `arcis`/`arcium-anchor`/`-client`/`-macros` `=0.11.1`, `anchor-lang 1.0.x` /
Solana 3.x, `arcium` CLI 0.11.2, `@arcium-hq/client 0.11.2` (frontend), devnet cluster offset **456**.

**Deployment status (Phase 3 complete):** program `9chPz3vJDeU7gr4zBtDreJUpVLKbqwrKoQBQQjT1SF5X`
deployed to **devnet** with MXE + all 6 comp-defs initialized. A full hand has run end-to-end
through the live MPC network (shuffle → deal → bet → reveals → showdown), with dealt cards
matching the showdown-revealed cards and the pot conserved. Circuits are hosted (OffChain source)
at `github.com/criptocbas/hiddenhand-arcium-circuit` — **if you edit a circuit you MUST
`arcium build` → re-push the exact `.arcis` to that repo → redeploy, or computations abort.**

**Frontend:** `app/lib/arcium.ts` (replaces `lib/inco.ts`) — x25519 key derivation, MXE pubkey,
RescueCipher decrypt, Arcium queue-account set, event scanning. `usePokerGame.ts` drives the 6
MPC steps (public API preserved). Builds under Turbopack via `next.config.ts` aliases
(anchor-core-shim + crypto/fs/child_process polyfills — see the file). Reproducible integration
test: `app/scripts/devnet-full-hand.cjs`. RPC must be reliable (Helius) via `NEXT_PUBLIC_SOLANA_RPC`;
public devnet drops Arcium txs.

---

## Conversation Context (IMPORTANT - READ FIRST)

> ⚠️ Historical (hackathon-era Inco/VRF design) — superseded by the Arcium banner above.

This project was started for the **Solana Privacy Hack** hackathon (Jan 12-30, 2025). The user and Claude collaboratively designed and built the initial program structure.

### Key Decisions Made
1. **Project Choice**: We evaluated 5 privacy project ideas and chose **Privacy Poker** because:
   - User's escrow3 experience translates well (state machines, multi-party)
   - Clear scope (Texas Hold'em rules are standardized)
   - Exciting demo potential
   - Targets MagicBlock ($5K) + Open Track ($18K) = $23K bounty potential

2. **Privacy Approach**: Hybrid **MagicBlock VRF + Inco TEE** for ultimate privacy:
   - **MagicBlock VRF**: Provably fair card shuffling with verifiable randomness
   - **Inco TEE**: confidential computing (TEE) for card privacy
     - Cards encrypted as u128 handles on-chain
     - Only card owner can decrypt (via allowances)
     - Ed25519 signature verification for reveals
     - **Cryptographic guarantee**: Cards are ALWAYS encrypted, even during computation

3. **Manual Inco CPI**: Built custom CPI module (`src/inco_cpi.rs`) for Inco integration to avoid SDK version conflicts.

### What's Built
- Full poker game state machine (7 phases)
- Table creation with configurable blinds
- Player join/leave with USDC buy-in (SPL token support)
- Betting logic (Fold/Check/Call/Raise/AllIn)
- Pot management and action rotation
- Card encoding (0-51) with Inco TEE encryption
- **Hand evaluation algorithm** (best 5 from 7 cards)
- **Showdown with pot distribution** (handles split pots)
- **48 passing unit tests**
- **MagicBlock VRF Integration** - Provably fair card shuffling
- **Inco TEE Encryption** - Hole cards encrypted as u128 handles
- **Ed25519 Signature Verification** - Secure card reveals at showdown
- **Complete Next.js Frontend** - Playable poker UI with wallet integration
- **Client-side Decryption** - Players decrypt their own cards via Inco SDK
- **Authority AFK Recovery** - Non-authority players can continue game after 60s timeout
- **Spectator Mode** - Read-only table view without wallet connection
- **Upgraded Lobby** - Grid/list toggle, stake/size filters, Quick Play
- **Rake System** - Tiered percentage with cap, auto-calculated from blind level
- **USDC Denomination** - Tables use SPL tokens (default: USDC on devnet)
- **On-Chain Hand Replay** - 5 program events create full action-by-action audit trail, frontend reconstructs timelines from past transactions
- **MagicBlock Session Keys** - Popup-free gameplay: player_action, reveal_cards, reveal_community use session keys for instant signing
- **Mobile-Responsive Design** - Full mobile support: landscape table with compact seats/cards, fixed bottom action bar, rotation prompt, safe area handling, touch-optimized controls

### Session Keys (Popup-Free Gameplay)

Uses MagicBlock's `session-keys` crate (v3.0.10, program `KeyspM2ssCJbqUhQ4k7sveSiY4WjnYsrXkC8oDbwde5`).

**How it works:** Player creates a session (one wallet popup) which generates an ephemeral keypair. All gameplay transactions are signed by the ephemeral key -- zero popups. Session is scoped to the HiddenHand program only with configurable expiry (max 7 days, default 4 hours).

**Instructions with session support** (via `#[derive(Session)]` + `#[session_auth_or(...)]`):
- `player_action` -- fold/check/call/raise/all-in (hot path)
- `reveal_cards` -- showdown card reveals
- `reveal_community` -- community card reveals

**Instructions WITHOUT session support** (require real wallet):
- `join_table` / `leave_table` -- real USDC transfers
- `grant_own_allowance` -- Inco CPI needs real wallet as fee payer
- All other instructions (create_table, showdown, start_hand, etc.)

**IDL field rename**: `player_action` and `reveal_cards` renamed their signer field from `player` to `signer`. `reveal_community` keeps `caller`. All three now have an optional `session_token` account.

**Frontend**: `useSessionKey.ts` hook manages session lifecycle. `usePokerGame.ts` accepts optional `SessionKeyParam` and routes transactions through session signing when active. `SessionStatus.tsx` component shows session state.

**Security model**: Session key can only call HiddenHand program. Cannot withdraw USDC (leave_table requires real wallet). Worst case if compromised: attacker folds hands or makes bad bets. Max loss = table buy-in.

### Mobile-Responsive Architecture

The frontend supports mobile phones in both portrait and landscape orientations. The table page is optimized for landscape (like PokerStars/GGPoker mobile apps).

**Key mobile hooks** (`useIsMobile.ts`):
- `useIsMobileLandscape()` — true when `innerHeight <= 500 && landscape`. Triggers compact table mode.
- `useIsMobilePortrait()` — true when `portrait && width <= 768`. Triggers rotation overlay.
- `useIsTouch()` — true when `(hover: none) and (pointer: coarse)`. Hides keyboard shortcut hints.

**Table page mobile behavior:**
- `RotateDeviceOverlay` appears on portrait mobile, suggesting landscape. Dismissible.
- `PokerTable` uses tighter `SEAT_POSITIONS_MOBILE` (seats at 6%/94% instead of 12%/88%) and seat width shrinks from `w-36` to `w-[5.5rem]`.
- `PlayerSeat` accepts `compact` prop: xs cards, smaller padding/text/badges.
- `Card.tsx` has `xs` size (36x50px) in addition to sm/md/lg.
- `ActionPanel` in `mobile` mode renders as a fixed bottom bar with 4 inline buttons and an expandable raise drawer (slides up). Hidden keyboard hints via `.kbd-hint` CSS class.

**CSS utilities** (`globals.css`):
- `.safe-bottom/.safe-left/.safe-right/.safe-top` — `env(safe-area-inset-*)` for notched devices
- `.touch-target` — min 48px on touch devices
- `.kbd-hint` — hidden on `(hover: none) and (pointer: coarse)`
- `.no-overscroll` — prevents pull-to-refresh on table page
- `.scrollbar-hide` — hides scrollbar on horizontal filter pill rows

**Viewport meta** (`layout.tsx`): `viewport-fit=cover, maximum-scale=1, user-scalable=no` prevents accidental zoom during gameplay.

**Modals on mobile**: `CreateTableModal` and `QuickPlayModal` use `items-end sm:items-center` and `rounded-t-*` to slide up from bottom as sheets on mobile, centered on desktop.

### On-Chain Events & Hand Replay

5 events in `events.rs` create a complete audit trail for every hand:

| Event | Emitted in | When |
|-------|-----------|------|
| `HandStarted` | `callback_shuffle.rs` | VRF shuffle complete, blinds posted |
| `ActionTaken` | `player_action.rs`, `timeout_player.rs`, `timeout_reveal.rs` | Every player action or timeout |
| `CommunityCardsRevealed` | `reveal_community.rs` | Flop/turn/river revealed |
| `ShowdownReveal` | `reveal_cards.rs` | Player reveals hole cards |
| `HandCompleted` | `showdown.rs` | Hand finishes, pot distributed |

**Phase encoding**: `GamePhase as u8` cast (Dealing=0, PreFlop=1, Flop=2, Turn=3, River=4, Showdown=5, Settled=6). This relies on enum variant ordering in `hand.rs` — **do not reorder GamePhase variants**.

**ActionTaken.action_type encoding**: 0=Fold, 1=Check, 2=Call, 3=Raise, 4=AllIn, 5=TimeoutFold, 6=TimeoutCheck.

**Frontend sync gotcha (IMPORTANT)**: `useHandHistory.ts` has hardcoded SHA-256 discriminators (`EVENT_DISCRIMINATORS`) and hand-written binary parsers for each event. If you rename an event struct, add/remove/reorder its fields, or change field types, you **must**:
1. Regenerate the discriminator: `echo -n "event:<EventName>" | sha256sum` (first 8 bytes)
2. Update the corresponding `parse*FromBuffer` function to match the new layout
3. Run `anchor build` and copy the updated IDL to `app/lib/idl/`

**Historical reconstruction**: `useHandHistory` calls `getSignaturesForAddress(tablePDA)` on mount to fetch past transactions, parses all `"Program data:"` log lines by discriminator, and rebuilds both `history` (HandCompleted) and `handTimelines` (all other events). This works because the table PDA appears in every instruction's account list.

### Game Liveness (AFK Recovery)

The game includes robust timeout mechanisms to prevent games from getting stuck if a player or the table authority goes AFK:

1. **Player Action Timeout** (`timeout_player`): Force fold inactive players after timeout
2. **Showdown Reveal Timeout** (`timeout_reveal`): Auto-fold players who don't reveal cards at showdown
3. **Community Card Reveal Timeout** (`reveal_community`): Any player can reveal community cards after 60s if authority is AFK
4. **Community Card Allowances** (`grant_community_allowances`): All players receive decryption access for community cards after VRF shuffle

**Technical Details:**
- Timeout checks use Solana cluster time (not local time) to avoid clock synchronization issues
- 60-second timeout constant: `ALLOWANCE_TIMEOUT_SECONDS` in `constants.rs`
- Frontend uses `getBlockTime()` RPC call for accurate timeout validation

### Spectator Mode Architecture

The spectator system lets anyone watch live tables without connecting a wallet.

**Privacy invariant (CRITICAL):** Encrypted u128 Inco TEE card handles must NEVER be passed to spectator-rendered components. The `useTableState` hook enforces this at the data layer — all player hole cards are returned as `[null, null]`. Only revealed showdown cards (public on-chain data) are shown. This is not just a UI concern; it's the core privacy guarantee.

**Read-only Anchor provider pattern:** Both `useTableState` and `useLobby` create their own Anchor `Program` instance using a dummy wallet when no real wallet is connected. This allows on-chain reads without wallet connection:
```ts
const dummyWallet = {
  publicKey: PublicKey.default,
  signTransaction: async (tx) => tx,
  signAllTransactions: async (txs) => txs,
};
const provider = new AnchorProvider(connection, wallet ?? dummyWallet, opts);
```

**dataSize filter (GOTCHA):** `useLobby` filters `table.all()` with `dataSize: 177` to skip old devnet table accounts that predate the USDC migration (they're 33 bytes shorter and cause deserialization errors). **If the Table struct changes, this constant MUST be updated to match `Table::SIZE` in `table.rs`.**

### Rake System

Rake is a percentage of each pot, capped per hand, tiered by blind level. Set at table creation via `getRakeForBlinds()` in `app/lib/rake.ts`. Collected on-chain during `showdown` into `table.accumulated_rake`. Authority withdraws via `collect_rake` instruction.

| Tier | Max BB | Rake | Cap |
|------|--------|------|-----|
| Micro | $0.10 | 5% | $1 |
| Low | $1 | 4.5% | $2 |
| Medium | $5 | 4% | $5 |
| High | $25 | 3% | $15 |
| Nosebleed | $25+ | 2.5% | $25 |

### What's Next
1. ~~Write tests for current poker logic~~ Done
2. ~~Integrate MagicBlock VRF for shuffling~~ Done
3. ~~Integrate Inco TEE for card encryption~~ Done
4. ~~Build Next.js frontend with game UI~~ Done
5. ~~Ed25519 signature verification for secure reveals~~ Done
6. ~~Spectator mode and lobby upgrade~~ Done
7. Record demo video and submit

---

## Overview
HiddenHand is a fully on-chain Texas Hold'em poker game with cryptographic privacy guarantees. Player hole cards are encrypted using Inco Lightning's TEE-based confidential computing, ensuring that only the card owner can see their hand while the game remains provably fair.

**Hackathon**: Solana Privacy Hack (Jan 12-30, 2025)
**Submission Due**: February 1, 2025
**Target Bounties**: Inco Gaming ($2K) + Open Track ($18K) = $20K potential

## Tech Stack

### Smart Contract (Anchor/Rust)
- **Location**: `/programs/hiddenhand/`
- **Framework**: Anchor 0.32.1
- **Network**: Solana Devnet
- **Program ID**: `5fcckjDn8wzRSodJbQVpHeuWZ8x4B3htKv1WEMx36XJe`

### MagicBlock VRF Integration
- **VRF SDK**: `ephemeral-vrf-sdk = "0.2.1"` - Verifiable random function
- **VRF Features**: Provably fair shuffling, callback-based randomness
- **Status**: Integrated and working (42 tests passing)

### Inco TEE Integration
- **Inco Lightning**: TEE-based confidential encryption for hole cards
- **Encryption**: Cards stored as u128 handles on-chain
- **Decryption**: Client-side via Inco SDK with wallet signing
- **Ed25519 Verification**: Covalidator signatures verify card reveals at showdown

### Frontend (Next.js)
- **Location**: `/app/`
- **Framework**: Next.js 15 with TypeScript
- **Wallet**: Solana Wallet Adapter
- **Status**: Complete and playable

## Game Architecture

### Card Representation
- Cards encoded as 0-51 (u8)
- Suit: card / 13 (0=Hearts, 1=Diamonds, 2=Clubs, 3=Spades)
- Rank: card % 13 (0=2, 1=3, ..., 8=10, 9=J, 10=Q, 11=K, 12=A)
- Stored as encrypted `u128` handles via Inco TEE

### Game Phases
```
Dealing → PreFlop → Flop → Turn → River → Showdown → Settled
```

### PDAs
- **Table**: `["table", table_id]`
- **Player Seat**: `["seat", table_pubkey, seat_index]`
- **Hand State**: `["hand", table_pubkey, hand_number]`
- **Deck State**: `["deck", table_pubkey, hand_number]`
- **Vault**: `["vault", table_pubkey]` — SPL TokenAccount, authority = table PDA

### Token Architecture (USDC / SPL)

Each table is denominated in a single SPL token (stored as `token_mint: Pubkey` on Table).

**Vault pattern**: The vault is an SPL `TokenAccount` PDA at `["vault", table_key]` with `token::authority = table PDA`. The **table PDA signs transfers** using `TABLE_SEED` signer seeds — not the vault's own seeds.

**Internal chip ledger**: `PlayerSeat.chips`, `HandState.pot`, and `Table.accumulated_rake` are abstract u64 values manipulated in-memory. **Real token transfers only happen at 4 entry/exit points**: `join_table` (deposit), `leave_table` (withdraw), `collect_rake` (authority), `close_inactive_table` (emergency refund). The 15+ gameplay instructions (betting, dealing, showdown, etc.) never touch tokens.

**USDC mint addresses**:
- Devnet: `4zMMC9srt5Ri5X14GAgXhaHii3GnPAEERYPJgZJDncDU` (6 decimals)
- Mainnet: `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v` (6 decimals)

**Anchor types**: Uses `InterfaceAccount<'info, TokenAccount>` and `Interface<'info, TokenInterface>` for forward-compatibility with both SPL Token and Token-2022.

**Native SOL is not supported** — was removed when switching from SystemAccount vaults to TokenAccount vaults. SOL tables would require wrapped SOL (wSOL) with auto-wrap/unwrap in the frontend.

## Current Instructions (18 total)

### Core Game Instructions

| Instruction | Description | Status |
|-------------|-------------|--------|
| `create_table` | Create poker table with blinds config | Done |
| `join_table` | Join table with SPL token buy-in | Done |
| `leave_table` | Cash out and leave (SPL transfer) | Done |
| `start_hand` | Begin new hand, init deck | Done |
| `player_action` | Fold/Check/Call/Raise/AllIn | Done |
| `showdown` | Evaluate hands, determine winner, distribute pot | Done |

### MagicBlock VRF Instructions (Provably Fair Shuffling)

| Instruction | Description | Status |
|-------------|-------------|--------|
| `request_shuffle` | Request VRF randomness for card shuffle | Done |
| `callback_shuffle` | VRF callback - atomic shuffle + encrypt with Inco | Done |

### Inco TEE Instructions (Encrypted Cards)

| Instruction | Description | Status |
|-------------|-------------|--------|
| `encrypt_hole_cards` | Encrypt dealt cards for a player | Done |
| `grant_card_allowance` | Authority grants decryption allowance | Done |
| `grant_own_allowance` | Player grants own allowance after timeout | Done |
| `grant_community_allowances` | Grant community card access to player | Done |
| `reveal_cards` | Reveal hole cards at showdown (Ed25519 verified) | Done |
| `reveal_community` | Reveal community cards (Ed25519 verified) | Done |

### Rake & Table Management Instructions

| Instruction | Description | Status |
|-------------|-------------|--------|
| `collect_rake` | Authority withdraws accumulated rake from vault | Done |
| `timeout_player` | Force fold inactive player | Done |
| `timeout_reveal` | Force reveal timeout at showdown | Done |
| `close_inactive_table` | Close abandoned table, return funds | Done |

## File Structure

```
hiddenhand/
├── programs/
│   └── hiddenhand/src/           # Main poker program
│       ├── lib.rs                # Program entry (18 instructions)
│       ├── constants.rs          # PDA seeds, game constants
│       ├── error.rs              # 30+ custom errors
│       ├── inco_cpi.rs           # Manual Inco CPI (no SDK)
│       ├── state/
│       │   ├── table.rs          # Table config, seat management
│       │   ├── hand.rs           # Hand phases, pot, betting round
│       │   ├── player.rs         # Player seat, chips, hole cards (u128 handles)
│       │   ├── deck.rs           # Deck state, card utilities
│       │   └── hand_eval.rs      # Hand evaluation (best 5 from 7)
│       └── instructions/
│           ├── create_table.rs
│           ├── join_table.rs
│           ├── leave_table.rs
│           ├── start_hand.rs
│           ├── player_action.rs
│           ├── showdown.rs             # Winner determination
│           ├── reveal_cards.rs         # Ed25519 verified card reveal
│           ├── reveal_community.rs     # Ed25519 verified community card reveal
│           ├── timeout_player.rs       # Force fold inactive players
│           ├── timeout_reveal.rs       # Force reveal at showdown
│           ├── close_inactive_table.rs # Return funds from abandoned table
│           ├── request_shuffle.rs      # VRF randomness request
│           ├── callback_shuffle.rs     # VRF callback - atomic shuffle + encrypt
│           ├── encrypt_hole_cards.rs   # Inco TEE encryption
│           ├── grant_own_allowance.rs  # Player grants decryption access
│           └── grant_community_allowances.rs  # Community card access for AFK recovery
├── app/                          # Next.js frontend (complete)
│   ├── app/
│   │   ├── page.tsx             # Landing page
│   │   ├── layout.tsx           # Root layout (viewport meta, safe areas)
│   │   ├── globals.css          # Global styles (mobile utilities, animations)
│   │   ├── lobby/page.tsx       # Table lobby (filters, Quick Play, view toggle)
│   │   ├── table/[tableId]/page.tsx  # Table page (player + spectator modes)
│   │   ├── leaderboard/page.tsx # Leaderboard with mobile card layout
│   │   ├── player/[walletAddress]/page.tsx  # Player profile + stats
│   │   └── responsible-gaming/page.tsx      # Responsible gaming info
│   ├── components/
│   │   ├── PokerTable.tsx       # Table visualization (mobile seat positions)
│   │   ├── PlayerSeat.tsx       # Player seat UI (compact mode for mobile)
│   │   ├── ActionPanel.tsx      # Betting controls (mobile bottom bar + raise drawer)
│   │   ├── Card.tsx             # Card rendering (xs/sm/md/lg sizes, flip animation)
│   │   ├── SpectatorView.tsx    # Read-only spectator view (privacy-safe)
│   │   ├── RotateDeviceOverlay.tsx  # Portrait rotation prompt for table page
│   │   ├── HandReplayer.tsx     # Visual hand replay modal
│   │   ├── OnChainHandHistory.tsx   # On-chain event timeline
│   │   ├── SwapModal.tsx        # Jupiter swap integration
│   │   ├── SessionStatus.tsx    # Session key state display
│   │   ├── stats/
│   │   │   ├── PlayerHUD.tsx    # Hover/tap stats tooltip on seats
│   │   │   └── ProfitChart.tsx  # SVG profit chart
│   │   └── lobby/
│   │       ├── TableList.tsx    # Grid/list view container
│   │       ├── TableCard.tsx    # Grid view card
│   │       ├── TableRow.tsx     # List view row
│   │       ├── CreateTableModal.tsx  # Table creation (bottom sheet on mobile)
│   │       └── QuickPlayModal.tsx    # Quick Play (bottom sheet on mobile)
│   ├── hooks/
│   │   ├── usePokerGame.ts      # Full game logic (wallet required)
│   │   ├── usePokerProgram.ts   # Anchor program hook (wallet required)
│   │   ├── useTableState.ts     # Read-only table state (no wallet needed)
│   │   ├── useLobby.ts          # Lobby data + filters (no wallet needed)
│   │   ├── useHandHistory.ts    # On-chain event parsing
│   │   ├── useIsMobile.ts       # Mobile detection hooks (landscape, touch, portrait)
│   │   ├── useTokenBalance.ts   # SPL token balance polling
│   │   ├── useSessionKey.ts     # MagicBlock session key lifecycle
│   │   └── usePlayerStats.ts    # On-chain player statistics
│   └── lib/
│       ├── inco.ts              # Inco SDK integration
│       ├── tokens.ts            # SPL token definitions (USDC)
│       ├── rake.ts              # Rake schedule and calculation
│       └── idl/                 # Program IDL
├── tests/
│   └── hiddenhand.ts            # Integration tests
├── marketing/                   # Pitch materials
├── Anchor.toml
└── CLAUDE.md
```

## Development Commands

```bash
# Build
anchor build

# Test
anchor test

# Deploy to devnet
anchor deploy --provider.cluster devnet

# Frontend (when created)
cd app && npm run dev
```

## Key Resources

### MagicBlock VRF (Provably Fair Shuffling)
- [MagicBlock Docs](https://docs.magicblock.gg/)
- [VRF SDK](https://crates.io/crates/ephemeral-vrf-sdk)
- [Example: Roll Dice](https://github.com/magicblock-labs/roll-dice) - VRF pattern

### Inco TEE (Card Encryption)
- [Inco Network](https://inco.org/)
- [Inco Solana SDK](https://www.npmjs.com/package/@inco/solana-sdk)

### Hackathon
- [Hackathon Page](https://solana.com/privacyhack)
- [Privacy on Solana GitHub](https://github.com/catmcgee/privacy-on-solana)

## Hackathon Info

**Timeline**:
- Jan 12: Opening ceremony
- Jan 12-16: Workshops
- Jan 12-30: Hacking
- Feb 1: Submissions due
- Feb 10: Winners announced

**Submission Requirements**:
- Open-source code
- Deployed to devnet/mainnet
- 3-minute demo video
- Documentation

## Design Notes

- Dark theme with poker aesthetic (green felt, gold accents)
- Card reveal animations with 3D flip effect
- Sound effects for chips/cards/actions
- Mobile-first responsive: landscape table, bottom action bar, compact seats
- Safe area support for notched phones (iPhone X+, Dynamic Island)
- Touch-optimized: 48px minimum targets, no hover-only interactions
- Reduced motion support via `prefers-reduced-motion`

## User Background

The user has built a milestone-based escrow platform on Solana (escrow3) with:
- Full Anchor/Rust program
- Complex state machines
- Multi-party coordination
- Next.js frontend with wallet integration

This experience directly applies to HiddenHand's poker mechanics.

---
> Source: [HiddenHandPoker/HiddenHand](https://github.com/HiddenHandPoker/HiddenHand) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
