## gojjo-mcp

> This project creates a custom Model Context Protocol (MCP) server that provides Claude Code with up-to-date documentation for a specific Django + Vue.js technology stack. The server automatically fetches current versions and documentation for your libraries, including custom PyPI packages.

# Gojjo Django Vue MCP Documentation Server

## Overview

This project creates a custom Model Context Protocol (MCP) server that provides Claude Code with up-to-date documentation for a specific Django + Vue.js technology stack. The server automatically fetches current versions and documentation for your libraries, including custom PyPI packages.

## Problem Solved

**Claude Code Issues:**

- Suggests outdated library versions
- No knowledge of custom PyPI libraries (like `aida-permissions`)
- Generic context that includes unnecessary frameworks

**Our Solution:**

- Real-time documentation for your exact tech stack
- Current version information from PyPI/NPM
- Custom library documentation integration
- Automated updates via web scraping
- Shared team access via hosted server

## Technology Stack Covered

### Django Backend

- Django (latest stable)
- Django REST Framework (DRF)
- drf-spectacular
- django-cors-headers
- django-filter
- django-allauth
- djangorestframework-simplejwt
- stripe
- twilio
- django-storages[google]
- django-anymail[sendgrid]
- gunicorn
- psycopg[c]
- redis
- celery
- django-celery-beat
- flower
- pytest-django
- factory
- **aida-permissions** (custom library)
- pytest
- coverage
- factory-boy

### Vue.js Frontend

- Vue 3
- Vue Router
- Pinia (state management)
- Axios
- Tailwind CSS
- TipTap (rich text editor)
- Vite
- Jest
- Cypress
- vue-test-utils
- Vitest
- ESLint
- Playwright

## Architecture

```markdown
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Claude Code   │───▶│   MCP Server     │───▶│  Documentation  │
│                 │    │  (mcp.domain.com)│    │   Sources       │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                                │                        │
                                ▼                        │
                       ┌──────────────────┐             │
                       │   Rate Limiting  │             │
                       │   & Caching      │             │
                       └──────────────────┘             │
                                                        ▼
                                              ┌─────────────────┐
                                              │ PyPI/NPM APIs   │
                                              │ GitHub Releases │
                                              │ Official Docs   │
                                              └─────────────────┘
```

## Implementation Plan

### Phase 1: Core MCP Server

1. **Basic MCP Server Structure**
   - Create `django_vue_mcp_server.py` with resource handlers
   - Implement PyPI/NPM API integration
   - Add caching layer for performance

2. **Documentation Fetching**
   - Real-time version checking
   - Custom library documentation (aida-permissions)
   - Integration examples and common patterns

3. **Local Testing**
   - Test MCP protocol communication
   - Verify Claude Code integration
   - Test all supported libraries

### Phase 2: Production Deployment

1. **Hetzner Server Setup**
   - Ubuntu server configuration
   - Docker containerization
   - Redis for caching

2. **Domain & SSL**
   - Configure `mcp.gojjoapps.com` subdomain
   - Let's Encrypt SSL certificate
   - Nginx reverse proxy

3. **HTTP API Wrapper**
   - REST endpoints for MCP resources
   - Health checks and monitoring
   - Webhook endpoints for auto-updates

### Phase 3: Security & Scaling

1. **Rate Limiting**
   - Anonymous: 100 requests/hour
   - API key: 1000 requests/hour
   - Open source projects: 5000 requests/hour

2. **Cost Protection**
   - Resource usage monitoring
   - Automatic scaling limits
   - Alert system for overages

3. **Open Source Preparation**
   - Documentation for self-hosting
   - API key system for public use
   - Community contribution guidelines

## File Structure

```markdown
django-vue-mcp-server/
├── CLAUDE.md                    # This file
├── README.md                    # Public documentation
├── requirements.txt             # Python dependencies
├── docker-compose.yml           # Production deployment
├── Dockerfile                   # Container configuration
├── nginx.conf                   # Reverse proxy config
├── systemd/                     # Service files
│   └── mcp-docs.service
├── src/
│   ├── django_vue_mcp_server.py    # Core MCP server
│   ├── production_mcp_server.py    # HTTP wrapper
│   ├── mcp_http_client.py          # Client for Claude Code
│   ├── auto_updater.py             # Automated doc updates
│   ├── rate_limiter.py             # Security layer
│   ├── auth.py                     # API key management
│   └── monitoring.py               # Usage tracking
├── docs/
│   ├── installation.md
│   ├── api.md
│   └── self-hosting.md
└── tests/
    ├── test_mcp_server.py
    ├── test_rate_limiting.py
    └── test_integrations.py
```

