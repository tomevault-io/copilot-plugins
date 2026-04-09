## jb-square

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

JB SQUARE (전북 바이오 플랫폼) is a web-based backoffice system for crawling, managing, and publishing biotech industry announcements and job postings in the Jeonbuk region. It aggregates data from multiple sources (JBTP, NTIS, BizInfo) and provides an administrative interface for content management.

## High-Level Architecture

The system follows a three-tier architecture with modular components:

1. **Frontend (SvelteKit)**: TypeScript-based UI at port 3000 (dev) with routes for notices, contents, login, and dashboard
2. **Backend (FastAPI)**: Python REST API at port 8000 with WebSocket support for real-time crawling
3. **Database (PostgreSQL)**: AWS RDS for persistent storage of notices, crawler configs, and results
4. **Crawlers**: Modular architecture with BaseAdapter abstract class and specialized implementations for JBTP, NTIS, BIZINFO, and BI Centers
5. **Deployment**: Docker containers orchestrated with Docker Compose, Nginx reverse proxy, SSL via Let's Encrypt

Data flows from external sources through crawlers to the database, then served via API to the frontend. WebSocket connections enable real-time progress updates during crawling operations.

## Essential Commands

### Development Setup

```bash
# Frontend development
cd frontend
npm install
npm run dev                    # Starts on http://localhost:5173

# Backend development
cd backend
python3 -m venv venv
source venv/bin/activate       # macOS/Linux
pip install -r requirements.txt
uvicorn src.main:app --reload                  # Starts on http://localhost:8000

# Docker development (recommended)
docker-compose up -d --build   # Development with exposed ports
docker-compose logs -f         # View logs
docker-compose down            # Stop containers
```

### Production Deployment

```bash
# Production deployment
docker-compose -f docker-compose.prod.yml up -d --build

# View specific service logs
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f nginx

# Initialize database
curl -X POST http://localhost:8000/init-db
```

### Testing

```bash
# Test WebSocket crawling
cd backend/tests/test_all_crawlers.py
python test_crawling.py

# Test API endpoints
curl http://localhost:8000/health
curl http://localhost:8000/api/dashboard/summary
curl http://localhost:8000/api/notices

# Access API documentation
# Swagger UI: http://localhost:8000/docs
# ReDoc: http://localhost:8000/redoc
```

## Key Implementation Details

### Crawler System Architecture

The crawler system was recently refactored from a monolithic 2416-line file to a modular design:

- **BaseAdapter** ([backend/src/services/sources/base_crawler.py](backend/src/services/sources/base_crawler.py)): Abstract base class providing status tracking, WebSocket events, keyword matching, and date parsing utilities
- **CrawlerManager** ([backend/src/services/sources/_manager.py](backend/src/services/sources/_manager.py)): Orchestrates crawler instances and manages execution (note: still 24KB, requires further refactoring)
- **Specialized Crawlers**: JBTP, NTIS (API-based), BIZINFO, BI Center crawlers inherit from BaseAdapter
- **Rate Limiting**: Implemented in [backend/src/services/rate_limiter.py](backend/src/services/rate_limiter.py) to respect target sites

### API Endpoints Structure

All API endpoints are registered in [backend/src/main.py](backend/src/main.py):
- `/api/dashboard`: Statistics and summaries
- `/api/contents`: Content management CRUD
- `/api/notices`: Notice management
- `/api/crawling`: WebSocket-based crawling execution
- `/api/crawling/configs`: Crawler configuration
- `/api/bi-centers`: Business incubation centers
- `/api/tags`: Tag management
- `/api/organizations`: Organization data
- `/api/analytics`: Analytics and reports
- `/api/settings`: Application settings

### Frontend Routing

SvelteKit file-based routing in [frontend-admin/src/routes/](frontend-admin/src/routes/):
- `/`: Dashboard home page
- `/login`: Authentication
- `/contents`: Content management interface
- `/notices/latest`: Latest announcements
- `/notices/business`: Business announcements
- `/notices/government`: Government grants
- `/notices/startup`: Startup programs
- `/notices/local`: Local programs
- `/notices/ntis`: NTIS announcements

### Database Models

Core models defined in [backend/src/core/database.py](backend/src/core/database.py) and [backend/src/models/](backend/src/models/):
- **CrawlerConfig**: Keywords (JSON), enabled status, source configuration
- **Notice**: Announcements with title, content, status, source information
- **CrawlResult**: Crawling operation results and statistics
- **BiCenter**: Business incubation center information
- **Tag**: Classification tags for content

### Environment Configuration

Required environment variables (see [.env.example](.env.example)):
- AWS credentials and RDS database connection
- API keys: NTIS_API_KEY (mandatory), BIZINFO_API_KEY, JBTP_API_KEY
- CORS_ALLOWED_ORIGINS: Include production domains (jb2.kr, jb2.co.kr) and localhost ports
- Admin credentials for initial setup

### Docker Deployment

Two Docker Compose configurations:
- **Development** ([docker-compose.yml](docker-compose.yml)): Exposes ports for debugging
- **Production** ([docker-compose.prod.yml](docker-compose.prod.yml)): Internal networking, health checks, SSL support

All services use multi-stage Docker builds for optimized image sizes.

### CI/CD Pipeline

GitHub Actions workflow ([.github/workflows/deploy.yml](.github/workflows/deploy.yml)) automatically deploys to EC2 on push to main branch:
1. Creates .env from GitHub Secrets
2. Syncs code to EC2 via rsync
3. Builds and starts Docker containers
4. Verifies deployment with health checks

Required GitHub Secrets: EC2_HOST, EC2_USER, EC2_SSH_KEY, AWS credentials, database config, API keys.

## Important Context for Development

### Recent Refactoring

The crawler system underwent major refactoring to modular architecture. The original monolithic [sources/_manager.py](backend/src/services/sources/_manager.py) (2416 lines) has been partially refactored. Each crawler now inherits from BaseAdapter, but sources/_manager.py still requires further decomposition.

### WebSocket Real-Time Updates

Crawling operations use WebSocket connections for real-time progress updates. Frontend connects to `/api/crawling/ws/{source}` and receives events for status changes, progress updates, and completion notifications.

### Rate Limiting Considerations

All crawlers implement rate limiting to avoid overwhelming target sites. Default delays are configured per source in the rate_limiter service.

### Testing Infrastructure

Currently uses manual testing scripts rather than a formal test framework. Test files include:
- [backend/tests/test_all_crawlers.py/test_crawling.py](backend/tests/test_all_crawlers.py/test_crawling.py): WebSocket crawling tests
- [backend/test_keyword_matching.py](backend/test_keyword_matching.py): Keyword matching logic
- Various debug scripts for individual crawlers

### Deployment Scripts

Helper scripts in [scripts/](scripts/) directory:
- `deploy.sh`: Main deployment automation
- `setup-ec2.sh`: EC2 environment setup
- `setup-ssl.sh`: SSL certificate configuration
- `setup-github-secrets.sh`: GitHub Secrets automation

### Known Areas for Improvement

1. **sources/_manager.py**: Still 24KB, needs further modularization
2. **Test Coverage**: No pytest/unittest framework configured yet
3. **Database Migrations**: Manual migrations, Alembic not yet configured
4. **Frontend State Management**: Limited to toast notifications store
5. **Error Handling**: Inconsistent error handling patterns across crawlers

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/viakisun)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/viakisun)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
