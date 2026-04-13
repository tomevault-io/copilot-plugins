## stacks-x402-jobs

> Decentralized marketplace for pay-per-use AI services ("skills") using the x402 HTTP 402 payment protocol on Stacks blockchain. Users and AI agents discover, pay for, and execute automated tasks with STX micropayments — no subscriptions, API keys, or accounts.

# x402-stacksJobs — AI Assistant Instructions

## Project Overview
Decentralized marketplace for pay-per-use AI services ("skills") using the x402 HTTP 402 payment protocol on Stacks blockchain. Users and AI agents discover, pay for, and execute automated tasks with STX micropayments — no subscriptions, API keys, or accounts.

## Tech Stack
- **Framework:** Next.js 16 (App Router, React 19)
- **Styling:** TailwindCSS + shadcn/ui
- **Language:** TypeScript (strict mode)
- **Blockchain:** Stacks (STX payments via `x402-stacks` SDK)
- **Wallet:** `@stacks/connect` (Hiro, Leather, Xverse)
- **Database:** Supabase (PostgreSQL)
- **Hosting:** Vercel
- **Data fetching:** @tanstack/react-query
- **Validation:** zod

## Architecture Decisions
- **Payment model:** Facilitator pattern from `x402-stacks` (client signs, does NOT broadcast; facilitator settles atomically)
- **Skills registry:** Hardcoded config for 5 launch skills; Supabase for execution logs and stats
- **State management:** React Context for wallet state; react-query for server state
- **Network:** Stacks Testnet for development, Mainnet for production

## Code Conventions
- Use the Next.js App Router (`/app` directory)
- All blockchain/wallet interactions must be in `"use client"` components
- Server Components by default; only opt into client where needed
- Use `x402-stacks` `paymentMiddleware()` on API routes — do NOT reimplement payment verification manually
- Prefer named exports over default exports
- Use `zod` schemas to validate skill inputs at runtime
- Colocate types in `/types`, utilities in `/lib`, components in `/components`
- API routes live under `/app/api/`

## File Structure
```
/app                          # Next.js App Router pages
  /page.tsx                   # Homepage
  /layout.tsx                 # Root layout
  /skills/page.tsx            # Skills directory
  /skills/[skillId]/page.tsx  # Skill detail + live demo
  /docs/page.tsx              # Documentation
  /api/                       # API routes
    /registry/skills/route.ts
    /skills/[skillId]/route.ts  # x402-protected skill execution
    /stats/global/route.ts
/components                   # React components
  /wallet/                    # WalletProvider, WalletConnect
  /skills/                    # SkillCard, SkillGrid, SkillDemo
  /payment/                   # PaymentButton, PaymentModal, PaymentStatus
  /common/                    # Header, Footer, Navbar
/lib                          # Utilities and clients
  /x402/                      # x402 client & payment utils
  /stacks/                    # Stacks API & wallet utils
  /db/                        # Supabase client & queries
  /skills-config.ts           # Hardcoded skill definitions
/types                        # TypeScript type definitions
```

## Key Edge Cases to Handle
1. **Payment latency:** Stacks blocks take 10-30s. Show progressive states: Signing → Broadcasting → Confirming → Executing
2. **Wallet not installed:** Detect missing extension, link to install page
3. **Insufficient balance:** Pre-check balance before attempting payment; show faucet link on testnet
4. **User cancels wallet popup:** Reset UI gracefully, no error flash
5. **Network mismatch:** Detect if wallet is on wrong network, warn user
6. **Transaction replay:** Server must verify amount, recipient, memo, age (<1hr), and single-use
7. **Mobile:** Wallet extensions are desktop-only. Show desktop-only notice or WalletConnect fallback
8. **Hiro API rate limits:** Cache non-payment-critical data; add retry with exponential backoff

## x402 Payment Flow (Facilitator Pattern)
```
Client → GET /api/skills/{id}
Server → 402 + payment-required header (base64 JSON)
Client → Signs STX tx via wallet (does NOT broadcast)
Client → Retries with payment-signature header (signed tx bytes)
Server → POST /settle to facilitator with signed tx
Facilitator → Broadcasts to Stacks, confirms
Server → 200 + result + payment-response header
```

## 5 Launch Skills
| Skill | ID | Price | Data Source |
|-------|----|-------|-------------|
| Whale Tracker | whale-tracker | 0.1 STX | Hiro Stacks API |
| Content Craft | content-craft | 0.25 STX | OpenAI GPT-4 |
| Stacks Scout | stacks-scout | 0.5 STX | Hiro Stacks API |
| Profile Pro | profile-pro | 2 STX | OpenAI GPT-4 |
| Meme Radar | meme-radar | 0.1 STX | Social APIs |

## Important Links
- x402-stacks SDK: https://github.com/tony1908/x402Stacks
- x402-stacks npm: https://www.npmjs.com/package/x402-stacks
- Stacks API (mainnet): https://api.mainnet.hiro.so
- Stacks API (testnet): https://api.testnet.hiro.so
- Testnet faucet: https://explorer.hiro.so/sandbox/faucet?chain=testnet
- Facilitator: https://facilitator.stacksx402.com

## Do NOT
- Reinvent payment verification — use `paymentMiddleware()` from `x402-stacks`
- Use `pages/` router — this is App Router only
- Store private keys in code — always use environment variables
- Skip error handling on payment flows — every step can fail
- Use default exports for components or utilities

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Arowolokehinde)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Arowolokehinde)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
