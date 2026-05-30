## stripe-test-cards

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file TypeScript library providing a typed, comprehensive reference of all Stripe test card numbers. Intended to be imported directly into test suites that use Stripe's sandbox/test API keys.

## Commands

```bash
npm run build      # compile to dist/ (ESM + CJS + .d.ts) via tsup
npm run typecheck  # tsc type-check without emitting
npm publish        # publish to npm (run build first)
```

## Structure

The entire library lives in `index.ts`. Named `index.ts` rather than `stripe-test-cards.ts` so that direct source imports work without an explicit path (TypeScript cannot import a file whose name contains hyphens as a bare identifier). The package name already provides the `stripe-test-cards` context. There are no dependencies; `tsup` compiles it to `dist/` for publishing.

The file is organized into sections:

1. **Types** — Four exported interfaces: `StripeTestCard` (base), `StripeDeclineCard`, `Stripe3DSCard`, `StripeCountryCard`
2. **Card collections** — Internal `const` objects (not individually exported), each typed with `as const satisfies`, composed into a single default export named `cards`:
   - `cards.standard` — Always-succeeding cards across major networks (Visa, Mastercard, Amex, Discover, JCB, UnionPay, etc.)
   - `cards.coBranded` — Cartes Bancaires and EFTPOS Australia co-branded variants
   - `cards.declined` — Cards that trigger specific decline codes
   - `cards.radar` — Cards that simulate Stripe Radar fraud-scoring outcomes
   - `cards.dispute` — Cards that trigger dispute/chargeback flows
   - `cards.disputeDeflection` — Visa RDR, CE3, Ethoca Alerts
   - `cards.threeDS` — 3D Secure / SCA scenarios (required, supported, frictionless, challenge flows)
   - `cards.terminal` — In-person / PIN scenarios
   - `cards.refund` — Async refund state transitions
   - `cards.balance` — Cards that bypass pending balance
   - `cards.country` — Country-specific billing cards (keys are uppercase ISO 3166-1 alpha-2 codes)
   - `cards.all` — Flat spread of all cards for parameterised tests
3. **Usage** — Single default import; all inner keys are camelCase:
   ```ts
   import cards from 'stripe-test-cards';
   cards.standard.visa.number
   cards.declined.insufficientFunds.number
   cards.threeDS.requiredIe.number
   ```

## Conventions

- Each card entry has `number`, `brand`, `cvc`, `expiry`, and `description`. Decline cards add `errorCode`/`declineCode`; 3DS cards add `threeDSStatus`/`outcome`; country cards add `country`.
- Optional `pmId` (`pm_card_*`) and `tokenId` (`tok_*`) fields are included where Stripe provides programmatic identifiers.
- All collections use `as const satisfies <Type>` to get both literal inference and type-checking at the same time.
- Helper constants `CVC3`, `CVC4`, and `FUTURE` are used internally to avoid repeating literal strings.

---
> Source: [qqilihq/stripe-test-cards](https://github.com/qqilihq/stripe-test-cards) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