## Key Features

### 1. Real-Time Documentation

- Fetches current versions from PyPI/NPM APIs
- Scrapes release notes and changelog information
- Caches results for performance (6-hour refresh)

### 2. Custom Library Integration

- Special handling for `aida-permissions`
- Integration examples with Django/DRF
- Common usage patterns and troubleshooting

### 3. Smart Caching

- Multi-tier caching strategy (hot/warm/cold)
- Redis for distributed caching
- Nginx proxy caching for static content

### 4. Security & Cost Control

- Rate limiting by IP and API key
- Resource usage monitoring
- Automatic alerts for unusual usage
- DDoS protection via Cloudflare

### 5. Team Collaboration

- Shared server accessible to all developers
- Webhook integration for auto-updates
- Version history and change tracking

## Development Commands

### Local Development

```bash
# Install dependencies
pip install -r requirements.txt

# Run MCP server locally
python src/django_vue_mcp_server.py

# Test MCP protocol
python src/mcp_http_client.py

# Run with auto-updates
python src/production_mcp_server.py
```

### Testing

```bash
# Unit tests
pytest tests/

# Integration tests
pytest tests/test_integrations.py

# Load testing
python tests/load_test.py
```

### Deployment

```bash
# Deploy to Hetzner
docker-compose -f docker-compose.yml up -d

# Update documentation
curl -X POST https://mcp.gojjoapps.com/webhook/update

# Check server health
curl https://mcp.gojjoapps.com/health
```

## Claude Code Integration

### Configuration File

Create `~/.config/claude-code/mcp-config.json`:

```json
{
  "servers": {
    "gojjo-django-vue-docs": {
      "command": "python",
      "args": ["/path/to/mcp_http_client.py"],
      "env": {
        "MCP_SERVER_URL": "https://mcp.gojjoapps.com"
      }
    }
  }
}
```

### Usage in Claude Code

Once configured, Claude Code will automatically have access to:

- Current version information for all stack libraries
- Integration examples between Django and Vue
- Custom `aida-permissions` library documentation
- Best practices and common patterns
- Troubleshooting guides

## Expected Benefits

### For Development Team

- **Accurate Suggestions**: Claude Code suggests current versions
- **Custom Library Knowledge**: Full awareness of `aida-permissions`
- **Integration Patterns**: Django + Vue best practices
- **Reduced Context Switching**: No need to check documentation separately

### For Open Source Community

- **Free Public Access**: Rate-limited but generous usage
- **Self-Hosting Option**: Full control when needed
- **Extensible Design**: Easy to add new libraries
- **Community Contributions**: Open to pull requests

## Monitoring & Maintenance

### Automated Updates

- **Daily**: Version checks for all libraries
- **Weekly**: Deep documentation scraping
- **On Webhook**: Immediate updates for critical changes

### Alerting

- **Usage Spikes**: Unusual traffic patterns
- **Error Rates**: Failed API calls or documentation fetches
- **Cost Thresholds**: Resource usage approaching limits

### Performance Metrics

- **Response Time**: Average API response latency
- **Cache Hit Rate**: Efficiency of caching layer
- **Uptime**: Server availability monitoring

## Next Steps

1. **Build Core Server**: Implement the basic MCP server with library support
2. **Test Locally**: Verify Claude Code integration works correctly
3. **Deploy to Hetzner**: Set up production environment with security
4. **Open Source Release**: Publish repository and documentation
5. **Community Engagement**: Share with Django/Vue communities

## Success Metrics

- **Technical**: Claude Code suggests correct current versions 95%+ of the time
- **Usage**: >100 developers using the service within 6 months  
- **Community**: >50 GitHub stars and active contributors
- **Cost**: Monthly server costs <$20 with reasonable usage

---

**Remember**: This MCP server is designed to be a living, breathing documentation source that evolves with your stack. The goal is to make Claude Code as knowledgeable about your specific technology choices as a senior developer on your team.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hmesfin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
