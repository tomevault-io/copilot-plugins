## soexpensive

> **Finnish supermarket price comparison web application**

# SoExpensive - Claude Documentation

**Finnish supermarket price comparison web application**
**Live:** https://soexpensive.jaime.win / https://soexpensive.vercel.app

---

## 📋 Quick Reference

### Key Facts
- **Status:** Deployed & Functional (MVP phase)
- **Tech Stack:** Node.js + Express + PostgreSQL + Vanilla JS
- **Deployment:** Vercel (free tier)
- **Git Workflow:** Single branch (`claude/price-comparison-app-km6Wj`)
- **Performance:** LCP 0.80s ✅ | CLS 0 ✅ | INP 32ms ✅

### Important Constraints
- ❌ **No manual data entry** - Automation required
- ❌ **No paid services** - Free tier only
- ❌ **No web scraping yet** - Blocked by stores, using sample data
- ✅ **One real data source:** K-Citymarket CSV (manual collection for now)
- ✅ **Monospace terminal aesthetic** - Design principle to maintain

---

## 🏗️ Architecture

### Technology Stack

**Backend:**
- Runtime: Node.js (serverless on Vercel)
- Framework: Express.js v4.18.2
- Database: PostgreSQL (Vercel Postgres, 256MB free tier)
- Price Storage: INTEGER (cents) for precision

**Frontend:**
- Pure vanilla JavaScript (no frameworks)
- CSS3 with monospace terminal aesthetic
- JetBrains Mono font
- ASCII charts for visualization
- 5-minute client-side caching (localStorage)

**Deployment:**
- Platform: Vercel
- Function Limits: 10s timeout, 1024MB memory (free tier)
- Auto-deploy: Every commit to `claude/price-comparison-app-km6Wj`
- Domain: soexpensive.jaime.win (custom)

### Project Structure
```
soexpensive/
├── api/                    # Vercel serverless functions
│   ├── index.js           # Main entry (exports server.js)
│   ├── products.js        # Product listing
│   ├── stores.js          # Store listing
│   ├── fetch-prices.js    # Price updates (sample data)
│   ├── import-prices.js   # CSV price import
│   ├── seed.js            # Database seeding
│   ├── cleanup.js         # Database reset
│   └── health.js          # Health check
├── public/                # Static frontend
│   ├── index.html         # Main UI
│   ├── admin.html         # Admin panel
│   ├── app.js             # Client JS (447 lines)
│   ├── styles.css         # Main styles (437 lines)
│   ├── monospace.css      # Terminal aesthetic
│   └── reset.css          # CSS reset
├── server.js              # Express app (171 lines)
├── db.js                  # PostgreSQL layer (204 lines)
├── seed.js                # Local seeding script (224 lines)
├── prices.csv             # Real K-Citymarket prices (37 products)
├── sample-prices.json     # Sample data for other stores
├── vercel.json            # Vercel config
└── package.json           # Dependencies
```

---

## 💾 Database Schema

### Current Schema (PostgreSQL)

```sql
-- Products table
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  category TEXT NOT NULL,
  unit TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(name, category, unit)  -- Prevents duplicate products ✅
);

-- Stores table
CREATE TABLE stores (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL UNIQUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Prices table
CREATE TABLE prices (
  id SERIAL PRIMARY KEY,
  product_id INTEGER NOT NULL REFERENCES products(id),
  store_id INTEGER NOT NULL REFERENCES stores(id),
  price_cents INTEGER NOT NULL,  -- Stored as cents for precision
  recorded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(product_id, store_id)  -- ⚠️ BLOCKS PRICE HISTORY
);

-- Index for performance
CREATE INDEX idx_prices_product_store
  ON prices(product_id, store_id, recorded_at);
```

### ⚠️ CRITICAL ISSUE: Price History Blocked

**Problem:** The `UNIQUE(product_id, store_id)` constraint in the `prices` table prevents storing historical price data. Each product-store pair can only have ONE price, which gets overwritten on updates.

**Impact:**
- ❌ Cannot track "Milk was €1.09 last week, now €1.15"
- ❌ Cannot show price trends over time
- ❌ Cannot detect price changes or inflation

**Fix Required:**
- Remove `UNIQUE(product_id, store_id)` constraint
- Update queries to use `ORDER BY recorded_at DESC LIMIT 1` for latest prices
- Modify `addPrice()` function to INSERT instead of UPSERT

**User Decision:** APPROVED - Enable price history tracking

---

## 🏪 Data Sources

### Current Data Strategy

