## trading-dashboard

> 1. Backend: `StrategyWalletsService.STRATEGY_KEYS` + `DISPLAY_NAMES` + `normalizeStrategy()`

# Trading Dashboard -- Rules

## Adding a New Strategy Wallet
1. Backend: `StrategyWalletsService.STRATEGY_KEYS` + `DISPLAY_NAMES` + `normalizeStrategy()`
2. Frontend: `StrategyFilter` type + `STRATEGY_COLORS` + filter dropdown + skeleton
3. Each strategy starts with 10,00,000 (10 Lakh) initial capital
4. Wallets auto-created by trade execution in Redis

## Display Standards
- Missing data: show `DM` (Data Missing) -- styled `text-slate-500 italic text-[10px]`
- Error states: show `ERR` badge -- styled `bg-red-500/20 text-red-400 text-[10px]`
- Never show empty strings, `-`, or `N/A`

## Build & Restart

### Backend
```bash
ps aux | grep "spring-boot:run" | grep -v grep
kill -9 <PID>
cd /home/ubuntu/trading-dashboard/backend && nohup mvn spring-boot:run >> nohup.out 2>&1 &  # use `>>` append, NOT `>`, to preserve forensic log history across restarts
tail -100f nohup.out
```

### Frontend
```bash
ps aux | grep "npm run dev" | grep -v grep
kill -9 <PID>
cd /home/ubuntu/trading-dashboard/frontend && nohup npm run dev > nohup.out 2>&1 &
tail -100f nohup.out
```

## Key Files
- Backend: `StrategyWalletsService.java` -- Wallet summary from Redis
- Backend: `StrategyWalletsController.java` -- REST API for wallet data
- Frontend: `StrategyWalletsPage.tsx` -- Strategy wallet cards
- Frontend: `api.ts` -- API client

---
> Source: [Shiv7/trading-dashboard](https://github.com/Shiv7/trading-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
