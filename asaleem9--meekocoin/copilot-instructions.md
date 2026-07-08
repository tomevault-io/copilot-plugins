## meekocoin

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MeekoCoin is a Solana SPL memecoin with a Next.js landing page. The project uses npm workspaces with two packages: `token` (Solana scripts) and `web` (Next.js site).

**Live Token:** `9AqPGi9n7unEA8C6T6ujHxXsg1ywb1Ro6fitw9daMGNa` (mainnet, mint and freeze authority both revoked)

**Live Site:** https://meekocoin.xyz (also served at meekocoin.vercel.app). meekocoin.com is NOT registered — don't reference it. Canonical URL lives in `SITE_URL` in `web/lib/constants.ts`.

## Commands

### Token Scripts (require Solana CLI + funded wallet)
```bash
# Set network via environment variable
NETWORK=devnet npm run token:create          # Create token with metadata on Arweave
NETWORK=devnet npm run token:mint            # Mint full supply to deployer wallet
NETWORK=devnet npm run token:revoke          # Permanently revoke mint authority (IRREVERSIBLE)
NETWORK=devnet npm run token:revoke-freeze   # Permanently revoke freeze authority (IRREVERSIBLE)

# Update metadata (logo/description)
NETWORK=mainnet npm run token:update

# Trigger DexScreener indexing: one dust swap (0.002 SOL) against the MEEKO/SOL
# Raydium CPMM pool — DexScreener indexes a pair after its first trade.
# Needs the deployer wallet funded with ≥0.01 SOL; prompts "yes" before spending.
NETWORK=mainnet npm run token:trigger-index
```

### Web Development
```bash
npm run web:dev      # Start dev server (localhost:3000)
npm run web:build    # Production build
```

### Install Dependencies
```bash
npm install          # Installs both workspaces
```

## Deployment

The repo root is linked to Vercel project `meekocoin` and connected to the `asaleem9/MeekoCoin` GitHub repo: **every push to `main` auto-deploys to production.** Manual deploys: `vercel --prod` from the repo root (never from `web/`). Build settings live in root `vercel.json` (builds the `web` workspace, output `web/.next`).

## Architecture

### Token Scripts (`token/src/`)
- **config.ts** - Central configuration: token name, symbol, supply (420.69M), RPC endpoints, keypair path
- **create-token.ts** - Uploads logo/metadata to Arweave via Irys, creates SPL token mint with Metaplex metadata
- **mint-tokens.ts** - Mints entire supply to deployer wallet
- **revoke-authority.ts** - Permanently disables minting (uses `mpl-toolbox` setAuthority)
- **revoke-freeze.ts** - Permanently disables freezing wallets (uses `mpl-toolbox` setAuthority)
- **update-metadata.ts** - Updates on-chain metadata (logo, description) without affecting supply

All scripts read the `NETWORK` env var and keypair from `~/.config/solana/id.json`. `NETWORK` must be exactly `devnet` or `mainnet` — anything else throws at startup. The scripts have safety guards: create/mint refuse to run against an existing mint, and both revoke scripts require typing `yes`.

### Web (`web/`)
Next.js 14 App Router. Scroll-driven single-page experience: GSAP 3 (ScrollTrigger, SplitText, DrawSVG, Physics2D, ScrambleText — all free since 3.13) + Lenis smooth scrolling + React Three Fiber v8 / drei v9 (WebGL hero) + Remotion `<Player>` background loops + Tailwind ("acid chrome" theme: void/acid/zap/ice/chrome palette).

Key files:
- `lib/constants.ts` - Single source of truth for `CONTRACT_ADDRESS`, `SITE_URL`, and `JUPITER_SWAP_URL` — never hardcode these in components
- `lib/gsap.ts` - The ONLY place `gsap.registerPlugin` runs; every component imports gsap from here. Also holds `SECTION_PRIORITY` (descending `refreshPriority` per section — required because dynamic imports make ScrollTrigger creation order nondeterministic)
- `lib/scroll.ts` - Module-level Lenis handle (`getLenis`/`lockScroll`/`unlockScroll`) + `isAppReady` one-shot state. Entrance animations gated on the `app:ready` event MUST check `isAppReady()` first — if their `useGSAP` rebuilds after the preloader finished, the event never re-fires and paused `from()` tweens leave elements invisible (`immediateRender` applies hidden start-states even when paused)
- `components/providers/MotionProvider.tsx` - Motion tiers: `full` (everything) / `lite` (coarse pointer, small screens, weak devices: no pins, no WebGL) / `off` (prefers-reduced-motion: static page, native scroll). Every section reads the tier and adapts
- `components/providers/SmoothScroll.tsx` - Lenis in window mode only (wrapper/transform mode breaks pins and fixed overlays); `anchors: true` handles `#contract`/`#how-to-buy`
- `components/sections/` - The 8 page acts; each owns its ScrollTriggers via `useGSAP({ scope, dependencies: [tier], revertOnUpdate: true })`
- `components/sections/LiveChart.tsx` - Queries the DexScreener API on mount; shows a "charts awaken soon" fallback until a MEEKO pair is indexed, then renders the embed automatically. Never wrap the iframe in animated transforms
- `components/eggs/EggLayer.tsx` + `lib/eggs.ts` - Hidden easter eggs (Konami DEGEN MODE, typing "meow", 60s idle sleep, dizzy coin, 42%/69% scroll "nice.", roadmap grass hover, console ASCII, 4:20 sticker, footer 5-click, `#treats` hash)
- CSS gradient text (`.text-holo`/`.text-chrome`) must be applied to SplitText **chars** (via `charClassName` on `SplitHeading`), not the parent — background-clip dies against the mask wrappers

## Gotchas

- `.gitignore` blanket-ignores `*.json` to protect Solana keypairs, with explicit `!` exceptions. **Any new JSON file must be added to the exception list or git will silently ignore it** (this once dropped `package-lock.json` from the repo).
- Port 3000 is sometimes occupied by a stale dev server; `next dev` then silently picks 3001 — check the startup log for the actual port.
- **`three` is pinned exactly at `0.170.0`** (web deps + root `overrides`). drei 9.x bundles `three-mesh-bvh ^0.7`, which breaks at runtime with three ≥ 0.172. Never bump three without also moving off drei v9 (which itself requires React 19 / fiber v9). The root `overrides` entry exists because drei's deps (`maath`, `stats-gl`) otherwise pull in a second, newer `@types/three` that breaks type-checking.
- No CSS `scroll-behavior: smooth` anywhere — it double-eases against Lenis. Same reason `<main>` uses `overflow-x-clip`, not `overflow-x-hidden`.

## Token Configuration

Edit `token/src/config.ts` to change:
- Token name/symbol/description
- Total supply (currently 420,690,000 with 9 decimals)
- External links (website, twitter)
- Logo path

## Solana Tooling

Uses Metaplex Umi framework with:
- `@metaplex-foundation/mpl-token-metadata` - Token metadata program
- `@metaplex-foundation/mpl-toolbox` - Authority management
- `@metaplex-foundation/umi-uploader-irys` - Arweave uploads via Irys

---
> Source: [asaleem9/MeekoCoin](https://github.com/asaleem9/MeekoCoin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