**6 Finnish Stores:**
1. S-Market
2. Prisma
3. K-Citymarket (real data)
4. K-Supermarket
5. Lidl
6. Alepa

**Data Quality Status:**
- ✅ **K-Citymarket:** Real prices from CSV (37 products, collected 2025-12-28)
- ⚠️ **All Others:** Sample/estimate data (not accurate)
- 📊 **Total Products:** 67 across 14 categories

### Data Collection Approaches (Attempted)

**1. Automated Web Scraping** ❌ FAILED
- **Reason:** Finnish stores block automated requests
- **Attempts:** Puppeteer + Chromium
- **Blocker:** Vercel free tier timeout (10s) too short
- **Status:** Abandoned for now

**2. Manual CSV Collection** ✅ CURRENT
- **How:** Manually collect prices, format as CSV
- **Format:** `product_name,price_cents,store_name,updated`
- **Issues:** Not sustainable, time-consuming
- **Status:** Used for K-Citymarket data

**3. Claude Browser Extension** 🔄 EXPLORING
- **Idea:** Use Claude Computer Use to navigate store websites
- **Workflow:** Generate structured CSV from browser session
- **Status:** Being tested by user

### CSV Data Format

**User's Scraped Data Format:**
```
PRODUCT NAME | PRICE (€/unit) | PRICE (€/kg)
Pirkka banaani | 0.30 | 1.69
Kurkku Suomi | 0.70 | 1.99
```

**Current Import Format (prices.csv):**
```csv
product_name,price_cents,store_name,updated
Maito,99,K-Citymarket,2025-12-28
Leipä,135,K-Citymarket,2025-12-28
```

**Note:** Need to build CSV parser for user's scraped format (pipe-delimited, euros not cents)

---

## 🎨 Design & Performance

### Monospace Terminal Aesthetic

**Core Principles:**
- Monospace fonts throughout (JetBrains Mono, Courier New)
- ASCII art decorations (╔═══╗, ┌─, →, ✓)
- Character-based layout (using `ch` units)
- Black/white/teal color scheme
- Sharp corners, no shadows
- Terminal-style borders

**DO NOT:**
- Use modern gradients or rounded corners
- Add emoji (except where explicitly requested)
- Use sans-serif fonts
- Add box shadows or fancy effects
- Break the monospace aesthetic

### Performance Metrics (MUST MAINTAIN)

**Current Performance:**
- **LCP:** 0.80s ✅ (Largest Contentful Paint)
- **CLS:** 0 ✅ (Cumulative Layout Shift)
- **INP:** 32ms ✅ (Interaction to Next Paint)
- **LCP Element:** `pre#asciiChart`

**Optimization Techniques:**
- Client-side caching (5-min TTL in localStorage)
- Pre-calculated min/max prices in `preprocessProducts()`
- LATERAL join for latest prices query
- Debounced search input (300ms)
- Vercel CDN with 1-hour cache headers

**When Making Changes:**
- Always test performance locally first
- Never add heavy dependencies
- Keep bundle size minimal
- Maintain vanilla JS approach

---

## 🔌 API Endpoints

### Public Endpoints

**GET `/api/health`**
- Health check with database status
- Returns: `{status, database: {hasPostgresUrl, state}, timestamp}`
- States: `not_initialized`, `initialized_empty`, `initialized_with_data`

**GET `/api/products`**
- All products with latest prices
- Returns: Array of products with prices object per store
- Query: Uses LATERAL join for performance
- Format: Prices in euros (converted from cents)

**GET `/api/products/:id/history`**
- Price history for specific product
- Returns: Array of {store_name, price_cents, recorded_at}
- Currently limited due to UNIQUE constraint

**GET `/api/stores`**
- List all stores
- Returns: Array of {id, name, created_at}

**POST `/api/prices`**
- Add/update single price entry
- Body: `{product_id, store_id, price_cents}`
- Uses: UPSERT with ON CONFLICT

**POST `/api/fetch-prices`**
- Update prices from sample data
- Uses fuzzy matching for product names
- Returns: Detailed results with matched/failed counts

**POST `/api/import-prices`**
- Import prices from JSON
- Body: `{prices: [{product, store, price}]}`
- Returns: Success/failure counts

**GET `/api/seed`**
- Populate database with sample data
- Creates: 6 stores, 67 products, 6 months history
- Returns: Seeding statistics

**POST `/api/cleanup`**
- Clear all database tables
- Dangerous: No confirmation required
- Returns: Deleted counts

---

## 🚀 Deployment Workflow

### Git Strategy

**Single Branch Workflow:**
- Main branch: `claude/price-comparison-app-km6Wj`
- No feature branches
- Direct commits to main branch
- User prefers simplicity over complex git workflows

