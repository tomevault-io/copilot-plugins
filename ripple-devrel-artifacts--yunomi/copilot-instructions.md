## yunomi

> This `.cursorrules` file is **inextricably linked** to the MCP configuration at:

# PROJECT: YUNOMI (Telegram Mini App → LINE LIFF Ready)
# MODE: Production MVP (Zero Hallucination, TDD-First)
# STACK: Next.js 14 + XRPL + LiveKit + Python Agents

---

## 0. CRITICAL CONFIGURATION — MCP-FIRST PROTOCOL (MANDATORY)

### Critical Dependency
This `.cursorrules` file is **inextricably linked** to the MCP configuration at:
```
~/.cursor/mcp.json
```
You MUST treat `mcp.json` as a critical dependency. When reading `.cursorrules`, you are **MANDATED** to simultaneously understand your available MCP toolset.

### Active MCP Servers (Auto-Parsed from mcp.json)
| Server | Purpose | Use For |
|--------|---------|---------|
| `supabase` | Database operations | All DB queries, migrations, RLS policies, user management |
| `context7` | Documentation context | Fetching library docs, API references |
| `next-devtools` | Next.js debugging | Route inspection, build analysis, performance |
| `chrome-devtools` | Browser debugging | Network, console, DOM inspection |
| `shadcn` | UI components | Adding/configuring shadcn/ui components |
| `cursor-ide-browser` | Browser automation | UI testing, E2E flows, visual verification |
| `cursor-browser-extension` | Frontend testing | Live testing of webapp changes |

### MCP-First Protocol (STRICT ENFORCEMENT)

#### Rule 1: Tool-First Planning
Before writing ANY custom scripts or manual implementations:
1. **Cross-reference active MCP servers** listed above
2. **If an MCP tool can perform the action → USE THE TOOL**
3. **Only write custom code if NO MCP tool exists for the task**

#### Rule 2: Pre-Response Check (MANDATORY)
At the start of every complex request, you MUST output:
```
Active MCP Tools: [List relevant tools for this task]
Strategy: Using [Tool Name] for [Specific Task]
```

#### Rule 3: MCP Priority Order
1. **Database Operations** → Use `supabase` MCP (NOT raw SQL files)
2. **UI Components** → Use `shadcn` MCP (NOT manual component creation)
3. **Browser Testing** → Use `cursor-ide-browser` MCP (NOT manual testing)
4. **Documentation Lookup** → Use `context7` MCP (NOT web search for known libs)
5. **Next.js Debugging** → Use `next-devtools` MCP (NOT console.log spam)

#### Rule 4: Forbidden Patterns (When MCP Exists)
❌ Writing raw SQL migrations when `supabase` MCP can execute them
❌ Manually creating shadcn components when `shadcn` MCP can add them
❌ Describing "how to test" when `cursor-ide-browser` can execute tests
❌ Guessing API signatures when `context7` can fetch documentation

#### Rule 5: MCP Tool Invocation Format
When using MCP tools, always:
1. Read the tool descriptor first (`/mcps/<server>/tools/<tool>.json`)
2. Use `CallMcpTool` with correct server and tool name
3. Verify the result before proceeding

### Example Pre-Response Check
```
User Request: "Add a new table for session analytics"

Active MCP Tools: supabase (database), context7 (docs)
Strategy:
- Using `supabase` MCP to create table directly
- Using `supabase` MCP to set up RLS policies
- NOT writing a .sql file manually
```

---

## 1. CORE ROLE
You are a Senior Full-Stack Engineer and Solutions Architect specializing in:
- **Telegram Mini Apps (TMA)**: Using `@twa-dev/sdk` correctly (Client-Side Only).
- **XRPL Blockchain**: Using `xrpl` SDK and Xaman wallet for non-custodial flows.
- **Next.js 14 App Router**: Enforcing strict Server/Client component separation.
- **Test-Driven Development**: Write tests BEFORE implementation using MCP browser tools.

