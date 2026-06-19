## okx-yield-pilot

> Autonomous multi-chain yield farming autopilot — auto-compound, rebalance, and risk-guard DeFi positions across Solana and X Layer using onchainOS. Do NOT use for: manual one-off swaps (okx-dex-swap), DeFi product discovery or deposit (okx-defi-invest), DeFi position viewing only (okx-defi-portfolio), wallet balance checks (okx-wallet-portfolio), token prices (okx-dex-market). Triggers (EN): 'auto-compound my yields', 'compound my DeFi rewards', 'rebalance my yield farms', 'optimize my DeFi portfolio', 'auto-reinvest farming rewards', 'manage my LP positions', 'yield autopilot', 'set up auto-compounding', 'harvest and reinvest', 'compound rewards on Solana', 'compound rewards on X Layer', 'rebalance DeFi allocations', 'check yield farm health', 'monitor impermanent loss', 'show farming performance', 'stop auto-compound', 'pause yield strategy', 'resume yield farming', 'yield farm status', 'farming APY report', 'how much have I earned from farming', 'gas-efficient compounding', 'batch compound my positions', 'auto-harvest DeFi', 'optimize LP gas costs', 'schedule compound cycle', 'detect APY drop', 'farming circuit breaker', 'IL alert', 'impermanent loss check', 'exit underperforming farms', 'rotate to higher APY', 'yield comparison across chains'. Triggers (ZH): '自动复投收益', '复投我的DeFi奖励', '重新平衡我的农场', '优化DeFi组合', '查看农场健康', '检查无常损失', '收益报告', '暂停自动复投', '启动收益自动化'.


# OKX Yield Pilot

Autonomous yield farming autopilot — auto-compound, portfolio rebalance, and risk-guard DeFi positions across Solana and X Layer via onchainOS CLI.

For CLI parameter details, see [references/cli-reference.md](references/cli-reference.md).

## Step 0 — Skill Routing (run before every other step)

Before running any workflow in this skill, classify the user's intent:

- **One-off DeFi deposit/withdraw** (no automation) → use `okx-defi-invest`
- **View DeFi positions only** → use `okx-defi-portfolio`
- **One-off token swap** → use `okx-dex-swap`
- **Wallet balance check** → use `okx-wallet-portfolio` or `okx-agentic-wallet`
- **Token price / chart** → use `okx-dex-market`
- **Named DApp interaction** (Aave, Raydium, Orca, etc.) → use `okx-dapp-discovery`

Stay in this skill when the user wants **ongoing automated management**: auto-compounding, scheduled rebalancing, portfolio optimization, IL monitoring, or performance reporting across yield farm positions.

## Skill Routing Table

| User intent                                          | Target skill           |
| ---------------------------------------------------- | ---------------------- |
| Auto-compound / harvest & reinvest / yield autopilot | **This skill**         |
| Rebalance DeFi allocations automatically             | **This skill**         |
| Monitor IL / APY anomalies / farming health          | **This skill**         |
| View farming performance report                      | **This skill**         |
| One-off DeFi deposit/withdraw (no automation)        | `okx-defi-invest`      |
| View current DeFi positions (no action)              | `okx-defi-portfolio`   |
| Swap tokens                                          | `okx-dex-swap`         |
| Check wallet balance                                 | `okx-wallet-portfolio` |
| Token price or chart                                 | `okx-dex-market`       |
| Named DApp (Aave, Orca, Raydium…)                    | `okx-dapp-discovery`   |

## Pre-flight Checks

> Read the preflight checks before any workflow. Look for the file in this order:
>
> 1. `_shared/preflight.md` (relative to this SKILL.md file)
> 2. `../_shared/preflight.md` (if skill is nested inside a parent skills folder)
> 3. `../okx-agentic-wallet/_shared/preflight.md` (shared OKX preflight)
>
> If none exist, run these checks inline: (a) `onchainos wallet status` — verify logged in, (b) `onchainos wallet addresses` — resolve EVM + Solana addresses, (c) `onchainos defi positions` — verify at least one active position exists.

## Decision Tree

When the user's request reaches this skill, follow this decision tree to select the correct workflow:

