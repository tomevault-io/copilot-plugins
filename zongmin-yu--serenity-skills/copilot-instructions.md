## serenity-skills

> AI infrastructure and semiconductor supply chain analysis, distilled from the public posts of Serenity ([@aleabitoreddit](https://x.com/aleabitoreddit)).

# Serenity Guidelines

AI infrastructure and semiconductor supply chain analysis, distilled from the public posts of Serenity ([@aleabitoreddit](https://x.com/aleabitoreddit)).

When trillions flow into AI buildout, some tiny company with no substitutes will be the bottleneck. Trace the capex from hyperscalers down through semiconductors, photonics, materials, and feedstock to find it before the market does.

## Quick Filter

Run these first. Two "No" answers means this is probably not a chokepoint.

| # | Principle | Question | No = Stop |
|---|-----------|----------|-----------|
| 1 | **Forced Demand** | Is someone *forced* to buy this input to ship their product? | No forced buyer = no chokepoint |
| 2 | **Size Mismatch** | Is the supplier's market cap < 1% of the annual capex it enables? | No mismatch = probably priced in |
| 3 | **No Substitute** | Is there no production-ready alternative within 24 months? | Substitute in qualification = expiring chokepoint |
| 4 | **Outside Voice** | Have ≥ 3 independent parties confirmed the constraint in 90 days? | Nobody else sees it = too early or wrong |

## Process (after passing the filter)

1. **Follow the Money** — Where is capex flowing? Which layer is heating up? (Also valid: enter from a geopolitical event or policy action that forces supply chain restructuring.)
2. **Decompose the BOM** — Break down the product. Which components take the largest cost share? (If BOM is not public, reconstruct from known supplier relationships + analyst reports + community intelligence.)
3. **Who Makes It?** — Count suppliers. How many can make this at required quality and volume?
4. **Find the Monopoly** — Supplier count ≤ 3 + small MC + high switching costs = chokepoint. (If upstream producers are large oligopolies, go one layer above to the smaller equipment/testing/controller companies they depend on.)
5. **Verify Irreplaceability** — Check substitutes. Qualification cycle? Capacity ramp timeline? (For early architecture transitions: qualification progress > trailing revenue.)
6. **Go Deeper** — Who supplies the supplier? Bottleneck within the bottleneck?
7. **Historical Pricing** — Similar scarce materials spiked how much? Cost of project delay?
8. **External Validation** — Earnings calls, analyst coverage, policy actions, combat/deployment validation?

## Variants

- **Architecture Migration** — Current-gen maxed out → next-gen needs different components. Evaluate by qualification timeline, not TTM revenue.
- **Operational Chokepoint** — Bottleneck is not a material but who can profitably monetize capacity: gross margins > raw capacity, financing structure > contract headline.
- **Event-Driven Entry** — Start from geopolitical event / export control → decompose mission BOM → find the small Western substitute.
- **Opaque Supply Chains** — No published BOM → reconstruct from public relationships, conferences, hiring, community intelligence.

## Scope

**Works for:** AI infrastructure (semi, photonics, HBM, packaging, substrates, feedstock), defense/critical materials, new architecture transitions, qualification-barrier industries.

**Does not work for:** Software platforms, macro/liquidity trades, fungible commodities, broad sector ETFs without component-level drilling, already-priced-in bottlenecks.

---

*Distilled from Serenity ([@aleabitoreddit](https://x.com/aleabitoreddit)). Not affiliated. Not financial advice.*

---
> Source: [zongmin-yu/serenity-skills](https://github.com/zongmin-yu/serenity-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
