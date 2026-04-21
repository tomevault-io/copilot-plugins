## csi-to-icc

> This is a web application that helps civil engineers and construction professionals map CSI (Construction Specifications Institute) MasterFormat codes to ICC (International Code Council) building code sections.

# CSI to ICC Code Mapping - Project Context

## Project Overview

This is a web application that helps civil engineers and construction professionals map CSI (Construction Specifications Institute) MasterFormat codes to ICC (International Code Council) building code sections.

**Problem**: Converting CSI specification codes to relevant ICC compliance codes is a manual, time-consuming process requiring extensive cross-referencing.

**Solution**: A web application where users can:
- Enter a CSI code
- Filter by state and year
- Get relevant ICC code sections with direct links to official documentation

## Tech Stack

### Backend
- **Language**: Python 3.13+
- **Framework**: FastAPI (REST API)
- **Database**: PostgreSQL
- **ORM**: SQLAlchemy 2.0
- **Migrations**: Alembic
- **Validation**: Pydantic v2

### Frontend
- **Framework**: Next.js 15 (App Router)
- **Language**: TypeScript
- **UI Library**: React 19
- **Styling**: Tailwind CSS 3.4.1
- **HTTP Client**: fetch API

## Project Structure

```
csi-to-icc/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py              # FastAPI application entry point
│   │   ├── database.py          # Database connection & session management
│   │   ├── models.py            # SQLAlchemy ORM models (5 tables)
│   │   ├── schemas.py           # Pydantic schemas for API validation
│   │   ├── crud.py              # Database CRUD operations
│   │   └── routers/
│   │       ├── __init__.py
│   │       └── codes.py         # API endpoints (21 endpoints)
│   ├── scripts/
│   │   ├── ipc_scraper_all.py   # Selenium-based ICC code scraper
│   │   ├── populate_icc_data.py # Database population from JSON
│   │   ├── populate_all_chapters.py  # Batch chapter processor
│   │   ├── extracted_data/
│   │   │   └── ipc_2018/        # 13 chapters of IPC 2018 (JSON)
│   │   ├── icc_credentials.env  # ICC login credentials (not in git)
│   │   └── README.md            # Scraper setup & usage guide
│   ├── alembic/                 # Database migrations
│   │   └── versions/
│   │       └── bc374c7d93a2_initial_schema.py
│   ├── venv/                    # Python virtual environment (not in git)
│   ├── requirements.txt         # Python dependencies
│   ├── add_sample_data.py       # Test data population script
│   ├── .env.example             # Environment variables template
│   └── .env                     # Actual environment variables (not in git)
├── frontend/
│   ├── app/
│   │   ├── page.tsx             # Main search interface (288 lines)
│   │   ├── layout.tsx           # Root layout
│   │   └── globals.css          # Tailwind configuration
│   ├── package.json             # Next.js 15, React 19, Tailwind 3.4
│   └── tailwind.config.ts       # Custom engineering-themed colors
├── docker-compose.yml           # Production Docker setup
├── docker-compose.dev.yml       # Development Docker setup
├── Makefile                     # Development commands
├── DOCKER.md                    # Docker deployment guide
├── .gitignore
├── CLAUDE.md                    # This file
└── README.md                    # User-facing documentation
```

## Frontend Implementation

### Search Interface (`app/page.tsx`)
**Features**:
- 3-field search form:
  - CSI Code input (required, e.g., "03 30 00")
  - State filter (optional, 2-letter format like "CO", "CA")
  - Year dropdown (optional, 2024/2021/2018)
- Real-time validation and error handling
- Loading states with spinner animation
- Empty state with example code suggestion

**Results Display**:
- CSI code information card (code, division, title, description)
- List of related ICC code sections
- Each section shows:
  - Document code and year (e.g., "IPC 2018")
  - Chapter number
  - Section number and title
  - Description text
  - Direct link to codes.iccsafe.org

