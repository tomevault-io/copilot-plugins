## bark-meme-io

> BARK is an AI-powered meme token risk auditor purpose-built for Four.Meme on BNB Chain. It auto-scans every new token launch, scores rug pull risk using AI, and learns from scam patterns over time via persistent memory.

# BARK — Blockchain Audit & Risk Keeper

## Project Overview

BARK is an AI-powered meme token risk auditor purpose-built for Four.Meme on BNB Chain. It auto-scans every new token launch, scores rug pull risk using AI, and learns from scam patterns over time via persistent memory.

**Tagline:** "We bark before you get bitten."

**Hackathon:** Four.Meme AI Sprint (DoraHacks) — Deadline April 30, 2026
**Prize Pool:** $50,000 USD main + $2,000 Pieverse bounty + $3,000 dgrid credits

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  FRONTEND (Next.js + Tailwind)           │
│  Live Feed / Token Search / Risk Dashboard / Score Card  │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│              BARK API (TypeScript / Node.js)              │
│                                                          │
│  ┌─────────────┐  ┌──────────────┐  ┌───────────────┐   │
│  │ Contract     │  │ Holder       │  │ Social        │   │
│  │ Analyzer     │  │ Analyzer     │  │ Analyzer      │   │
│  │ (GoPlus API) │  │ (BSCScan)    │  │ (optional)    │   │
│  └──────┬──────┘  └──────┬───────┘  └──────┬────────┘   │
│         └────────────────┼──────────────────┘            │
│                          ▼                               │
│              ┌───────────────────┐                       │
│              │   RISK SCORING    │                       │
│              │   (dgrid AI LLM)  │                       │
│              │   Score: 0-100    │                       │
│              └────────┬──────────┘                       │
└───────────────────────┼──────────────────────────────────┘
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
┌──────────────┐ ┌────────────┐ ┌──────────────┐
│   Unibase    │ │  Pieverse   │ │  BSC         │
│   Membase    │ │  x402       │ │  Contracts   │
│   (memory)   │ │  (payments) │ │  (registry)  │
└──────────────┘ └────────────┘ └──────────────┘
```

## Tech Stack

| Layer          | Technology                                      |
| -------------- | ----------------------------------------------- |
| Frontend       | Next.js 14+ (App Router) + Tailwind CSS         |
| Backend/API    | TypeScript / Node.js                             |
| AI Inference   | dgrid AI Gateway (`https://api.dgrid.ai/v1`)     |
| Memory         | Unibase Membase SDK (Python/TS)                  |
| Identity       | ERC-8004 (Unibase agent identity)                |
| Payments       | Pieverse x402 + pieUSD (gasless)                 |
| Contracts      | Solidity + Foundry (BSC testnet)                 |
| Data Sources   | GoPlus API, BSCScan API, DexScreener API         |
| Font Stack     | Inter (body), Space Grotesk (headings/scores), JetBrains Mono (addresses) |

## Brand Identity

### Colors
```
Primary:
  Deep Navy      #0A1628   — background
  Electric Amber #F59E0B   — primary accent

Score Colors:
  Danger Red     #EF4444   — DANGER (0-39)
  Warning Orange #F97316   — CAUTION (40-69)
  Safe Green     #22C55E   — SAFE (70-100)

Neutral:
  Slate Gray     #94A3B8   — secondary text
  White          #F8FAFC   — primary text on dark bg

Gradient (hero/banner):
  Navy → Dark Purple  #0A1628 → #1E1B4B
```

### Design Direction
- Dark theme (AI/security tool convention)
- Clean, geometric, modern tech aesthetic
- NOT pixel art — professional security dashboard
- Think DexScreener / DEXTools aesthetic
- Mascot: Shiba Inu guard dog (alert, vigilant, not cutesy)

### Score Badge Design
```
🟢 SAFE    (70-100) — Green badge
🟡 CAUTION (40-69)  — Orange badge
🔴 DANGER  (0-39)   — Red badge
```

## Four.Meme Integration Details

### Contract
- **TokenManager2:** `0x5c952063c7fc8610FFDB798152D69F0B9550762b` (BSC mainnet)
- **TokenPurchase event:** signature hash `0x7db52723a3b2cdd6164364b3b766e65e540d7be48ffa89582956d8eaebe62942`
- All event parameters are **non-indexed** — must manually decode data field (each param = 32 bytes / 64 hex chars)

### Token Lifecycle on Four.Meme
1. Creator deploys token (fixed 10B supply, no custom functions, no taxes)
2. Token trades on internal bonding curve
3. At ~24 BNB accumulated → auto-graduates to PancakeSwap with liquidity pair
4. Post-graduation: trades on PancakeSwap (external market)

