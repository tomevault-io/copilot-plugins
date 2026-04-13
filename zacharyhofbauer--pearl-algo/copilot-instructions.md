## pearl-algo

> **WARNING: This is a LIVE 24/7 automated futures trading system. Every code change can cause real financial loss.**

# PearlAlgo - Claude Code Instructions

**WARNING: This is a LIVE 24/7 automated futures trading system. Every code change can cause real financial loss.**

## Architecture

- **Market data:** IBKR gateway (data only, NOT used for execution)
- **Order execution:** Tradovate (paper account = source of truth for trades)
- **Notifications:** Removed (Telegram code deleted)
- **Entry point:** `pearl.sh` (master control), `python -m pearlalgo.market_agent.main`
- **Canonical live config:** `config/live/tradovate_paper.yaml`
- **Current live state root:** `/home/pearlalgo/var/pearl-algo/state/data/agent_state/MNQ` (repo `data/` symlinks here)
- **Prop firm:** MFF compliance via TraderSyncer (copies demo -> live)

## Critical Safety Rules

**NEVER change these without explicit user approval:**

| Setting | Required Value | Why |
|---------|---------------|-----|
| `execution.armed` | current value | Controls live order submission |
| `execution.enabled` | current value | Master execution switch |
| `execution.mode` | current value | Paper vs live |
| `max_positions` | current value | Position limit |
| `max_position_size_per_order` | 1 | 1 contract per order, adds allowed |
| `max_position_size` | 5 | MFF max 5 MNQ total |
| `guardrails.*` | current values | Minimal execution safety without legacy signal gating |
| `virtual_pnl.*` | disabled | Not used, Tradovate is source of truth |
| `ibkr.execution` | inactive | IBKR is data-only |

## Config Rules

- **YAML duplicate keys are SILENT** — last key wins, no error.
- Canonical runtime edits belong in `config/live/tradovate_paper.yaml`.
- Always validate the canonical runtime config after editing: `python -c "import yaml; yaml.safe_load(open('config/live/tradovate_paper.yaml'))"`

## Forbidden Actions

1. Do NOT re-enable IBKR execution
2. Do NOT increase contract sizes above 1
3. Do NOT disable execution guardrails or drawdown limits
4. Do NOT enable virtual PnL
5. Do NOT reintroduce legacy time / direction / regime signal gates without user approval
6. Do NOT restart the trading service without user approval

## Testing

- Run `python -m pytest tests/ -x -q` before any changes
- Validate YAML configs after editing
- Check `logs/` for errors after changes

## Key Files

| File | Purpose |
|------|---------|
| `src/pearlalgo/market_agent/service.py` | Main service orchestrator |
| `src/pearlalgo/market_agent/service_loop.py` | Core trading loop |
| `src/pearlalgo/market_agent/performance_tracker.py` | Trade tracking + trades.db |
| `src/pearlalgo/market_agent/signal_handler.py` | Signal processing |
| `src/pearlalgo/execution/tradovate/adapter.py` | Order execution adapter |
| `src/pearlalgo/execution/tradovate/client.py` | Tradovate API client |
| `src/pearlalgo/strategies/composite_intraday/engine.py` | Canonical live strategy bundle |
| `src/pearlalgo/trading_bots/signal_generator.py` | Core signal generation engine (indicators, regime detection, entry logic) |
| `src/pearlalgo/market_agent/state_manager.py` | Signal state machine |
| `config/live/tradovate_paper.yaml` | Canonical live runtime configuration |
| `config/accounts/tradovate_paper.yaml` | Legacy compatibility overlay; canonical live config is `config/live/tradovate_paper.yaml` |
| `apps/pearl-algo-app/` | Next.js web dashboard (standalone mode, port 3001) |
| `src/pearlalgo/api/server.py` | FastAPI API server (port 8001) |
| `src/pearlalgo/api/indicator_service.py` | Indicator calculations (EMA, VWAP AA, BB, ATR) |
| `apps/pearl-algo-app/hooks/useChartManager.ts` | Chart series initialization and refs |
| `apps/pearl-algo-app/lib/schemas.ts` | Zod validation for WS/API data |

## Data Insights (from 922-trade analysis)

