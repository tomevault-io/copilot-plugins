## tend

> - **Bags Hackathon** — $1M prize pool, 100 winners ($10K-$100K each)

# Tend — AI-managed revenue and growth control plane for Bags creator tokens

## Hackathon Context
- **Bags Hackathon** — $1M prize pool, 100 winners ($10K-$100K each)
- **Deadline**: 2026-06-01 (rolling review, ship early)
- **Tracks**: Claude Skills + AI Agents
- **Judging**: Onchain performance (volume, active traders, market cap) + App traction (DAU, MRR, GitHub stars)
- **Rule**: "Ideas alone don't qualify. You must deploy a working product with real users and real transactions."
- **Judges/mentors**: Solana, Helius, Meteora, Privy, DFlow, Birdeye

## Positioning
Tend transforms Bags.fm fee-sharing into a programmable growth engine:
- Services capture a % of trading fees on-chain
- An AI agent analyzes token/market state, decides what to do, executes, logs its reasoning
- The dashboard shows inputs → decision → action → impact → tx link
- MCP server lets creators manage everything through Claude Desktop

**Tend is NOT "MCP server with bots". Tend IS an AI-managed growth control plane for Bags tokens.**

## Stack
- Monorepo: npm workspaces (`packages/shared`, `packages/mcp-server`, `packages/agent`, `packages/frontend`)
- MCP: `@modelcontextprotocol/sdk` (STDIO transport)
- Solana: `@solana/web3.js` + `@bagsfm/bags-sdk`
- Frontend: Next.js 15, Tailwind v4
- Agent: Node.js scheduler + `@anthropic-ai/sdk` for decisional AI

## Build
```bash
npm run build           # all packages
npm run build:shared    # shared types + SDK wrapper
npm run build:mcp       # MCP server
npm run dev:dashboard   # Next.js dev
npm run dev:agent       # Agent runtime
```

## Key Architecture
- `packages/shared/src/bags-client.ts` — All Bags SDK interactions, handles tx signing (partialSign for claims)
- `packages/shared/src/db/` — Drizzle schema + Neon Postgres client (opt-in via `DATABASE_URL`; state.json remains default until migration lands)
- `packages/mcp-server/src/services/orchestrator.ts` — Fee-share config management (on-chain first, state second)
- `packages/mcp-server/src/state/` — StateManager (persists to `~/.tend/state.json`, unified wallet pool)
- `packages/mcp-server/src/tools/` — 7 MCP tools (1 prompt, 1 resource) covering the creator workflow
- `packages/shared/src/squads-client.ts` — Squads v4 PDA derivation, ix builders, SpendingLimit state reader (mandatory custody on the payout path)
- `packages/agent/src/payout-executor.ts` — refuses any payout without a Squads ref (no legacy admin-transfer fallback)
- `packages/agent/src/treasury-health.ts` — surplus check across all campaigns; scheduler gates accrual + withdrawals when the shared admin wallet runs low
- `packages/agent/src/` — Buyback agent + fee claimer + scheduler

## Conventions
- All shared types in `@tend/shared`
- Never `console.log` in MCP server (STDIO) — use `console.error` for debug
- BPS = basis points (100 = 1%, 10000 = 100%)
- All amounts in lamports internally, format with `formatSol()` for display

---

## DECISION FRAMEWORK — Every feature must pass this test

Before building or recommending ANY feature, ask:

1. **Does it improve a hackathon KPI?** (volume, active traders, DAU, MRR, GitHub stars)
2. **Is it visible in under 60 seconds in a demo?**
3. **Does it increase Tend's credibility as a premium product?**
4. **Does it produce real on-chain signals, or just complexity?**

If the answer to all 4 is "no", **don't build it**.

Prefer **2 real agents** over 6 fictional ones.
Prefer **real metrics** over decorated dashboards.
Prefer **working end-to-end flows** over impressive architecture diagrams.

---

## STRICT RULES — DO NOT VIOLATE

### Rule 1: No feature claims without working code
- A service in `service-registry.ts` MUST NOT have `status: "available"` unless there is real, tested execution logic in `packages/agent/src/` or equivalent.
- If the code only does generic `claimFees()`, the service is `"coming-soon"`, not `"available"`.
- "Available" means: user can add it → agent autonomously executes the specific strategy → results are observable in dashboard AND on-chain.

### Rule 2: No unused dependencies
- Every dependency in every `package.json` MUST be imported somewhere in the source code of that package.
- Aspirational dependencies (planned but not yet used) → REMOVE from package.json. Add when the import exists.

### Rule 3: No hardcoded status in UI
- The frontend MUST NOT hardcode service names, statuses ("ACTIVE", "LIVE"), or metrics.
- Data must come from API/state. If unavailable, show loading/empty state — not a fake badge.

### Rule 4: One write path per entity
- ONE canonical way to add/remove a service. Other surfaces call into it, not reimplement it.
- Known violation to fix: orchestrator (MCP) vs `/api/services/add` vs `/api/services/prepare+submit` all write state differently.

### Rule 5: State consistency
- `walletPool` mutations MUST be persisted to `state.json` immediately via `save()`. No separate wallets.json.
- Services added via any surface (MCP, API, dashboard) must be readable from any other surface.
- If a multi-tx sequence partially fails, DO NOT persist local state.

### Rule 6: Metrics must be real
- `totalFeesClaimed`, `totalFeesEarned`, `actionsPerformed` MUST be updated by the code that actually performs claims/actions.
- If a field is not updated by runtime code, remove it from UI. No decorative metrics.

