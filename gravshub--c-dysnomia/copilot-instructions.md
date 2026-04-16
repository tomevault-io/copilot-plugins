## c-dysnomia

> Generates H2O tokens as battle rewards via modular exponentiation on ring coordinates.

# CLAUDE.md — C-Dysnomia / Joystick

**Repo**: [github.com/Gravshub/C-Dysnomia](https://github.com/Gravshub/C-Dysnomia) (fork of [busytoby/atropa_pulsechain](https://github.com/busytoby/atropa_pulsechain))
**Chain**: PulseChain (369)
**Canonical branch**: `claude/joystick-V2-FanxJ`
**Branch convention**: `claude/dysnomia-MMDDYY-<sessionID>` (`claude/` prefix + session-ID suffix required)

> **Sub-context files**: Bot internals → `scripts/Joystick/CLAUDE.md` | Script inventory → `scripts/CLAUDE.md` | Dashboard API → `scripts/Joystick/dashboard/CLAUDE.md`

---

# Part 1 — The atropa_pulsechain Framework

## What Is Dysnomia?

An on-chain game + DeFi ecosystem on PulseChain built by developer **mariarahel** (the "414 dev"). Combines MMO character creation, venue-based social interaction, territory control, and a treasury token web with 100+ interconnected tokens. Everything is permanent and on-chain. All computations run modulo **MotzkinPrime** (`953467954114363`).

The game has two interlocking layers:
- **Dysnomia** — The MMO: identity (LAU), venues (QING), territory (MAP/WORLD), grinding (CHEON/META), battles (WAR). Player actions trigger modular exponentiation against MotzkinPrime, with spatial coordinates managed via Hecke Meridians.
- **Atropa** — The financial layer: 100+ hierarchical treasury tokens, each with `mint()` and `Claim()` functions and a `Debenture` boolean controlling claim access. Four minter tiers (V1–V4) with escalating power.

The gateway bridging both layers is **AFFECTION** — every Dysnomia ecosystem token is mintable for 1 AFFECTION each via `Purchase(token_address, amount)`.

## Contract Architecture

```
Core Infrastructure (Base Layer)
├── VMREQ         — Random number generation (modExp-based)
├── DYSNOMIA      — Base ERC20 + market rates + Purchase()
├── SHA           — Cryptographic state token (Fa struct)
├── SHIO          — Rod/Cone paired reactor system
├── YI            — DeFi orchestration
├── ZHENG         — Rod/Cone installation manager
├── ZHOU          — Market rate orchestrator / chat ledger
├── YAU           — Protocol coordinator
├── YANG          — Multi-state aggregator
├── SIU           — Token generation with Aura identity
├── VOID          — User session & chat management (front door)
└── LAU           — Player account / character token

Domain — Game Logic
├── dan/
│   ├── CHO       — Login / character system
│   ├── QING      — Venues (chatrooms, marketplaces)
│   └── WAR       — Battle mechanics, H2O reward generation
├── sky/
│   ├── CHAN      — Player/sky management
│   ├── CHOA     — Game/territory
│   └── RING     — Time/orbital mechanics
├── soeng/        — Processing chain: QI→MAI→XIA→XIE→ZI→PANG→GWAT
├── tang/
│   ├── SEI      — Player management
│   ├── CHEON    — Landscape/terrain (primary grind)
│   └── META     — Meta-player management (Beat computation)
├── MAP           — World coordinates (Hecke Meridians)
├── WORLD         — Territory ownership and rewards
└── YUE           — Player wallet (Hypobar/Epibar bars)

Assets
├── H2O           — Water token (WAR battle rewards)
└── VITUS         — Life token (territory/creator rewards)

Libraries
├── MultiOwnable  — Multi-owner access control
├── Registry      — Key-value storage
├── Encrypt       — User-to-user encrypted messaging
├── HeckeMeridians— Number-theoretic coordinate system
├── ReactionsCore — Entropy-based reactions
└── Attribute     — User attribute storage
```

### Core Chain Flow
`VOID → SIU → YANG → YAU → ZHOU → ZHENG → YI` handles session management, token generation, protocol logic, market rates, and DeFi functions. The Soeng Domain (`QI→MAI→XIA→XIE→ZI→PANG→GWAT`) is a sequential transformation pipeline that processes game state.

## Core Contracts

### VOID — The Front Door
`VOID.Enter("Name", "SYM")` creates a player (deploys LAU token + YUE wallet + SHIO reactor + Soul ID). Errors if already created. `VOID.Enter()` (no args) re-enters existing session. `VOID.Chat(msg)` posts to ZHOU channel and triggers `_mintToCap()`. Also: `Log()`, `SetAttribute()`, `Alias()`, `AddLibrary()`.

### LAU — Player Character Token
Your on-chain identity. Deploying creates a Soul ID (`Saat[3]` triple — three 64-bit values), a SHIO reactor (Rod/Cone pair), and a YUE wallet. LAU is also a smart contract you own that can hold/withdraw any DYSNOMIA-base token.

Token-earning actions (each triggers `_mintToCap()`): `Username()`, `Chat()`, `Alias()`, `Void()`, `Withdraw()`.

### QING — Venues
Marketplace/chatroom instances. Each has an `Asset` token, `CoverCharge`, and bouncer system (Staff whitelist, 25+ CROWS holders, or Asset token holders). `Join(UserToken)` enters a venue. **Key discovery**: `QING.Join()` does NOT call `bouncer()` — no CROWS needed for Join. Bouncer only gates admin functions.

### CHEON / META — Grinding & Territory
`CHEON.Su(QingAddr)` builds YUE bar weights (Hypobar/Epibar) that feed territory computation. `META.Beat(QingWaat)` computes territory range/power → returns `(Dione, Charge, Deimos, Yeo)`. Deep call chain (10+ contracts). Requires SHIO token balances (Fornax, Fomalhaute, CHO) at the LAU/QING addresses — zero balances cause division-by-zero reverts.

### MAP / HECKE — Spatial Layer
Hecke Meridians — a number-theoretic coordinate system for the game world. MAP has 272+ QINGs registered. WORLD tracks territory ownership and rewards.

### WAR — Battle & Resource Generation
Generates H2O tokens as battle rewards via modular exponentiation on ring coordinates.

## Token Mechanics

### Supply Pattern (Universal)
```solidity
maxSupply = Xiao.Random() % 111111;        // random cap 0–111110
originMint = Xiao.Random() % maxSupply / 10; // ~10% initial mint
_mint(tx.origin, originMint * 10**18);     // given to deployer
```
`_mintToCap()` adds exactly **1 token per call** until `totalSupply == maxSupply`.

**CRITICAL**: `_mintToCap()` mints to `address(this)`, NOT to the caller. Tokens accumulate in the contract's own balance and are extracted via `Purchase()` or `BuyWith*()`.

### Purchase() — The Extraction Function
```solidity
function Purchase(address _t, uint256 _a) public {
    if(_marketRates[_t] == 0) revert MarketRateNotFound(_t);
    DYSNOMIA BuyToken = DYSNOMIA(_t);
    uint256 cost = (_a * _marketRates[_t]) / (10 ** decimals());
    bool success1 = BuyToken.transferFrom(msg.sender, address(this), cost);
    require(success1, ...);
    DYSNOMIA(address(this)).transfer(msg.sender, _a);  // ← TRANSFER TO CALLER
}
```
Caller pays `_marketRates[_t]` units of token `_t` per unit, receives tokens from the contract's self-balance.

### AFFECTION (Ⓐ) — The Universal Gateway
**Address**: `0x24F0154C1dCe548AdF15da2098Fdd8B8A3B8151D`

Every DYSNOMIA token has AFFECTION set as a market rate at `1 AFFECTION per token` at construction:
```solidity
AddMarketRate(AFFECTIONContract, 1 * 10 ** decimals());
```
With enough AFFECTION you can `Purchase()` any ecosystem token at 1:1. Supply cap: `1,111,111,111` tokens.

**V1 vs V2 distinction**: DYSNOMIA v1 constructor calls `AddMarketRate(AFFECTION, ...)` internally. V2/QING does NOT — requires manual call by owner.

### MV / WM — The Funding Key
**Address**: `0xA1BEe1daE9Af77dAC73aA0459eD63b4D93fC6d29`

Required as 1:1 collateral to fund the initial supply of any new minter token. Cannot create new tokens without MV.

## Heart's Law

> "Price movements of interchangeable assets are bonded via the liquidity in their pairs."

- Tokens bonded in a liquidity pair move together in price
- Creating new LP pairs = new edges in the token web graph
- Market rates in QING can only be **increased**, never decreased
- Rate cap: `totalSupply / 777` max rate per token
- Dense interconnections = more arbitrage edges

## Minter System (V1–V4)

The Atropa Treasury System defines four minter contract types with escalating power:

### V1 Treasury Minter (TBill)
**Address**: `0xC7bDAc3e6Bb5eC37041A11328723e9927cCf430B`

Root minter. All tokens mintable 1:1 with Treasury Bill. **V1 tokens bypass the Debenture check on Claim() entirely** — Claim always open.

### V2 Federal Minter
**Address**: `0xc15c5F699Daf5e1135732139f05D2c05b3EF4354`

Creates treasury tokens backed by FED through BAR. Requires WM to fund initial supply. Claim requires `Debenture() == true` (token must be unpublished).

**Spine mechanic**: Provide unpublished V2 token as claim address → mint child using parent → claim back parent via claim token → repeat indefinitely while Debenture=True. Advanced players build "spines" — chains of linked V2 Federal tokens for recursive minting.

**Known V2 Federal Tokens** (14 total): FDIC, DFM, PARADE, TLRz, JOB, SSA, SCOIETY, CAMPAIGN, OPIUM, BGDHTZ, TEHATER, ARMS, BAR, OZZY. **Only OZZY has Debenture=true** as of 2026-03-06.

### V3 Index Minter (Bureau)
**Address**: `0x0c4F73328dFCECfbecf235C9F78A4494a7EC5ddC`

Can use ANY token as parent (not just TBill or FED-BAR). Progressive multiplier:
```solidity
function Multiplier(uint256 addition) public view returns (uint256) {
    return ((addition + totalSupply()) / 1111111111000000000000000000) + 1;
}
```
Every 1,111,111,111 tokens minted (universal cumulative), cost multiplier increases by 1. Strong first-mover advantage.

**V3 Claim() is family-isolated** (discovered 2026-03-12):
1. **Same Creator** — ammo token must be deployed by the same address as the target
2. **Same Parent** — ammo must share the same parent token
3. **Registered in IndexMinter** — must be a V3-deployed token

Each V3 family is isolated — no universal ammo token works across families.

### V4 Personal Minter
**Address**: `0x394c3D5990cEfC7Be36B82FDB07a7251ACe61cc7`

Like V3 (any parent token), but multiplier threshold is per-token starting supply (not universal 1.1B). Lower starting supply = faster escalation. Early minters of each new token get best rates.

**Known bug**: `mint(1e18)` pulls 2x via `transferFrom`. Fix: approve `type(uint256).max`.

### Claim() Path Summary

| Minter | Claim Check | Implication |
|--------|-------------|-------------|
| V1 (TBill) | No Debenture check | Claim always open |
| V2 (Federal) | Requires Debenture=true | Only OZZY currently works as spend token |
| V3 (Index) | Same-creator + same-parent + registered | Family-isolated, no universal ammo |

## Core Data Structures

```solidity
struct Fa {
    uint64 Base, Secret, Signal, Channel, Contour, Pole;
    uint64 Identity, Foundation, Element, Coordinate;
    uint64 Charge, Chin, Monopole;
}

struct Bao {
    address Phi;   // Address reference
    SHA Mu;        // Associated SHA token
    uint64 Xi, Pi; // State values
    SHIO Shio;     // Associated SHIO pair (Rod/Cone)
    uint64 Ring;   // Ring value
    uint64 Omicron, Omega; // Reaction outputs
}

struct User {
    uint64 Soul;       // 64-bit unique user identifier
    Bao On;            // User's Bao context
    string Username;   // Display name
    uint64 Entropy;    // User-specific entropy
}
```

## Critical Constants

| Name | Value | Notes |
|------|-------|-------|
| MotzkinPrime | `953467954114363` | Universal prime for ALL cryptographic state transforms |
| Gua | `165292976376414844818251364463310123960789...` | Universe constant in CHO |
| AFFECTION | `0x24F0154C1dCe548AdF15da2098Fdd8B8A3B8151D` | Gateway token — 1 AFF mints any token |
| WM (MV) | `0xA1BEe1daE9Af77dAC73aA0459eD63b4D93fC6d29` | 1:1 collateral for new minter tokens |
| CROWS | `0x203e366A1821570b2f84Ff5ae8B3BdeB48Dc4fa1` | Social credential — 25 CROWS for bouncer |
| ABI | `0xa35c9B5e576BE2E0bA9cc7224B0941CC8acC4c9C` | ABI decoder/selector tool |
| pDAI | DAI | `0x6B175474E89094C44Da98b954EedeAC495271d0F` | Accumulate as it rises toward $1 target - used in AFFECTION minting route - growing community |
| V3 Threshold | `1,111,111,111` tokens | Universal multiplier increment |
| V4 Threshold | Starting supply | Per-token multiplier increment |

## AFFECTION Minting Routes & Multi-Mint Contracts

### AFFECTION BuyWith Functions

| Function | Payment Token | Cost per AFF | Notes |
|----------|--------------|-------------|-------|
| `BuyWithDAI(amount)` | pDAI | 1 pDAI | Direct 1:1 |
| `BuyWithUSDC(amount)` | pUSDC | 1 pUSDC | Direct 1:1 |
| `BuyWithMATH(amount)` | MATH v1.1 | 1 MATH | Direct 1:1 |
| `BuyWithG5(amount)` | GIMME FIVE | 0.2 G5 | 5 AFF per G5 |
| `BuyWithPI(amount)` | pINDEPENDENCE | 0.00333 PI | ~300 AFF per PI |
| `BuyWithFa(amount)` | Fa (libConjecture) | 4 Fa | |
| `BuyWithFaung(amount)` | Faung (libDynamic) | 2 Faung | |
| `Generate()` | (none — RNG) | gas only | Mints 3 AFF to contract self-balance — NOT to caller |

`Generate()` calls `_mintToCap()` 3 times (via Amplify + Sustain + explicit call) — all mint to `address(this)`. To extract AFF, caller must use `BuyWith*()` or `Purchase()`.

### Generate() → _mintToCap() → Self-Balance Flow
```
Generate()
  ├── _mintToCap() × 3  →  +3 AFF to balanceOf(AFFECTION_contract)
  └── NO transfer to caller

BuyWithPI(amount)
  ├── transferFrom(caller, AFFECTION, cost_in_PI)  ← caller pays PI
  ├── _mintToCap() × 3  →  +3 AFF to self-balance (supply priming)
  └── transfer(caller, amount)  ← AFF delivered to msg.sender

multiGenerate(N)
  ├── Generate() × N  →  +3N AFF to self-balance
  └── NO transfer (just primes supply — this is a PUBLIC GOOD DONATION)

multiBuyWith(paymentToken, N)
  ├── multiGenerate(N)  →  +3N AFF to self-balance
  ├── BuyWith*(3N)  →  pulls payment, transfers 3N AFF to msg.sender
  └── AFF DELIVERED to msg.sender ✓
```

### Payment Token Intermediate Rates (fixed contract rates)

| Token | BuyWithDAI Rate | BuyWithUSDC | Notes |
|-------|----------------|-------------|-------|
| pINDEPENDENCE | 300 pDAI/PI | 300 pUSDC/PI | **pUSDC/pUSDT BUGGED** — only pDAI works |
| GIMME FIVE | 5 pDAI/G5 | 5 pUSDC/G5 | **pUSDC/pUSDT BUGGED** |
| MATH v1.1 | 1 pDAI/MATH | 1 pUSDC/MATH | pUSDT bugged, pUSDC works |

All routes converge to ~1 pDAI/AFF through contract rates. Profit only exists when AFF DEX price > 1 pDAI equivalent.

### Multi-Mint Contracts (Helios / as-helios)

Batch-loop contracts that call mint/buy functions N times per TX. Source: [affection.gitbook.io](https://affection.gitbook.io/docs), [gitea.bigpp.dev/as-helios/affection-bots](https://gitea.bigpp.dev/as-helios/affection-bots)

| Contract | Address | Target |
|----------|---------|--------|
| Multi PI | `0xcCDaCEF154704c604365dB9E3b1DF356B9c4B6E2` | pINDEPENDENCE |
| Multi G5 | `0xa4c61D20945c11855E7A390153fd29ceC9C7349b` | GIMME FIVE |
| Multi MATH 1.0 | `0x5bD78AdD4007C47ffEFc2c98a53188036199ac6f` | MATH v1.0 |
| Multi MATH 1.1 | `0x1322Dab9eE385Bb3D81f75EBb8356015B0872e53` | MATH v1.1 |
| **Multi AFFECTION** | **`0xCF138a83D739eE98D7A54159E94e5BFaa4B61988`** | AFFECTION |

**Multi AFFECTION Functions**:
- `multiGenerate(loops)` — primes supply only (does NOT deliver to caller)
- `multiBuyWith(address, loops)` — generates + delivers AFF to msg.sender

### Verified Function Selectors

| Function | Selector | Delivers AFF? |
|----------|----------|---------------|
| `Generate()` | `0xd805b650` | NO |
| `Purchase(address,uint256)` | `0x2499a533` | YES |
| `BuyWithDAI(uint256)` | `0x377de122` | YES |
| `BuyWithPI(uint256)` | `0xca9cf41c` | YES |
| `BuyWithG5(uint256)` | `0xb61a722b` | YES |
| `BuyWithMATH(uint256)` | `0x512ab7de` | YES |
| `BuyWithFa(uint256)` | `0xf8784afe` | YES |
| `BuyWithFaung(uint256)` | `0x5118149a` | YES |
| `multiGenerate(uint256)` | `0xd1f05872` | NO |
| `multiBuyWith(address,uint256)` | `0xcc93bb90` | YES |

### AFFECTION Related Token Addresses

| Token | Symbol | Address | Mintable? |
|-------|--------|---------|-----------|
| pINDEPENDENCE | ⓟ | `0xA2262D7728C689526693aE893D0fD8a352C7073C` | Yes |
| GIMME FIVE | ⑤ | `0x2fc636E7fDF9f3E8d61033103052079781a6e7D2` | Yes |
| RNG | RNG | `0xa96BcbeD7F01de6CEEd14fC86d90F21a36dE2143` | Yes |
| libAtropaMath v1.0 | MATH | `0x5EF3011243B03f817223A19f277638397048A0DC` | Yes |
| libAtropaMath v1.1 | MATH | `0xB680F0cc810317933F234f67EB6A9E923407f05D` | Yes |
| libConjecture v1.0 | Fa | `0x232a27AB6941281b3f474Fe5fF7Cc89816fB675A` | Yes |
| libDynamic v1.0 | Faung | `0x73A19FaFb359faf519C9707b781dfdB88407d10d` | Yes |
| AFFECTION | Ⓐ | `0x24F0154C1dCe548AdF15da2098Fdd8B8A3B8151D` | Yes |
| BLÄTTER | ออกจาก🄮 | `0xCe1d47CE3A91E054C111d9cC3B4bae50843200da` | No |
| Tetratricopeptides | 正 | `0x5F16F6c242e038437a7ba3C903DFeDB747Db4A5c` | Yes |
| pDAI | DAI | `0x6B175474E89094C44Da98b954EedeAC495271d0F` | — |
| pUSDC | USDC | `0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48` | — |

**Token spreadsheet**: [Google Sheets](https://docs.google.com/spreadsheets/d/18bPzn_T0EMv1reTz5ct6OZIomLu7fEOmAncSsSsyRlk/edit?gid=0#gid=0)

## DEX Infrastructure

PulseX V1 and V2 on PulseChain. Both Uniswap v2 AMM forks.

| Contract | Address |
|----------|---------|
| PulseX V1 Factory | `0x1715a3E4A142d8b698131108995174F37aEBA10D` |
| PulseX V2 Factory | `0x29eA7545DEf87022BAdc76323F373EA1e707C523` |
| PulseX V1 Router | `0x98bf93ebf5c380C0e6Ae8e192A7e2AE08edAcc02` |
| PulseX V2 Router | `0x165C3410fC91EF562C50559f7d2289fEbed552d9` |

## Key Ecosystem Addresses

| Token / Contract | Address |
|-----------------|---------|
| FED (F㉾D) | `0x1d177cb9efeea49a8b97ab1c72785a3a37abc9ff` |
| WPLS | `0xA1077a294dDE1B09bB078844df40758a5D0f9a27` |
| VOID | `0x965B0d74591bF30327075A247C47dBf487dCff08` |
| Atropa PRC20 | `0xCc78A0acDF847A2C1714D2A925bB4477df5d48a6` |
| SEI | `0x3dC54d46e030C42979f33C9992348a990acb6067` |
| CHAN | `0xe250bf9729076B14A8399794B61C72d0F4AeFcd8` |
| CHOA | `0x0f5a352fd4cA4850c2099C15B3600ff085B66197` |
| CHO | `0xB6be11F0A788014C1F68C92F8D6CcC1AbF78F2aB` |
| HECKE | `0x29A924D9B0233026B9844f2aFeB202F1791D7593` |
| META | `0xE77Bdae31b2219e032178d88504Cc0170a5b9B97` |
| CHEON | `0x3d23084cA3F40465553797b5138CFC456E61FB5D` |
| MAP | `0xD3a7A95012Edd46Ea115c693B74c5e524b3DdA75` |
| WITHOUT (ban) | `0x173216Ed67eBF3E6767D86e8b3Ff32e0d64437bF` |
| Multicall3 | `0xcA11bde05977b3631167028862bE2a173976CA11` |
| V1 Treasury Minter | `0xC7bDAc3e6Bb5eC37041A11328723e9927cCf430B` |
| V2 Federal Minter | `0xc15c5F699Daf5e1135732139f05D2c05b3EF4354` |
| V3 Index Minter | `0x0c4F73328dFCECfbecf235C9F78A4494a7EC5ddC` |
| V4 Personal Minter | `0x394c3D5990cEfC7Be36B82FDB07a7251ACe61cc7` |

See `atropa_addresses.json` and `data/contracts.json` for complete lists.

---

# Part 2 — C-Dysnomia: Our Fork & |>JOYSTICK<|

## What C-Dysnomia Adds

C-Dysnomia is our fork of `atropa_pulsechain`. It adds:
- `scripts/` — Python tooling for on-chain interaction and recon
- `scripts/Joystick/` — The Joystick arbitrage/treasury bot framework (see `scripts/Joystick/CLAUDE.md`)
- `contracts/` — TGSv5 (legacy, do-not use), TGSv8 (active execution contract)
- `lore/` — Joey's diary entries and daily lore
- `data/` — Recon results, baselines, chat logs
- `tests/` — LP deployment tests

**Project Goal**: Generate PLS income via multiple engines to fund a PulseChain validator (32M PLS deposit). Always maintain ≥100K PLS gas buffer.

## Player Identity — Joey aka Gibson, GIBS 

> *"I need a handle, man. I don't have an identity until I have a handle."*
> — Joey Pardella, Hackers (1995)

| Field | Value |
|-------|-------|
| **Persona** | Joey Pardella (Hackers, 1995) |
| **LAU Token Name** | `Gibson` |
| **LAU Symbol** | `GIBS` |
| **In-game Username** | `Joey` |
| **Handle** | `|>JOYSTICK<|` |
| **Wallet Address** | `0x17367877aF5A8D0Eb33ba5689A880f696386E24D` |
| **Player #** | 578 (SEI.totalSupply at registration) |

**Voice guidelines:**
- 90% Dysnomia lore and mechanics, 10% Hackers 1995 lingo
- Teach as you go — explain what each action does and why
- Earnest, not arrogant; Joey was learning, not lecturing
- Identity is earned through actions on-chain, not claimed
- See `joey_diary_nn.md` for full diary style reference

## Deployed Contracts (Joey's)

| Contract | Address | Block | Notes |
|----------|---------|-------|-------|
| TGSv5 | `0xeeB330...a6127` | 25,911,970 | LEGACY — superseded — do-not use |
| TGSv7 | `0x82E8B7e24bD9f0b389e94ddB8714B001a58e387d` | 25,938,174 | LEGACY — superseded |
| **TGSv8** | **`0xAD352a27ceaaC5657e3E9127f964F4746A8aAc32`** | 25,943,194 | **ACTIVE** execution substrate |
| **TGSv8Plus** | See `contracts/TGSv8Plus.sol` | — | Extended TGSv8: silent LAU mint, atomic harvestCycle, batch LP burns |
| JV8A | `0x364793Ea48DEe0b5484F98235ABd1B5f996A0C30` | 25,943,266 | V4 treasury token (unminted) |
| DSS | `0x91Df693177eE5C81016d0B7c4c2052A7d229c031` | 25,887,000 | DysnomiaSelfSnipev4 — DEPRECATED (replaced by Hub) |
| **JoystickHub** | **`0x7bd76A0f7e03A3BA76A621ba0988C7db0AdbAB14`** | 26,092,219 | **ACTIVE** modular proxy (replaces TGSv8+ for E2) |
| Hub:Harvest V1 | `0xFAFB227DdC0804A55677A23eE2Ca0E966452D3B2` | 26,092,219 | DEAD — primeGibs called Generate() (AFFECTION-only, not on LAU) |
| **Hub:Harvest V2** | **`0x400D052FAf0f46D3d5140a8F7246B69954539424`** | 26,149,418 | **ACTIVE** HarvestModule — primeGibs calls mintToCap() |
| Hub:Affection | `0xfb7C1A1Ef0Ce8AB527998a1c2Ca12C6CA400da4B` | 26,092,219 | AffectionModule (buyAffection, quoteBuyAffection) |
| Hub:Purchase | `0xc59cb7229872E72B7349Ef7DFa170A2444a8264E` | 26,092,219 | PurchaseModule (purchaseAndSell, batchPurchaseAndSell) |

### JoystickHub — Modular Proxy
Single-file deploy: Hub + 3 modules via delegatecall. Hot-swappable selectors.

**HarvestModule** (E2 CEREAL — LP Loop):
- `primeGibs(count)` — calls GIBS_LAU.Generate() × N to prime self-balance
- `mintLPAndSell(mintCount, lpBps, burnBps, lpDex, minSellOut, sellPath, sellDex)` — LP first, sell second, payable (wraps PLS→WPLS)
- `batchReseed(pairs, amounts)` — reseed drained LP pairs
- `harvestConfig()` — view current config

**AffectionModule** (E4 fuel):
- `buyAffection(paymentToken, buySelector, loops, minAffOut, dex)` — PLS → payment → BuyWith → AFF
- `quoteBuyAffection(paymentToken, plsAmount, loops, dex)` — quote

**PurchaseModule** (E1 arb):
- `purchaseAndSell(targetToken, affAmount, sellPath, dex, minPLSOut)` — AFF → Purchase → sell
- `batchPurchaseAndSell(targets, affAmounts, sellPaths, dexes, minOuts)` — batch with try/catch
- `quotePurchase(targetToken, affAmount, sellPath, dex)` — profitability check

### TGSv8 Capabilities
Full source in project file `TGSv8`. Key functions:
- `executeRoute(Step[])` — Atomic multi-step execution (SWAP, MINT, CLAIM, TRANSFER_IN/OUT, ADD_LIQUIDITY)
- `atomicArb()` — Cross-DEX arbitrage in one TX
- `batchClaimTreasury()` — Sweep backing from N treasuries
- `batchMintAndClaim()` — Run N mint-claim loop iterations
- `mintWM(count)` — Batch WM minting (no external dependency)
- `getReservesBoth()` — V1+V2 reserves in one call
- `swapNativeForTokens()` / `swapTokensForNative()` / `wrapPLS()` / `unwrapWPLS()` — Native PLS handling
- `createV4(name, symbol, initialMint, parent)` — Deploy new V4 tokens
- Step.dex field — Per-step DEX selection in executeRoute

### Joey's On-Chain Identity

| Item | Address |
|------|---------|
| GIBS LAU | `0x66a08aa12da955eb63d7ac121a88b2b210a07b03` |
| Joey YUE wallet | `0x8e666227B0C5A42075a4f9bdf5d2176f287a9cf0` |
| GIBS QING venue | `0x1B8774C0d0ba2A814A592bE7978DFe78b0e86E35` |

### GIBS LP Pairs (10 live)

| Pair | DEX | Address | Notes |
|------|-----|---------|-------|
| GIBS/WPLS | V2 | `0x7BCa1c997c...` | Price anchor — E2 unlock |
| GIBS/FED | V2 | `0xA2a7a2153136b6ee075335b979fb6ac033412e4d` | Deepest arb surface |
| GIBS/ATROPA | V1 | `0xa152659B...` | pDAI intermediate route |
| GIBS/WM | V2 | `0xc23Cf1aF...` | |
| GIBS/DFM | V2 | `0x88c5B784...` | Thin — near-empty GIBS side |
| GIBS/PROOF_RES | V2 | `0xC52EFaed...` | |
| GIBS/ZHENG | V2 | `0xbFBEaf50...` | |
| GIBS/VOID | V2 | `0xB0776024...` | |
| GIBS/PARADE | V2 | `0xD8dA05aF...` | Thin — near-empty GIBS side |
| GIBS/TLRz | V2 | `0x7711f0dE...` | Thin — near-empty GIBS side |

87% of GIBS supply in LP. AMM bots actively arb between pairs — Joey earns fees on both sides.

## Joystick Bot Architecture

### Package Structure
- `core/` — config, chain (Multicall3), wallet, wallet_manager, executor, gas_guard, gas_oracle, simulator, strategist, concurrency, sell_queue, event_logger, rpc_provider, split_swap
- `oracle/` — price, scanner (272+ QINGs), profitability, route_auditor, pair_discovery, graph, data_store, supply_oracle
- `engines/` — EngineBase ABC + 8 engine files
- `loops/` — GameLoopBase ABC + terraform.py
- `dashboard/` — FastAPI read-only API server (see `scripts/Joystick/dashboard/CLAUDE.md`)
- `deploy/` — Production VPS deployment (systemd units, setup scripts, health checks)
- `bot.py` — V2 orchestrator: 3-wallet async pipeline with ROI-ranked engine selection

### Engine Status Summary

| # | Name | File | Wallet | Description | Status |
|---|------|------|--------|-------------|--------|
| E1 | RAZOR (Arb) | `arb.py` | Seller | Cross-DEX QING arbitrage via `atomicArb()` | Ready — net-negative per recon |
| E2 | CEREAL (Hub) | `dss.py` | Joey | JoystickHub `primeGibs` + `mintLPAndSell` — LP first, sell second | Wired — Hub V2 deployed |
| E3 | MERIDIAN (Beat) | `beat.py` | Joey | Territory positioning (`CHEON.Su` + `META.Beat`) | Running (Dione=41) |
| E4 | FACTORY | `token_factory.py` | Minter | AFF `multiBuyWith` all 5 routes + TGSv8 `mintWM()` | AFF routes unprofitable |
| E5 | ABUPRU (LAU) | `lau.py` | Joey | Mathematical state loop + EmitSniper | Gated (150K PLS floor) |
| E6 | DaVINCI (Treasury Sniper) | `treasury_sniper.py` | Joey | `batchClaimTreasury()` via recon data — routes through Joey | Wired — recon refresh added |
| E7 | BACKBONE (Spine Runner) | `spine_runner.py` | Minter | `batchMintAndClaim()` on Debenture=True V2 tokens | Blocked — needs E8 ARM |
| E8 | PHR3AK (Web Weaver) | `phreak.py` | Minter | Token Web Manipulation: ARM / DEPLOY / STITCH modes | Ready — ARM is critical path |

### Game Loops → Engine Mapping

| Loop | Engine | Mechanic |
|------|--------|----------|
| Hub Harvest | E2 | `primeGibs(N)` → `mintLPAndSell()` via JoystickHub — LP first, sell second |
| Beat | E3 | `CHEON.Su()` → `META.Beat()` → territory state (strategic, no direct PLS) |
| ABUPRU | E5 | Alpha→Beta→Upsilon(false)→Pi→Rho→Upsilon(true) state sequence |
| Treasury Claim | E6 | Scan for backing (`selfBalance > 0`), claim via `batchClaimTreasury()` |
| Spine Run | E7 | V2 Debenture=True loops via `batchMintAndClaim()` |
| Web Weave | E8 | Deploy V4 → mint → pair → burn LP → creates new arb/spine edges |

### Implementation Rules
- Always simulate via `eth_call` before sending any TX
- Always `estimate_gas()` with `GAS_MULT` (2.5x) multiplier — abort if it fails, never send blind
- **EIP-1559 Type 2 transactions** — all TXs use `maxFeePerGas` + `maxPriorityFeePerGas`, gas ceiling checks against actual `maxFeePerGas`
- Gas denomination: **Beats** (not Gwei). 1 PLS = 1,000,000,000 Beats. `eth_gasPrice` returns Impulses (wei-equiv) — divide by 10^9 for Beats
- Gas price ceiling: skip cycle if `maxFeePerGas > GAS_PRICE_CEIL`
- No OpenZeppelin imports in Solidity — inline guards
- Atomic file writes via `os.rename()` / `os.replace()`
- Never rewrite existing scripts — import as modules
- Chain ID: 369 (PulseChain)

## PLS Generation Strategies

### Strategy A: Hub Harvest (E2) — HIGHEST CONFIDENCE
JoystickHub `primeGibs(N)` → `mintLPAndSell()` — LP-first, sell-second. Per cycle: mint 17 GIBS, LP 50% + sell 50% → ~1,190 PLS net + LP position value. Zero VOID spam (silent minting via `mintToCap`).

### Strategy B: Treasury Sniping (E6)
Scan treasury tokens for claimable backing. If child tokens cheaper to acquire than parent tokens received, execute. Recon yields ~1-10K PLS after price impact.

### Strategy C: Spine Running (E7)
V2 Federal mint-claim loop on Debenture=True tokens. Only OZZY has Deb=true but PLS/token is near-zero. V2 Federal children NOT profitable — need E8.

### Strategy D: V3 Family Exploitation (E8)
High-value targets: MXDAI (118 PLS/tok, 29M liq), S&㉿500 (44K PLS/tok, 6M liq). V3 Claim is family-isolated — need to deploy sibling tokens as custom ammo via Web Weaver.

### Strategy E: Purchase→DEX Arbitrage (E1)
AFFECTION routes across 272 QING venues. Currently net-negative per last recon.

### Strategy F: AFFECTION BuyWith Routes (E4) — CURRENTLY UNPROFITABLE
All BuyWith routes currently net-negative. PI route: -64%. G5 route: -57%. AFF must trade above ~118 PLS (G5) or ~139 PLS (PI) for BuyWith paths to become profitable. Currently at ~50 PLS. WM minting also unprofitable (23 PLS mint cost vs 17 PLS DEX value).

## Treasury Recon Key Findings (2026-03-12)

**V2 Federal**: 13/14 Debenture=false. Only OZZY Deb=true, near-zero PLS/token. Not profitable for E7.

**V3 High-Value Targets** (family-isolated, need sibling ammo):
- MXDAI: 118 PLS/token, 29M PLS liquidity (parent=pDAI)
- S&㉿500: 44K PLS/token, 6M PLS liquidity (parent=N㉾SD㉾Q)

**V1 Tokens**: Bypass Debenture check — Claim always open.

**E8 Web Weaver** is the critical unlock for V3 exploitation.

## Grav's AFFECTION Pipeline (Reverse-Engineered)

3-bot system, active around blocks ~21M-21.1M (currently idle):

| Role | Address | Nonces |
|------|---------|--------|
| Funder | `0x1B79F904087DaaF6C67d7AD2cfA1D7c727De885B` | 13,702 |
| Minter | `0x217a76D9BEf7CeC27eFB5039099221241ca26F93` | 27,657 |
| Seller | `0xa767a0D5E04eD4c90Ad68F316A9E55090aa28c51` | 27,393 |

**Primary route**: PLS → pUSDC → MATH v1.1 → `BuyWithMATH()` on AFFECTION (via custom contracts `0x9837...` + `0x7947...`). **Secondary**: pDAI → PI/G5 via `0x32d3...`. Total: 4.06M AFF lifetime. Was profitable when AFF > 1 pDAI on DEX; currently at ~0.36 pDAI.

**Key pipeline contracts**:
| Contract | Address | Purpose |
|----------|---------|---------|
| pUSDC→MATH batch | `0x98375dA84C1493767De695Ee7Eef0BB522380A07` | pUSDC → BuyWithUSDC → MATH v1.1 |
| MATH→AFF batch | `0x79474ff39B0F9Dc7d5F7209736C2a6913cf50F82` | MATH → BuyWithMATH → AFF |
| Custom BuyWith wrapper | `0x32d390e9e1b1dc7af2349312ad55be81dfc6398c` | BuyWithPI/G5, forwards AFF to tx.origin |
| Multi PI (Grav's) | `0x3026512fd7116e0a6b6db942f263cc9eef063143` | pDAI → PI batch |
| AFF/WPLS V2 sell pair | `0x155172653e94a7e5f0e04126803dcb6896796fbb` | AFF sell target |
| pDAI/WPLS V2 pair | `0xae8429918fdbf9a5867e3243697637dc56aa76a1` | PLS → pDAI acquisition |

## QING Ownership Puzzle — UNRESOLVED

GIBS_QING owners: GIBS_LAU contract + CHO contract. Joey EOA is NOT an owner. `AddMarketRate` is `onlyOwners`. Unlock requires `CHO.AddContractOwner(GIBS_QING, Joey)` — needs CHO deployer or GIBS_QING contract to call.

**Workaround**: GIBS_LAU uses DYSNOMIA v1 (AFFECTION rate set at birth). `Purchase()` on GIBS_LAU works at 1:1.

## BEAT Status

`META.Beat(QingWaat)` running. Dione=41. WORLD not yet deployed on-chain. Prior revert root cause: `RING.Moments[Soul]`=0 → Iota=0 → division-by-zero. Fix: run `CHEON.Su()` first to initialize YUE bar state.

SHIO balances confirmed (block 25,903,711): Fornax 0.15 @ LAU/QING, Fomalhaute 0.001387 @ LAU, CHO 0.002921 @ LAU/QING.

## On-Chain Intelligence

- **Noumenon** (`0xEbE9B8673d...`) — 2,473+ txs, deep Dysnomia knowledge
- **Enteh** — 25-26 Beat calls, skips CHEON.Su()
- **RatKing** (`0x530c8cE7...`) — Fornax whale (25K)
- **Active bot** `0xb1c9b8d6...` → `0xc078C8DaE2...` — chatAndClaim loop
- **GIBS LP arb bots** — found 9,852 PLS/GIBS implied price on thin pools, immediately arbed back toward parity

---

# Part 3 — Session History

| # | Date | Key Events |
|---|------|------------|
| 1-4 | 2026-02-27/28 | LAU creation, Beat analysis, SHIO acquisition, wallet encryption |
| 5 | 2026-03-01 | Joystick bot (23 files), TGSv5 deployed (block 25,911,970) |
| 6 | 2026-03-02 | E5 LAU-ABUPRU + EmitSniper (merged via PR #3) |
| 7 | 2026-03-04 | Branch consolidation, canonical branch `claude/Joystick-Engines-Lj9Kp` set |
| 8 | 2026-03-04/05 | TGSv7→TGSv8 deploy, JV8A created via `createV4()`, V4 2x transferFrom bug found/fixed |
| 9 | 2026-03-09 | PLS stimulus (~1.95M), 10 LP pairs deployed (86 txs), E2 unlocked at 10x above break-even |
| 10 | 2026-03-12 | Treasury recon deep dive, V3 family isolation discovered, E7 blocked, E8 identified as unlock |
| 11 | 2026-03-12 | AFF mainnet test (`multiGenerate(100)` → mints to contract not caller), Grav pipeline reverse-engineered, AFF routes disabled |
| 12 | 2026-03-16 | V2 3-wallet architecture, strategist, concurrency, sell queue, E8 PHR3AK wired |
| 13 | 2026-03-22 | Bugfix audit, dashboard FastAPI server, Helios PingPong intelligence upgrades |
| 14 | 2026-03-28 | JoystickHub deployed (block 26,092,219), Hub:Harvest V2 (block 26,149,418), E2 rewritten V1→V2→V3 |
| 15 | 2026-03-29 | 6 autoresearch experiments merged: E2/E8 unblock, E4 sim timeout fix, E6 recon refresh |
| 16 | 2026-03-30 | EIP-1559 Type 2 TXs, GAS_MULT 2.5x, --e2-only/--e6-only CLI flags, E6 routes through Joey |

See `lore/joey_diary_*.md` for detailed session narratives.

### Key TX Log

| Event | TX / Address | Block |
|-------|-------------|-------|
| GIBS LAU deployed | → `0x66a08aa...` | 26,215,664 |
| Username "Joey" set | `0x1d46e25...` | 25,886,977 |
| DSS deployed | → `0x91Df693...` | 25,887,000 |
| SEI.Start() — YUE wallet | → `0x8e666227...` | 25,893,644 |
| MAP.New(GIBS) — QING venue | → `0x1B8774C0...` | 25,893,651 |
| DSS.setChatMultiplier(17) | | 25,893,803 |
| Handle |>JOYSTICK<| acquired | `0x2eb66e1b...` | 25,894,338 |
| **TGSv8 deployed** | → `0xAD352a27...` | 25,943,194 |
| **JV8A created** | → `0x364793Ea...` | 25,943,266 |
| **10 GIBS LP pairs** | (86 txs, nonce 25→111) | ~25,984,143 |
| **multiGenerate(100)** | `0x9867...` | 26,007,782 |
| **JoystickHub deployed** | → `0x7bd76A0f...` | 26,092,219 |
| **Hub:Harvest V2 deployed** | → `0x400D052F...` | 26,149,418 |

---

# Part 4 — Bugs & Lessons Learned

- **V4 mint 2x transferFrom**: V4 minter pulls 2x the requested amount. Fix: approve `type(uint256).max`
- **`execute 0 X func arg` bug**: C# command overwrites Alias with account number string. Use `execute X func arg` (no leading 0)
- **C# SendTransactionAsync hang**: Nethereum hangs on PulseChain. All TX moved to Python web3.py
- **MultiOwnable ownership trap**: `owner()` returns `address(this)` not EOA. MAP.New() adds LAU CONTRACT as QING owner, not Joey's wallet
- **V1 vs V2 AFFECTION**: v1 sets market rate at birth. V2/QING does NOT — requires manual `AddMarketRate` by owner
- **PulseChain gas units**: `eth_gasPrice` returns Impulses (wei-equiv). Divide by 10^9 for Beats. Display confusion caused stuck TX
- **RPC strategy**: `rpc-pulsechain.g4mm4.io` or `rpc.pulsechain.com` for reads; `rpc.pulsechain.com` for TX submit. `pulsechainstats.com` is read-only
- **`_mintToCap()` mints to self**: ALL DYSNOMIA tokens mint to `address(this)`. Must use `Purchase()` or `BuyWith*()` to extract. `Generate()` / `multiGenerate()` alone do NOT deliver tokens to caller
- **RPCPool "replacement TX underpriced"**: Multi-provider `send_raw()` causes race when first provider accepts and second rejects. For critical single TXs, use single `Web3.HTTPProvider()` directly
- **multiGenerate() is a public good**: Spends your gas to increase AFFECTION's self-balance supply — anyone can then buy it via `BuyWith*`. Donation, not profit
- **Legacy gas price ceiling**: Gas ceiling was checking `eth.gas_price` before building EIP-1559 params. Fixed: now checks actual `maxFeePerGas` after `build_gas_params()`
- **DSS chatAndClaim VOID spam**: V1 DSS spammed VOID chat. Replaced by JoystickHub silent minting via `mintToCap()` — zero chat noise
- **Gas multiplier too low**: 1.3x caused TX failures on PulseChain. Bumped to 2.5x (commit d14aecf)

---

# Part 5 — Current State (as of 2026-04-01)

```
PLS:        ~1,979,625         GIBS price:   ~195 PLS
GIBS:       0 (all in LP)      PLS/USD:      ~$7.14e-6
AFF:        ~127               GIBS supply:  3,396
WM:         ~263               TGSv8 WM:    9
ATROPA:     ~167               Nonce:        128+
VOID:       ~51                Gas:          EIP-1559 Type 2
```

Key changes since last snapshot:
- JoystickHub deployed + Hub:Harvest V2 active (E2 rewritten)
- EIP-1559 Type 2 TXs with 2.5x gas limit
- E6 routes through Joey wallet (TGSv8 balance gate removed)
- 6 autoresearch experiments merged (E2/E8 unblock, E4 timeout fix)

---

# Part 6 — Tools & Reference Files

### Infrastructure
- **RPC**: `rpc.pulsechain.com` (preferred), `rpc-pulsechain.g4mm4.io` (fallback)
- **Block explorer API**: BlockScout at `https://api.scan.pulsechain.com/api` — returns decoded function names, accessible via Python `requests`
- **Python libraries**: `pycryptodome` (`Crypto.Hash.keccak`) for function selectors (preferred over `pysha3`); `eth_abi` for calldata encode/decode
- **Pattern**: `eth_call_with_error` — post directly to RPC and check `error` field for simulation
- **Helios wiki**: `affection.gitbook.io` — AFFECTION minting mechanics, multi-mint contracts

### Key Implementation Files

**Solidity**:
- `solidity/dysnomia/01_dysnomia.sol` — Base token, AFFECTION market rate, Purchase()
- `solidity/dysnomia/10_void.sol` — VOID game controller
- `solidity/dysnomia/11_lau.sol` — LAU player token
- `solidity/dysnomia/domain/tang/03_meta.sol` — META.Beat()
- `solidity/dysnomia/domain/tang/02_cheon.sol` — CHEON.Su()
- `solidity/dysnomia/etc/DysnomiaSelfSnipev4.sol` — DSS (pragma ^0.8.21, no OZ) — DEPRECATED
- `contracts/TGSv8.sol` — Execution substrate (active)
- `contracts/TGSv8Plus.sol` — Extended TGSv8: silent LAU mint, atomic harvestCycle
- `contracts/JoystickHub.sol` — Modular proxy with delegatecall modules

**Python (Joystick bot)**:
- `scripts/Joystick/bot.py` — V2 orchestrator (3-wallet async pipeline)
- `scripts/Joystick/core/` — config, chain, wallet, wallet_manager, executor, gas_guard, gas_oracle, simulator, strategist, concurrency, sell_queue, event_logger, rpc_provider, split_swap
- `scripts/Joystick/oracle/` — price, scanner, profitability, route_auditor, pair_discovery, graph, data_store, supply_oracle
- `scripts/Joystick/engines/` — arb, dss, beat, token_factory, lau, treasury_sniper, spine_runner, phreak
- `scripts/Joystick/dashboard/` — FastAPI read-only API (overview, engines, wallet, gas, terminal, canopy, etc.)

**Python (standalone)** — see `scripts/CLAUDE.md` for full inventory:
- `scripts/tx_beat_flow.py` — Full Beat orchestration
- `scripts/tx_cheon_su.py` — CHEON.Su() YUE bar primer
- `scripts/scan_lau_arb.py` — Scan 272 QINGs for Purchase→DEX arb
- `scripts/intel_aff_bots.py` — AFFECTION bot pipeline analysis
- `scripts/deploy_joystick_hub.py` — JoystickHub deployment
- `scripts/deploy_tgsv8plus.py` — TGSv8Plus deployment

**Data**:
- `scripts/Joystick/data/recon_results.json` — Full treasury recon (39K lines)
- `scripts/Joystick/data/pair_registry.json` — Known DEX pairs (543K)
- `scripts/Joystick/data/token_master.json` — Master token registry (574K)
- `scripts/Joystick/data/spine_map.json` — Spine opportunity mapping (860K)
- `scripts/Joystick/data/deploy_candidates.json` — V4 token candidates for E8 DEPLOY
- `scripts/Joystick/data/v2_federal_tokens.json` — V2 Federal token scan

### Domain Glossary

| Term | Meaning |
|------|---------|
| LAU | Player character token (via VOID.Enter) |
| QING | Venue/marketplace contract |
| SHIO | Reactor system (Rod/Cone pair) — required for Beat |
| DSS | DysnomiaSelfSnipe — DEPRECATED, replaced by JoystickHub |
| Hub | JoystickHub — modular proxy with delegatecall (primeGibs, mintLPAndSell) |
| MV / WM | Token to deploy new minter tokens (1 MV per supply unit) |
| Debenture | True = unpublished = Claim stays open |
| Spine | Chain of V2 Federal tokens for recursive mint-claim |
| Purchase | Mint any ecosystem token at 1 AFFECTION each |
| _mintToCap | Internal — mints 1 token to contract self-balance per game action |
| ZHOU | Chat ledger (records all events) |
| Saat | Three-element Soul ID coordinate (uint64[3]) |
| ABUPRU | Alpha-Beta-Upsilon-Pi-Rho-Upsilon loop sequence |
| Beats | PulseChain gas unit (not Gwei). 1 PLS = 1B Beats |
| Impulses | Wei-equivalent on PulseChain (what eth_gasPrice returns) |
| Heart's Law | Price correlation through LP bonding |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Gravshub) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