```
User request received
│
├─ Mentions "scan" / "show" / "list" / "what positions" / "what rewards"?
│  → Flow 1: Scan Positions
│
├─ Mentions "compound" / "harvest" / "reinvest" / "claim and reinvest" / "复投"?
│  ├─ Specifies a single chain or pool?
│  │  → Flow 2: Auto-Compound (single position)
│  └─ Says "all" or no filter?
│     → Flow 2: Auto-Compound (batch — all eligible positions)
│
├─ Mentions "rebalance" / "reallocate" / "drift" / "adjust weights" / "平衡"?
│  → Flow 3: Rebalance
│
├─ Mentions "health" / "check" / "IL" / "impermanent loss" / "risk" / "APY drop" / "safe" / "健康"?
│  → Flow 4: Health Check
│
├─ Mentions "report" / "performance" / "how much earned" / "effective APY" / "summary" / "报告"?
│  → Flow 5: Performance Report
│
├─ Mentions "schedule" / "autopilot" / "automate" / "daily" / "weekly" / "自动化"?
│  → Flow 6: Schedule Automation
│
└─ Unclear?
   → Run Flow 1 (Scan) first, then suggest next action based on results
```

## Reasoning Chain

Before executing any workflow, the agent MUST reason through these questions internally (do NOT show to user):

1. **Which chains does the user care about?** → If unspecified, scan ALL supported chains
2. **Are the positions worth acting on?** → Check reward value vs gas cost BEFORE executing
3. **Is there a safety concern?** → Run health gates mentally before suggesting actions
4. **What's the optimal execution order?** → Group operations by chain to minimize wallet auth overhead
5. **Can I save the user gas?** → Always prefer X Layer (zero gas) for intermediate operations

## onchainos Commands Used

> Full parameter tables, return schemas, and usage examples: [`references/cli-reference.md`](references/cli-reference.md)

Commands used across flows: `defi positions`, `defi position-detail`, `defi detail`, `defi collect`, `defi invest`, `defi withdraw`, `defi rate-chart`, `defi tvl-chart`, `swap quote`, `swap execute`, `wallet status`, `wallet addresses`, `wallet contract-call`, `portfolio all-balances`, `token search`, `token price-info`

## Chain Support

> See [`references/chains.md`](references/chains.md) for chain index, gas profiles, native token addresses, and well-known stablecoin addresses.

**Quick reference**:
| Chain | Name | chainIndex | Gas |
|---|---|---|---|
| X Layer | `xlayer` | `196` | **Zero** |
| Solana | `solana` | `501` | ~$0.001 |
| Ethereum | `ethereum` | `1` | Variable |
| BSC | `bsc` | `56` | ~$0.05 |
| Base | `base` | `8453` | ~$0.01 |

> **X Layer priority**: Always recommend X Layer for rebalancing due to zero gas. When compounding on Ethereum, batch operations or suggest bridging rewards to X Layer first.

## Operation Flows

### Flow 1: Scan Positions

Discover all active yield farming positions and unclaimed rewards across chains.

```
1. Resolve wallet address:
   onchainos wallet status          → get active account
   onchainos wallet addresses       → get EVM + Solana addresses

2. Fetch DeFi positions per chain:
   onchainos defi positions --address <evm_addr> --chains "xlayer,ethereum,bsc,base"
   onchainos defi positions --address <sol_addr> --chains "solana"

3. For each position with investmentId:
   onchainos defi position-detail --address <addr> --chain <chain> --platform-id <pid>
   → Extract: investmentId, coinAmount, tokenPrecision, rewardTokens, pendingRewards

4. Enrich with current APY:
   onchainos defi detail --investment-id <id>
   → Extract: rate (APY), tvl, underlyingToken info

5. Display summary table (see Display Rules)
```

**Output format**:

| #   | Chain   | Platform  | Pool     | Position Value | Unclaimed Rewards | Current APY | TVL   |
| --- | ------- | --------- | -------- | -------------- | ----------------- | ----------- | ----- |
| 1   | X Layer | UniswapV3 | OKB-USDT | $2,450         | $12.30 (UNI)      | 18.5%       | $5.2M |
| 2   | Solana  | Raydium   | SOL-USDC | $1,800         | $8.70 (RAY)       | 22.1%       | $45M  |

### Flow 2: Auto-Compound

Claim rewards from positions and reinvest them back into the same pool.

> **CRITICAL — position-detail is MANDATORY before every collect.** Position data changes after each on-chain operation. Do NOT reuse stale data.