**Deployment Process:**
1. Make changes locally
2. Commit to `claude/price-comparison-app-km6Wj`
3. Push to origin
4. Vercel auto-deploys (triggers on push)
5. Live in ~30 seconds

**Git Commands (with retry for network failures):**
```bash
# Push with retry (4 attempts, exponential backoff)
git push -u origin claude/price-comparison-app-km6Wj

# If push fails: retry at 2s, 4s, 8s, 16s intervals
# Branch MUST start with 'claude/' and match session ID
```

### Vercel Configuration

**vercel.json:**
```json
{
  "rewrites": [
    {"source": "/admin", "destination": "/admin.html"},
    {"source": "/scraper", "destination": "/scraper.html"}
  ],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [{"key": "Cache-Control", "value": "public, max-age=3600, must-revalidate"}]
    }
  ]
}
```

**Environment Variables (Auto-set by Vercel):**
- `POSTGRES_URL` - Database connection string
- `POSTGRES_PRISMA_URL` - Pooling URL
- `POSTGRES_URL_NON_POOLING` - Direct connection
- Plus: `POSTGRES_USER`, `POSTGRES_HOST`, `POSTGRES_PASSWORD`, `POSTGRES_DATABASE`

**Free Tier Limits:**
- 100GB bandwidth/month
- 60 hours Postgres compute/month
- 256MB database storage
- 10s function timeout
- 1024MB function memory

---

## 🔧 Development Guide

### Local Setup

```bash
# Clone and install
git clone <repo>
cd soexpensive
npm install

# Pull Vercel environment variables
vercel env pull

# Seed database
node seed.js

# Run locally
npm run dev  # or npm start

# Access at http://localhost:3000
```

### Database Operations

**Initialize (automatic on first request):**
```javascript
const db = require('./db');
await db.initDatabase();  // Creates tables and indexes
```

**Check status:**
```javascript
const hasData = await db.isDatabaseInitialized();
```

**Seed with sample data:**
```bash
node seed.js
```

**Or via API:**
```bash
curl https://soexpensive.vercel.app/api/seed
```

### Admin Panel

**URL:** https://soexpensive.jaime.win/admin

**Features:**
- Database diagnostics
- Cleanup and reseed
- Price update triggers
- Health checks
- CSV upload (planned)

**Security:** ⚠️ Currently NO authentication
- Security through obscurity (for now)
- User approved: Add auth later, not urgent

---

## 🚨 Known Issues & TODOs

### Critical Issues

1. **Price History Blocked** [HIGH PRIORITY]
   - Status: Identified, fix approved by user
   - Fix: Remove UNIQUE constraint on prices table
   - Impact: Enables trend tracking

2. **No Real Price Data** [MEDIUM]
   - Only K-Citymarket has real data (37 products)
   - Other 5 stores use sample estimates
   - Solution: Exploring Claude browser extension

3. **Admin Panel Security** [LOW]
   - No authentication required
   - Publicly accessible cleanup/seed endpoints
   - User decision: Add later

### Planned Improvements

**Data Collection:**
- [ ] Build CSV parser for pipe-delimited format
- [ ] Add bulk CSV upload to admin panel
- [ ] Support GitHub-based CSV workflow
- [ ] Explore Claude browser extension for scraping

**Database:**
- [ ] Remove UNIQUE constraint for history
- [ ] Add proper time-series queries
- [ ] Show "last verified" timestamps in UI
- [ ] Data freshness indicators (real vs sample)

**Code Quality:**
- [ ] Add tests (Jest/Vitest)
- [ ] Add ESLint + Prettier
- [ ] GitHub Actions CI/CD
- [ ] Consider TypeScript (low priority)

**Features:**
- [ ] Price change alerts
- [ ] Trend visualization (beyond ASCII)
- [ ] Category filtering in UI
- [ ] Shopping list feature
- [ ] Export to CSV

### Performance Optimizations (Already Done ✅)

- ✅ Client-side caching (5-min TTL)
- ✅ Product preprocessing for min/max prices
- ✅ Debounced search input
- ✅ LATERAL join for latest prices
- ✅ Optimized CSS (monospace, minimal JS)

---

## 📚 Important Code Patterns

### Database Query Pattern (Latest Prices)

