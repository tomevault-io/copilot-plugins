## watchhunt-site

> > **ChronoHunt** is a watch marketplace aggregator with a comprehensive watch database. This document provides essential context for AI assistants working on the project.

# ChronoHunt Project Guide

> **ChronoHunt** is a watch marketplace aggregator with a comprehensive watch database. This document provides essential context for AI assistants working on the project.

## Quick Start

```bash
# Start PostgreSQL (Docker)
docker start watches-db

# Start development server
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/watches" npm run dev

# Open http://localhost:3000
```

## Project Structure

| Directory | Purpose |
|-----------|---------|
| `src/app/api/` | Next.js API routes (Prisma ORM) |
| `src/components/` | React components (WatchGrid, FilterSidebar, etc) |
| `src/context/` | FilterContext for filter state |
| `scripts/scraper/` | **Scraper engines and CLI** |
| `scripts/scraper/config/` | Brand configurations (100+ brands) |
| `scripts/scraper/engines/` | ShopifyScraper, HtmlScraper, SpaScraper |
| `scripts/scraper/scrapers/` | Custom brand-specific scrapers |
| `scripts/scraper/utils/` | Parsers (movement, color, specs) |
| `prisma/` | Database schema (PostgreSQL) |
| `data/` | JSON files (accessories, movements, static-filters) |
| `tests/unit/` | Unit tests (Vitest) |
| `.claude/agents/` | Claude Code agent prompts |
| `.claude/commands/` | Claude Code slash commands |

## Tech Stack

- **Frontend:** Next.js 16 + React 19 + TailwindCSS 4
- **Backend:** Next.js API Routes + Prisma ORM
- **Database:** PostgreSQL (Docker)
- **Scraping:** Custom engines (Shopify JSON, HTML, SPA, Custom)
- **Testing:** Vitest (145+ tests passing)

---

## Scraper Architecture (v2)

The scraper system uses a **configuration-driven engine pattern**. Instead of 15+ unique scraper files, brands are defined by type and config.

### Brand Configuration

Location: `scripts/scraper/config/brand-configs.ts`

```typescript
// 100+ brands configured across 4 engine types
export const brandConfigs: Record<string, BrandConfig> = {
    'san-martin':  { type: 'CUSTOM', scraperPath: './scrapers/san-martin.js', ... },
    'squale':      { type: 'SHOPIFY', baseUrl: 'https://www.gnomonwatches.com', ... },
    'helm':        { type: 'SPA', collectionUrl: '/watches.html', waitForSelector: '.wsite-multicol-col', ... },
    'wise':        { type: 'SHOPIFY', baseUrl: 'https://wisetimepiece.com', ... },
};
```

### Scraper Engines

| Engine | Use Case | Dependencies |
|--------|----------|--------------|
| `ShopifyScraper` | Shopify stores (most brands) | fetch + Cheerio |
| `HtmlScraper` | Static HTML sites | fetch + Cheerio |
| `SpaScraper` | JS-heavy sites, WooCommerce, Weebly | Puppeteer |
| `CUSTOM` | Brand-specific logic | Per-scraper |

### SPA Scraper Features

- **Selector validation**: Warns if `productItem` selector matches >100 elements
- **Anti-bot measures**: Puppeteer stealth, random delays, Cloudflare handling
- **Lazy image loading**: Checks `data-src`, `srcset` attributes
- **Auto-scroll**: Loads lazy content before parsing

### Custom Scrapers

| Scraper | Target | Notes |
|---------|--------|-------|
| `islander.ts` | longislandwatch.com | BigCommerce, parses #tab-warranty |
| `squale-liw.ts` | longislandwatch.com/squale | Same pattern as Islander |
| `san-martin.ts` | sanmartin.watch | Custom variant handling |
| `nodus.ts` | noduswatches.com | NodeX clasp detection |

### CLI Usage

```bash
# Scrape all brands
npx tsx scripts/scraper/cli.ts --verbose

# Scrape specific brand
npx tsx scripts/scraper/cli.ts --brand cronos --verbose

# Dry run (no DB writes)
npx tsx scripts/scraper/cli.ts --brand san-martin --dry-run

# Clean and rescrape (deletes existing brand data first)
npx tsx scripts/scraper/cli.ts --brand squale --clean

# Scrape by engine type
npx tsx scripts/scraper/cli.ts --type SHOPIFY
```

### GitHub Actions

Scheduled scraping runs daily at 6 AM UTC via `.github/workflows/scraper.yml`. Requires `DATABASE_URL` secret in GitHub repository settings.

---

## Database

### Schema (PostgreSQL via Prisma)

| Model | Fields | Notes |
|-------|--------|-------|
| `Watch` | brand, model, reference, specs... | Main watch records |
| `Variant` | watchId, color, imageUrl, variantUrl | Normalized from JSON blob |

### Current Stats (2026-01-15)

| Metric | Value |
|--------|-------|
| **Total Watches** | 7,220 |
| **Configured Brands** | 101 |
| **Unit Tests** | 145+ passing |

**Top Brands:**
- Zelos (472), Sugess (312), Spinnaker (280), Kuoe (248)
- Seiko (243), Citizen (241), Orient (237), Addiesdive (210)
- Sidereus (180), Henry Archer (170), San Martin (167)

