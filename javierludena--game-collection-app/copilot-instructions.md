## game-collection-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a full-stack Node.js web application for managing and analyzing video game collections. It provides a REST API backend with JSON file storage (ready for PostgreSQL migration) and a web interface for collection management and analysis.

**Core Features**:
- Complete CRUD operations for game collection items
- Advanced collection analysis and value estimation
- Platform-specific statistics and recommendations
- Interactive web dashboard with charts and visualizations
- RESTful API for external integrations
- Optimized for Raspberry Pi deployment

## Architecture

**Backend Structure**:
```
src/
├── config/          # Database connection and app configuration
├── models/          # Data models (Platform, CollectionItem, Category, Region) - PostgreSQL ready
├── services/        # Business logic (DataService, EbayPriceService, PriceEstimationService)
├── controllers/     # API controllers (Collection, Analysis, View) - Uses DataService
├── middleware/      # Custom middleware and validation
├── database/        # Database migrations and seeders (for future PostgreSQL migration)
└── utils/           # Helper utilities
```

**Key Technologies**:
- **Backend**: Node.js + Express.js
- **Storage**: JSON files with DataService (PostgreSQL-ready architecture)
- **Frontend**: EJS templates with vanilla JavaScript
- **Charts**: Chart.js for data visualization
- **Validation**: express-validator
- **Security**: Helmet, CORS, rate limiting

## Data Structure

**Current Storage (JSON Files)**:
- `data/collection.json` - Main collection items with embedded platform/category/region strings
- `data/platforms.json` - Platform definitions (Nintendo Switch, PlayStation, etc.)
- `data/market_prices.json` - Price database for value estimation (ready for scraping integration)

**Future PostgreSQL Schema**:
- `platforms` - Gaming platforms (PlayStation, Nintendo, etc.)
- `categories` - Item types (Games, Systems, Controllers, Accessories) 
- `regions` - Geographic regions (PAL España, NTSC USA, NTSC-J)
- `collection_items` - Main collection data with foreign key relationships
- `market_prices` - Price database for value estimation
- `publishers` & `developers` - Game publisher/developer information

**Current Data Model**:
```json
{
  "id": "uuid-string",
  "title": "Game Title",
  "platform": "Nintendo Switch", 
  "category": "Games",
  "region": "PAL España",
  "publisher": "Publisher Name",
  "developer": "Developer Name",
  "releaseType": "Official",
  "ownershipCondition": "CIB",
  "acquisitionDate": "2024-12-27",
  "estimatedValue": 50,
  "notes": "Game description"
}
```

## API Endpoints

**Collection Management** (All Working with DataService):
- `GET /api/collection` - List all items with filtering
- `POST /api/collection` - Create new collection item ✅ FIXED
- `GET /api/collection/:id` - Get item by ID
- `PUT /api/collection/:id` - Update existing item
- `DELETE /api/collection/:id` - Delete item
- `GET /api/collection/search?q=term` - Search collection

**Analysis & Statistics**:
- `GET /api/stats` - Basic collection statistics
- `GET /api/analysis/platforms` - Platform-wise analysis  
- `GET /api/analysis/timeline` - Acquisition timeline data
- `GET /api/filters` - Get all filter options (platforms, categories, regions) ✅ FIXED

**Web Views**:
- `GET /` - Dashboard with analytics
- `GET /collection` - Collection view with filters
- `GET /add` - Add new item form

## Service Layer

**DataService** (Currently Active):
- JSON file-based storage for collection and platforms
- Complete CRUD operations with UUID generation
- Built-in filtering, search, and statistics
- Platform analysis and timeline data
- Ready for PostgreSQL migration

**CollectionService** (PostgreSQL Ready):
- Model-based operations with foreign key relationships
- Advanced validation and constraints
- Ready for production database migration

**PriceEstimationService** (Active):
- eBay API integration for international price data
- Local database for Spanish/European market fallback
- No web scraping - 100% legal approach
- Platform-specific filtering and confidence scoring

**EbayPriceService** (Active):
- eBay Finding API integration for international prices
- Multi-platform support with category filtering
- Currency conversion (USD/GBP to EUR)
- Detailed filtering by condition and region

**CollectionAnalysisService** (Future Enhancement):
- Combined price estimation service integrating all sources
- Platform efficiency calculations with real market data
- Collection value analysis and trend tracking
- Strategic recommendations generation

## Key Business Logic

**Price Estimation**: 
- Multi-source data aggregation (Web scraping + eBay API)
- Weighted averaging based on source confidence
- Condition-based pricing adjustments (CIB > Boxed > Loose)
- Fallback to local price database for Spanish market

