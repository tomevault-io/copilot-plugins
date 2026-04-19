## bedrot-data-ecosystem

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Path Warning ⚠️

**CRITICAL**: The correct ecosystem path has changed!

**OLD/WRONG**: `C:\Users\Earth\BEDROT PRODUCTIONS\BEDROT DATA LAKE`  
**NEW/CORRECT**: `C:\Users\Earth\BEDROT PRODUCTIONS\bedrot-data-ecosystem`

Component paths:
- Data Lake: `bedrot-data-ecosystem\data_lake\`
- Data Warehouse: `bedrot-data-ecosystem\data_warehouse\`
- Dashboard: `bedrot-data-ecosystem\data_dashboard\`

Always verify you're in the correct directory before running commands!

## Project Overview

BEDROT Data Ecosystem is a production-grade analytics infrastructure for music industry data, consisting of three integrated components:

- **Data Lake** (`data_lake/`): Multi-zone ETL architecture for ingesting data from Spotify, TikTok, Meta Ads, DistroKid, Linktree, TooLost, YouTube, MailChimp
- **Data Warehouse** (`data_warehouse/`): SQLite-based analytical database with normalized tables for streaming, financial, and social media metrics  
- **Data Dashboard** (`data_dashboard/`): Real-time Next.js/FastAPI dashboard displaying 20+ KPIs with live WebSocket updates

## Zone Architecture

### Data Lake Zones
1. **1_landing/** - Raw data ingestion, timestamped files from extractors
2. **2_raw/** - Validated, immutable copies in NDJSON/CSV format
3. **3_staging/** - Cleaned, transformed, standardized data
4. **4_curated/** - Business-ready datasets for analytics/dashboards
5. **5_archive/** - Long-term storage (7+ year retention)
6. **6_automated_cronjob/** - Master pipeline orchestration scripts

### Service Implementation Status
| Service | Extractors | Cleaners | Priority | Critical Notes |
|---------|------------|----------|----------|----------------|
| TooLost | ✅ Multiple variants | ✅ All 3 | CRITICAL | JWT expires every 7 days! |
| Spotify | ✅ audience_extractor | ✅ All 3 | HIGH | Artists API |
| TikTok | ✅ 2 accounts | ✅ All 3 | HIGH | zonea0 + pig1987 |
| DistroKid | ✅ dk_auth | ✅ All 3 | MEDIUM | Streams + financials |
| Linktree | ✅ analytics_extractor | ✅ All 3 | MEDIUM | GraphQL |
| MetaAds | ✅ daily_campaigns | ✅ All 3 | LOW | Graph API |
| Instagram | ❌ Empty dirs | ❌ | - | Not implemented |
| YouTube | ❌ Empty dirs | ❌ | - | Not implemented |
| MailChimp | ❌ Empty dirs | ❌ | - | Not implemented |

## Important Context: Semi-Manual Authentication Design

### Authentication Philosophy
**CRITICAL**: The BEDROT Data Lake operates as a **semi-manual system by design**. This is not a limitation but a deliberate choice for compliance and security:

1. **Extractors NEVER authenticate automatically** - All authentication is handled manually by the user
2. **2FA Compliance** - All services use two-factor authentication, requiring human interaction
3. **Cookie Management is Convenience, Not Critical** - Expired cookies mean manual re-authentication, not system failure
4. **User-Driven Process** - The system respects service terms by requiring explicit user authentication

### Cookie Management Role
The cookie system exists to **reduce** manual work, not eliminate it:
- **Storage**: Preserves authentication state between runs
- **Monitoring**: Warns when cookies are expiring
- **Refresh Assistance**: Helps guide re-authentication when needed
- **NOT Critical Infrastructure**: Cookie failures are inconveniences, not emergencies

### Interpreting Cookie-Related Failures
When you see extraction failures due to cookies:
1. **This is EXPECTED behavior** - The system is working as designed
2. **Not a critical error** - Just means manual authentication is needed
3. **Part of the workflow** - Semi-manual means user intervention is normal
4. **Compliance-friendly** - Ensures we're not automating around security measures

## Development Commands

### Testing
```bash
# Run all tests with coverage
pytest -ra --cov=src --cov-report=term-missing

# Run tests for specific service
pytest tests/spotify/ -v

# Run cookie refresh tests
pytest tests/test_cookie_refresh.py -v
pytest tests/test_integration.py -v
pytest tests/test_e2e_scenarios.py -v
```

### Code Quality
```bash
# Format code
black src/
isort src/

# Lint code  
flake8 src/
mypy src/
```

### Data Pipeline Execution

#### New Semi-Manual Pipeline with Authentication (Recommended)
```bash
# Interactive mode - prompts for manual auth when needed
cd "C:\Users\Earth\BEDROT PRODUCTIONS\bedrot-data-ecosystem\data_lake"
cronjob\run_bedrot_pipeline.bat

# Automated mode - skips services needing auth
cronjob\run_bedrot_pipeline.bat --automated

