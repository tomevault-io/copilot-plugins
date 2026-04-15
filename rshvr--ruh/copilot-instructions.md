## ruh

> This document provides comprehensive documentation for the entire Ruh codebase, including all function-level flows, file relationships, and architectural decisions.

# Ruh - AI Product Safety Analyzer

## SOURCE OF TRUTH - Complete System Documentation

This document provides comprehensive documentation for the entire Ruh codebase, including all function-level flows, file relationships, and architectural decisions.

---

## Project Overview

**Ruh** is an AI-powered Chrome extension that analyzes product safety by detecting allergens, PFAS compounds, and other harmful substances in consumer products. The system consists of two main components:

1. **Backend**: Python FastAPI server using Claude AI Agent SDK for product analysis
2. **Extension**: Svelte 5 Chrome extension (Manifest V3) for user interface

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         USER (Chrome Browser)                    │
└─────────────────────────────────────────────────────────────────┘
                                 │
                                 ↓
┌─────────────────────────────────────────────────────────────────┐
│  EXTENSION (/extension)                                          │
│  ├─ Content Script (content.ts) - Injected into Amazon pages    │
│  ├─ Background Worker (background.ts) - Service worker          │
│  └─ Sidebar App (Sidebar.svelte) - Analysis UI in iframe        │
└─────────────────────────────────────────────────────────────────┘
                                 │
                                 ↓ HTTP POST /api/analyze
┌─────────────────────────────────────────────────────────────────┐
│  BACKEND (/backend)                                              │
│  ├─ API Layer (FastAPI) - HTTP endpoints                        │
│  ├─ Domain Layer - Business logic & harm scoring                │
│  └─ Infrastructure Layer - Claude AI, DB, scrapers              │
└─────────────────────────────────────────────────────────────────┘
                                 │
                    ┌────────────┴────────────┐
                    ↓                         ↓
            ┌──────────────┐          ┌──────────────┐
            │  Claude AI   │          │  Supabase    │
            │  (Anthropic) │          │  (PostgreSQL)│
            └──────────────┘          └──────────────┘
```

---

## Complete Function-Level Flow Diagram

### MAIN FEATURE: Product Analysis (User Click → Display Results)

```
═══════════════════════════════════════════════════════════════════
STEP 1: USER VISITS AMAZON PRODUCT PAGE
═══════════════════════════════════════════════════════════════════

[User navigates to Amazon product page]
  ↓
📄 extension/src/content/content.ts::init()
  ├─ Calls: isAmazonProductPage(window.location.href) → boolean
  ├─ Stores: currentProductUrl = window.location.href
  ├─ Sets up: chrome.runtime.onMessage listener
  └─ Calls: startAnalysis()

📄 extension/src/content/content.ts::startAnalysis()
  ├─ Reads: import.meta.env.VITE_API_BASE_URL
  ├─ Reads: import.meta.env.VITE_API_KEY
  ├─ Makes: fetch(API_BASE_URL + '/api/analyze', {
  │         method: 'POST',
  │         headers: { Authorization: `Bearer ${API_KEY}` },
  │         body: JSON.stringify({ product_url: currentProductUrl })
  │       })
  ├─ Stores: state.data = await response.json()
  ├─ Extracts: harmScore = state.data.analysis.product_analysis.overall_score
  └─ Calls: injectTriggerButton(harmScore)

📄 extension/src/content/content.ts::injectTriggerButton(score: number)
  ├─ Creates: <button> element with donut chart SVG
  ├─ Attaches: click event → openSidebar()
  ├─ Injects: Button into page DOM (above product title)
  └─ Returns: void

═══════════════════════════════════════════════════════════════════
STEP 2: BACKEND API PROCESSING (Concurrent with Step 1)
═══════════════════════════════════════════════════════════════════

[HTTP POST /api/analyze arrives at FastAPI]
  ↓
📄 backend/src/api/main.py::app (FastAPI application)
  ├─ Middleware: CORS handler
  ├─ Routes: /api/health → health.router
  └─ Routes: /api/analyze → analyze.router