**Efficiency Scoring**: Combines game count (70%) and CIB percentage (30%) 
**Recommendations**: Platform-specific advice based on collection composition and value trends

## Development Commands

**Node.js Version**:
```bash
# This project requires Node.js 24+ for web scraping dependencies
nvm use 24                    # Switch to Node.js 24 (if using nvm)
# or use your Node.js version manager of choice
```

**Setup**:
```bash
npm install                    # Install dependencies (requires Node 24+)
# No additional setup needed - JSON files included
```

**Development**:
```bash
npm run dev                   # Start with nodemon (may need manual restart)
npm run dev:simple            # Start with plain node (easier to stop with Ctrl+C)  
npm run start                 # Production mode
npm test                      # Run test suite (if available)
npm run lint                  # Check code style
```

**Debugging Process Issues**:
```bash
# Check what's running on port 3000
netstat -ano | findstr :3000

# Kill specific process by PID (not all node processes!)
taskkill //F //PID <process_id>

# Kill only your app, not Claude! :)
```

## Environment Configuration

**Current (JSON Storage + Web Scraping)**:
- `PORT` - Server port (default: 3000)
- `NODE_ENV` - Environment mode
- **Requires Node.js 24+** for modern web scraping dependencies
- No database credentials needed

**Future (PostgreSQL Migration)**:
- `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD` - PostgreSQL connection
- `RASPBERRY_PI_MODE=true` - Enables resource optimization for Pi deployment

**Price Estimation (100% Legal)**:
- **eBay API** - Official API for international price data with `EBAY_APP_ID` (free tier: 5000 calls/day)
- **Local Database** - Curated Spanish/European market prices for popular games (fallback)

## ⚖️ Legal & Ethical Approach - No Web Scraping

**✅ Current Implementation (100% Legal):**
- ✅ **Official eBay API** - Fully authorized and legal
- ✅ **Local Database** - Curated manual data, no legal issues
- ✅ **No web scraping** - Zero legal risk
- ✅ **Terms of Service compliant** - All data sources are authorized

**🛡️ Why This Approach:**
- **Zero legal risk** - No terms of service violations
- **Stable and reliable** - APIs don't change structure frequently
- **Better performance** - No browser overhead or detection avoidance
- **Community friendly** - No server load on external websites
- **Future-proof** - Official APIs have better long-term support

**📊 Data Sources:**
1. **eBay Finding API** - Official, free tier (5000 calls/day)
2. **Local Database** - Manually curated Spanish/European market prices
3. **Community Contributions** - Future expansion through user submissions

**🔧 Testing Price Estimation:**
- `GET /api/price/estimate?title=GameTitle&platform=Platform` - Legal price estimation
- All price data comes from authorized sources only

## Raspberry Pi Deployment Notes

- Connection pool size reduced to 5 connections
- Memory usage optimized for limited resources
- Docker configuration included for easy deployment
- Supports ARM64 architecture
- Includes systemd service configuration

## Quick Start

**Setup:**
```bash
npm install                    # Install dependencies
npm run dev:simple             # Start development server (recommended)
# OR npm run dev               # With nodemon (harder to stop)
```

**Access:** http://localhost:3000

**Test adding games:** Works via web interface `/add` or API `POST /api/collection`

## Project Status

✅ **FULLY WORKING** - Modern web application with complete functionality:
- ✅ Dashboard with interactive charts and analytics
- ✅ Collection management with advanced filtering  
- ✅ Beautiful UX with glassmorphism design and dark/light themes
- ✅ Real-time search and dual view modes (list/card)
- ✅ Complete CRUD operations via API and web interface
- ✅ Game catalog autocomplete system with 100+ titles
- ✅ **Real-time price estimation** from eBay API and local database (100% legal)
- ✅ All API endpoints functional with DataService
- ✅ JSON-based storage with 5 sample games included

## Data Storage

Uses JSON files instead of PostgreSQL for simplicity:
- `data/collection.json` - Main collection data (5 sample games included)
- `data/platforms.json` - Platform definitions (10+ platforms)
- `data/catalog.json` - Game catalog database (100+ popular games for autocomplete)
- All data persisted automatically on changes
- Ready for PostgreSQL migration when needed

## Key Features

**Modern UX:**
- Responsive glassmorphism design with CSS Grid/Flexbox
- Dark/light theme toggle with localStorage persistence
- Interactive Chart.js dashboard with doughnut, bar, and line charts
- Real-time search with debouncing and keyboard shortcuts (Ctrl+K, Ctrl+N, V)
- Toast notifications and smooth animations throughout
- Loading screens and microinteractions