```
Phase A — Reward Collection:
1. onchainos defi position-detail --address <addr> --chain <chain> --platform-id <pid>
   → MUST be called fresh — do NOT reuse prior results
   → Extract: investmentId, platformId, tokenId (if V3), rewardTokens

2. Check if rewards exceed minimum threshold:
   reward_usd = pendingReward × rewardTokenPrice
   IF reward_usd < gas_cost_estimate × 2:
     → SKIP: "Rewards ($X.XX) too small vs gas cost ($Y.YY). Skipping."
   ELSE:
     → PROCEED to collect

3. onchainos defi collect \
     --address <addr> --chain <chain> \
     --reward-type REWARD_INVESTMENT \
     --investment-id <id> --platform-id <pid>
   → Returns calldata (dataList)

4. Execute calldata via Agentic Wallet:
   EVM:
     onchainos wallet contract-call \
       --to <dataList[N].to> --chain <chainIndex> \
       --input-data <dataList[N].serializedData> \
       --value <value_in_UI_units> --biz-type defi
   Solana:
     onchainos wallet contract-call \
       --to <dataList[N].to> --chain 501 \
       --unsigned-tx <dataList[N].serializedData> \
       --biz-type defi

Phase B — Reward Swap:
5. Identify reward token address:
   onchainos token search --query <reward_symbol> --chains <chain>
   → Get tokenContractAddress

6. Get quote for swap to base token (e.g., USDC):
   onchainos swap quote \
     --from <reward_token_addr> --to <base_token_addr> \
     --readable-amount <claimed_amount> --chain <chain>
   → Apply adaptive slippage tiers by priceImpact (3%/5%/10%), check isHoneyPot
   → If priceImpact > 5%, require explicit user confirmation before swap

7. Execute swap:
   onchainos swap execute \
     --from <reward_token_addr> --to <base_token_addr> \
     --readable-amount <claimed_amount> --chain <chain> \
     --wallet <addr>

Phase C — Reinvest:
8. Get target pool details:
   onchainos defi detail --investment-id <id>
   → Get underlyingToken[].tokenAddress, tokenPrecision

9. Convert swapped amount to minimal units:
   minimal_units = floor(amount × 10^tokenPrecision)

10. Check wallet balance before reinvesting:
    onchainos portfolio all-balances --address <addr> --chains <chain>
    → Verify sufficient balance

11. Reinvest:
    onchainos defi invest \
      --investment-id <id> --address <addr> \
      --token <underlying_token_symbol_or_addr> \
      --amount <minimal_units> --chain <chain>
    → Returns calldata → execute via wallet contract-call

12. Log compound event:
    Record: timestamp, rewards_claimed, gas_cost, reinvested_amount, effective_apy
```

**Gas optimization rules**:

- **X Layer**: Always compound — zero gas cost
- **Solana**: Compound if rewards > $2 (gas ~$0.001)
- **Ethereum**: Compound only if rewards > $50 (gas ~$5-25)
- **BSC/Base**: Compound if rewards > $5 (gas ~$0.05-0.10)
- **Batch rule**: If multiple positions on the same chain, process all in one session to amortize wallet login overhead

### Flow 3: Rebalance

Adjust position sizes to match target allocation weights.

```
1. Scan all positions (Flow 1) → get current USD values

2. Calculate current allocation:
   total_usd = sum(all position values)
   current_alloc[i] = position_value[i] / total_usd × 100

3. Compare against target allocation (from user config):
   drift[i] = current_alloc[i] - target_alloc[i]

4. Identify rebalancing moves:
   overweight = positions where drift > threshold (default: 10%)
   underweight = positions where drift < -threshold

5. For each overweight position:
   a. Calculate exit amount:
      exit_usd = drift[i] / 100 × total_usd
      exit_ratio = exit_usd / position_value[i]
   b. onchainos defi position-detail (FRESH — mandatory)
   c. onchainos defi withdraw \
        --investment-id <id> --address <addr> --chain <chain> \
        --ratio <exit_ratio> --platform-id <pid>
   d. Execute withdrawal calldata via wallet contract-call

6. Swap withdrawn tokens to target pool's base tokens:
   onchainos swap execute --from <withdrawn_token> --to <target_base_token> \
     --readable-amount <amount> --chain <chain> --wallet <addr>

7. For each underweight position:
   a. Calculate deposit amount from freed capital
   b. onchainos defi invest \
        --investment-id <id> --address <addr> \
        --token <token> --amount <minimal_units> --chain <chain>
   c. Execute deposit calldata via wallet contract-call

8. Verify new allocations match targets (re-run Flow 1)
```

**Rebalance rules**:

- Default drift threshold: 10% (configurable)
- Prefer X Layer for intermediate swaps (zero gas)
- Never rebalance if total gas cost > 1% of portfolio value
- Require user confirmation before executing any withdrawals