### Detection Approach
- Poll Four.Meme contract events for new token creations (every 10 seconds for MVP)
- Each new token → auto-trigger audit pipeline
- Can also use Bitquery APIs or BSCScan API for event monitoring

## Data Sources

### GoPlus Security API (FREE — primary contract analysis)
- Endpoint: `https://api.gopluslabs.io/api/v1/token_security/{chain_id}?contract_addresses={address}`
- Chain ID for BSC: `56`
- Returns: honeypot check, mint function, owner powers, proxy, blacklist, tax info, 30+ security checks
- Use as INPUT data — we add AI reasoning layer on top

### BSCScan API
- Deployer wallet history (previous contracts created)
- Top token holders
- Contract source/verification status
- Transaction history

### DexScreener API
- Liquidity data
- Volume
- Price
- Pair info post-graduation

### Unibase Membase (historical scam pattern memory)
```python
# Python SDK
pip install git+https://github.com/unibaseio/membase.git

from membase.memory.buffered_memory import BufferedMemory
from membase.memory.message import Message

memory = BufferedMemory(membase_account="bark-agent", auto_upload_to_hub=True)
memory.add(Message(name="bark-agent", content="audit result JSON", role="assistant", metadata=""))
```

## dgrid AI Gateway Integration

### Setup (OpenAI SDK compatible — drop-in replacement)
```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  baseURL: 'https://api.dgrid.ai/v1',
  apiKey: process.env.DGRID_API_KEY,
});

const result = await openai.chat.completions.create({
  model: 'openai/gpt-4o-mini', // cheap + fast for scoring
  messages: [
    { role: 'system', content: SCORING_SYSTEM_PROMPT },
    { role: 'user', content: JSON.stringify(tokenData) }
  ],
});
```

### Also supports Claude-native format
```typescript
// POST https://api.dgrid.ai/v1/messages
// Headers: Authorization: Bearer <DGRID_API_KEY>; anthropic-version: 2023-06-01
// Model: claude-3-5-sonnet-20241022
```

### Available models
- Format: `[provider]/[model-name]` (e.g., `openai/gpt-4o`, `openai/gpt-4o-mini`)
- Full list: https://dgrid.ai/models

### dgrid x402 API (bonus narrative — agent pays for its own inference)
- Pay-per-inference via x402 on BSC
- Payment token: USD1 (`0x8d0D000Ee44948FC98c9B98A4FA4921476f08B0d`)
- Flow: request → 402 → sign → retry with `x-payment` header → response
- Docs: https://docs.dgrid.ai/x402

## Scoring Algorithm

### Weight Distribution
```
Contract Security (40%)
├── Honeypot check                10%
├── Mint function                  8%
├── Owner powers (pause/blacklist) 8%
├── Proxy/upgradeable              7%
├── Source verified                 7%

Holder Analysis (30%)
├── Top 10 wallet concentration   12%
├── Sniper detection (block 0-1)   8%
├── Deployer previous rugs        10%

Liquidity/Market (20%)
├── Liquidity lock status          8%
├── Bonding curve progress         7%
├── LP size post-graduation        5%

Social/Meta (10%)
├── Twitter account age            4%
├── Telegram existence             3%
├── Website domain age             3%
```

### LLM Scoring Prompt
```
You are BARK, an expert meme token risk analyst on BNB Chain.
Given the following data about a token launched on Four.Meme, provide:

1. risk_score: integer 0-100 (higher = safer)
2. risk_category: "SAFE" (70-100) | "CAUTION" (40-69) | "DANGER" (0-39)
3. top_risks: array of top 3 risk factors, each with title and description
4. summary: one-paragraph plain English explanation of why this token is or isn't safe
5. similar_scams: if memory contains similar patterns, reference them

Respond in JSON only. No markdown, no preamble.

TOKEN DATA:
{contract_analysis, holder_data, deployer_history, liquidity_data, social_data, memory_patterns}
```

## Smart Contracts (Solidity / Foundry)

### AuditRegistry.sol
```solidity
// Store audit results on-chain (only high-confidence scores)
struct AuditReport {
    uint8 score;           // 0-100
    uint256 timestamp;
    bytes32 reportHash;    // IPFS hash of full report
    address auditorAgent;  // BARK agent address (ERC-8004)
}

mapping(address => AuditReport) public audits;

function recordAudit(address token, uint8 score, bytes32 reportHash) external;
function getScore(address token) external view returns (uint8);
function flagToken(address token) external; // community flagging
```

### x402 Payment Integration (Pieverse bounty)
- Free tier: basic score (SAFE/CAUTION/DANGER) + top 3 risk factors
- Paid tier (0.50 pieUSD via x402): full detailed report, deployer history, similar scam comparison, AI narrative
- Payment flow: HTTP request → 402 Payment Required → pay via pieUSD (gasless) → receive full report