📄 backend/src/api/routes/analyze.py::analyze_product(
      request: AnalysisRequest,
      credentials: HTTPAuthorizationCredentials
    ) → AnalysisResponse
  │
  ├─ Calls: verify_api_key(credentials) → None | raises HTTPException
  │   └─ 📄 backend/src/api/auth.py::verify_api_key(credentials)
  │       ├─ Reads: settings.api_key
  │       ├─ Compares: credentials.credentials == settings.api_key
  │       └─ Returns: None OR raises HTTPException(401)
  │
  ├─ Initializes: db = DatabaseService()
  ├─ Generates: url_hash = db.generate_url_hash(request.product_url)
  │   └─ 📄 backend/src/infrastructure/database.py::generate_url_hash(url: str) → str
  │       └─ Returns: hashlib.sha256(url.encode()).hexdigest()
  │
  ├─ Checks Cache: cached = db.get_cached_analysis(url_hash)
  │   └─ 📄 backend/src/infrastructure/database.py::get_cached_analysis(url_hash: str) → Dict | None
  │       ├─ Calls: supabase.table('product_analyses').select('*').eq('url_hash', url_hash).execute()
  │       └─ Returns: data[0] if exists else None
  │
  ├─ IF CACHE HIT:
  │   └─ Returns: AnalysisResponse(cached data)
  │
  ├─ IF CACHE MISS - SCRAPING PATH:
  │   │
  │   ├─ Initializes: scraper_service = ProductScraperService()
  │   ├─ Calls: scraped = scraper_service.try_scrape(product_url)
  │   │   └─ 📄 backend/src/infrastructure/product_scraper.py::try_scrape(url: str) → ScrapedProduct | None
  │   │       ├─ Calls: scraper = ScraperFactory.get_scraper(url)
  │   │       │   └─ 📄 backend/src/infrastructure/scrapers/factory.py::get_scraper(url: str) → BaseScraper | None
  │   │       │       ├─ Checks: if 'amazon.com' in url
  │   │       │       └─ Returns: AmazonScraper() OR None
  │   │       │
  │   │       ├─ IF scraper exists:
  │   │       │   └─ Calls: scraper.scrape(url)
  │   │       │       └─ 📄 backend/src/infrastructure/scrapers/amazon.py::scrape(url: str) → ScrapedProduct
  │   │       │           ├─ Makes: httpx.AsyncClient().get(url, headers={...})
  │   │       │           ├─ Parses: soup = BeautifulSoup(html, 'lxml')
  │   │       │           ├─ Extracts: title = soup.select_one('#productTitle').text
  │   │       │           ├─ Extracts: brand = soup.select_one('#bylineInfo').text
  │   │       │           ├─ Extracts: ingredients = find sections with keywords
  │   │       │           ├─ Calculates: confidence score (0-1)
  │   │       │           └─ Returns: ScrapedProduct(raw_html_product, confidence)
  │   │       │
  │   │       └─ Returns: ScrapedProduct OR None (if failed)
  │   │
  │   ├─ IF scraped AND confidence > 0.3:
  │   │   │
  │   │   ├─ Calls: product_data = claude_query.extract_product_data(scraped.raw_html_product)
  │   │   │   └─ 📄 backend/src/infrastructure/claude_query.py::extract_product_data(html: str) → Dict
  │   │   │       ├─ Initializes: client = Anthropic(api_key=settings.anthropic_api_key)
  │   │   │       ├─ Calls: response = client.messages.create(
  │   │   │       │         model="claude-sonnet-4-5-20250929",
  │   │   │       │         messages=[{role: "user", content: prompt + html}],
  │   │   │       │         max_tokens=4096
  │   │   │       │       )
  │   │   │       ├─ Parses: JSON from response.content[0].text
  │   │   │       └─ Returns: {product_name, brand, ingredients, materials, ...}
  │   │   │
  │   │   └─ Calls: analysis_data = claude_agent.analyze_extracted_product(product_data, url)
  │   │       └─ 📄 backend/src/infrastructure/claude_agent.py::analyze_extracted_product(
  │   │                 product_data: Dict, url: str
  │   │             ) → Dict
  │   │           ├─ Initializes: client = Anthropic(api_key=settings.anthropic_api_key)
  │   │           ├─ Defines: tools = [web_search_tool]
  │   │           ├─ Calls: response = client.messages.create(
  │   │           │         model="claude-sonnet-4-5-20250929",
  │   │           │         messages=[{role: "user", content: prompt + product_data}],
  │   │           │         tools=[web_search],
  │   │           │         max_tokens=8192
  │   │           │       )
  │   │           ├─ Handles: Tool use loop (web_search requests)
  │   │           │   └─ For each web_search:
  │   │           │       ├─ Makes: httpx.get('https://google.serper.dev/search', ...)
  │   │           │       └─ Returns: search results to Claude
  │   │           ├─ Parses: Final JSON response
  │   │           └─ Returns: {allergens_detected, pfas_detected, other_concerns, confidence}
  │   │
  │   ├─ IF scraping FAILED OR low confidence:
  │   │   └─ Calls: analysis_data = claude_agent.analyze_product(product_url)
  │   │       └─ 📄 backend/src/infrastructure/claude_agent.py::analyze_product(url: str) → Dict
  │   │           ├─ Similar to analyze_extracted_product but with web_fetch tool
  │   │           ├─ Claude fetches the product page itself
  │   │           └─ Returns: analysis_data
  │   │
  │   ├─ Calculates: harm_score = HarmScoreCalculator.calculate(analysis_data)
  │   │   └─ 📄 backend/src/domain/harm_calculator.py::HarmScoreCalculator.calculate(
  │   │             analysis_data: Dict[str, Any]
  │   │         ) → int
  │   │       ├─ Initializes: total_score = 0
  │   │       ├─ For allergens_detected:
  │   │       │   └─ Adds: severity points (5-30 per allergen)
  │   │       ├─ For pfas_detected:
  │   │       │   └─ Adds: 40 points per PFAS compound
  │   │       ├─ For other_concerns:
  │   │       │   └─ Adds: points based on toxicity level
  │   │       ├─ Applies: category multipliers (pesticides, cleaners)
  │   │       ├─ Caps: max(min(total_score, 100), 0)
  │   │       └─ Returns: harm_score (0-100)
  │   │
  │   ├─ Builds: analysis = ProductAnalysis(
  │   │           product_name=...,
  │   │           overall_score=100 - harm_score,
  │   │           allergens_detected=...,
  │   │           pfas_detected=...,
  │   │           ...
  │   │         )
  │   │
  │   ├─ Stores: db.store_analysis(url_hash, product_url, analysis_response)
  │   │   └─ 📄 backend/src/infrastructure/database.py::store_analysis(
  │   │             url_hash: str, url: str, response: AnalysisResponse
  │   │         ) → bool
  │   │       ├─ Prepares: db_data = {url_hash, product_url, product_name, ...}
  │   │       ├─ Calls: supabase.table('product_analyses').upsert(db_data).execute()
  │   │       └─ Returns: True OR False (on error)
  │   │
  │   └─ Returns: AnalysisResponse(
  │               analysis=analysis,
  │               alternatives=[],
  │               cached=False,
  │               url_hash=url_hash
  │             )

