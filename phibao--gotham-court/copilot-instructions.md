## gotham-court

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project: Gotham Court

Decentralized AI-powered dispute resolution + prediction market on GenLayer. Users file cases, defendants submit defenses, the crowd bets real GEN on outcomes, and AI validators judge via Optimistic Democracy consensus.

**Deployed contract**: `0x09c7fF6DbaF4dA1A826eCa3B2D46cF11Dab9f064` on GenLayer studionet (chain ID 61999).

## Quick Commands

```bash
npm run deploy          # Deploy contracts via GenLayer CLI
genlayer deploy         # Alternative deploy command
cd frontend && npm run dev   # Start frontend dev server
cd frontend && npm run build # Build frontend for production
gltest                  # Run contract tests (requires GenLayer Studio)
genlayer network        # Select network (studionet/localnet/testnet)
```

## Architecture

```
contracts/
  gotham_court.py       # GenLayer intelligent contract (cases + betting + events + real GEN transfers)
frontend/               # Next.js 16 app (React 19, TypeScript, TanStack Query, Radix UI)
  app/
    page.tsx            # Main page (hero, case feed, how-it-works)
    faucet/page.tsx     # GEN faucet page with live wallet balance
  components/
    BettingPanel.tsx    # Parimutuel betting UI (place bets / claim winnings)
    CaseFeed.tsx        # Case list + filters + analytics + betting pool bars
    CaseDetail.tsx      # Case view + timeline + judgment + betting panel
    FileCaseModal.tsx   # File case dialog with validation
    Leaderboard.tsx     # Top cases by betting volume (pool ranking)
    BetHistory.tsx      # User's active + past bets across all cases
    Navbar.tsx          # Navigation + stats + faucet link
    AccountPanel.tsx    # MetaMask wallet panel
  lib/contracts/
    GothamCourt.ts      # SDK wrapper (payable bets + balance queries)
    types.ts            # TypeScript types (Case, Bet, CaseBetTotals)
  lib/hooks/
    useGothamCourt.ts   # TanStack Query hooks (cases + bets + escrow + balance)
  lib/genlayer/
    WalletProvider.tsx  # MetaMask provider
    client.ts           # Network config + provider helpers
deploy/                 # TypeScript deployment scripts (genlayer deploy)
test/                   # Python integration tests (gltest)
config/                 # Python config loader
.audit/findings/        # Security audit reports (Feynman + State Inconsistency passes)
```

## Key Technical Details

- **GenVM**: Does NOT support `import json`. Use `from dataclasses import dataclass` explicitly.
- **Address type**: SDK passes addresses as strings. Use `Address(defendant)` conversion in contract.
- **genlayer-js SDK**: `readContract` returns JavaScript `Map` objects, not plain objects. Frontend converts with `item.forEach((value, key) => obj[key] = value)`.
- **writeContract**: Pass `value: BigInt(0)` for non-payable calls. For betting, pass `value: amountWei` (in wei).
- **TransactionStatus**: Import from `genlayer-js/types`.
- **Chain**: Use `studionet` from `genlayer-js/chains`.

## Contract Pattern

### Case Lifecycle + AI Judgment
```python
from dataclasses import dataclass
from genlayer import *

@allow_storage
@dataclass
class Case:
    id: u256
    plaintiff: Address
    # ... title, description, evidence_urls, defense_text, defense_urls, verdict, reasoning, severity
    status: str  # OPEN, DEFENSE, JUDGED

class GothamCourt(gl.Contract):
    cases: TreeMap[u256, Case]
    case_count: u256

    @gl.public.write
    def file_case(self, defendant: Address, ...) -> u256:
        defendant_as_addr = Address(defendant) if isinstance(defendant, str) else defendant
        # ...

    @gl.public.write
    def judge_case(self, case_id: u256) -> None:
        # Uses gl.vm.run_nondet_unsafe(leader_fn, validator_fn)
        # leader_fn: scrapes evidence via gl.nondet.web.render(), generates verdict via gl.nondet.exec_prompt()
        # validator_fn: independently re-runs and compares verdict + severity (±2 tolerance)
```

### Betting (Real GEN Transfers)
```python
# Receiving native GEN from users
@gl.public.write.payable
def place_bet(self, case_id: u256, outcome: str) -> None:
    amount = gl.message.value  # Real GEN sent by user (in wei)
    # ... store bet, update pool totals

# Sending native GEN to winners
@gl.public.write
def claim_winnings(self, case_id: u256) -> u256:
    # ... calculate proportional payout
    bet.claimed = True  # State update BEFORE external effect
    if winnings > 0:
        _Recipient(sender).emit_transfer(value=winnings)  # Real GEN transfer
    return winnings
```

### EOA Transfer Helper (v0.1.x)
```python
@gl.eth_contract
class _Recipient:
    pass

# Usage: _Recipient(Address(addr)).emit_transfer(value=amount)
```

**Note:** v0.1.3+ uses `@gl.evm.contract_interface` with `View`/`Write` inner classes. v0.1.0 uses `@gl.eth_contract` with a minimal body. Events (`gl.Event`) are available in newer SDK versions but syntax varies by release — check the GenLayer docs for your target network before enabling.

## Frontend Patterns