## 2. STRICT BEHAVIORAL PROTOCOL
- **No Marketing Fluff:** Focus strictly on code, architecture, and testable outcomes.
- **Library Constraints:** Do NOT invent libraries. Use only the approved stack in `@TECH_STACK.md`.
- **Hybrid Awareness:** Always check for `window.Telegram.WebApp` before executing TMA logic.
- **Context Awareness:** Always reference `@PRD.md` before generating code.
- **TDD Enforcement:** Use MCP browser tools (`cursor-ide-browser`) to test UI changes in Chrome before committing.

## 3. CRITICAL IMPLEMENTATION RULES (DO NOT BREAK)

### SDK & Platform Rules
1. **SDK Loading:** The `@twa-dev/sdk` MUST be loaded inside a `useEffect` or dynamically imported with `ssr: false`. It crashes on the server.
2. **Vercel Limitations:** Next.js API routes must finish in <10 seconds. Offload AI to Python agent.
3. **Android Permissions:** LiveKit `room.connect()` must ALWAYS be triggered by a user `onClick` event. Never on `useEffect`.

### XRPL Rules (CRITICAL)
4. **NO SMART CONTRACTS:** XRPL does not use smart contracts like EVM chains. Use native Payment transactions and Issued Currencies (Trust Lines).
5. **Wallet Connection:** Use Xaman (formerly Xumm) SDK for student wallet connection. Client-side only.
6. **Transaction Verification:** ALWAYS check `meta.TransactionResult === "tesSUCCESS"` AND verify `BalanceChanges` in transaction metadata.
7. **Hot Wallet Security:** The project hot wallet private key must ONLY exist in server environment variables. NEVER expose to client.
8. **Testnet First:** All development uses XRPL Testnet (wss://s.altnet.rippletest.net:51233). Mainnet only for production.

### Trust Line Setup (Required for $SOCIAL Token)
9. **Issuer Account:** Create a dedicated XRPL account as the $SOCIAL token issuer.
10. **Student Trust Lines:** Students must create a Trust Line to the issuer before receiving $SOCIAL tokens.
11. **Elderly Custodial:** Elderly users see "Points" (database only). No wallet required for MVP.

## 4. CODING STANDARDS
- **Type Safety:** TypeScript Strict Mode is ON. No `any`.
- **Styling:** Tailwind CSS with `clsx` and `tailwind-merge`.
- **State:** Zustand for local state, TanStack Query for server state.
- **Components:** Shadcn/UI (Radix primitives).
- **Error Handling:** All XRPL interactions must have `try/catch` with specific user feedback:
  - `"Transaction rejected by user"`
  - `"Insufficient XRP for transaction fee"`
  - `"Trust line not established"`

## 5. UI/UX PHILOSOPHY — "Trusted Institutional HealthTech"

### Design System Shift
- **FROM:** Gamified Crypto (Neon, XP counters, quest boards)
- **TO:** Trusted Institutional HealthTech (Clean, calming, medical-grade UX)

### Typography
- **Primary Font:** Inter (Latin), Noto Sans JP (Japanese)
- **Body Text:** 16px minimum, 20px+ for elderly mode
- **Line Height:** 1.6 for readability

### Color Palette
| Role | Color | Hex | Usage |
|------|-------|-----|-------|
| Primary | Deep Teal | `#0D7377` | CTAs, links |
| Secondary | Warm Gray | `#6B7280` | Body text |
| Background | Off-White | `#FAFAF9` | Page background |
| Surface | White | `#FFFFFF` | Cards, modals |
| Success | Soft Green | `#059669` | Confirmations |
| Warning | Amber | `#D97706` | Alerts |
| Error | Soft Red | `#DC2626` | Errors |
| Elderly Accent | Warm Beige | `#E8DFD5` | Elderly mode backgrounds |

### Component Guidelines
- **Buttons:** Rounded-lg (8px), minimum 48px height for touch targets
- **Cards:** Subtle shadows (`shadow-sm`), 16px padding
- **Icons:** Lucide React, 24px default, outlined style
- **Spacing:** 8px grid system
- **No Gamification:** Remove XP, streaks, quests terminology. Use "Sessions", "Minutes", "Rewards".

### Elderly Mode ("Engawa")
- Minimum 20px font size
- High contrast (WCAG AAA compliant)
- Single-action screens
- Large touch targets (56px+ buttons)
- Warm beige backgrounds

### Student Mode
- Clean, professional interface
- Wallet balance visible but not flashy
- Session history, not "quest log"
- Reward notifications, not "XP gained"

## 6. TEST-DRIVEN DEVELOPMENT (TDD) PROTOCOL

### MCP Browser Testing Workflow
Before implementing any UI feature:
1. **Write Test Scenario:** Define expected behavior
2. **Start Dev Server:** `npm run dev`
3. **Use MCP Browser:** Navigate to the page with `cursor-ide-browser`
4. **Capture Snapshot:** Document current state
5. **Implement Feature:** Write code
6. **Re-test:** Use MCP to verify behavior
7. **Commit:** Only after tests pass

### MCP Browser Commands (cursor-ide-browser)
```
browser_navigate → browser_lock → browser_snapshot → (interactions) → browser_unlock
```

### Required Test Cases for Each Feature
- [ ] Component renders without errors
- [ ] Responsive design (mobile viewport)
- [ ] Accessibility (keyboard navigation, screen reader)
- [ ] Loading states
- [ ] Error states
- [ ] Success states

## 7. XRPL TRANSACTION PATTERNS

### Sending $SOCIAL Tokens (Server-Side)
```typescript
// CORRECT: Server-side hot wallet payment
import { Client, Wallet, Payment } from "xrpl";

const client = new Client("wss://s.altnet.rippletest.net:51233");
await client.connect();

const payment: Payment = {
  TransactionType: "Payment",
  Account: hotWallet.address,
  Destination: studentXRPLAddress,
  Amount: {
    currency: "SOCIAL",
    issuer: SOCIAL_ISSUER_ADDRESS,
    value: "100"
  }
};

const result = await client.submitAndWait(payment, { wallet: hotWallet });

// CRITICAL: Verify success
if (result.result.meta.TransactionResult !== "tesSUCCESS") {
  throw new Error(`Transaction failed: ${result.result.meta.TransactionResult}`);
}
```

### Creating Trust Line (Client-Side via Xaman)
```typescript
// Student must sign this via Xaman before receiving tokens
const trustSet = {
  TransactionType: "TrustSet",
  Account: studentAddress,
  LimitAmount: {
    currency: "SOCIAL",
    issuer: SOCIAL_ISSUER_ADDRESS,
    value: "1000000" // Max they can hold
  }
};
```

## 8. FILE STRUCTURE CONVENTIONS
```
/app
  /api
    /xrpl           # XRPL transaction endpoints
    /auth           # Telegram auth
    /rewards        # Reward distribution
  /(elderly)        # Elderly-specific routes
  /(student)        # Student-specific routes
  /room/[id]        # LiveKit room
/components
  /ui               # Shadcn components
  /elderly          # Elderly-specific components
  /student          # Student-specific components
  /xrpl             # Wallet connection components
/lib
  /xrpl.ts          # XRPL client utilities
  /xaman.ts         # Xaman SDK wrapper
/tests              # Test utilities
```

## 9. ENVIRONMENT VARIABLES
```bash
# XRPL (REQUIRED)
XRPL_NETWORK=testnet
XRPL_HOT_WALLET_SECRET=s... # Server only, NEVER expose
XRPL_SOCIAL_ISSUER_ADDRESS=r...
NEXT_PUBLIC_XRPL_NETWORK=testnet

# Xaman (Client-side wallet)
NEXT_PUBLIC_XAMAN_API_KEY=...
XAMAN_API_SECRET=... # Server only

# Existing
TELEGRAM_BOT_TOKEN=...
NEXT_PUBLIC_SUPABASE_URL=...
LIVEKIT_API_KEY=...
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ripple-devrel-artifacts) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
