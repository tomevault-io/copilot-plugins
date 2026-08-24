## sam-copybot-poly-kalshi

> This is a copy-trading platform that monitors Polymarket trades via Alchemy webhooks and automatically replicates them with precision and speed.

# Polymarket Copybot Platform

This is a copy-trading platform that monitors Polymarket trades via Alchemy webhooks and automatically replicates them with precision and speed.

## Product Overview

### Core Purpose
- **Automatic Trade Copying**: Monitor specific wallet addresses on Polymarket and automatically copy their trades
- **Webhook-Driven**: Receives real-time transaction events from Alchemy blockchain webhooks
- **Precision Required**: 100% accuracy in market, outcome, price, shares, and side - this is real money
- **Speed Matters**: Execute copy trades quickly to minimize price slippage between original trade and copy
- **Event Timing**: Track game/event start times to set proper order expiry (default: 10 minutes before start)

### Data Flow
```
Alchemy Webhook → ApiRequestLog → PolymarketEnricher → PolymarketTrade → Copy Order Execution
```

1. Alchemy sends webhook when tracked wallet makes a transaction
2. `ApiRequestLog` stores the raw webhook payload
3. `PolymarketEnricher` enriches transaction with Polymarket API data
4. `PolymarketTrade` stores validated trade data ready for copying
5. Copy order gets executed on Polymarket (future feature)

## Critical Rules

### 1. Data Accuracy is Non-Negotiable
- **Never use fallback values**: No "Unknown Market", "Unknown Outcome", or default values
- **Fail-fast principle**: If any critical data is missing or uncertain, return `nil` and skip the trade
- **Critical fields**: market_title, outcome, price, shares/size, side (BUY/SELL), slug, outcome_index
- **Validate before saving**: All critical fields must be present and validated

### 2. API Documentation First
- **Always web_search before implementing**: Check latest API docs for Polymarket and Alchemy
- **Verify payload structures**: Don't assume - confirm the actual response format
- **API changes happen**: What worked yesterday may not work today

### 3. Database Schema Adherence
- **Check db/schema.rb first**: Before accessing any model attribute, verify it exists in the schema
- **Never assume fields exist**: This prevents runtime errors in production

### 4. Detailed Logging
- Use `log_debug` extensively in enrichment process
- Log when API calls succeed/fail
- Log when data is missing or validation fails
- This helps diagnose why trades aren't processing

## API Endpoints Reference

### Polymarket Gamma API
Base URL: `https://gamma-api.polymarket.com`

**Market Lookup:**
- `GET /markets/{slug}` - Get market details by slug (most reliable)
- `GET /markets?token_id=0x{token_id}` - Find market by token ID
- `GET /markets?condition_id=0x{condition_id}` - Find market by condition ID

**Response includes:**
- `question` or `title` - Market title
- `tokens[]` - Array of outcome tokens with `token_id`, `outcome`, `tokenId`
- `slug` or `market_slug` - Market URL slug
- `game_start_time` or `gameStartTime` - Event start time (for sports)
- `end_date_iso` or `endDate` - Market close time
- `category` or `tags[]` - Market category
- `description` - Market description

### Polymarket Data API  
Base URL: `https://data-api.polymarket.com`

**User Activity:**
- `GET /activity?user={address}&limit={limit}&sortBy=TIMESTAMP&sortDirection=DESC`
- Returns user's blockchain activity including trades

**User Trades:**
- `GET /trades?user={address}&limit={limit}&sortBy=TIMESTAMP&sortDirection=DESC`
- Returns enriched trade data with market info

**Trade Response Fields:**
- `type` - TRADE, SPLIT, MERGE, REDEEM
- `title` - Market title
- `outcome` - Outcome name (Yes/No or specific outcome)
- `side` - BUY or SELL
- `price` - Price per share (0-1)
- `size` - Number of shares
- `usdcSize` - Amount in USDC
- `timestamp` - Unix timestamp
- `transactionHash` - Blockchain tx hash
- `proxyWallet` - User's proxy wallet address
- `conditionId` - Token condition ID
- `slug` - Market slug
- `outcomeIndex` - Token index (0 or 1 for binary)

### Alchemy Webhooks
**Address Activity Webhooks:**
- Tracks all blockchain activity for specified addresses
- Webhook payload includes:
  - `event.activity[]` - Array of transaction activities
  - ERC1155 token transfers (Polymarket outcome tokens)
  - ERC20 token transfers (USDC payments)
  - Contract interactions with Polymarket exchange/CTF contracts

**Key Activity Fields:**
- `category` - "erc1155", "token", "internal"
- `asset` - Token symbol (e.g., "USDCE")
- `value` - Transfer amount
- `fromAddress` / `toAddress` - Transaction parties
- `erc1155Metadata[].tokenId` - The token ID (maps to market outcome)
- `erc1155Metadata[].value` - Amount in hex (divide by 1,000,000 for shares)

## Code Examples

### Validating Trade Data
```ruby
# BAD - Uses fallback values
data[:market] = market_info[:market] || "Unknown Market"

# GOOD - Fail fast if missing
unless market_info[:market].present?
  log_debug("⚠️ CRITICAL: Missing market title - skipping trade")
  return nil
end
data[:market] = market_info[:market]
```

### API Lookup Pattern
```ruby
# Always validate we got the data we need
market = @client.market_by_slug(slug)
return nil unless market

market_title = market["question"] || market["title"]
unless market_title.present?
  log_debug("⚠️ CRITICAL: Market missing title")
  return nil
end
```

### Token to Outcome Matching
```ruby
# Must find the exact token in the market's token array
tokens = market["tokens"] || []
matching_token = tokens.find do |t|
  t_id = (t["token_id"] || t["tokenId"]).to_s.downcase.gsub(/^0x/, '')
  t_id == clean_token_id
end

return nil unless matching_token
outcome = matching_token["outcome"] || matching_token["name"]
return nil unless outcome.present?
```

## Service Architecture

### PolymarketClient
- HTTP client for Polymarket APIs
- Methods: `market_by_slug`, `market_by_condition_id`, `activity_for_user`, `trades_for_user`
- Always returns `nil` on failure, never raises exceptions
- Uses debug logging to trace API calls

### PolymarketEnricher  
- Enriches blockchain transaction data with market metadata
- Two paths: API matches (fast) or blockchain parsing (slower but complete)
- Validates all data before returning
- Returns `nil` for entire enrichment if critical data missing
- Never saves trades with incomplete data

### Key Methods Flow
1. `enrich_and_save(api_log)` - Main entry point
2. `enrich_log(api_log)` - Extract and enrich transaction data
3. `format_trade_data()` - Format from API or blockchain activities
4. `fetch_market_info_by_token()` - Get full market metadata
5. `fetch_full_market_data()` - Comprehensive market details with validation

## Future Features
- Direct wallet management (add/remove wallets to track)
- Automated order execution on Polymarket
- Configurable order expiry offsets
- Trade filtering (only copy certain markets/categories)
- Position sizing (copy with different amounts)

## Testing Approach
- Test with real webhook payloads from Alchemy
- Verify enrichment produces complete, accurate data
- Check that failed enrichments return `nil` properly
- Validate trade data matches source wallet's actual trade

---

For Rails UI conventions (forms, views, icons, etc.), see `.cursor/rails-ui-rules.md`

---
> Source: [ortusclub/sam_copybot_poly_kalshi](https://github.com/ortusclub/sam_copybot_poly_kalshi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
