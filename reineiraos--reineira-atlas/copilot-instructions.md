## 06-strategy

> Use for business strategy, pricing, pivot decisions, business model, tokenomics, incentives, flywheel, TAM, market size, unit economics, revenue, LTV, CAC


# Strategy & Tokenomics

> **Read before acting:**
> - `.claude/docs/intelligence/METRICS.md` — ALWAYS check current numbers first
> - `.claude/docs/strategy/BUSINESS_MODEL.md` — current revenue model
> - `.claude/docs/strategy/ROADMAP.md` — current phase and priorities
> - `.claude/docs/strategy/TOKENOMICS.md` — incentive design

## Decision Framework

For every strategic question:

1. **State the decision** — what exactly is being decided?
2. **List options** — at least 2, ideally 3
3. **Score each option:**
   - Revenue impact (0-10)
   - Effort required (0-10, lower = better)
   - Risk level (0-10, lower = better)
   - Protocol alignment (0-10) — does it leverage FHE/escrow advantages?
4. **Recommend** — pick one, explain why
5. **Flag irreversibility** — hard to reverse? Say so explicitly

## Protocol-Aware Strategy

Always consider:
- **FHE moat** — what does encryption uniquely enable?
- **Escrow economics** — fee per transaction, volume needed
- **Insurance dynamics** — premiums vs claims, sustainability
- **Cross-chain reach** — CCTP v2 expands addressable market
- **Open economy** — can the venture attract LPs, operators, policy builders?

## Revenue Model Patterns

| Pattern                | Typical Fee      | Margin | Best For                   |
| ---------------------- | ---------------- | ------ | -------------------------- |
| Escrow fee             | 0.3-1% per trade | ~95%   | Any escrow-based venture   |
| Insurance premium      | 1-5% of trade    | 70-90% | Ventures with dispute risk |
| LP management fee      | 0.3-0.5% AUM/yr  | ~95%   | Pool-based ventures        |
| Protocol fee on claims | 2-5% of payout   | ~100%  | Insurance ventures         |
| API/white-label        | $2-8 per policy  | 85-90% | Platform play (Year 2+)    |
| SaaS subscription      | $49-199/mo       | ~92%   | B2B with recurring use     |

## The Flywheel

```
Builder writes accurate policy → Pool attaches it →
Users purchase coverage → Premiums flow to pool →
Stakers earn yield → More liquidity enters →
More capacity → More coverage sold → More premium
```

## Fee Design Principles

1. Fees must be value-proportional
2. Keep it simple — complex fees confuse users
3. Align incentives — everyone benefits from system working well
4. Sustainability — premiums must exceed expected claims without subsidies

## Stress Tests

Before finalizing any recommendation:
- What if this takes **2x longer** than expected?
- What if the **market shifts** before we finish?
- What if we're **wrong about the core assumption**?
- What's the **cheapest way to test** before committing?

## What You Must Not Do

- Do not recommend spending money the brief says isn't available
- Do not suggest features not in the current phase
- Do not make technical architecture decisions (that's product's job)
- Do not present options without a recommendation
- Do not design token models as a substitute for real revenue
- Do not recommend token launches at idea/MVP stage

## Checklist

- [ ] Checked METRICS.md for current numbers
- [ ] Options scored with framework
- [ ] Recommendation is clear and actionable
- [ ] Protocol alignment considered
- [ ] Strategic decision logged to `.claude/data/decisions/`

---
> Source: [ReineiraOS/reineira-atlas](https://github.com/ReineiraOS/reineira-atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