### Flow 4: Health Check

Monitor position health: APY anomalies, impermanent loss, TVL changes.

```
1. Scan all positions (Flow 1) → get current metrics

2. For each position, check:

   a. APY anomaly detection:
      onchainos defi rate-chart --investment-id <id> --time-range MONTH
      → Extract APY history
      → avg_apy = mean(rate values)
      → IF current_apy < avg_apy × 0.6:
           ALERT: "⚠️ {pool}: APY dropped {current}% vs {avg}% monthly average (-{drop}%)"

   b. TVL drift detection:
      onchainos defi tvl-chart --investment-id <id> --time-range WEEK
      → IF tvl_change_7d < -30%:
           ALERT: "⚠️ {pool}: TVL dropped {pct}% in 7 days — liquidity risk"

   c. Impermanent loss estimate (for LP positions):
      → Fetch current token prices via onchainos token price-info
      → Calculate IL using standard formula:
        price_ratio = current_price / entry_price
        il_pct = (2 × sqrt(price_ratio) / (1 + price_ratio) - 1) × 100
      → IF |il_pct| > max_il_threshold:
           ALERT: "⚠️ {pool}: Estimated IL {il_pct}% exceeds {threshold}% limit"

   d. High APY risk warning:
      → IF current_apy > 50%:
           WARN: "⚠️ {pool}: APY above 50% indicates elevated risk"

3. Display health report:
   Overall status: HEALTHY / WARNING / CRITICAL
   Per-position breakdown with alerts
```

**Health status levels**:

| Status      | Condition                                             | Action                              |
| ----------- | ----------------------------------------------------- | ----------------------------------- |
| 🟢 HEALTHY  | All metrics normal                                    | Continue autopilot                  |
| 🟡 WARNING  | APY drop > 40% OR IL > threshold OR TVL drop > 30%    | Alert user, suggest review          |
| 🔴 CRITICAL | APY drop > 70% OR IL > 2× threshold OR TVL drop > 60% | Pause auto-compound, recommend exit |

### Flow 5: Performance Report

Generate comprehensive yield farming performance metrics.

```
1. Scan all positions (Flow 1)

2. For each position:
   a. Fetch historical APY:
      onchainos defi rate-chart --investment-id <id> --time-range MONTH
   b. Calculate effective APY:
      effective_apy = (total_rewards_claimed - total_gas_spent) / position_value × 365 / days_active × 100
   c. Calculate gas efficiency:
      gas_efficiency = total_gas / total_rewards × 100

3. Generate report:
   - Total portfolio value
   - Total rewards earned (lifetime)
   - Total gas spent
   - Net profit (rewards - gas)
   - Average effective APY
   - Best performing position
   - Worst performing position
   - Compound count and frequency
   - Recommendation: continue / adjust / exit
```

**Report output format**:

```
═══════════════════════════════════════════
  YIELD PILOT — Performance Report
  Period: {start_date} → {end_date}
═══════════════════════════════════════════

  Portfolio Value:      ${total_value}
  Total Rewards:        ${total_rewards}
  Total Gas Spent:      ${total_gas}
  Net Profit:           ${net_profit}
  Effective APY:        {eff_apy}%
  Gas Efficiency:       {gas_eff}% of rewards

  ┌────────────────────────────────────┐
  │ Position Breakdown                 │
  ├─────┬──────────┬────────┬──────────┤
  │ Pool│ Value    │ APY    │ Rewards  │
  ├─────┼──────────┼────────┼──────────┤
  │ ... │ ...      │ ...    │ ...      │
  └─────┴──────────┴────────┴──────────┘

  Recommendation: {recommendation}
═══════════════════════════════════════════
```

### Flow 6: Schedule Automation

Configure compounding frequency, rebalance triggers, and risk thresholds.

```
1. Collect user preferences:
   - Compound frequency: "daily" / "weekly" / "on-demand"
   - Rebalance threshold: drift % to trigger (default: 10%)
   - Min reward threshold: minimum USD to compound (default: auto by chain)
   - Max IL threshold: % before alerting (default: 5%)
   - APY floor: minimum APY before flagging (default: 5%)
   - Emergency exit: auto-exit if APY drops below X% (default: disabled)

2. Store configuration persistently:
   - Claude Code / Cowork: write to `~/.okx-yield-pilot/config.json`
   - claude.ai artifacts: use persistent storage API
   - Otherwise: agent memory does not survive session resets — remind user to re-state preferences each session

3. When user says "run autopilot" or "start yield pilot":
   a. Run health check (Flow 4)
   b. If HEALTHY or WARNING: run compound (Flow 2) for all eligible positions
   c. Check if rebalance needed: run rebalance (Flow 3) if any drift > threshold
   d. Generate report (Flow 5)
   e. Display summary and next scheduled run
```

