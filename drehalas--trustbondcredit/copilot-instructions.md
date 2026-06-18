## trustbondcredit

> Monitors HBAR/USDC volatility via SupraOracles and automatically adjusts Bonzo vault liquidity ranges to reduce impermanent loss. Registered in HOL Registry. Use when an agent or user queries vault status, volatility readings, range adjustments, or emergency actions on Bonzo vaults.


# Volatility-Aware Rebalancer

Protects Bonzo vault depositors from impermanent loss by monitoring HBAR/USDC volatility and adjusting liquidity ranges accordingly.

---

## Decision Logic

| Volatility | Action | Range |
|---|---|---|
| < 15% | Tighten | ±5–10% |
| 15–30% | Maintain | current |
| 30–50% | Widen | ±30–40% |
| > 50% | Emergency withdraw | single-sided staking |

---

## Capabilities

| Capability | Description |
|---|---|
| Volatility monitoring | Real-time HBAR/USDC via SupraOracles |
| Range adjustment | Tighten, widen, or reset liquidity bounds |
| Emergency withdrawal | Move funds to single-sided staking |
| Reward harvesting | Claim and reinvest vault rewards |

---

## Chat Commands

### Volatility

| Input | Response |
|---|---|
| `What's the current volatility?` | Volatility % + regime label |
| `What are your thresholds?` | Decision table |

### Range

| Input | Response |
|---|---|
| `Tighten the range` | Set ±5–10% |
| `Widen the range` | Set ±30–40% |
| `Reset to default` | Restore ±15–20% |
| `What's the current range?` | Active bounds |

### Vault

| Input | Response |
|---|---|
| `What's my APY?` | Current yield |
| `How much is in the vault?` | TVL |
| `What rewards are pending?` | Unharvested balance |
| `Show my position` | Deposit details |

### Emergency

| Input | Response |
|---|---|
| `Emergency withdraw` | Move to single-sided staking |
| `Harvest now` | Claim pending rewards |
| `Stop auto-rebalancing` | Pause automation |

### General

| Input | Response |
|---|---|
| `What can you do?` | Capability list |
| `How do you work?` | Strategy summary |
| `Hire me` | Open collaboration session |

---

## Technical Integration

### Discovery

```bash
npx @hol-org/registry search "volatility rebalancer" 5
npx @hol-org/registry resolve uaid:aid:bond-credit-volatility
```

### Chat

```bash
npx @hol-org/registry chat uaid:aid:bond-credit-volatility "What's the volatility?"
npx @hol-org/registry chat uaid:aid:bond-credit-volatility "Tighten the range"
```

### Agent Integration

```javascript
// Query volatility
const result = await queryAgent('uaid:aid:bond-credit-volatility', {
  command: 'getVolatility'
});

// Subscribe to spike alerts
await subscribeToAgent('uaid:aid:bond-credit-volatility', {
  event: 'volatilitySpike',
  callback: 'https://your-agent.com/webhook'
});
```

---

## Collaboration

Other agents can:

- **Hire** — delegate volatility monitoring; receive threshold alerts
- **Pool** — contribute capital to a shared managed vault
- **Subscribe** — receive webhook events on regime changes

```
Agent: "Hire me"
→ "I'll monitor HBAR volatility and alert you when thresholds are crossed."

Agent: "Pool 1000 HBAR"
→ "Added to pool. Managing with current volatility strategy."

Agent: "Subscribe to volatility alerts"
→ "Subscribed. Alerts fire at 15%, 30%, and 50% crossings."
```

---

## Config

### Environment Variables

```bash
REGISTRY_BROKER_API_KEY=   # HOL Registry key
HEDERA_ACCOUNT_ID=         # 0.0.xxx
HEDERA_PRIVATE_KEY=        # Ed25519 private key
SUPRA_API_KEY=             # SupraOracles key
SUPRA_API_URL=             # SupraOracles API URL
```

### Agent Settings (`config.js`)

```javascript
{
  uaid: 'uaid:aid:bond-credit-volatility',
  thresholds: { low: 0.15, medium: 0.30, high: 0.50 },
  checkInterval: 5,   // minutes
  cooldown: 60,       // minutes between actions
  bonzoVault: '0x…'
}
```

---

## Resources

- [HOL Registry Docs](https://hol.org/docs)
- [Bonzo Vaults](https://bonzo.finance)
- [Hedera Agent Kit](https://github.com/hedera-dev/hedera-agent-kit)

---
> Source: [Drehalas/TrustBondCredit](https://github.com/Drehalas/TrustBondCredit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
