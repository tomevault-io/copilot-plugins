## digiops-currency

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

WSO2 Digital Wallet is a React microapp that runs inside a mobile SuperApp WebView. It manages WSO2 Coin (O2C) ERC20 token operations on a private blockchain — wallet creation/recovery, token transfers, balance checking, and transaction history.

Part of the `digiops-currency` monorepo under `apps/wso2-digital-wallet/`. Related services include `operations/transaction-service/` (NestJS backend).

## Commands

```bash
npm start          # Dev server at localhost:3000
npm run build      # Production build to /build
npm test           # Tests in watch mode (react-scripts test)
npm test -- --watchAll=false   # Single test run (CI mode)
npm test -- --watchAll=false src/path/to/File.test.js   # Single test file
```

No standalone lint script — ESLint runs via CRA's `react-scripts` (config: `react-app` + `react-app/jest` in `package.json`).

## Environment

Copy `.env.example` to `.env` and set:
- `REACT_APP_RPC_ENDPOINT` — Blockchain RPC URL
- `REACT_APP_CONTRACT_ADDRESS` — ERC20 token contract address
- `REACT_APP_CHAIN_ID` — Private chain ID
- `REACT_APP_WALLET_SERVICE_BASE_URL` — Backend wallet service URL

## Architecture

### SuperApp Registration (`microapp.json`)
Manifest consumed by the host SuperApp. `appId: com.wso2.superapp.microapp.wallet` identifies the microapp; `versions[]` lists downloadable bundle URLs. `package.json` sets `"homepage": "."` so the built bundle uses relative asset paths and can be served from any WebView path.

### Native Bridge Layer (`src/microapp-bridge.js`)
All communication with the host SuperApp goes through `window.nativebridge` / `window.ReactNativeWebView`. This provides JWT auth tokens, encrypted local storage, QR scanning, alerts, and navigation. The bridge uses callback-based APIs wrapped in helper functions.

- `src/helpers/bridge.js` — `waitForBridge(maxWait)` polls until both `window.nativebridge` and `window.ReactNativeWebView` are present; pages call this before any native operation.
- `src/helpers/storage.js` — Promise wrappers over the bridge's callback-based storage. Bridge serializes as `btoa(JSON.stringify(value))` on write and `JSON.parse(atob(value))` on read; always use these wrappers, not the bridge functions directly.
- `requestOpenMicroApp(targetAppId, launchData)` — navigates the SuperApp to another microapp; used by the parking payment flow to return to the caller app.

### Service Layer (`src/services/`)
- **`blockchain.service.js`** — Ethers.js v5 wrapper. `getRPCProvider()` creates `StaticJsonRpcProvider` with JWT Bearer auth in headers and caches the provider for 30 s (single-flight to prevent concurrent token fetches). Handles balance queries, token transfers (`gasPrice: '0'` — free private chain), and transaction history via Transfer event log filtering. `getRecentTransactions` (legacy, scans last 10k blocks) is superseded by `getTokenTransfersByAddress` / `getTransactionHistory` (paginated, uses `queryFilter` with concurrency cap `RPC_CONCURRENCY = 8`).
- **`wallet.service.js`** — REST API calls to the backend wallet service for registering/managing wallet addresses.
- **`query-hooks.js`** — balance (`staleTime: 30s`) and block number (`staleTime: 10s`) React Query hooks.
- **`query-history-hooks.js`** — `useWalletTransactionHistory`: server-side offset pagination, `staleTime: 60s`, no polling.
- **`src/hooks/useTransactionHistory.js`** — `useTransactionHistory`: fetches all transactions up to `MAX_TRANSFER_PAGE = 200` in one call, then filters and paginates client-side. This is what the History page uses.

### Authentication
JWT token is retrieved from the native bridge (`src/helpers/auth.js`) and injected into both RPC provider headers and backend API calls.

### Storage
Sensitive data (private keys, wallet addresses) is stored via the native bridge's encrypted storage, not browser localStorage. Keys defined in `src/constants/configs.js` → `STORAGE_KEYS`.

### Routing (`src/pages/Pages.js`)
Hash-based routing via React Router v6. Key routes: `/` (home/dashboard), `/create-wallet`, `/recover-wallet`, `/send`, `/confirm-assets-send`, `/receive`, `/profile`, `/history`.

### Data Flow Pattern
```
Page Component → Service Function → React Query Hook → Ethers.js/Fetch → Native Bridge (auth)
```

### Parking Payment Integration
`src/helpers/parkingPaymentFlow.js` handles a special flow where launch data from the SuperApp pre-fills the SendAssets page for parking meter payments.

## Key Technical Details

- **Ethers v5 pinned**: `ethers@5.7.0` (exact). v6 has incompatible APIs — `StaticJsonRpcProvider`, `BigNumber`, `utils.parseUnits` all moved or changed. Do not upgrade or mix v6 import patterns.
- **Gas**: All transfers use `gasPrice: '0'` and `gasLimit: 5000000` (private chain, no fees)
- **Wallet generation**: Client-side via `ethers.Wallet.createRandom()` (BIP39 12-word seed phrase)
- **UI framework**: Ant Design v5 + Reactstrap, with dark/light theme switching via `react-css-theme-switcher`
- **Contract ABI**: Hardcoded in `src/constants/configs.js` (ERC20 with airDrop and authorized wallet extensions)
- **No centralized state store**: Uses React Query for server state, component-level useState for UI state

---
> Source: [wso2/digiops-currency](https://github.com/wso2/digiops-currency) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
