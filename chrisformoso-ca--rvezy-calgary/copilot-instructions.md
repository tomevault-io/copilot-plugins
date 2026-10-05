## rvezy-calgary

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an RVezy market analysis system for analyzing RV rental listings in the Calgary area. The project extracts and structures data from PandaScraper CSV exports to provide competitive intelligence, pricing optimization, and investment recommendations.

## Key Business Context

- User operates a 2021 Travel Trailer (Super-lite/Conquest 197BH) on RVezy at $97/night
- 60/40 profit split with RV owner
- Looking to optimize pricing and identify investment opportunities
- Focus on Calgary and surrounding Alberta markets

## Commands

### Run Analysis Scripts
```bash
# Extract data from raw CSV to SQLite database
python3 scripts/extract_rvezy_data.py

# Generate comprehensive dashboard with all visualizations
python3 scripts/generate_comprehensive_dashboard.py

# Analyze multi-RV owners and rental businesses
python3 scripts/analyze_multi_owners.py

# Get pricing recommendations for current listing
python3 scripts/pricing_optimizer.py

# Analyze investment opportunities
python3 scripts/investment_analyzer.py

# Seasonal revenue analysis
python3 scripts/seasonal_revenue_analyzer.py

# Add-on and amenity premium analysis
python3 scripts/addon_amenity_analyzer.py

# Top performer analysis
python3 scripts/top_performer_analyzer.py

# Query the database for custom analysis
python3 scripts/query_database.py
```

### Dashboard and Visualization
```bash
# Generate dashboard data and HTML
python3 scripts/generate_comprehensive_dashboard.py

# Serve dashboard locally on port 8080
cd output && python3 -m http.server 8080

# Access dashboard at: http://localhost:8080/dashboard_comprehensive.html
```

### Database Operations
```bash
# Access SQLite database directly
sqlite3 data/processed/rvezy_listings.db

# Export specific queries
sqlite3 data/processed/rvezy_listings.db "SELECT * FROM listings WHERE location_city='Calgary'" > calgary_data.csv

# Common queries
sqlite3 data/processed/rvezy_listings.db ".tables"  # List all tables
sqlite3 data/processed/rvezy_listings.db ".schema listings"  # Show table structure
```

## Architecture

### Data Pipeline
1. **Input**: PandaScraper CSV (`data/raw/RVEzy Listings Text 06302025.csv`)
   - Contains URL, Title, Description, Content fields
   - Content field has unstructured text requiring regex extraction

2. **Processing**: `scripts/extract_rvezy_data.py`
   - RVezyDataExtractor class handles ETL
   - Regex patterns extract 30+ data points per listing
   - Creates normalized SQLite database

3. **Storage**: SQLite database with 7 normalized tables
   - `listings` - Main RV information (30 fields)
   - `hosts` - Owner details and ratings
   - `pricing` - Discount structures
   - `amenities` - Feature list
   - `listing_amenities` - Many-to-many junction
   - `addons` - Optional services
   - `beds` - Sleeping configurations

### Analysis Scripts
- Each script is standalone and queries the SQLite database
- Scripts output both console reports and JSON/CSV exports
- Designed for iterative analysis and reporting

### Dashboard System
- `generate_comprehensive_dashboard.py` creates both data (`comprehensive_dashboard_data.json`) and HTML (`dashboard_comprehensive.html`)
- Dashboard includes 7 tabs: Market Overview, Price Analysis, Specs Analysis, Add-Ons Revenue, Top Performers, Investment Analysis, Host Intelligence
- Interactive features: price threshold slider, sortable tables, clickable charts, **ROI calculator**
- Responsive design with real-time filtering capabilities
- **NEW: ROI Calculator** in Investment tab for evaluating potential RV purchases with real market data

## Database Schema

Key relationships:
- `hosts` (1) -> (many) `listings`
- `listings` (many) <-> (many) `amenities` via `listing_amenities`
- `listings` (1) -> (many) `pricing`, `addons`, `beds`

Important fields:
- `listings.base_price` - Nightly rate in CAD
- `listings.location_city` - Primary geographic filter
- `listings.rv_type` - Travel Trailer, Class A/B/C, Fifth Wheel, Toy Hauler
- `listings.overall_rating` - Average review score
- `listings.num_reviews` - Proxy for booking frequency

## Data Insights

Current dataset (June 2025):
- 489 listings with pricing data
- 334 Travel Trailers (68%)
- 93 Class C motorhomes
- 32 Campervans
- 22 Fifth Wheels
- 8 Tent Trailers (competitive budget segment)
- Price range: $75-$489/night
- 30+ hosts operate multiple RVs (rental businesses)
- Average 120-day summer revenue: $11,640-$41,280 depending on RV type

## Common Analysis Patterns

When analyzing pricing:
- Filter by `location_city` for geographic relevance
- Group by `rv_type` for category comparisons
- Join with `hosts` table for superhost premium analysis
- Use `num_reviews` as demand proxy

When identifying opportunities:
- Look for underserved size/capacity combinations
- Analyze multi-owner portfolios for business models
- Compare feature premiums via amenities joins
- Calculate ROI using estimated purchase prices

## Key Business Metrics

- **Success Rate**: Listings with 20+ reviews (proxy for consistent bookings)
- **Summer Revenue**: Base price × 120 days (typical rental season)
- **ROI Calculation**: (Summer Revenue × 70% operating margin) ÷ Used Purchase Price
- **Your Position**: $97/night is 25th percentile among Travel Trailers (only 85 cheaper out of 334)

## Dashboard Features

### Price Threshold Analysis
- Interactive slider to explore price segments
- Bubble chart shows price vs demand (reviews)
- Identifies premium pricing outliers

### Investment Opportunity Matrix
- Visualizes market size vs price range opportunities
- Color-coded by success rate
- Bubble size represents average demand

### Multi-Owner Intelligence
- Fleet composition and pricing strategies
- Summer revenue calculations (120 days)
- Expandable portfolio views

### ROI Calculator (NEW)
- Interactive calculator in Investment Analysis tab
- Input: Purchase price, RV specs, storage costs, insurance
- Calculates ROI based on category averages and actual market data
- Shows similar listings for comparison
- Revenue projections for 50%, 70%, 90% occupancy scenarios
- Full cost breakdown including RVezy fees, maintenance, cleaning
- Your 60/40 profit split factored in

## Dashboard Deployment

### Viewing Locally
```bash
cd output && python3 -m http.server 8080
# Access at http://localhost:8080/dashboard_comprehensive.html
```

### Deploying to GitHub Pages
```bash
# Copy files for deployment
cp output/dashboard_comprehensive.html output/index.html

# Option 1: Deploy to gh-pages branch
git checkout -b gh-pages
cp output/index.html output/comprehensive_dashboard_data.json .
git add index.html comprehensive_dashboard_data.json
git commit -m "Deploy dashboard"
git push origin gh-pages

# Option 2: Deploy from docs folder
mkdir -p docs
cp output/index.html output/comprehensive_dashboard_data.json docs/
git add docs/
git commit -m "Add dashboard to docs"
git push origin main
```

Access at: https://chrisformoso-ca.github.io/rvezy-calgary/

### Other Deployment Options
- **Netlify**: Drag and drop the output folder at app.netlify.com/drop
- **Local Network**: Host on your Proxmox server with nginx
- **PWA**: Can be made into an offline-capable Progressive Web App

See DEPLOYMENT.md for detailed deployment instructions.

---
> Source: [chrisformoso-ca/rvezy-calgary](https://github.com/chrisformoso-ca/rvezy-calgary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