```javascript
// Get latest price per product-store pair
const result = await sql`
  SELECT
    p.id, p.name, p.category, p.unit,
    s.name as store_name,
    pr.price_cents,
    pr.recorded_at
  FROM products p
  CROSS JOIN stores s
  LEFT JOIN LATERAL (
    SELECT product_id, store_id, price_cents, recorded_at
    FROM prices
    WHERE product_id = p.id AND store_id = s.id
    ORDER BY recorded_at DESC
    LIMIT 1  -- Latest price only
  ) pr ON true
  ORDER BY p.category, p.name, s.name
`;
```

### Fuzzy Product Matching

```javascript
// Normalize Finnish characters for matching
function normalizeProductName(name) {
  return name
    .toLowerCase()
    .replace(/[^\wåäö\s]/g, '')
    .trim();
}

// Find product match
function findProductMatch(scrapedName, dbProducts) {
  const normalized = normalizeProductName(scrapedName);
  return dbProducts.find(p => {
    const dbNorm = normalizeProductName(p.name);
    return normalized.includes(dbNorm) || dbNorm.includes(normalized);
  });
}
```

### Price Storage (Cents)

```javascript
// Always convert to cents before storing
const priceCents = Math.round(priceEuros * 100);

await sql`
  INSERT INTO prices (product_id, store_id, price_cents)
  VALUES (${productId}, ${storeId}, ${priceCents})
`;

// Convert back to euros for display
const priceEuros = parseFloat((priceCents / 100).toFixed(2));
```

### Client-Side Caching

```javascript
const CACHE_TTL = 5 * 60 * 1000; // 5 minutes
const CACHE_KEY_PRODUCTS = 'soexpensive_products';
const CACHE_KEY_TIMESTAMP = 'soexpensive_timestamp';

function isCacheValid() {
  const timestamp = localStorage.getItem(CACHE_KEY_TIMESTAMP);
  if (!timestamp) return false;
  const age = Date.now() - parseInt(timestamp);
  return age < CACHE_TTL;
}
```

---

## 🎯 User Goals & Vision

**Current Phase:** MVP / Experimentation
- Not focused on monetization yet
- Goal: Top quality result
- Public but limited audience
- Learning project with potential

**Design Philosophy:**
- Minimalist, fast, functional
- Terminal aesthetic (unique identity)
- No bloat, no heavy frameworks
- Privacy-focused (no tracking)

**Constraints:**
- Zero budget (free tier everything)
- Minimal manual work (automation preferred)
- High performance standards
- Clean, maintainable code

---

## 📖 Quick Reference for Claude

### When Starting a New Session

1. **Read this file first** (CLAUDE.md)
2. Check git status: `git status`
3. Review recent commits: `git log --oneline -10`
4. Check live site: https://soexpensive.jaime.win
5. Review current branch: `claude/price-comparison-app-km6Wj`

### Before Making Changes

1. ✅ Will this break performance metrics?
2. ✅ Does it fit the monospace aesthetic?
3. ✅ Is it compatible with Vercel free tier?
4. ✅ Have I tested locally first?
5. ✅ Did I read existing code patterns?

### When Committing

1. Use clear, descriptive commit messages
2. Test deployment URL after push
3. Verify performance metrics remain good
4. Check Vercel dashboard for build status

### When Adding Features

1. Prefer vanilla JS over frameworks
2. Keep bundle size minimal
3. Maintain terminal aesthetic
4. Add to TODO list in this file
5. Document in relevant section

---

## 📝 Changelog

**2025-12-28:**
- Added real K-Citymarket prices (37 products)
- Fixed database schema to use INTEGER price_cents
- Updated admin panel for sample price workflow
- Identified UNIQUE constraint blocking price history

**2025-12-26:**
- Deployed monospace terminal redesign
- Added database seeding infrastructure
- Created verification tools
- Fixed routing and headers in vercel.json

**Earlier:**
- Initial deployment to Vercel
- PostgreSQL migration from SQLite
- Sample data implementation
- Basic price comparison UI

---

## 🆘 Common Commands

```bash
# Development
npm start                    # Run server locally
node seed.js                 # Seed database
vercel env pull              # Pull environment variables

# Git
git status                   # Check status
git log --oneline -10        # Recent commits
git add .                    # Stage changes
git commit -m "message"      # Commit
git push -u origin claude/price-comparison-app-km6Wj  # Deploy

# Database
node seed.js                 # Local seeding
curl https://soexpensive.jaime.win/api/seed  # Remote seeding
curl https://soexpensive.jaime.win/api/health  # Check health
```

---

**Last Updated:** 2025-12-28
**Branch:** `claude/price-comparison-app-km6Wj`
**Status:** ✅ Deployed & Functional (MVP)
**Next Priority:** Fix UNIQUE constraint for price history

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jaimejim)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jaimejim)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
