## bimboh

> Bimboh is a memecoin hunting platform with multi-component architecture:

# AGENTS.md - Bimboh Development Guide

## Architecture
Bimboh is a memecoin hunting platform with multi-component architecture:
- `frontend/` - Next.js app with ShadCN/UI, TypeScript, TailwindCSS, Solana wallet integration
- `scraper/` - Python web scraper using Selenium for TikTok data collection  
- `js-scraper/` - Node.js Puppeteer scraper for alternative TikTok scraping
- `twitter/` - Node.js bot for automated Twitter posting using twitter-api-v2
- `bitquery/` - Node.js Solana blockchain data fetcher with Web3.js integration
- Database: Supabase for data storage and API

## Commands
### Frontend (in frontend/)
- `npm run dev` - Start development server
- `npm run build` - Build production
- `npm run start` - Start production server  
- `npm run lint` - Run ESLint

### Python Scraper (in scraper/)
- `python main.py` - Run TikTok scraper

### Node.js Components  
- No specific test commands defined (use `node index.mjs` or similar to run)

## Code Style
- Frontend: TypeScript strict mode, ESLint with Next.js rules, ShadCN components
- Path aliases: `@/*` maps to frontend root
- Python: Standard Python imports, logging configured, environment variables via dotenv
- Node.js: ES modules (.mjs), async/await patterns, dotenv for config
- All projects use environment variables for sensitive data (Supabase, API keys)

---
> Source: [Afoxcute/bimboh](https://github.com/Afoxcute/bimboh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