- Overnight sessions (18:00-08:30 ET) historically lose money (-$4,477 net)
- Short trades have poor win rate compared to longs
- Legacy signal gating is intentionally OFF on the canonical live path — user wants strategy decisions without hour/direction/regime vetoes
- Execution should remain disarmed until you explicitly re-arm it.
- Tradovate Paper is the sole live execution account; IBKR remains data-only.

## Pearl Algo Memory Bridge (REQUIRED at session end)

Write what was built to Pearl Algo's memory so the agent stays in sync with code changes:

**From px-core, the bridge runs remotely:**
```bash
# At end of every session, SSH to Mac and write journal entry:
ssh pearlassistant@$(tailscale ip -4 2>/dev/null || echo 'PEARL-Macbook')   "bash ~/.openclaw/pearl-workspace/scripts/cc-session-bridge.sh '$(cat <<SUMMARY
- [what was built/changed]
- [key decisions]
- [files changed]
- [status: tests passing/failing]
SUMMARY
)' 'pearl-algo'" 2>/dev/null || echo 'Bridge unavailable - manually update Pearl Algo MEMORY.md'
```

**What to include:**
- Algorithm changes and why
- Risk/circuit breaker logic changes
- Test results
- Anything Pearl Algo agent needs to know

---

## Webapp UI/UX Design System

**You are the sole design and frontend engineer for pearlalgo.io.** Every webapp change must meet professional fintech/trading-terminal standards. Think TradingView, Bloomberg Terminal, Arc Browser — dense, functional, beautiful.

### Design Philosophy

1. **Information density over whitespace** — This is a trading dashboard, not a marketing site. Every pixel should earn its place. Pack data tight but keep it scannable.
2. **Visual hierarchy through color and weight, not size** — Use `--text-primary` / `--text-secondary` / `--text-tertiary` / `--text-muted` to create hierarchy. Don't make things bigger — make important things brighter.
3. **Consistency is taste** — Every panel, card, label, and number must follow the same patterns. If one panel uses `--font-size-xs` for labels, ALL panels use `--font-size-xs` for labels.
4. **Motion is information** — Transitions signal state changes. Use `--duration-fast` for micro-interactions, `--duration-normal` for panel transitions. Never animate decoratively.
5. **Dark theme is the only theme** — `--bg-primary: #131722` is the canvas. Never introduce light backgrounds.

### Design Tokens (MUST use — never hardcode colors/sizes)

All values live in `styles/tokens.css`. Reference them by CSS variable name:

