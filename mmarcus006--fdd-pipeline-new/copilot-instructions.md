## fdd-pipeline-new

> This is an automated Franchise Disclosure Document (FDD) processing pipeline that scrapes, downloads, processes, and extracts structured data from FDD documents filed with state franchise portals.

# FDD Pipeline - Project Documentation

## Overview
This is an automated Franchise Disclosure Document (FDD) processing pipeline that scrapes, downloads, processes, and extracts structured data from FDD documents filed with state franchise portals.

## Key Features
- **Web Scraping**: Automated scraping of FDD documents from state portals (Minnesota CARDS, Wisconsin DFI)
- **Document Processing**: PDF processing using MinerU for layout analysis and section detection
- **Data Extraction**: LLM-powered extraction of structured data from specific FDD items (5, 6, 7, 19, 20, 21)
- **Database Storage**: Structured storage in Supabase with full data lineage tracking
- **Google Drive Integration**: Automatic document storage and organization in Google Drive
- **Workflow Orchestration**: Prefect-based workflow management with retry logic and monitoring

## Architecture (Current Implementation)

### Core Components

1. **Web Scrapers** (NOTE: Architecture in transition):
   - **Planned structure** (`scrapers/` - not yet implemented):
     - `base/base_scraper.py`: Abstract base class with common scraping functionality
     - `base/exceptions.py`: Custom exception hierarchy
     - `states/minnesota.py`: MinnesotaScraper class
     - `states/wisconsin.py`: WisconsinScraper class
     - `utils/scraping_utils.py`: Common utilities
   - **Current implementation** (`franchise_scrapers/`):
     - `MN_Scraper.py`: Standalone Minnesota scraper
     - `WI_Scraper.py`: Standalone Wisconsin scraper

2. **Document Processing** (`processing/`):
   - `mineru/mineru_processing.py`: Task wrapper for MinerU Web API
   - `mineru/mineru_web_api.py`: MinerU Web API client with browser authentication
   - `segmentation/document_segmentation.py`: FDD section detection and boundary extraction
   - `segmentation/enhanced_detector.py`: Advanced section detection using Claude
   - `pdf/pdf_extractor.py`: Basic PDF text extraction utilities

3. **Data Extraction** (`processing/extraction/`):
   - `llm_extraction.py`: Multi-model LLM framework with routing and fallback
   - `multimodal.py`: Handles image and table extraction
   - Supports Gemini, OpenAI, and Ollama models
   - Item-specific extraction for FDD sections

4. **Data Models** (`models/`):
   - Pydantic models for all database entities
   - Item-specific response models (Item5Fees, Item6OtherFees, etc.)
   - Composite models for complex data structures
   - JSON storage models for flexible item data

5. **Workflows** (`workflows/`):
   - `base_state_flow.py`: Generic state scraping flow (unified implementation)
   - `state_configs.py`: State-specific configurations
   - `process_single_pdf.py`: Single PDF processing flow
   - `complete_pipeline.py`: End-to-end orchestration flow

6. **Storage** (`storage/`):
   - `google_drive.py`: Google Drive integration for document storage
   - `database/manager.py`: Database connection and operations management

7. **Validation** (`validation/`):
   - `schema_validation.py`: Pydantic schema validation
   - `business_rules.py`: Business logic validation

8. **API Layer** (`src/api/`):
   - `main.py`: FastAPI application with REST endpoints
   - `run.py`: Uvicorn server runner
   - Endpoints for document processing and data retrieval

## Database Schema

### Core Tables
- **franchisors**: Canonical franchise entities with deduplication
- **fdds**: FDD documents with versioning and supersession tracking
- **fdd_sections**: Document sections after segmentation
- **scrape_metadata**: Web scraping audit trail

### Extracted Data Tables
- **item5_fees**: Initial franchise fees
- **item6_other_fees**: Other fees and costs
- **item7_investment**: Estimated initial investment
- **item19_fpr**: Financial performance representations
- **item20_outlets**: Outlet and franchisee information
- **item21_financials**: Financial statements

## Common Tasks

### 1. Run Complete Pipeline
```bash
# Run for all configured states
python main.py run-all

# Run for specific state (now uses unified base flow)
python main.py scrape --state minnesota
python main.py scrape --state wisconsin

# With options
python main.py scrape --state all --limit 10 --test-mode
```

### 2. Process Single PDF
```bash
python main.py process-pdf --path /path/to/fdd.pdf
```

### 3. Check Pipeline Health
```bash
python main.py health-check
```

### 4. Deploy Workflows to Prefect
```bash
python main.py orchestrate --deploy --schedule
```

### 5. Run Tests
```bash
# Unit tests
pytest tests/ -m unit

# Integration tests  
pytest tests/ -m integration

# All tests
pytest tests/
```

### 6. Database Migrations
```sql
-- Apply migrations in order
psql -d your_database -f migrations/001_initial_schema.sql
psql -d your_database -f migrations/002_structured_data_tables.sql
-- etc...
```