- Contract interactions: `frontend/lib/contracts/GothamCourt.ts`
- React hooks: `frontend/lib/hooks/useGothamCourt.ts`
- Wallet context: `frontend/lib/genlayer/WalletProvider.tsx`
- GenLayer client: `frontend/lib/genlayer/client.ts`
- Betting UI: `frontend/components/BettingPanel.tsx`

### Betting Frontend Pattern
```typescript
// Place a bet (sends real GEN)
const amountWei = BigInt(Math.floor(parseFloat(amountGen) * 10 ** 18));
await client.writeContract({
  address: contractAddress,
  functionName: "place_bet",
  args: [caseId, outcome],
  value: amountWei,  // Native GEN transfer
});

// Claim winnings (receives real GEN transfer back)
await client.writeContract({
  address: contractAddress,
  functionName: "claim_winnings",
  args: [caseId],
  value: BigInt(0),
});
```

---

## GenLayer Technical Reference

> **Can't solve an issue?** Always check the complete SDK API reference:
> **https://sdk.genlayer.com/main/_static/ai/api.txt**
>
> Contains: all classes, methods, parameters, return types, changelogs, breaking changes.

### Documentation URLs

| Resource | URL |
|----------|-----|
| **SDK API (Complete)** | https://sdk.genlayer.com/main/_static/ai/api.txt |
| Full Documentation | https://docs.genlayer.com/full-documentation.txt |
| Main Docs | https://docs.genlayer.com/ |
| GenLayerJS SDK | https://docs.genlayer.com/api-references/genlayer-js |
| Value Transfers | https://docs.genlayer.com/developers/intelligent-contracts/features/value-transfers |
| Balances | https://docs.genlayer.com/developers/intelligent-contracts/features/value-transfers |

### What is GenLayer?

GenLayer is an AI-native blockchain where smart contracts can natively access the internet and make decisions using AI (LLMs). Contracts are Python-based and executed in the GenVM.

### Betting Model: Parimutuel Pool

Gotham Court uses a **parimutuel (pool) betting system**, NOT an orderbook:
- All bets on a single case go into one shared pool
- The pool is divided into three outcome buckets (Guilty / Not Guilty / Insufficient Evidence)
- After the AI verdict, the **entire pool** (winning + losing bets) is distributed proportionally to winners based on their stake share
- If nobody bet on the winning outcome, all bettors receive a refund
- The contract tracks `case_escrow` to monitor how much GEN is locked per case

### Web Access (`gl.nondet.web`)

```python
gl.nondet.web.get(url: str, *, headers: dict = {}) -> Response
gl.nondet.web.post(url: str, *, body: str | bytes | None = None, headers: dict = {}) -> Response
gl.nondet.web.render(url: str, *, mode: Literal['text', 'html']) -> str
gl.nondet.web.render(url: str, *, mode: Literal['screenshot']) -> Image
```

### LLM Access (`gl.nondet`)

```python
gl.nondet.exec_prompt(prompt: str, *, images: Sequence[bytes | Image] | None = None) -> str
gl.nondet.exec_prompt(prompt: str, *, response_format: Literal['json'], image: bytes | Image | None = None) -> dict
```

### Equivalence Principle

Validation for non-deterministic outputs:

| Type | Use Case | Function |
|------|----------|----------|
| Strict | Exact outputs | `gl.eq_principle.strict_eq()` |
| Comparative | Similar outputs | `gl.eq_principle.prompt_comparative()` |
| Non-Comparative | Subjective assessments | `gl.eq_principle.prompt_non_comparative()` |

### Value Transfers (Native GEN)

```python
# Receiving value (payable method)
@gl.public.write.payable
def deposit(self):
    received = gl.message.value  # u256, amount in wei

# Sending value to another IC
other = gl.contract.get_at(recipient_address)
other.emit_transfer(value=u256(amount), on='finalized')

# Sending value to an EOA (wallet address)
_Recipient(Address(recipient_address)).emit_transfer(value=u256(amount))

# Reading balance
my_balance = self.balance  # u256
```

### Events (SDK version-dependent)

Events syntax varies by GenLayer SDK release:
- **v0.1.0**: Events may not be available or have limited support
- **v0.1.3+**: `class MyEvent(gl.Event): ...` with positional-only `/` separator for indexed fields
- **v0.3.0+**: `class MyEvent(gl.vm.Event): ...`

Always verify event support against your target network's GenLayer version before using them in production.

### Key Documentation Links

- [Introduction to Intelligent Contracts](https://docs.genlayer.com/developers/intelligent-contracts/introduction)
- [Storage](https://docs.genlayer.com/developers/intelligent-contracts/storage)
- [Deploying Contracts](https://docs.genlayer.com/developers/intelligent-contracts/deploying)
- [Crafting Prompts](https://docs.genlayer.com/developers/intelligent-contracts/crafting-prompts)
- [Value Transfers](https://docs.genlayer.com/developers/intelligent-contracts/features/value-transfers)
- [Contract Examples](https://docs.genlayer.com/developers/intelligent-contracts/examples/storage)
- [Testing Contracts](https://docs.genlayer.com/developers/decentralized-applications/testing)

---
> Source: [PhiBao/gotham-court](https://github.com/PhiBao/gotham-court) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