| Token | Usage |
|-------|-------|
| `--bg-primary` (#131722) | Page background |
| `--bg-secondary` / `--bg-card` (#1e222d) | Cards, panels, containers |
| `--bg-elevated` (#2a2e39) | Hover states, active items, elevated surfaces |
| `--text-primary` (#d1d4dc) | Numbers, values, primary content |
| `--text-secondary` (#787b86) | Labels, headers, secondary info |
| `--text-tertiary` (#5d6067) | Timestamps, tertiary info |
| `--text-muted` (#4a4e59) | Disabled, decorative text |
| `--accent-cyan` (#2962ff) | Primary action, links, active states |
| `--accent-green` (#26a69a) | Profit, long, success, online |
| `--accent-red` (#ef5350) | Loss, short, error, offline |
| `--accent-yellow` (#f57f17) | Warning, caution |
| `--accent-purple` (#ab47bc) | Special/ML indicators |
| `--border-color` (#2a2e39) | Panel borders |
| `--border-subtle` (#363a45) | Inner separators |

**Semantic aliases — use these for meaning:**
- `--color-profit` / `--color-loss` for P&L
- `--color-long` / `--color-short` for direction
- `--color-status-online` / `--color-status-warning` / `--color-status-offline` for health
- `--color-info` / `--color-success` / `--color-warning` / `--color-error` for UI feedback

### Typography Rules

- **Font:** `--font-mono` everywhere (SF Mono > Monaco > Consolas)
- **Data values:** `--font-size-md` (12px), `--font-weight-medium` (500)
- **Labels:** `--font-size-xs` (10px), `--font-weight-medium` (500), `--text-secondary`
- **Panel titles:** `--panel-title-font-size` (10px), uppercase, `--panel-title-color`
- **Large numbers (P&L, price):** `--font-size-lg` (14px) or `--font-size-xl` (18px), `--font-weight-bold`
- **Never use font sizes outside the token scale**

### Spacing Rules

- Use `--space-*` tokens (2px, 4px, 6px, 8px, 12px, 16px, 24px)
- Panel internal padding: `--space-4` (8px) to `--space-5` (12px)
- Gap between panels: `--space-4` (8px)
- Between label and value: `--space-1` (2px) to `--space-2` (4px)
- **Never use arbitrary pixel values for spacing**

### Component Patterns

**Panel/Card:**
```css
background: var(--bg-card);
border: 1px solid var(--border-color);
border-radius: var(--radius-md);
padding: var(--space-5);
```

**Panel Header:**
```css
font-size: var(--panel-title-font-size);
font-weight: var(--panel-title-font-weight);
color: var(--panel-title-color);
text-transform: uppercase;
letter-spacing: 0.05em;
margin-bottom: var(--space-3);
```

**Status Dot:**
```css
width: 6px; height: 6px;
border-radius: var(--radius-full);
background: var(--color-status-online); /* or warning/offline */
```

**Data Row (label + value):**
```css
.label { color: var(--text-secondary); font-size: var(--font-size-xs); }
.value { color: var(--text-primary); font-size: var(--font-size-md); font-weight: 500; }
```

**P&L Display:**
```css
color: value >= 0 ? var(--color-profit) : var(--color-loss);
font-weight: var(--font-weight-bold);
/* Always prefix with + or - sign, always show 2 decimal places */
```

### Responsive Breakpoints

| Breakpoint | Target | Behavior |
|-----------|--------|----------|
| <=480px | Small phone | Stack everything, compact padding |
| <=640px | Large phone | Nav collapses, cards stack |
| <=768px | Tablet | Mobile layout, 2-col stats |
| <=1024px | Desktop | Full layout, max-width kicks in |
| <=1440px | Wide | Extra breathing room |

### CSS Architecture

- **All styles in `styles/`** — never inline styles in JSX, never CSS-in-JS
- **Component CSS:** `styles/components/_component-name.css`
- **Layouts:** `styles/layouts/_layout-name.css`
- **Tokens:** `styles/tokens.css` (source of truth)
- **Globals:** `app/globals.css` (imports tokens + components)
- **No Tailwind, no utility classes** — semantic CSS classes only

### Screenshot-Driven Workflow (Chrome DevTools MCP)

Claude has access to Chrome DevTools MCP (headless) and Playwright MCP. Use them:

1. **After every visual change**, take a screenshot of `http://localhost:3001` to verify the result
2. **Before starting**, screenshot the current state as a baseline
3. **Never ship blind** — if you can't screenshot (server down, etc.), note it explicitly
4. **Use Playwright** for testing responsive layouts at different viewport widths
5. **Use Chrome DevTools** for DOM inspection, Lighthouse audits, and evaluating runtime behavior

### Accessibility Hard Requirements (WCAG 2.1 AA)

These are **not optional** — every UI element must meet these:

| Requirement | Standard | How to Verify |
|-------------|----------|---------------|
| Body text contrast | >= 4.5:1 ratio | Chrome DevTools Lighthouse audit |
| Large text contrast (>=18px bold / >=24px) | >= 3:1 ratio | Chrome DevTools color picker |
| UI component contrast (borders, icons) | >= 3:1 ratio | Manual check against `--bg-primary` |
| Touch/click targets | >= 44x44px with 8px spacing | Inspect element dimensions |
| Focus states | Visible on ALL interactive elements | Tab through the page |
| Input labels | Every input has an associated `<label>` | Inspect DOM |
| Semantic HTML | Use `<nav>`, `<main>`, `<section>`, `<button>` | ARIA only when native HTML can't do it |
| Color is not sole indicator | Use icons/patterns alongside color | Check with grayscale filter |

### SaaS Dashboard Patterns

- **Layout:** top bar + sidebar + main content (current pattern — keep it)
- **Data displays:** semantic colors WITH icons/patterns for colorblind users (e.g., triangles with green/red)
- **Empty states:** action-oriented messaging ("No trades today. The market opens at 18:00 ET.")
- **Notifications/toasts:** 500ms per word + 3 seconds base duration
- **Tables:** sticky headers, alternating row hints via subtle bg shift, sortable columns where useful

### Before Every Webapp Change

1. **Read `styles/tokens.css`** — know the current design tokens
2. **Read the component's CSS file** — understand existing patterns
3. **Read neighboring components** — match their visual style
4. **Use existing `components/ui/` primitives** where possible (StatDisplay, InfoTooltip)
5. **New reusable UI patterns -> add to `components/ui/`** with their own CSS

### Pre-Delivery Checklist

After making webapp changes, verify ALL of the following before considering the task done:

- [ ] **Screenshot taken** — use Chrome DevTools MCP to capture the result at `http://localhost:3001`
- [ ] **Contrast ratios pass** — body text >= 4.5:1, large text >= 3:1, UI components >= 3:1
- [ ] **Touch targets >= 44x44px** — all buttons, links, interactive elements
- [ ] **Test at 480px** — mobile layout works, nothing overflows or overlaps
- [ ] **Test at 1440px** — desktop layout uses space well, no stretched elements
- [ ] **No hardcoded colors** — all colors use `var(--token-name)`
- [ ] **No hardcoded spacing** — all spacing uses `var(--space-*)` tokens
- [ ] **Focus states work** — tab through interactive elements, all show visible focus
- [ ] **Build succeeds** — `npm run build` passes in `apps/pearl-algo-app/`
- [ ] **Restart webapp service** — `sudo systemctl restart pearlalgo-webapp.service`

### UX Principles for Trading Dashboards

- **Numbers align right** — financial data, P&L, prices always right-aligned
- **Green = profit/long, Red = loss/short** — never swap or reassign these
- **Stale data is dangerous** — always show data freshness (timestamps, status dots, staleness indicators)
- **Errors are loud** — red background tint, not just red text
- **Loading states are explicit** — skeleton, spinner, or "—" placeholder. Never show stale data as if fresh
- **Zero is neutral** — $0.00 P&L uses `--text-primary`, not green or red
- **Negative numbers get a minus sign, positive get a plus** — always explicit
- **Time formats:** 24h for timestamps (14:32:05), relative for recency ("2m ago")
- **Currency:** Always prefix $ and show 2 decimal places for USD amounts
- **Responsive is required** — every layout must work from 480px phone to 1440px desktop

### Reference Sites (Study These for Inspiration)

- **TradingView** — chart interaction patterns, panel layouts, data density
- **Bloomberg Terminal** — information density, color coding, keyboard-driven UX
- **Grafana** — dashboard grid layouts, time-series display, status indicators
- **Linear** — polish, transitions, command palette patterns
- **Arc Browser** — modern dark UI, subtle gradients, clean typography

### Anti-Patterns (NEVER Do These)

- Hardcoded hex colors — use tokens
- Hardcoded pixel values for spacing — use `--space-*` tokens
- Inline styles in JSX
- CSS-in-JS or styled-components
- Light mode or light backgrounds
- Decorative animations with no informational purpose
- Marketing-style layouts (big hero sections, excessive whitespace)
- Inconsistent panel styling (different border radius, padding, header styles)
- Using `px` for font sizes instead of tokens
- Green for negative values or red for positive values
- Showing stale data without a staleness indicator

---

## Restart Doctrine (updated 2026-03-19)

**systemd is the ONLY process manager. Never use pkill on pearlalgo services.**

| Command | Use when |
|---|---|
| `./pearl.sh soft-restart` | Code change, config update, agent issue — no gateway touch, no 2FA |
| `./pearl.sh hard-restart` | Gateway crashed or wifi/power cycle — may trigger IBKR 2FA |

**How it works:**
- `systemctl stop` waits for confirmed dead before next step (no race condition)
- Start order: openclaw -> agent -> api -> webapp
- Stop order: webapp -> api -> agent -> openclaw (reverse)
- With `--gateway`: stop gateway first, start + wait for API-ready, then services

**Never:**
- `pkill -f pearlalgo` — bypasses systemd, causes duplicate instances
- `systemctl restart pearlalgo-agent` directly — skips cleanup and ordering
- Run `scripts/maintenance/nuke_services.sh` — archived, dangerous

**Old aliases (still work for backwards compat):**
- `soft-restart` = `restart`
- `hard-restart` = `restart --gateway`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zacharyhofbauer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zacharyhofbauer)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
