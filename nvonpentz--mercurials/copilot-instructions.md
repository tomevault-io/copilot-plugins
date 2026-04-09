## mercurials

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mercurials is a full-stack Web3 application for on-chain generative art NFT auctions. It combines Solidity smart contracts with a Next.js frontend. NFTs feature procedurally generated SVG artwork with deterministic attributes from seed data, and use Variable Rate Gradual Dutch Auction (VRGDA) pricing.

## Commands

### Frontend Development
```bash
npm run dev          # Start Next.js dev server (http://localhost:3000)
npm run build        # Build production bundle
npm run lint         # Format with Prettier and run ESLint
```

### Smart Contract Development
```bash
cd contracts
forge build          # Compile contracts (uses --via-ir)
forge test           # Run test suite
forge test --match-test testNextToken -vvv  # Run single test with verbosity
```

### Local Development Workflow
```bash
# Terminal 1: Start local blockchain
anvil --block-time 12

# Terminal 2: Deploy contracts
cd contracts
forge script script/Deploy.sol --rpc-url http://localhost:8545 --broadcast --private-key 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80

# Terminal 3: Start frontend
npm run dev

# Optional: Auto-redeploy on contract changes
npm run watch
```

## Architecture

### Smart Contracts (`/contracts`)
- **Mercurials.sol**: Main ERC721 contract inheriting `LinearVRGDA` for pricing and `ReentrancyGuard` for security. Core functions:
  - `mint(uint256 tokenId, bytes32 blockHash)` - Mint with payment
  - `nextToken()` - Returns next token details (ID, SVG, price, TTL)
  - `tokenURI()` - On-chain metadata with deterministic SVG generation
- **Dependencies**: Git submodules in `lib/` (forge-std, openzeppelin-contracts, VRGDAs, solmate)
- **Deployment artifacts**: Written to `deploys/` with chain-specific addresses and ABIs

### Frontend (`/pages`, `/components`, `/utils`)
- **Tech stack**: Next.js 12, React 18, wagmi, RainbowKit, ethers
- **Chain config**: `utils/config.tsx` imports deployment artifacts and exports chain-specific contract addresses/ABIs
- **Key pattern**: Wagmi hooks (useContractRead, useContractWrite) with RainbowKit wallet connection
- **Components**: MintButton (transactions), PriceInfo (VRGDA pricing), ExpiresIn (token TTL), TransactionInfo (minted details)

### Deployment Flow
1. `Deploy.sol` script deploys contract and extracts ABI from Forge output
2. Artifacts written to `deploys/mercurials.{chainId}.address.json` and `deploys/mercurials.{chainId}.compilerOutput.json`
3. Frontend `config.tsx` imports these for multi-chain support (mainnet: 1, goerli: 5, local: 31337)

## Environment Setup

Copy `.env.example` to `.env` and set `NEXT_PUBLIC_ALCHEMY_API_KEY` for RPC access.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nvonpentz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nvonpentz)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