## Configuration

### Environment Variables
```bash
# Database
SUPABASE_URL=your_supabase_url
SUPABASE_ANON_KEY=your_anon_key
SUPABASE_SERVICE_KEY=your_service_key

# LLM APIs
GEMINI_API_KEY=your_gemini_key
OPENAI_API_KEY=your_openai_key  # Optional, for fallback
OLLAMA_BASE_URL=http://localhost:11434  # For local models

# Google Drive
GDRIVE_CREDS_JSON=gdrive_cred.json  # Path to service account JSON
GDRIVE_FOLDER_ID=root_folder_id

# MinerU Web API
MINERU_AUTH_FILE=mineru_auth.json  # Browser auth storage

# Section Detection
USE_ENHANCED_SECTION_DETECTION=true
ENHANCED_DETECTION_CONFIDENCE_THRESHOLD=0.7
ENHANCED_DETECTION_MIN_FUZZY_SCORE=80

# Application Settings
DEBUG=false
LOG_LEVEL=INFO
MAX_CONCURRENT_EXTRACTIONS=5
```

### Key Settings (config.py)
- Retry attempts: 3 (configurable per task)
- Request timeout: 30 seconds
- Document processing timeout: 5 minutes
- LLM extraction timeout: 60 seconds per section
- MinerU processing timeout: 300 seconds
- Enhanced section detection: Enabled by default
- Concurrent extractions: 5 (prevents rate limiting)

## Refactoring Status (In Progress)

### Major Changes Planned
1. **State Scraping Flows Consolidation**:
   - Created `workflows/base_state_flow.py` with generic state scraping logic
   - Created `workflows/state_configs.py` for state-specific configurations
   - NOTE: These files expect `scrapers/` module structure that doesn't exist yet
   - Current scrapers in `franchise_scrapers/` don't match expected architecture

2. **Files Cleaned Up**:
   - Deleted duplicate test files
   - Archived individual migration files (kept combined versions)
   - Removed all .DS_Store files
   - Updated .gitignore to prevent future accumulation

### Adding New States
To add a new state (e.g., California):

**Option 1: Using current implementation**
1. Create scraper file: `franchise_scrapers/CA_Scraper.py` following MN/WI pattern
2. Implement standalone scraper with Playwright

**Option 2: Using planned architecture (requires setup)**
1. Create the missing `scrapers/` directory structure
2. Create scraper class: `scrapers/states/california.py` extending `BaseScraper`
3. Add configuration to `workflows/state_configs.py`:
   ```python
   CALIFORNIA_CONFIG = StateConfig(
       state_code="CA",
       state_name="California",
       scraper_class=CaliforniaScraper,
       folder_name="California FDDs",
       portal_name="CA DBO"
   )
   ```
4. Update `STATE_CONFIGS` dictionary in `state_configs.py`
5. Update `scrapers/states/__init__.py` to export the new scraper

## Performance Considerations

### Scraping
- Implements exponential backoff with jitter
- Respects rate limits with configurable delays
- Browser resource pooling for efficiency

### Document Processing
- Streaming downloads for large PDFs
- Chunk-based processing for memory efficiency
- Parallel section extraction when possible

### LLM Extraction
- Model routing based on section complexity
- Token usage tracking and cost optimization
- Fallback chains for reliability

## Monitoring & Logging
- Structured logging with correlation IDs
- Prefect flow run tracking
- Database audit trails for all operations
- Extraction metrics and model performance tracking

## Security Notes
- All API keys stored as environment variables
- Database connections use service keys (RLS bypassed)
- Document hashes for deduplication and integrity
- No sensitive data logged

## Troubleshooting

### Common Issues

1. **MinerU Authentication Failed**
   - Delete `mineru_auth.json` and re-authenticate
   - Ensure Playwright browsers are installed: `playwright install chromium`
   - Check if MinerU Web API is accessible

2. **Database Connection Issues**
   - Verify Supabase URL and service key
   - Check if tables exist: `python main.py health-check`
   - Run migrations if needed

3. **Scraping Failures**
   - State portals may change their structure
   - Check browser automation with `DEBUG=true`
   - Verify network connectivity to state portals

4. **LLM Extraction Errors**
   - Ensure API keys are valid
   - Check rate limits for your API tier
   - For Ollama, ensure models are pulled: `ollama pull llama3`

5. **Google Drive Upload Issues**
   - Verify service account credentials
   - Check folder permissions
   - Ensure quota hasn't been exceeded

### Debug Mode

Enable detailed logging:
```bash
DEBUG=true LOG_LEVEL=DEBUG python main.py scrape --state minnesota
```

## Future Enhancements
1. Add more state portals (CA, NY, IL)
2. Implement incremental scraping (only new documents)
3. Add data validation and quality scoring
4. Enhance API layer with authentication
5. Implement automated testing for scrapers
6. Add document change detection and diff tracking
7. Create admin dashboard for monitoring
8. Add webhook notifications for pipeline events

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mmarcus006)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mmarcus006)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