### Rule 7: README and docs must match code
- Version numbers, tool counts, feature lists in README must match the actual codebase.
- Update README in the same commit as any feature addition or removal.

### Rule 8: Deployment model
- Frontend on Vercel = read-only API routes (Bags SDK on-chain reads only).
- All writes (add/remove service) require either wallet-sign flow or MCP server running locally.
- `~/.tend/state.json` is local-only state. Do NOT pretend it works on serverless.

### Rule 9: Before saying "done" or "ready"
Verify:
1. Happy path works end-to-end (not just one function)
2. State persists across process restarts
3. UI reflects real state, not hardcoded values
4. Feature is reachable from every surface that claims to support it

### Rule 10: Commits must not introduce drift
- Never commit a registry entry without the corresponding implementation.
- Never commit a UI component that displays fake data as if real.
- Never commit a dependency without code that imports it.

### Rule 11: AI must be real
- Every "AI" label must correspond to a real LLM API call in the runtime.
- The agent's decisions must be logged with: inputs, reasoning, action taken, outcome.
- No "AI theater" — if it's a deterministic script, call it a bot, not an agent.

### Rule 12: Agent decisions must be bounded
- Every AI decision must have a finite, explicit action space (e.g., buy/hold/partial-buy).
- Every financial action must have guardrails (max amount, min threshold, cooldown).
- The agent MUST NOT have unbounded authority over funds.

---

## ROADMAP (ordered by priority)

### Phase 1: Integrity + Reliability (CURRENT)
- [ ] Fix service registry: only `buyback-bot` is `"available"`, rest → `"coming-soon"`
- [ ] Remove unused deps (`@anthropic-ai/sdk`, `ai`) from agent package.json
- [ ] Fix `wallets.json` persistence bug (assignWallet must persist immediately)
- [ ] Unify write paths: one canonical add/remove service flow
- [ ] Fix `serviceWallets` vs `walletPool` bifurcation
- [ ] Update metrics (totalFeesClaimed, actionsPerformed) from actual runtime
- [ ] Remove hardcoded "ACTIVE" badges from landing page
- [ ] Update README to match reality

### Phase 2: Decisional Buyback Agent
- [ ] Add `@anthropic-ai/sdk` back WITH real code that imports it
- [ ] Buyback pipeline: claim → collect market snapshot (price, volume, holders, momentum) → Claude API call → bounded decision (buy now / hold / partial buy + amount) → execute → log (inputs, reasoning, tx, outcome)
- [ ] Decision log stored in state, exposed via MCP tool `agent_decision_log`
- [ ] Guardrails: max buy amount, min fee threshold, cooldown between buys, slippage limit
- [ ] Dashboard: show decision history (inputs → decision → action → tx link → impact)

### Phase 3: Observability Dashboard
- [ ] Real-time service status from on-chain data (not local state)
- [ ] Decision feed: each agent action with rationale visible
- [ ] Before/after metrics: volume change, price impact per buyback
- [ ] Token health score (computed, not decorative)

### Phase 4: Deploy + Traction
- [ ] Deploy frontend to Vercel (read-only API routes)
- [ ] Run buyback agent on $TEND for 1+ week → real on-chain transactions
- [ ] Onboard 3-5 real users (wallet connect + explore tokens)
- [ ] Demo video 3-5 min showing: dashboard live → Claude Desktop MCP → agent decisions → Solscan proof
- [ ] Submit to Bags hackathon

### NOT building (explicit)
- Fee Compounder (no LP integration code — would be fake)
- Growth Agent (no concrete action space — would be AI theater)
- Chat AI in frontend (feature on a chair — no organic integration)
- Comprehensive test suite (2-3 critical tests only: state restart, claim flow, wallet assignment)

---

## Postgres (Neon) migration runbook

State lives in `~/.tend/state.json` by default. To switch to Neon Postgres:

1. Create a Neon project (free tier is fine) and grab the pooled `DATABASE_URL`.
2. Put it in `.env.local` at the repo root — never commit.
3. Apply the schema: `npm run db:push -w packages/shared` (one-shot CREATE TABLE).
4. Dry-run the migration to sanity-check the row mapping:
   `npm run build:agent && node --env-file=.env.local packages/agent/build/migrate-state-to-db.js --dry-run`.
5. Apply it: `node --env-file=.env.local packages/agent/build/migrate-state-to-db.js [--force]`.
6. Verify parity before touching the flag:
   `node --env-file=.env.local packages/agent/build/verify-db-state.js` (exit 0 = safe to flip).
7. Flip the flag on the agent host (Render): `TEND_STATE_BACKEND=db`.
8. Optional — flip the same flag on the frontend (Vercel) so API routes read directly from Postgres instead of the `/state` proxy.

Rollback: unset `TEND_STATE_BACKEND` (defaults to `file`). Both backends expose the same `withStateLock` / `loadState` API — call sites don't change.

Schema lives in `packages/shared/src/db/schema.ts`; migrations generated by `npm run db:generate -w packages/shared` land in `packages/shared/drizzle/`. Concurrent write contention retries up to 3× on SQLSTATE 40001 before surfacing.

## MCP Server Config
Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:
```json
{
  "mcpServers": {
    "tend": {
      "command": "node",
      "args": ["<path>/packages/mcp-server/build/index.js"],
      "env": {
        "BAGS_API_KEY": "...",
        "SOLANA_RPC_URL": "...",
        "TEND_PRIVATE_KEY": "..."
      }
    }
  }
}
```

---
> Source: [RedGnad/Tend](https://github.com/RedGnad/Tend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