═══════════════════════════════════════════════════════════════════
STEP 3: USER CLICKS TRIGGER BUTTON → SIDEBAR OPENS
═══════════════════════════════════════════════════════════════════

[User clicks floating donut chart button]
  ↓
📄 extension/src/content/content.ts::openSidebar()
  ├─ Creates: iframe = document.createElement('iframe')
  ├─ Sets: iframe.src = chrome.runtime.getURL('src/sidebar.html')
  ├─ Injects: document.body.appendChild(iframe)
  ├─ Waits: iframe onload event
  ├─ Sends: iframe.contentWindow.postMessage({
  │          type: 'ANALYSIS_DATA',
  │          data: state.data
  │        }, '*')
  └─ Hides: trigger button (display: none)

📄 extension/src/sidebar.html (loaded in iframe)
  └─ Loads: <script type="module" src="/sidebar.js"></script>

📄 extension/src/sidebar.ts::initApp()
  ├─ Gets: app = document.getElementById('app')
  └─ Calls: mount(Sidebar, { target: app })

📄 extension/src/Sidebar.svelte::onMount()
  ├─ Sets up: chrome.runtime.onMessage listener
  ├─ Sets up: window.addEventListener('message', handleMessage)
  └─ Defines: handleMessage(event: MessageEvent)