# Alternative with data warehouse ETL
cronjob\run_pipeline_with_auth.bat
```

#### Legacy Pipeline Commands
```bash
# Run full data lake pipeline (Windows)
data_lake/cronjob/run_datalake_cron.bat

# Run pipeline without extractors (cleaners only)
data_lake/cronjob/run_datalake_cron_no_extractors.bat

# Manual Python execution requires PROJECT_ROOT environment variable
set PROJECT_ROOT=%cd%
python src/spotify/extractors/spotify_audience_extractor.py
```

#### Health Monitoring & Diagnostics
```bash
# Check pipeline health
python src\common\pipeline_health_monitor.py

# Check authentication status
python src\common\run_with_auth_check.py --check-only

# Run specific services with auth check
python src\common\run_with_auth_check.py spotify toolost

# Test pipeline components
python test_pipeline_components.py
```

### Data Dashboard

```bash
# Frontend (Next.js)
cd data_dashboard
npm install
npm run dev        # Port 3000 with custom Socket.IO server
npm run build
npm run start
npm run lint
npm run type-check

# Backend (FastAPI)
cd data_dashboard/backend
pip install -r requirements.txt
python main.py     # Port 8000 with WebSocket support
```

### Data Warehouse

```bash
cd data_warehouse
# Currently being reinitialized - structure pending
# Virtual environment preserved at .venv/
# Requirements preserved at requirements.txt
```

## Architecture

### System Data Flow
```
External APIs → Data Lake → Data Warehouse → Dashboard → Business Insights
     ↓             ↓             ↓              ↓              ↓
  [Collect]    [Process]    [Normalize]    [Visualize]    [Decide]
```

### Data Lake Zones
```
landing/ → raw/ → staging/ → curated/ → archive/
   ↓        ↓        ↓          ↓          ↓
[Ingest] [Validate] [Clean] [Transform] [Preserve]
```

### ⚠️ CRITICAL Data Caveat: Spotify Stream Double-Counting
**IMPORTANT**: The curated zone contains Spotify streaming data from TWO separate sources:
1. **tidy_daily_streams.csv** - Spotify + Apple Music streams from distributors (DistroKid/TooLost)
2. **spotify_audience_curated_*.csv** - Spotify-only metrics from Spotify for Artists API

**Never combine these datasets when calculating Spotify totals** - you would be double-counting!
- For total platform streams: Use `tidy_daily_streams` (includes both Spotify and Apple)
- For Spotify-specific analytics: Use `spotify_audience_curated` 
- **DO NOT** add values from both datasets together

### Key Components

**Data Lake Organization** (`src/<platform>/`):
- `extractors/` - API/web scraping scripts
- `cleaners/` - Zone promotion scripts (follow naming: `<service>_landing2raw.py`)
- `cookies/` - Authentication cookies

**Additional Directories**:
- `6_automated_cronjob/` - Automated pipeline execution scripts
- `src/common/cookie_refresh/` - Cookie refresh system with service strategies
- Root CSV files - Ad campaign and catalog data exports

**Data Warehouse** (Currently being reinitialized):
- SQLite database: `bedrot_analytics.db`
- Preserved virtual environment and requirements
- Will integrate with data_lake curated zone

**Dashboard Architecture**:
- Frontend: Next.js 14, React 18, TypeScript, Tailwind CSS
- Backend: FastAPI with WebSocket support
- Real-time updates via Socket.IO

## Cookie Authentication System

### Service-Specific Expiration
| Service | Max Age | Refresh Interval | Strategy | 2FA |
|---------|---------|------------------|----------|-----|
| TooLost | 7 days | 5 days | JWT Manual | No |
| Spotify | 30 days | 7 days | OAuth Manual | No |
| TikTok Zone A0 | 30 days | 7 days | QR/Manual | Yes |
| TikTok PIG1987 | 30 days | 7 days | QR/Manual | Yes |
| DistroKid | 90 days | 10 days | Automated | No |
| Linktree | 30 days | 14 days | Standard | No |
| MetaAds | 60 days | 30 days | OAuth | No |

### Cookie Management
```bash
# Check all services
python src/common/cookie_refresh/check_status.py

# Manual refresh
python src/common/cookie_refresh/manual_refresh.py --service toolost

# View dashboard (http://localhost:8080)
python src/common/cookie_refresh/start_dashboard.py
```

## Common Tasks

### Adding a New Data Source

1. Create directory structure:
   ```bash
   mkdir -p src/newservice/{extractors,cleaners,cookies}
   touch src/newservice/__init__.py
   ```

2. Implement extractor (`src/newservice/extractors/newservice_extractor.py`)
3. Create three cleaners following naming convention
4. Add tests in `tests/newservice/`
5. No registration needed - pipelines auto-discover

### Running a Single Test
```bash
# Specific test file
pytest tests/spotify/test_spotify_extractor.py -v

# Specific test function
pytest tests/spotify/test_spotify_extractor.py::test_extract_data -v