### Key Commands

```bash
# Generate Prisma client
npx prisma generate

# Push schema to DB
npx prisma db push

# Open Prisma Studio
DATABASE_URL="..." npx prisma studio

# Count watches
npx tsx -e "import{PrismaClient}from'@prisma/client';new PrismaClient().watch.count().then(console.log)"

# Regenerate static filters (REQUIRED after scraping)
npx tsx scripts/generate-static-filters.ts
```

---

## Testing

Unit tests are in `tests/unit/` and run with Vitest:

```bash
# Run all tests
npm test

# Run specific test file
npx vitest run tests/unit/database-integrity.test.ts
```

### Test Files

| File | Coverage |
|------|----------|
| `parser-functions.test.ts` | All 12 parser utility functions |
| `database-integrity.test.ts` | Data validation (prices, genders, materials) |
| `parser-logic.test.ts` | Material parsing edge cases |
| `color-utils.test.ts` | Color normalization |
| `watch-images.test.ts` | Image URL validation |
| `filter-params.test.ts` | URL filter building |

---

## Filters

The frontend supports these filters (stored in `FilterContext`):

- **Brand, Movement, Caliber**
- **Case Material, Strap Material/Type, Clasp Type**
- **Dial Color** (normalized from variants)
- **Gender** (mens, womens, unisex)
- **Water Resistance** (30m, 50m, 100m, 200m, etc)
- **Watch Type** (dive, dress, pilot, field, etc)
- **Price Range, Diameter Range**

Filters use **static pre-computed data** from `data/static-filters.json` for instant loading.

---

## Special Behaviors

### Movement Parsing Priority

The `parseMovement()` function checks calibers BEFORE keywords to prevent misclassification:
```typescript
// NH38 is Automatic, even with "hand winding" in description
if (lower.match(/(nh3[456789]|4r3[456789]|sw200|...)/)) return 'Automatic';
if (lower.includes('hand winding')) return 'Manual';  // Only if no caliber found
```

### SPA maxPages Override

Some sites (like Laco) return the same products on all `?page=N` URLs. Use `maxPages: 1` in the brand config to prevent duplicate scraping.

### Authorized Dealer Scraping

For complex official sites (e.g., Squale), scrape from authorized dealers instead:
- **Gnomon Watches** (Shopify): Clean JSON, reliable
- **Long Island Watch** (BigCommerce): Custom scraper, good spec parsing

### Variant Deep Linking

Each watch variant has its own `variantUrl` for linking directly to the color option on the original site.

---

## Workflows

### Git Commit

```bash
git add -A && git commit -m "message" && git push origin main
```

### Add New Shopify Brand

1. Check if Shopify: `curl -s "https://brand.com/products.json?limit=1" | head`
2. Add to `scripts/scraper/config/brand-configs.ts`
3. Test scrape: `npx tsx scripts/scraper/cli.ts --brand new-brand --dry-run --verbose`
4. If tests pass, run without `--dry-run`
5. Regenerate filters: `npx tsx scripts/generate-static-filters.ts`

### Add New SPA Brand

1. Create config with `type: 'SPA'`, `collectionUrl`, `waitForSelector`, and `selectors`
2. Inspect site in browser DevTools to find correct CSS selectors
3. Test with dry run and examine output for >100 element warnings
4. Run full scrape and regenerate filters

### Architecture Review

Use `/arch-review` command to run an architectural review of recent changes.

### Completeness Check

Use `/arewedone` command to run a structural completeness review and prepare commit.

---

## Recent Updates (2026-01-15)

### New Brands Added
- **Wise** (Thailand, 81 watches) - Shopify
- **Helm** (USA/China, 80 watches) - SPA/Weebly platform
- **Unimatic** (Italy, 28 watches) - SPA/WooCommerce
- **Sidereus** (Ireland, 180 watches) - SPA
- **Kuoe** (Japan, 248 watches) - SPA

### Scraper Improvements
- **Selector validation warning**: SpaScraper warns if >100 elements matched
- **Stable filter sorting**: Secondary alphabetical sort prevents noisy diffs
- **URL concatenation fix**: SPA configs now use relative paths for collectionUrl
- **JSDoc for countryOfOrigin**: Clarifies it means manufacturing location, not brand HQ

### Movement Parser Fix
- Fixed NH38/NH39 movements being misclassified as Manual
- Caliber detection now runs BEFORE keyword checks
- Updated 6 Hemel watches in database

### Claude Code Integration
- Added `.claude/agents/` with architecture-reviewer and structural-completeness-reviewer
- Added `.claude/commands/` with `/arch-review` and `/arewedone` workflows

---

## Important Notes

> [!CAUTION]
> Do NOT auto-run rescrape scripts without user confirmation. Always ask before running scrapers to prevent rate limiting and data corruption.

> [!TIP]
> Check for Shopify `products.json` endpoint first - it's much faster and more reliable than DOM scraping.

> [!NOTE]
> For sites with complex SPAs, consider scraping from authorized dealers (Gnomon, Long Island Watch) instead.

> [!IMPORTANT]
> Always run `npx tsx scripts/generate-static-filters.ts` after scraping to update the filter dropdowns.

---
> Source: [sashaziv-m/watchhunt-site](https://github.com/sashaziv-m/watchhunt-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