## Purrfect Claw Skill (Pieverse $2K bounty)

Skills are **markdown files** with YAML frontmatter, published to ClawHub.

### Skill Structure
```
skills/bark-token-audit/
└── SKILL.md
```

### SKILL.md
```markdown
---
name: bark_token_audit
description: Audit meme tokens on Four.Meme for rug pull risks. Returns risk score, top risk factors, and AI-generated safety analysis.
---

# BARK Token Audit

When the user provides a BSC token address or asks to check if a meme token
is safe, call the BARK API to analyze contract security, holder concentration,
deployer history, and return a risk score with explanation.

## Usage
- User says: "audit 0x1234..." or "is this token safe?" or "check this meme coin"
- Call: GET https://api.bark.tools/v1/audit/{token_address}
- Returns: risk score (0-100), category (SAFE/CAUTION/DANGER), top risks, summary

## Example Response
Score: 23/100 🔴 DANGER
Top Risks:
1. 89% supply held by 3 wallets
2. Hidden mint function detected
3. Deployer rugged 2 previous tokens

## Premium Report
For detailed analysis: GET https://api.bark.tools/v1/audit/{token_address}/full
Requires x402 payment (0.50 pieUSD via Pieverse)
```

Publish: `clawhub publish bark-token-audit`

## Feature Priority

### ESSENTIAL (MVP — must ship)
1. **Four.Meme Live Feed Scanner** — auto-detect & scan every new launch
2. **Core Risk Checks** — honeypot, mint, owner powers, holder concentration, liquidity lock (via GoPlus + BSCScan)
3. **AI Risk Scoring** — dgrid AI LLM generates score + natural language report
4. **Deployer Wallet History** — check if deployer has created scam tokens before
5. **Unibase Membase Integration** — store audit results, learn patterns over time
6. **Dashboard** — live feed view, token detail page, search by address

### NICE-TO-HAVE (if time allows)
7. Social Analyzer (Twitter age, Telegram check)
8. Purrfect Claw Skill (1 hour effort — easy $2K bounty pickup)
9. x402 Premium Reports (Pieverse integration)
10. On-chain Audit Registry (AuditRegistry.sol)
11. Alert system (Telegram/Discord bot notifications)
12. Comparison mode ("94% similar to $RUGPULL69")

## Monetization Model (for judges — not fully implemented)

**Freemium:**
- Free: live feed + basic score + top 3 risk factors
- Paid (x402 pieUSD ~$0.50): full report, deployer history, similar scam patterns, AI narrative

**Long-term vision:** B2B to Four.Meme — embed BARK as native safety layer in Four.Meme platform.

## Differentiation vs GoPlus / TokenSniffer

| Gap                  | GoPlus/TokenSniffer        | BARK                                          |
| -------------------- | -------------------------- | --------------------------------------------- |
| Scope                | Generic, all chains        | Purpose-built for Four.Meme                   |
| Memory               | Stateless, each scan fresh | Membase — learns scam patterns over time       |
| Intelligence         | Rule-based pass/fail       | LLM reasoning — explains WHY in natural lang   |
| Social               | No social analysis         | Twitter/Telegram/website checks                |
| Meme-specific        | No Four.Meme awareness     | Understands bonding curve lifecycle            |
| Live monitoring      | User must input address    | Auto-scans every new Four.Meme launch          |
| Cross-token intel    | Isolated per token         | "Deployer rugged 3 tokens last week"           |

Note: We USE GoPlus API as a data source input — not rebuilding what exists. We add AI reasoning + memory + meme-specific context on top.

## Project Structure

```
bark/
├── CLAUDE.md
├── packages/
│   ├── contracts/          # Foundry project
│   │   ├── foundry.toml
│   │   ├── src/
│   │   │   └── AuditRegistry.sol
│   │   └── test/
│   │       └── AuditRegistry.t.sol
│   ├── api/                # Backend API
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   ├── src/
│   │   │   ├── index.ts           # Express server entry
│   │   │   ├── analyzers/
│   │   │   │   ├── contract.ts    # GoPlus API integration
│   │   │   │   ├── holder.ts      # BSCScan holder analysis
│   │   │   │   ├── deployer.ts    # Deployer history check
│   │   │   │   ├── liquidity.ts   # DexScreener integration
│   │   │   │   └── social.ts      # Social analysis (optional)
│   │   │   ├── scoring/
│   │   │   │   ├── engine.ts      # Orchestrate analyzers + LLM
│   │   │   │   └── prompt.ts      # LLM prompt templates
│   │   │   ├── services/
│   │   │   │   ├── dgrid.ts       # dgrid AI Gateway client
│   │   │   │   ├── membase.ts     # Unibase Membase client
│   │   │   │   ├── scanner.ts     # Four.Meme live feed poller
│   │   │   │   └── x402.ts        # Pieverse x402 payment
│   │   │   └── types/
│   │   │       └── index.ts       # TypeScript interfaces
│   │   └── .env.example
│   └── web/                # Next.js frontend
│       ├── package.json
│       ├── next.config.js
│       ├── tailwind.config.ts
│       ├── app/
│       │   ├── layout.tsx
│       │   ├── page.tsx           # Live feed dashboard
│       │   └── token/
│       │       └── [address]/
│       │           └── page.tsx   # Token detail / report page
│       └── components/
│           ├── LiveFeed.tsx
│           ├── ScoreBadge.tsx
│           ├── RiskReport.tsx
│           ├── TokenCard.tsx
│           └── SearchBar.tsx
├── skills/                  # Purrfect Claw skill
│   └── bark-token-audit/
│       └── SKILL.md
└── README.md
```