# With coverage for module
pytest tests/spotify/ --cov=src.spotify --cov-report=term-missing
```

### Debugging Failed Extractors
```bash
# Set environment and run with debug
cd data_lake
set PROJECT_ROOT=%cd%
python src/spotify/extractors/spotify_audience_extractor.py --debug

# Check cookies
python -c "from common.cookies import validate_cookies; print(validate_cookies('spotify'))"

# View logs
type logs\extractor_YYYYMMDD.log
```

## Key Conventions

- **Environment**: `PROJECT_ROOT` must be set for all scripts
- **File Naming**: Timestamps as `YYYYMMDD_HHMMSS`
- **Data Formats**: NDJSON for structured data, CSV for tabular
- **Testing**: pytest with coverage targets
- **Authentication**: Cookie-based with automatic refresh system
- **Deployment**: Supports both Windows batch files and Linux scripts

## WebSocket Events (Dashboard)

```javascript
// Client → Server
socket.emit('subscribe', { channel: 'kpis' })
socket.emit('unsubscribe', { channel: 'kpis' })

// Server → Client  
socket.on('kpi:update', (data) => { /* New KPI data */ })
socket.on('connection:status', (status) => { /* Connected/Disconnected */ })
```

## Data Quality Standards

- SHA-256 hash deduplication tracked in `_hashes.json`
- Archive existing curated files before overwriting
- Validate data types and constraints in cleaners
- Log all operations with timestamps
- Handle errors gracefully with descriptive messages

## Performance Optimization

**Data Lake**:
- Parallel processing for different services
- Batch operations (1000 records/chunk)
- Generator patterns for large files

**Data Warehouse**:
- Indexes on frequently queried columns
- Regular VACUUM operations
- Batch inserts with transactions

**Dashboard**:
- React.memo for expensive components
- WebSocket throttling (max 1/second)
- Query result caching

## Security Considerations

- Store credentials in environment variables only
- Never commit cookies or API keys
- Hash PII data before storage
- Use HTTPS for all external APIs
- Implement authentication for dashboard access

## Current Status (September 2025)

### Pipeline Health
- **Working**: Check current status with `python src/common/pipeline_health_monitor.py`
- **Cookie Status**: Monitor with `python src/common/cookie_refresh/check_status.py`
- **Dashboard**: View at http://localhost:8080 via `python src/common/cookie_refresh/start_dashboard.py`

### Recent Changes (September 2025)
- Fixed TikTok zone.a0 date range selector to properly select 365 days
- Updated TikTok HTML selectors to match current UI structure
- Fixed browser profile conflicts between TikTok accounts
- Improved download button selectors for TikTok analytics
- Fixed MetaAds cleaner nlargest() syntax error

### Known Issues
- TooLost JWT expires every 7 days
- TikTok zone.a0 downloads may still show limited data (account limitation)
- Some services have data stuck in landing/raw zones
- Need weekly reminder for TooLost refresh

## Understanding Log Outputs

### Log File Locations
The data pipeline generates several types of logs in the `logs/` directory:

1. **Pipeline Execution Logs** (`pipeline_YYYYMMDD_HHMM.log`)
   - Main pipeline execution log from cron job runs
   - Contains step-by-step execution details
   - Includes cookie checks, extractor runs, cleaner runs, and health reports

2. **Pipeline Executor Logs** (Enhanced logging as of 2025-07-15)
   - `pipeline_executor.log` - Detailed execution log with timestamps
   - `pipeline_executor_errors.log` - Error-only log for quick debugging
   - `pipeline_executor_structured.jsonl` - JSON-formatted logs for programmatic parsing

3. **Service-Specific Logs**
   - `cookie_refresh.log` - Cookie refresh operations
   - `bedrot_pipeline.log` - Semi-manual pipeline runs
   - `bedrot_pipeline_errors.log` - Error-only view

### Common Error Resolutions

| Error Pattern | Likely Cause | Resolution | Severity |
|--------------|--------------|------------|----------|
| `TimeoutError: Page.wait_for_selector` | Page didn't load or login required | Run manual auth - this is NORMAL | Low |
| `No cookies found` | Missing authentication | Run extractor manually to login | Low |
| `Multiple extraction failures` | Cookies expired across services | Batch re-authenticate all services | Low |
| `attempted relative import` | Script run incorrectly | Use proper execution method | High |
| `no healthy upstream` | Network/proxy issue | Check network, disable proxy | High |
| `STALE data` | Extraction hasn't run recently | Run specific extractor with auth | Medium |

### Understanding Failure Severity

**Low Severity (Cookie/Auth Issues)**:
- Expected in semi-manual system
- Requires user intervention
- Not a system failure
- Part of normal workflow

**High Severity (System Issues)**:
- Actual infrastructure problems
- Code/configuration errors
- Network connectivity issues
- These need immediate fixing

**Medium Severity (Data Issues)**:
- Data quality concerns
- Processing bottlenecks
- May indicate auth OR system issues

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/blakedemarest) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