**Design**:
- Custom Tailwind color palette:
  - Primary: Sky blue (#0ea5e9) - engineering/blueprint inspired
  - Accent: Construction orange (#f97316)
  - Neutral: Slate grays for professional look
- Responsive layout (mobile-first, tablet, desktop)
- Card-based design with shadows and borders
- Custom scrollbar styling
- Hover states and interactive elements

**API Integration**:
- Connects to backend at `http://localhost:8000`
- Uses fetch API for POST requests to `/api/search`
- Error handling for network failures and API errors
- Type-safe TypeScript interfaces for API responses

## Database Schema

### Tables

1. **csi_codes** - CSI MasterFormat codes
   - `id` (PK)
   - `code` (unique, e.g., "03 30 00")
   - `division` (integer)
   - `title` (e.g., "Cast-in-Place Concrete")
   - `description`

2. **icc_documents** - ICC code documents (IBC, IRC, etc.)
   - `id` (PK)
   - `code` (e.g., "IBC", "IRC")
   - `year` (e.g., 2021, 2024)
   - `title`
   - `state` (nullable, for state-specific codes)
   - `base_url` (link to codes.iccsafe.org)

3. **icc_sections** - Specific sections within ICC documents
   - `id` (PK)
   - `document_id` (FK to icc_documents)
   - `section_number` (e.g., "1905.2")
   - `title`
   - `chapter`
   - `url` (direct link to section)
   - `description`

4. **csi_icc_mappings** - Many-to-many relationship
   - `id` (PK)
   - `csi_code_id` (FK)
   - `icc_section_id` (FK)
   - `relevance` (primary/secondary/reference)
   - `notes`
   - `created_at`

5. **state_amendments** - State-specific code amendments
   - `id` (PK)
   - `icc_section_id` (FK)
   - `state`
   - `amendment_text`
   - `effective_date`

## API Endpoints

### CSI Codes
- `GET /api/csi-codes` - List all CSI codes
- `GET /api/csi-codes/{code}` - Get specific CSI code
- `POST /api/csi-codes` - Create new CSI code
- `GET /api/csi-codes/{code}/icc-sections` - Get ICC sections for a CSI code (supports filters)

### ICC Documents
- `GET /api/icc-documents` - List ICC documents (with filters)
- `GET /api/icc-documents/{id}` - Get specific document
- `POST /api/icc-documents` - Create new document

### ICC Sections
- `GET /api/icc-sections/{id}` - Get specific section
- `POST /api/icc-sections` - Create new section
- `GET /api/icc-sections/{id}/amendments` - Get state amendments

### Mappings
- `POST /api/mappings` - Create CSI-to-ICC mapping

### Search
- `POST /api/search` - Main search endpoint
  - Body: `{ "csi_code": "03 30 00", "state": "CO", "year": 2024 }`

## Development Approach

### Phase 1: MVP (Mostly Complete)
✅ Backend API structure (21 endpoints)
✅ Database schema (5 tables with migrations)
✅ Basic CRUD operations
✅ Frontend UI (search interface with results display)
✅ IPC 2018 data extraction (13 chapters, 200+ sections)
✅ Docker deployment setup
⏳ CSI-to-ICC mapping population (sample data only)
⏳ Additional ICC codes (IBC, IRC, IECC, etc.)

### Phase 2: Data Population (In Progress)
- ✅ IPC 2018 scraping complete (13 chapters extracted, 1,087 sections loaded)
- ✅ CSI MasterFormat 2016 codes loaded (8,778 codes across all hierarchy levels)
- ⏳ Create CSI-to-IPC mappings (with expert verification)
- ⏳ Scrape additional ICC codes (IBC, IRC, IECC)
- ⏳ Add state-specific amendments
- Use LLMs to suggest mappings (with expert verification)
- Focus on most-used divisions (concrete, steel, wood, plumbing, etc.)

### Phase 3: Enhanced Features
- Natural language search
- User accounts and saved searches
- Community contributions (with moderation)
- Export functionality (CSV, PDF reports)
- Full-text search optimization
- Caching layer (Redis)
- Analytics dashboard

## Data Extraction & Population

### IPC 2018 (Completed)
**Status**: ✅ Full extraction complete

**Scraped Data**:
- 13 chapters from 2018 International Plumbing Code
- 200+ code sections with descriptions
- Direct URLs to codes.iccsafe.org for each section
- Stored as JSON files in `backend/scripts/extracted_data/ipc_2018/`

**Chapters Extracted**:
1. Scope and Administration
2. (Definitions - skipped by design)
3. General Requirements
4. Definitions
5. Materials
6. Water Supply and Distribution (largest, 1606 lines)
7. Sanitary Drainage
8. Vents
9. Inspection, Testing, and Procedures
10. Water Heaters
11. Energy Efficiency
12. Exterior Walls
13. Roof Assemblies
14. Referenced Standards

**Scraping Approach**:
- Selenium WebDriver with headless Chrome
- Parses codes.iccsafe.org JavaScript-rendered content
- 10-second wait times for dynamic content loading
- Detailed logging per chapter
- Handles authentication (requires ICC account)
- No login required (public access mode used)

**Scripts**:
- `ipc_scraper_all.py` - Main scraper (multi-chapter)
- `populate_icc_data.py` - Database population from JSON
- `populate_all_chapters.py` - Batch processor
- `backend/scripts/README.md` - 347-line setup guide

### CSI MasterFormat 2016 (Completed)
**Status**: ✅ Full population complete

**Data Source**:
- GitHub repository: [outer-labs/masterformat-json](https://github.com/outer-labs/masterformat-json)
- License: MIT (free to use)
- Version: MasterFormat 2016 Edition
- Supplemented with scraped data from CSI widget for intermediate hierarchy levels

**Loaded Data**:
- **8,778 unique codes** across all hierarchy levels:
  - Division level (35 codes): 00 00 00, 03 00 00, 22 00 00, etc.
  - Subdivision level (~400 codes): 03 30 00, 22 07 00, etc.
  - Detail/leaf level (~8,300 codes): 03 31 13, 22 07 19, etc.

**Data Structure**:
- Three-level hierarchy matching official CSI MasterFormat organization
- Each code includes: code number, division, title, and description
- Covers all 48 divisions (00-48)

**Scripts**:
- `csi_scraper.py` - Scraper for CSI widget (used for divisions and subdivisions)
- `populate_csi_codes.py` - Database population from multiple JSON sources
- Combined data from GitHub JSON and scraped intermediate levels

**Note**: This version uses MasterFormat 2016 Edition. While not the absolute latest, it provides comprehensive coverage of 8,778 codes and is suitable for production use. Future updates may incorporate MasterFormat 2020 or later editions as machine-readable formats become available.

### Future Data Extraction
- IBC (International Building Code) - 2018, 2021, 2024
- IRC (International Residential Code)
- IECC (International Energy Conservation Code)
- IMC (International Mechanical Code)
- State-specific amendments and adoptions

## Data Sources

### Legal & Available
- **codes.iccsafe.org** - Free public access to ICC codes (requires account)
- State government websites - Many publish adopted codes
- Your friend's expertise - Initial mappings
- LLMs - Draft suggestions (must be verified)

### Important Notes
- ICC code content is copyrighted but publicly viewable
- We link to official sources rather than hosting content
- State amendments are often public domain
- Manual verification is essential for accuracy
- Scraping is done ethically (public access, rate-limited)

## Development Setup

See README.md for setup instructions.

### Running Scripts Against Docker Database

When running population scripts locally (e.g., `populate_csi_codes.py`), they use the `DATABASE_URL` from your local `.env` file by default. To populate the **Docker container's database**, explicitly set the URL:

```bash
cd backend
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/csi_icc_db" python scripts/populate_csi_codes.py
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/csi_icc_db" python scripts/populate_all_chapters.py
```

### Data Persistence

- Data persists across `make dev-down` / `make dev` cycles (uses named volume `postgres_data_dev`)
- Only `make clean` or `make clean-volumes` will delete the database volume and data

## Key Design Decisions

1. **Direct Linking Model**: Map CSI codes to ICC section URLs rather than hosting copyrighted content
2. **Selenium-Based Scraping**: Use Selenium WebDriver to extract data from JavaScript-rendered pages on codes.iccsafe.org
3. **PostgreSQL**: Robust relational database with full-text search capabilities
4. **FastAPI**: Modern async Python framework with auto-generated API docs (OpenAPI/Swagger)
5. **Alembic**: Database version control for reproducible schema migrations
6. **Pydantic v2**: Type-safe API validation and serialization
7. **Next.js 15 App Router**: Server-side rendering, static generation, and optimal performance
8. **Modular Scripts**: Separate extraction (scraping) and population (database) for flexibility
9. **Docker Compose**: Containerized deployment for development and production environments

## Future Considerations

- Add full-text search (PostgreSQL `tsvector`)
- Implement caching (Redis) for popular queries
- Add analytics to track most-searched codes
- Consider ICC partnership for official data access
- Mobile app version

## Testing Strategy

- Unit tests for CRUD operations
- Integration tests for API endpoints
- Manual testing with real CSI codes
- Expert validation of mappings

## Deployment

### Production (Railway) ✅
- **Live URL**: https://csimap.up.railway.app/
- **Backend**: FastAPI on Railway (https://csi-to-icc-backend-production.up.railway.app)
- **Frontend**: Next.js 15 on Railway
- **Database**: Managed PostgreSQL on Railway
- **Deployment**: Automatic deploys from main branch via GitHub integration
- **Environment Variables**: Configured with Railway service references for dynamic URLs

### Local Development (Docker)
- **Development**: `docker-compose.dev.yml` with hot-reload
- **Production**: `docker-compose.yml` with optimized builds
- **Makefile Commands**: `make dev`, `make up`, `make logs`, etc.
- **Database**: PostgreSQL container with persistent volume
- **Documentation**: See DOCKER.md for full deployment guide

## Current Project Status

### Production Status: LIVE ✅
**Public URL**: https://csimap.up.railway.app/

### What's Working ✅
- **Backend API**: All 21 endpoints functional, deployed on Railway
- **Frontend**: Full search interface live with keyword-based matching
- **Database**: 5 tables with proper relationships, hosted on Railway PostgreSQL
- **IPC 2018 Data**: 1,087 sections fully loaded and searchable
- **CSI MasterFormat 2016**: 8,778 codes loaded across all hierarchy levels
- **Docker Setup**: Development and production environments configured
- **Scraping Infrastructure**: Selenium-based scraper with comprehensive documentation
- **Search Functionality**: Working end-to-end with keyword fallback (validated by users!)
- **Production Deployment**: Live on Railway with automatic GitHub deploys

### What Needs Work ⏳
- **Expert Mappings**: Manual CSI-to-ICC relationships (requires domain expert validation)
- **Additional ICC Codes**: Only IPC 2018 loaded, need IBC, IRC, IECC, IMC (high priority)
- **State Amendments**: No state-specific code modifications yet
- **Testing**: Unit and integration tests not implemented
- **Data Currency**: Using MasterFormat 2016 (consider updating to 2020+ when available)
- **Search Refinement**: Improve keyword matching algorithm with synonyms and weighting

### Next Steps (Priority Order)
1. **Add IBC 2024/2021**: Most commonly used ICC code, critical for expansion
2. **Create Expert Mappings**: Build CSI-to-ICC relationships with professional validation
3. **Add IRC and IMC**: Expand coverage to residential and mechanical codes
4. **Implement Analytics**: Track popular searches to prioritize mapping efforts
5. **Add State Amendments**: Research and add state-specific modifications
6. **User Accounts**: Optional login for saved searches and personal notes

See [TODO.md](TODO.md) for the full roadmap.

## Contact & Collaboration

This is a personal project built to help construction professionals and civil engineers work more efficiently.

**Tech Stack Summary**: FastAPI + PostgreSQL + Next.js 15 + Railway
**Current Phase**: Phase 2 (Data Expansion & Mapping Enhancement)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BenGottschall) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
