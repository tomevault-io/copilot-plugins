## mcp-hubspot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HubSpot MCP Server - A Model Context Protocol (MCP) server that enables AI assistants to interact with HubSpot CRM data. This is a Python-based MCP server designed for Docker deployment with built-in FAISS vector storage for semantic search and caching.

## Development Commands

### Installation and Setup
```bash
# Install in development mode
pip install -e .

# Run the server locally (requires HUBSPOT_ACCESS_TOKEN environment variable)
mcp-server-hubspot

# Or with explicit access token
mcp-server-hubspot --access-token YOUR_TOKEN
```

### Docker Commands
```bash
# Build Docker image
docker build -t mcp-hubspot .

# Build multi-platform image
docker buildx create --use
docker buildx build --platform linux/amd64,linux/arm64 -t buryhuang/mcp-hubspot:latest --push .

# Run Docker container
docker run -e HUBSPOT_ACCESS_TOKEN=your_token -v /path/to/storage:/storage buryhuang/mcp-hubspot:latest
```

### Testing
Tests are located in the `tests/` directory:
- `test_closed_tickets.py` - Tests for closed ticket retrieval
- `test_mcp_ticket_conversations.py` - Tests for ticket conversation threads
- `get_closed_ticket_conversations.py` - Utility script for testing conversation retrieval

Run tests directly with Python (no test runner configured in pyproject.toml).

## Architecture

### Three-Layer Architecture

1. **Server Layer** (`server.py`)
   - MCP protocol implementation using `mcp.server.stdio`
   - Tool registration and routing
   - Initialization of dependencies (HubSpot client, FAISS manager, embedding model)
   - Single entry point that composes all handlers

2. **Handler Layer** (`handlers/`)
   - Domain-specific handlers extending `BaseHandler`
   - Each handler manages MCP tool schemas and executes tool calls
   - Handlers: `CompanyHandler`, `ContactHandler`, `ConversationHandler`, `TicketHandler`, `SearchHandler`, `PropertyHandler`
   - Responsible for FAISS storage integration via `store_in_faiss_safely()`

3. **Client Layer** (`clients/`)
   - Direct HubSpot API interaction via `hubspot-api-client`
   - Domain-specific clients: `CompanyClient`, `ContactClient`, `ConversationClient`, `TicketClient`, `PropertyClient`
   - Composed by `HubSpotClient` which delegates to specialized clients

### Key Components

**FAISS Manager** (`faiss_manager.py`)
- Rolling storage by day (configurable max_days, default 7)
- Maintains separate indexes per date: `index_YYYY-MM-DD.faiss` and `metadata_YYYY-MM-DD.json`
- Automatically loads recent indexes on initialization and removes old ones
- Uses `faiss.IndexFlatL2` with configurable embedding dimension (default: 384)

**Embedding Model**
- Uses SentenceTransformer with `all-MiniLM-L6-v2` model (384 dimensions)
- Pre-downloaded in Docker image to `/app/models/all-MiniLM-L6-v2`
- Falls back to HuggingFace download if local model not found

**Storage System**
- Default storage directory: `/storage` (Docker) or `storage/` (local)
- Configurable via `HUBSPOT_STORAGE_DIR_LOCAL` environment variable
- `ThreadStorage` caches conversation threads in `storage/conversation_threads.json`
- FAISS indexes stored per-day for semantic search across HubSpot data

### Data Flow Pattern

1. Handler receives MCP tool call → validates arguments
2. Handler calls appropriate client method → gets HubSpot data
3. Handler stores data in FAISS → calls `store_in_faiss_safely()`
4. FAISS manager adds vectors and metadata → saves to disk
5. Handler returns formatted response to MCP client

### Important Patterns

**Error Handling**
- `ApiException` from `hubspot.crm.contacts.exceptions` is caught at server level
- Handlers use `core/error_handler.py` for consistent error responses
- All FAISS operations wrapped in try-catch with logging

**Formatters**
- `core/formatters.py` provides `convert_datetime_fields()` for HubSpot datetime normalization
- Used throughout clients to ensure consistent date formatting

**Duplicate Prevention**
- Contact and company creation include duplicate checking before API calls
- Uses property-based matching (email for contacts, domain/name for companies)

## Environment Variables

- `HUBSPOT_ACCESS_TOKEN` (required) - HubSpot API access token
- `HUBSPOT_STORAGE_DIR_LOCAL` (optional) - Override default storage directory, defaults to `/storage`

## Dependencies

Core dependencies (from pyproject.toml):
- `mcp>=1.4.1` - Model Context Protocol SDK
- `hubspot-api-client>=11.1.0` - Official HubSpot API client
- `faiss-cpu>=1.7.4` - Vector similarity search
- `sentence-transformers>=2.2.2` - Embedding generation
- `huggingface-hub==0.14.1` - Model downloads (pinned version)
- `python-dotenv>=1.0.1` - Environment variable loading

## HubSpot API Scopes Required

The access token must have these scopes:
- `crm.objects.contacts` (read/write)
- `crm.objects.companies` (read/write)
- `sales-email-read`

## MCP Tools Exposed

### Contact Tools
- `hubspot_create_contact` - Create contacts with duplicate prevention
- `hubspot_get_active_contacts` - Retrieve most recently active contacts
- `hubspot_get_contact` - Get a specific contact by ID (with optional properties filter)
- `hubspot_update_contact` - Update an existing contact record

### Company Tools
- `hubspot_create_company` - Create companies with duplicate prevention
- `hubspot_get_company_activity` - Retrieve activity for specific companies
- `hubspot_get_active_companies` - Retrieve most recently active companies
- `hubspot_get_company` - Get a specific company by ID (with optional properties filter)
- `hubspot_update_company` - Update an existing company record

### Conversation Tools
- `hubspot_get_recent_conversations` - Retrieve conversation threads with messages

### Ticket Tools
- `hubspot_get_tickets` - Get tickets with configurable criteria ("default" or "Closed")
- `hubspot_get_ticket_conversation_threads` - Get conversation threads for a specific ticket

### Property Tools
- `hubspot_get_property` - Get details of a specific HubSpot property definition
- `hubspot_update_property` - Update a HubSpot property definition (e.g., add dropdown options)
- `hubspot_create_property` - Create a new custom property in HubSpot

### Search Tools
- `hubspot_search_data` - Semantic search across stored HubSpot data via FAISS

---
> Source: [baryhuang/mcp-hubspot](https://github.com/baryhuang/mcp-hubspot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