📄 extension/src/Sidebar.svelte::handleMessage(event)
  ├─ Checks: if (event.data.type === 'ANALYSIS_DATA')
  ├─ Extracts: data = event.data.data
  ├─ Sets: analysis = data (reactive state)
  ├─ Sets: loading = false
  ├─ Calls: cache.set(productUrl, data)
  │   └─ 📄 extension/src/lib/cache.ts::set(key: string, value: AnalysisResponse)
  │       ├─ Opens: db = await openDB('eject-cache', 1)
  │       ├─ Stores: db.put('analyses', { key, value, timestamp })
  │       └─ Returns: void
  └─ Renders: <Sidebar {analysis} />

📄 extension/src/components/Sidebar.svelte (UI Component)
  ├─ Receives: analysis prop (AnalysisResponse)
  ├─ Extracts: productAnalysis = analysis.analysis.product_analysis
  ├─ Computes: harmScore = getHarmScore(productAnalysis)
  │   └─ 📄 extension/src/lib/utils.ts::getHarmScore(analysis: ProductAnalysis) → number
  │       └─ Returns: 100 - analysis.overall_score
  │
  ├─ Computes: riskLevel = getRiskLevel(harmScore)
  │   └─ 📄 extension/src/lib/utils.ts::getRiskLevel(score: number) → RiskLevel
  │       ├─ Returns: 'low' if score < 30
  │       ├─ Returns: 'medium' if score < 60
  │       └─ Returns: 'high' if score >= 60
  │
  ├─ Computes: riskClass = getRiskClass(riskLevel)
  │   └─ 📄 extension/src/lib/utils.ts::getRiskClass(level: RiskLevel) → string
  │       └─ Returns: CSS class name ('risk-low' | 'risk-medium' | 'risk-high')
  │
  ├─ Renders: Donut chart SVG with harmScore
  ├─ Renders: Product name and brand
  ├─ Renders: Allergens list (if any)
  ├─ Renders: PFAS list (if any)
  ├─ Renders: Other concerns list (if any)
  └─ Renders: Confidence score and timestamp
```

---

## File-Level Import Graph

### Backend Dependencies

```
run.py
  └─ src.infrastructure.config.settings

src/api/main.py
  ├─ src.infrastructure.config.settings
  └─ src.api.routes.{health, analyze}

src/api/auth.py
  └─ src.infrastructure.config.settings

src/api/routes/health.py
  └─ (no internal imports)

src/api/routes/analyze.py
  ├─ src.domain.models.*
  ├─ src.domain.harm_calculator.HarmScoreCalculator
  ├─ src.infrastructure.claude_agent.ProductSafetyAgent
  ├─ src.infrastructure.product_scraper.ProductScraperService
  ├─ src.infrastructure.claude_query.ClaudeQueryService
  ├─ src.infrastructure.database.DatabaseService
  └─ src.api.auth.verify_api_key

src/domain/models.py
  └─ (no internal imports - pure Pydantic models)

src/domain/harm_calculator.py
  └─ (no internal imports - pure logic)

src/infrastructure/config.py
  └─ (no internal imports - Pydantic settings)

src/infrastructure/database.py
  └─ src.infrastructure.config.settings

src/infrastructure/claude_agent.py
  └─ src.infrastructure.config.settings

src/infrastructure/claude_query.py
  ├─ src.infrastructure.config.settings
  └─ src.domain.models.ScrapedProduct

src/infrastructure/product_scraper.py
  ├─ src.infrastructure.scrapers.factory.ScraperFactory
  └─ src.domain.models.ScrapedProduct

src/infrastructure/scrapers/factory.py
  ├─ src.infrastructure.scrapers.base.BaseScraper
  └─ src.infrastructure.scrapers.amazon.AmazonScraper

src/infrastructure/scrapers/base.py
  └─ (abstract base - no imports)

