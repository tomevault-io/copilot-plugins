## dashboard-hm-sp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

VestPol Support Dashboard - A live operations dashboard for HallMonitor & SwitchPay support teams. Displays real-time metrics for telephony queues, Jira support tickets, Jira order pipelines, and e-conomic financial data.

## Commands

### Development
```bash
# Start backend API server (runs on http://localhost:3001)
npm run server

# Start frontend dev server (runs on http://localhost:5173)
npm run dev

# Build production bundle
npm run build

# Preview production build
npm run preview

# Lint TypeScript/React code
npm run lint
```

### Testing APIs Directly
```bash
# Health check
curl http://localhost:3001/api/health

# Telephony queue statistics
curl http://localhost:3001/api/telephony/support

# Jira support issues
curl http://localhost:3001/api/jira/support

# Jira orders pipeline
curl http://localhost:3001/api/jira/orders-pipeline

# e-conomic open posts
curl http://localhost:3001/api/economic/open-posts
```

## Architecture

### Frontend-Backend Split
- **Frontend**: React 19 + TypeScript + Vite, single-page dashboard in `src/App.tsx`
- **Backend**: Express API server in `server.mjs` that proxies/transforms data from external services
- API base URL configured in `src/config.ts`:
  - Development: `http://localhost:3001` (used by `npm run dev`)
  - Production: `http://192.168.1.130:3001` (used by `npm run build`)
  - Automatically switches based on Vite's build mode
- Vite configured with base path `/operations-dashboard/` in `vite.config.ts:7`

### Service Layer Pattern
Backend uses separate service modules in `services/` directory:
- **telephonyService.mjs**: Integrates with Uni-tel One-Connect via web scraping (Puppeteer) + CSV fallback
- **jiraService.mjs**: Dual Jira instance integration (HallMonitor + SwitchPay) using REST API v3
- **economicService.mjs**: e-conomic REST API integration for financial data
- **oneConnectScraper.mjs**: Puppeteer-based scraper for One-Connect dashboard
- **telephonyCsvParser.mjs**: CSV parser fallback for telephony data when scraping fails

### Multi-Tenant Data Structure
All API responses follow a dual-tenant pattern with `hallmonitor` and `switchpay` keys:
```javascript
{
  hallmonitor: { /* data */ },
  switchpay: { /* data */ }
}
```

### Telephony Data Integration Strategy
The telephony service has a unique hybrid approach:
1. **Primary method**: Web scraping One-Connect dashboard with Puppeteer (headless browser)
2. **Fallback method**: CSV file parsing from `data/telephony/*.csv`
3. **Cache layer**: Data cached with scheduled refresh times (09:00, 10:00, 11:00, 13:00, 14:00, 15:30)
4. **Mock data**: Returns safe defaults if both methods fail

This complexity exists because the CSTA API provided by Uni-tel only supports call control, NOT queue statistics (see `docs/telephony-api-requirements.md` for details).

### Jira Dual-Instance Architecture
The Jira service manages two completely separate Jira instances:
- **HallMonitor**: Uses `JIRA_HM_*` environment variables
- **SwitchPay**: Uses `JIRA_SP_*` environment variables

Each instance has separate project keys for Support and Orders. The service constructs Basic Auth headers per-instance and executes parallel API calls with `Promise.all()`.

### Auto-refresh Mechanism
- Frontend polls all 4 endpoints every 5 minutes (300000ms) via `setInterval` in `App.tsx:100`
- Backend telephony service uses time-based cache invalidation (not TTL-based)
- Last update timestamp displayed to users in `da-DK` locale format

## Environment Configuration

### Backend Configuration
Required environment variables in `.env` (see `.env.example`):

```bash
# One-Connect web scraper credentials
ONECONNECT_EMAIL=your_email
ONECONNECT_PASSWORD=your_password

# HallMonitor Jira
JIRA_HM_URL=https://hallmonitor.atlassian.net
JIRA_HM_EMAIL=your_email
JIRA_HM_API_TOKEN=your_token
JIRA_HM_SUPPORT_PROJECT_KEY=HS
JIRA_HM_ORDERS_PROJECT_KEY=HO

# SwitchPay Jira
JIRA_SP_URL=https://switchpaydev.atlassian.net
JIRA_SP_EMAIL=your_email
JIRA_SP_API_TOKEN=your_token
JIRA_SP_SUPPORT_PROJECT_KEY=SUP
JIRA_SP_ORDERS_PROJECT_KEY=ORDERS

# e-conomic API
ECO_APP_SECRET_TOKEN=your_token
ECO_AGREEMENT_GRANT_TOKEN=your_token
```

### Frontend Configuration
Frontend API URL is configured in `src/config.ts`. Update the `apiBaseUrl` values if your backend runs on different URLs:
```typescript
const config = {
  development: { apiBaseUrl: 'http://localhost:3001' },
  production: { apiBaseUrl: 'http://192.168.1.130:3001' },
};
```

## Key Implementation Details

### TypeScript Strictness
The project uses strict TypeScript (`tsconfig.json`):
- `strict: true`
- `noUnusedLocals: true`
- `noUnusedParameters: true`
- All React component props must be explicitly typed

### CORS Configuration
Backend enables CORS for all origins (`server.mjs:11`). In production, restrict to specific frontend domains.

### Error Handling Pattern
All service methods include try-catch with fallback to mock data:
```javascript
try {
  const realData = await fetchFromAPI();
  return realData;
} catch (error) {
  console.error('API error:', error.message);
  return getMockData();
}
```

This ensures the dashboard always renders, even with API failures.

### Puppeteer Browser Lifecycle
The One-Connect scraper maintains a persistent browser instance:
- Launched once on first scrape attempt
- Reused for subsequent scrapes
- Uses headless mode with specific Chrome flags for server compatibility
- Browser/page cleanup handled by Node.js process lifecycle

### JQL Query Strategy
Jira queries use `statusCategory != Done` instead of listing specific open statuses. This is more resilient to custom workflow configurations.

### Data Directory Structure
```
data/
└── telephony/
    └── *.csv  # CSV fallback files for telephony data
```

## Deployment Notes

- Frontend API URL configured in `src/config.ts` - automatically switches based on Vite build mode
- Backend credentials stored in `.env` file (gitignored) - never commit this file
- Vite base path set to `/operations-dashboard/` - affects asset paths in production
- Backend preloads telephony data on startup to warm the cache (`server.mjs:82-89`)
- Puppeteer requires Chrome/Chromium binaries available in deployment environment

## External API References

- **Jira REST API v3**: https://developer.atlassian.com/cloud/jira/platform/rest/v3/
- **e-conomic REST API**: https://restdocs.e-conomic.com/
- **Uni-tel CSTA API**: https://uni-tel.github.io/api-documentation/ (NOT used - only supports call control, not queue stats)

---
> Source: [vestermark10/dashboard-hm-sp](https://github.com/vestermark10/dashboard-hm-sp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