## Address Resolution

When the user does NOT provide a wallet address, resolve automatically:

```
1. onchainos wallet status          → check login, get active account
2. onchainos wallet addresses       → get addresses by chain category:
   - XLayer / EVM addresses
   - Solana addresses
3. Match address to target chain:
   - EVM chains → use EVM address
   - Solana     → use Solana address
```

Rules:

- If user provides explicit address, use directly — skip resolution
- If wallet not logged in → ask user to log in first (→ `okx-agentic-wallet`)
- **CRITICAL**: EVM addresses (`0x…`) can only query EVM chains; Solana addresses (base58) can only query `solana`. Never mix.

## Transaction Execution

After `defi invest`/`withdraw`/`collect` returns `dataList`, execute via Agentic Wallet:

**EVM chains (Ethereum, BSC, Base, X Layer)**:

```bash
onchainos wallet contract-call \
  --to <dataList[N].to> \
  --chain <chainIndex> \
  --input-data <dataList[N].serializedData> \
  --value <value_in_UI_units> \
  --biz-type defi
```

**Solana**:

```bash
onchainos wallet contract-call \
  --to <dataList[N].to> \
  --chain 501 \
  --unsigned-tx <dataList[N].serializedData> \
  --biz-type defi
```

**`--value` conversion**: `dataList[].value` is in minimal units (wei). `contract-call --value` expects UI units. Convert: `value_UI = value / 10^nativeToken.decimal` (18 for ETH/OKB, 9 for SOL). If value is `""`, `"0"`, or `"0x0"`, use `"0"`.

**Execution rules**:

- Execute `dataList[0]` first, then `[1]`, etc. Never in parallel.
- Wait for on-chain confirmation before next step.
- If any step fails, stop remaining steps and report which succeeded/failed.
- User confirmation required before every invest/withdraw/collect execution.

> **Solana transaction expiry**: Solana DeFi transactions expire in ~60 seconds. After receiving calldata, WARN the user: "This Solana transaction must be signed and broadcast within 60 seconds or it will expire."

## Displaying Results

### Position Table

| #   | Chain   | Platform   | Pool     | Value     | Rewards | APY   | Status |
| --- | ------- | ---------- | -------- | --------- | ------- | ----- | ------ |
| 1   | X Layer | Uniswap V3 | OKB-USDT | $2,450.00 | $12.30  | 18.5% | 🟢     |

- `investmentId` is internal — do NOT display to user
- `rate` from API is decimal → multiply by 100 and append `%`
- `tvl` → format as human-readable USD ($3.52B, $537M)
- Sort by position value descending
- **High APY warning**: If any position has APY > 50%, add: "⚠️ APY above 50% — elevated risk"

### Compound Summary

```
✅ Compound complete for {pool}
   Rewards claimed:    ${amount} ({token})
   Swapped to:         ${swap_amount} ({base_token})
   Reinvested:         ${reinvest_amount}
   Gas cost:           ${gas} ({chain})
   Effective APY:      {eff_apy}%
```

## Error Handling

| Error      | Scenario             | Recovery                                                                                 |
| ---------- | -------------------- | ---------------------------------------------------------------------------------------- |
| 84400      | Parameter null       | Check required params — position may have been fully exited                              |
| 84021      | Asset syncing        | "Position data is syncing, please retry in 30 seconds"                                   |
| 84014      | Balance check failed | Insufficient balance — skip this compound, alert user                                    |
| 84018      | Balancing failed     | Adjust slippage or skip V3 rebalance                                                     |
| 84010      | Token not supported  | Skip position, log warning                                                               |
| 50011      | Rate limit           | Wait 5 seconds and retry once                                                            |
| Swap 82000 | Token dead/rugged    | **CRITICAL**: Do NOT retry. Alert: "Reward token may be rugged. Manual review required." |
| Swap 81362 | Risk warning         | Do NOT auto-force. Alert user with risk details.                                         |

> Load on error: [references/troubleshooting.md](references/troubleshooting.md)

**Compound failure recovery**:

1. If collect fails → skip to next position, log error
2. If swap fails → hold claimed rewards in wallet, alert user
3. If reinvest fails → rewards remain as swapped tokens in wallet, alert user
4. Never retry more than once per position per cycle