src/infrastructure/scrapers/amazon.py
  ├─ src.infrastructure.scrapers.base.BaseScraper
  └─ src.domain.models.ScrapedProduct
```

### Extension Dependencies

```
sidebar.ts
  ├─ ./app.css
  └─ ./Sidebar.svelte

Sidebar.svelte
  ├─ ./components/Sidebar.svelte
  ├─ ./lib/api.{api}
  ├─ ./lib/cache.{cache}
  └─ ./types.{AnalysisResponse}

components/Sidebar.svelte
  ├─ @/types.*
  └─ @/lib/utils.*

content/content.ts
  └─ (no file imports - uses chrome API and import.meta.env)

background/background.ts
  └─ (no file imports - uses chrome API)

lib/api.ts
  └─ @/types.{AnalysisResponse}

lib/cache.ts
  └─ @/types.{AnalysisResponse, CachedAnalysis}

lib/utils.ts
  └─ @/types.{ProductAnalysis, RiskLevel}

types/index.ts
  └─ (no imports - pure type definitions)
```

---

## Cross-Directory Relationships

### Backend: Clean Architecture Pattern

```
API Layer (src/api/)
  ↓ calls
Domain Layer (src/domain/)
  ↓ uses
Infrastructure Layer (src/infrastructure/)
  ↓ calls
External Services (Anthropic, Supabase, Web)
```

**Key Dependency Rule**: Higher layers depend on lower layers. Infrastructure is called by Domain/API but never calls them back (dependency inversion).

### Extension: Component-Based Architecture

```
Content Script (content/)
  ↓ creates
Sidebar App (Sidebar.svelte)
  ↓ uses
Libraries (lib/)
  ↓ uses
Types (types/)
```

**Key Pattern**: Content script is isolated (no imports) to avoid bundling issues. Sidebar app handles all state management and API communication.

---

## Bloat Identification

### ⚠️ BLOAT: Legacy Migrations

**Location**: `/backend/migrations/`

**Evidence**:
- Contains outdated SQL files: `001_create_tables.sql`, `002_seed_knowledge_base.sql`
- Superseded by `/backend/supabase/migrations/`
- Old schema missing tables (toxic_substances) and columns

**Impact**: None (not used in production)

### ⚠️ BLOAT: Unused Application Layer

**Location**: `/backend/src/application/`

**Evidence**:
- Directory contains only empty `__init__.py`
- No code implements application layer pattern
- Business logic exists in `domain/` and `infrastructure/`

**Impact**: None (empty directory)

### ⚠️ DEVELOPMENT SCAFFOLDING: Empty Test Directories

**Locations**:
- `/backend/tests/unit/` (empty except `__init__.py`)
- `/backend/tests/integration/` (empty except `__init__.py`)

**Evidence**:
- Only E2E tests implemented (`tests/e2e/test_product_analysis.py`)
- Unit and integration test directories created but unused

**Impact**: None (future test scaffolding)

---

## Subdirectory Documentation

### Backend
- [Backend Overview](./backend/CLAUDE.md) - FastAPI server, clean architecture, Claude AI integration

### Extension
- [Extension Overview](./extension/CLAUDE.md) - Svelte 5 Chrome extension, Manifest V3, UI components

---

## Essential Files Summary

**Total Source Files**: 52
- **Backend**: 21 Python files
- **Extension**: 12 TypeScript/Svelte files
- **Database**: 4 SQL migration files (active)
- **Tests**: 4 test files
- **Bloat**: 3 files (5.8%)

**Codebase Health**: 94.2% of files are actively used and essential for the product to function.

---

## Key Technologies

### Backend Stack
- **Framework**: FastAPI (Python)
- **AI**: Anthropic Claude Sonnet 4.5 with Agent SDK
- **Database**: Supabase (PostgreSQL)
- **Scraping**: httpx + BeautifulSoup4
- **Testing**: pytest

### Extension Stack
- **Framework**: Svelte 5
- **Language**: TypeScript
- **Build**: Vite
- **Styling**: Tailwind CSS
- **Storage**: IndexedDB (idb library)
- **Manifest**: Chrome Extension Manifest V3

---

Last Updated: 2025-11-18

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RSHVR)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/RSHVR)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