## Environment Variables

```env
# dgrid AI Gateway
DGRID_API_KEY=

# BSCScan
BSCSCAN_API_KEY=

# GoPlus (free, no key needed for basic tier)
# GOPLUS_API_KEY=

# Unibase Membase
MEMBASE_ACCOUNT=bark-agent

# Pieverse x402
PIEVERSE_X402_ENDPOINT=

# BSC RPC
BSC_RPC_URL=https://bsc-dataseed.binance.org/

# App
PORT=3001
NEXT_PUBLIC_API_URL=http://localhost:3001
```

## Build & Run

```bash
# Install dependencies
cd packages/api && npm install
cd packages/web && npm install

# Run API server
cd packages/api && npm run dev

# Run frontend
cd packages/web && npm run dev

# Run contracts tests
cd packages/contracts && forge test

# Deploy contracts (BSC testnet)
cd packages/contracts && forge script script/Deploy.s.sol --rpc-url $BSC_TESTNET_RPC --broadcast
```

## Sprint Plan (April 16-30)

### Week 1 (Apr 16-22): Core Engine
- [ ] Setup monorepo structure
- [ ] GoPlus API integration (contract analyzer)
- [ ] BSCScan API integration (holder + deployer history)
- [ ] dgrid AI scoring engine
- [ ] Four.Meme scanner (poll for new tokens)
- [ ] Express API with /audit/:address endpoint
- [ ] AuditRegistry.sol + tests

### Week 2 (Apr 23-28): Frontend + Integrations
- [ ] Next.js dashboard — live feed page
- [ ] Token detail / report page
- [ ] ScoreBadge, RiskReport, TokenCard components
- [ ] Membase integration (store/query patterns)
- [ ] x402 premium report endpoint (Pieverse bounty)
- [ ] Purrfect Claw SKILL.md (publish to ClawHub)
- [ ] Deploy contracts to BSC testnet

### Polish (Apr 29-30): Demo & Submit
- [ ] Record 3-min demo video
- [ ] README with screenshots
- [ ] Submit to DoraHacks
- [ ] Verify bounty eligibility (dgrid + Pieverse)

## Demo Video Script (3 minutes)

```
0:00-0:20  Hook
"Every day, hundreds of meme tokens launch on Four.Meme.
Most are scams. BARK is the guard dog that barks before
you get bitten."

0:20-0:50  Live Feed
Show dashboard with tokens streaming in real-time.
Point at red-scored vs green-scored tokens.
"BARK auto-scans every new launch within seconds."

0:50-1:40  Deep Dive Audit
Click DANGER token. Show full report:
- Contract risks highlighted
- "Deployer has rugged 2 previous tokens"
- "89% supply in 3 wallets"
- AI narrative explanation
"Unlike GoPlus or TokenSniffer, BARK tells you WHY
and shows similar past scams from memory."

1:40-2:10  Memory / Learning
"Powered by Unibase Membase — BARK remembers every
scam pattern and gets smarter over time."
Show pattern match with historical data.

2:10-2:30  x402 Premium
Show free vs paid report.
Trigger x402 payment via Pieverse.
"Gasless micropayment — 0.50 pieUSD for full report."

2:30-2:50  Purrfect Claw + dgrid
Quick flash: audit via messaging app.
"AI inference powered by dgrid AI Gateway."

2:50-3:00  Architecture + Close
Flash architecture diagram.
"BARK — we bark before you get bitten."
```

## Submission Checklist

- [ ] GitHub repo (public) with documentation
- [ ] Demo video (3 min, uploaded to YouTube/Loom)
- [ ] Live deployed app (Vercel for frontend, VPS for API)
- [ ] Contracts deployed on BSC testnet with explorer links
- [ ] BUIDL submitted on DoraHacks
- [ ] Bounty tracks selected: Main + Pieverse + dgrid
- [ ] Purrfect Claw skill published to ClawHub/Pieverse Skill Store

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EndPx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