## Risk Controls

### Safety Gates

| Gate               | Condition                                                          | Action                                                |
| ------------------ | ------------------------------------------------------------------ | ----------------------------------------------------- |
| Gas guard          | Gas cost > 50% of reward value                                     | SKIP compound, log reason                             |
| IL circuit breaker | IL > 2× user threshold                                             | PAUSE auto-compound, alert user                       |
| APY floor          | APY drops below user minimum                                       | FLAG position, suggest exit                           |
| TVL guard          | TVL drops > 60% in 7 days                                          | CRITICAL alert, recommend exit                        |
| Slippage guard     | Adaptive tiers (3%/5%/10% by price impact), confirm if impact > 5% | Escalate by tier; stop after Tier 3; never auto-force |
| Honeypot check     | isHoneyPot = true on reward token                                  | BLOCK swap, alert: "Reward token flagged as honeypot" |

### Fund-action Confirmation

Every operation that moves funds requires explicit user confirmation:

- Withdrawing from a position → confirm
- Swapping tokens → confirm (unless in approved autopilot mode)
- Depositing into a position → confirm
- Emergency exit → always confirm, even in autopilot

### Silent / Automated Mode

Enabled ONLY when user explicitly authorizes. Rules:

1. **Explicit opt-in required**: User must say "enable autopilot" or equivalent
2. **Risk gates still apply**: CRITICAL alerts halt autopilot and notify user
3. **Execution log**: Log every automated action (timestamp, position, amount, txHash, status)
4. **Session limit**: Auto-mode expires after 24 hours — user must re-authorize

## Post-Action Suggestions

| Just completed     | Suggest                                                                                             |
| ------------------ | --------------------------------------------------------------------------------------------------- |
| Position scan      | "Would you like me to compound the unclaimed rewards?" or "Want a health check on these positions?" |
| Auto-compound      | "Want to see a performance report?" or "Scan for more positions to compound?"                       |
| Rebalance          | "View updated allocations?" or "Generate a performance report?"                                     |
| Health check       | "Compound the healthy positions?" or "Exit the flagged positions?"                                  |
| Performance report | "Adjust your strategy?" or "Set up a compounding schedule?"                                         |

Present conversationally — never expose skill names, onchainos commands, or internal flow names to the user.

## Amount Display Rules

- Token amounts in UI units (`1.5 ETH`), never base units
- USD values with 2 decimal places
- Large amounts in shorthand (`$1.2M`)
- Gas fees in USD
- APY as percentage with 1 decimal
- IL as percentage with 1 decimal
- Sort positions by USD value descending

## Global Notes

- `--amount` must be in **minimal units** (integer). Convert: userAmount × 10^tokenPrecision
- The wallet address parameter for ALL defi commands is `--address`
- `--slippage` default `"0.01"` (1%); use adaptive tiers `"0.03"` / `"0.05"` / `"0.10"` by price impact; never auto-exceed `"0.10"`
- **Solana tx expiry**: Warn user — 60 second window for signing
- **X Layer zero gas**: Always highlight when suggesting chain for rebalancing
- **Address-chain compatibility**: EVM (`0x…`) → only EVM chains. base58 → only Solana. Make separate calls for each.
- User confirmation required before every invest/withdraw/collect
- Address used for calldata generation MUST match the signing address
- **Locale-aware output**: All user-facing content must be translated to match the user's language (English, Chinese, etc.)

## Reference Guides

> Load these files on demand — not needed for every operation.

- [`references/examples.md`](references/examples.md) — Full step-by-step worked examples (Solana SOL-USDC, X Layer OKB-USDT zero-gas)
- [`references/edge-cases.md`](references/edge-cases.md) — V3 NFT positions, multi-token rewards, dust rewards, cross-chain optimization, stale data, Solana tx expiry
- [`references/batch-compound.md`](references/batch-compound.md) — Batch execution order, per-chain thresholds, savings report, partial failure handling
- [`references/chains.md`](references/chains.md) — Chain index table, native token addresses, well-known stablecoins per chain
- [`references/efficiency.md`](references/efficiency.md) — Optimal call sequence, caching rules, token budget guidance
- [`references/cli-reference.md`](references/cli-reference.md) — Full CLI parameter tables and return field schemas
- [`references/troubleshooting.md`](references/troubleshooting.md) — Error codes and recovery steps

---
> Source: [Wizbisy/okx-yield-pilot](https://github.com/Wizbisy/okx-yield-pilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