**Collection Management:**
- Advanced filtering by platform, category, condition with real-time updates
- Dual view modes (table/cards) with persistent user preference  
- Auto-value estimation based on comprehensive price database
- Quick add templates for Nintendo Switch, PlayStation, GameCube, Controllers
- **Game Catalog Autocomplete:** Intelligent game suggestions from 100+ title database
- **Real-time Price Estimation:** Auto-estimates prices from eBay API (legal) and local database (Spanish market)
- Form auto-save with localStorage backup
- Full CRUD operations with confirmation dialogs

**Analytics Dashboard:**
- Collection statistics (total items, games, systems, platforms, value)
- Platform efficiency scoring (70% games count + 30% CIB percentage)
- Value analysis with market price estimates in EUR
- Acquisition timeline tracking with monthly breakdowns
- Platform comparison table with CIB percentages and efficiency badges

**Price Database:**
- 100+ games with CIB and loose prices for Spanish/European market
- Covers PlayStation 1/2, GameCube, Nintendo 64, Switch, 3DS, DS, GBA
- Condition-based pricing (CIB > Boxed > Loose with percentage adjustments)
- Auto-estimation for new games based on title keywords and platform

## API Endpoints

**Collection Management:**
- `GET /collection` - Collection view with filters
- `GET /api/collection` - JSON API for items with filtering
- `POST /api/collection` - Add new item with automatic price scraping
- `PUT /api/collection/:id` - Update item
- `DELETE /api/collection/:id` - Delete item
- `GET /api/catalog/search` - Game catalog search for autocomplete

**Analytics:**
- `GET /api/stats` - Basic collection statistics
- `GET /api/analysis/platforms` - Platform analysis data
- `GET /api/analysis/timeline` - Acquisition timeline

## Views & Templates

All views are standalone (no layout inheritance issues):
- `dashboard.ejs` - Interactive analytics dashboard
- `collection.ejs` - Collection listing with filters and dual views
- `add-item.ejs` - Smart form with templates and auto-estimation
- `item-detail.ejs` - Individual item details with actions
- `error.ejs` - Elegant error pages with gaming facts

## Testing Features

**Basic Functionality**:
- `http://localhost:3000` - Dashboard with sample data charts
- `http://localhost:3000/collection` - Collection view (5 sample games)  
- `http://localhost:3000/add` - Add new item form with templates
- `http://localhost:3000/collection?search=zelda` - Search functionality
- `http://localhost:3000/collection?platform=Nintendo Switch` - Platform filter

**Price Scraping Test**:
1. Go to `/add` form
2. Start typing game title → See autocomplete suggestions
3. Select a popular game (e.g., "Zelda Breath of the Wild")
4. Leave "Estimated Value" empty and ensure "Auto-estimate" is checked ✅
5. Submit form → Watch console for scraping logs:
   ```
   🎯 Auto-estimating price for: Zelda Breath of the Wild (Nintendo Switch)
   🛒 Querying eBay API (legal)...
   💰 Final estimated price: €48 (85% confidence)
   ```
6. Success toast shows: "Game added! Price estimated: €48 (85% confidence from eBay API)"

## Data Migration

## Next Steps for Enhancement

### Immediate (Ready to Implement):
1. ✅ **Price Estimation Integration**: 
   - ✅ eBay API integration implemented (free tier - 5000 calls/day) (`EbayPriceService.js`)
   - ✅ Local database with Spanish market prices (`PriceEstimationService.js`)
   - ✅ Combined estimation service - 100% legal approach
2. **Advanced Analytics**: Implement CollectionAnalysisService with real market data
3. **Export Features**: CSV/JSON export of collection data

### Future Production:
1. **PostgreSQL Migration**: Switch from DataService to CollectionService
2. **User Authentication**: Multi-user support with login/register
3. **Image Uploads**: Game cover art and photo storage
4. **Docker Deployment**: Ready for Raspberry Pi or cloud hosting

### Price Scraping Documentation:
Complete implementation guide available in `docs/` folder:
- `docs/price-database-analysis.md` - Market analysis and strategy
- `docs/api-integration-guide.md` - Complete code implementation  
- `docs/scraping-best-practices.md` - Anti-detection and reliability
- `docs/data-migration-plan.md` - Database migration strategy

---
> Source: [javierludena/game-collection-app](https://github.com/javierludena/game-collection-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
