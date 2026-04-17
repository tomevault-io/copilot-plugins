## minervini-ticker

> This file helps Claude understand the project context and provides quick access to documentation.

# Claude Code Configuration

This file helps Claude understand the project context and provides quick access to documentation.

## Project Overview
- **Name**: Minervini Trend Index
- **Purpose**: Real-time synthetic stock index based on Mark Minervini's trend template criteria
- **Tech Stack**: Node.js, SQLite, Polygon.io API, WebSocket streaming

## API Documentation

### Polygon.io API
- **Main Documentation**: https://polygon.io/docs/stocks/getting-started
- **API Reference**: https://polygon.io/docs/stocks
- **Node.js Client**: https://github.com/polygon-io/client-js
- **Rate Limits**: https://polygon.io/docs/stocks/getting-started#rate-limits
- **Tickers Endpoint**: https://polygon.io/docs/stocks/list-ticker-symbols
- **Aggregates (OHLC)**: https://polygon.io/docs/stocks/get-aggregates
- **Technical Indicators**: https://polygon.io/docs/stocks/get-simple-moving-average-sma
- **Previous Close**: https://polygon.io/docs/stocks/get-previous-day-aggregates

### Mark Minervini Trend Template
- **8 Criteria Overview**: https://www.minervini.com/stock-selection-criteria
- **Trend Template Rules**:
  1. Price > 150-day and 200-day moving averages
  2. 150-day MA > 200-day MA
  3. 200-day MA trending upward (past 30 days)
  4. 50-day MA > 150-day and 200-day MAs
  5. Price within 25% of 52-week high
  6. Price at least 30% above 52-week low
  7. Relative strength > 70
  8. Daily volume > 100,000 shares

## Project Commands

### Data Setup
```bash
npm run get-tickers          # Fetch NYSE/NASDAQ tickers from Polygon
npm run setup-universe       # Initialize stock universe
```

### Screening
```bash
npm run screen               # Daily screening process
npm run create-synthetic     # Generate synthetic ticker with qualifying stocks
npm run test-synthetic       # Test with sample stocks (AAPL, MSFT, etc.)
```

### Development
```bash
npm run dev                  # Start with nodemon
npm run start                # Production start
npm run test-polygon         # Test Polygon API technical indicators
npm run lint                 # ESLint
npm run format               # Prettier
```

## Environment Variables

Required in `.env`:
```
POLYGON_API_KEY=your_api_key_here
NODE_ENV=development
LOG_LEVEL=info
PORT=3001
DB_PATH=./data/minervini_index.db
INDEX_BASE_VALUE=1000
REBALANCE_TIME=16:30
MIN_STOCK_PRICE=15.00
EXCHANGES=XNYS,XNAS
```

## Database Schema

### Tables
- `universe_stocks` - All NYSE/NASDAQ stocks
- `screening_results` - Daily Minervini screening results
- `rebalances` - Index rebalancing history
- `index_values` - Real-time index value tracking

## Key Files

### Core Logic
- `src/api/PolygonAPI.js` - Polygon.io integration
- `src/core/StockScreener.js` - Minervini criteria evaluation
- `src/core/MinerviniTrendIndex.js` - Main index orchestration
- `src/data/Database.js` - SQLite operations

### Scripts
- `src/scripts/createSyntheticTicker.js` - Generate synthetic ticker
- `src/scripts/getTickerList.js` - Fetch active stocks
- `src/scripts/dailyScreen.js` - Daily screening automation

## Common Tasks

When user asks to:
- **Add new stocks**: Use `npm run get-tickers`
- **Run screening**: Use `npm run create-synthetic`
- **Test functionality**: Use `npm run test-synthetic`
- **Check results**: Look in `./data/` directory
- **Debug API issues**: Check rate limits and API key
- **Modify criteria**: Edit `src/core/StockScreener.js`

## Troubleshooting

### API Rate Limits
- Free tier: 5 requests/minute
- Unlimited tier: 100 requests/second
- Adjust `rateLimiter` in `PolygonAPI.js`

### Database Issues
- Database file: `./data/minervini_index.db`
- Reset: Delete file and run initialization scripts

### No Qualifying Stocks
- Minervini criteria are strict
- Normal to have 0-50 stocks qualifying at any time
- Market conditions affect qualification rates

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kck924) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
