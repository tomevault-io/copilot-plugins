## taskme

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RAGFlow is an open-source RAG (Retrieval-Augmented Generation) engine. This repository contains "TaskMe" - an enhanced version based on RAGFlow v0.17 that focuses on knowledge base management and intelligent Agent functionality, with traditional chat features removed.

## Architecture

### Backend Structure
- **Framework**: Python Flask + Gunicorn
- **API Apps**: Located in `/api/apps/` with dynamic Blueprint registration
- **Core Engine**: RAG engine in `/rag/` directory
- **Services**: Database services in `/api/db/services/`

### Frontend Structure  
- **Framework**: React 18 + UmiJS 4 + Ant Design 5
- **State Management**: Zustand + React Query
- **Styling**: Less + TailwindCSS
- **Key Pages**: `/web/src/pages/` (knowledge, agent, file-manager, etc.)

### Database Architecture
- **MySQL**: Structured data (users, knowledge bases, conversations)
- **Elasticsearch/Infinity**: Document vector search
- **Redis**: Session management and caching
- **MinIO**: File storage

## Development Commands

### Server Startup
```bash
# Development mode (requires virtual environment)
source .venv/bin/activate
export PYTHONPATH=$(pwd)
./start_server.sh

# Production mode
gunicorn -c gunicorn.conf.py "api.ragflow_server:create_app()"

# Docker deployment
cd docker/
docker compose up -d
```

### Frontend Development
```bash
cd web/
npm install
npm run dev      # Development server
npm run build    # Production build
npm test         # Run tests
```

### Testing and Validation
```bash
# Python tests
pytest

# Test server startup (critical for validating changes)
source .venv/bin/activate && ./start_server.sh
```

## Critical Development Notes

### Development Server Status
**NOTE**: The development servers (both backend and frontend) are currently running in reload mode. Code changes are automatically reloaded without requiring manual server restart. Testing can be done immediately after making changes.

### Server Startup Testing
**IMPORTANT**: When servers are not running, always test server startup after making changes to the codebase. Many import errors and configuration issues only surface during server initialization. Use `./start_server.sh` to validate that all modules load correctly.

### Proxy Configuration
The startup script includes proxy cleanup to avoid SOCKS proxy issues that can cause import failures with httpx/ollama dependencies.

### API App Registration
- Apps in `/api/apps/` are dynamically loaded by `/api/apps/__init__.py`
- Each app file gets a `manager` Blueprint assigned at runtime via `register_page()`
- Never define `manager = None` in app files - this causes import errors
- Use `@manager.route()` decorators with `# noqa: F821` to suppress linting

### Table Knowledge Base Feature
New functionality for database/Excel knowledge bases:
- **Backend**: `/api/apps/table_kb_app.py`, `/rag/table_knowledge/`
- **Frontend**: `/web/src/pages/table-knowledge-base/`, `/web/src/components/table-knowledge-base/`
- **API Routes**: `/v1/kb/table/*` endpoints

### Configuration Files
- **Main Config**: `/conf/service_conf.yaml` (from template)
- **Frontend Config**: `/web/src/conf.json`
- **Environment**: Docker `.env` files
- **Dependencies**: `pyproject.toml` for Python, `package.json` for Node.js

## Project Structure Highlights

### Key Directories
```
/api/apps/           # Flask Blueprint applications
/rag/               # Core RAG engine and processing
/web/src/pages/     # React page components  
/web/src/services/  # API service layer
/conf/              # Configuration templates
/docker/            # Container deployment files
```

### Database Models
Located in `/api/db/db_models.py` with migration support. Notable additions include:
- `storage_type` and `table_config` fields in Knowledgebase model
- `TableKnowledgeBaseMetadata` model for enhanced table KB context

### Service Layer
Database services in `/api/db/services/` follow consistent patterns:
- Permission checks via `accessible()` and `accessible4deletion()`
- Error handling with `get_data_error_result()` and `server_error_response()`
- Standardized JSON responses via `get_json_result()`

## Development Environment

### Requirements
- **Python**: 3.10-3.12
- **Node.js**: >=18.20.4  
- **System**: CPU>=4 cores, RAM>=16GB, Disk>=50GB
- **Docker**: >=24.0.0 for container deployment

### Virtual Environment
The project uses `.venv/` for Python virtual environment. Always activate before development and ensure `PYTHONPATH` is set to the project root.

## Frontend-Backend Interaction Architecture

### API Design Patterns

#### 1. URL Structure and Routing
RAGFlow follows RESTful API design principles with consistent URL patterns:

```
Base URL: /v1/{module}/{action}
Examples:
- /v1/kb/list              # Knowledge base list
- /v1/kb/create            # Create knowledge base  
- /v1/kb/table/query       # Table KB query
- /v1/user/info            # User information
- /v1/llm/factories        # LLM factory list
```

**URL Pattern Rules:**
- Always prefixed with `/v1` for API versioning
- Module-based organization (`kb`, `user`, `llm`, `flow`, etc.)
- RESTful actions (`list`, `create`, `update`, `delete`, `detail`)
- Hierarchical for sub-modules (e.g., `/kb/table/*` for table knowledge bases)

#### 2. HTTP Methods and Actions
- **GET**: Data retrieval (lists, details, statistics)
- **POST**: Data creation and complex operations (queries, analysis)
- **PUT/PATCH**: Data updates (not commonly used, POST is preferred)
- **DELETE**: Resource deletion (also uses POST with action endpoints)

#### 3. Request/Response Data Format

**Standard Response Structure:**
```typescript
interface APIResponse<T> {
  code: number;          // Status code (0 = success, >0 = error)
  message: string;       // Human-readable message
  data?: T;             // Response payload
}
```

**Backend Response Utilities:**
```python
# Success response
get_json_result(code=0, message='success', data=result)

# Error responses  
get_data_error_result(code=RetCode.DATA_ERROR, message='Data missing')
server_error_response(exception)
```

**Frontend Request Configuration:**
```typescript
// Service layer configuration
const methods = {
  createKb: {
    url: create_kb,
    method: 'post',
  },
  getList: {
    url: kb_list, 
    method: 'get',
  }
};
```

### Authentication and Authorization

#### 1. Token-Based Authentication
- **Frontend**: Bearer token in Authorization header
- **Backend**: JWT token validation with double-decode fallback
- **Storage**: Session-based storage with filesystem sessions

#### 2. Permission System
- **User Roles**: Different access levels for knowledge bases
- **Access Control**: 
  - `accessible()` - Read permissions
  - `accessible4deletion()` - Write/admin permissions
- **Team Permissions**: Support for team-shared resources

### Error Handling Strategy

#### 1. Backend Error Handling
```python
# Centralized error handling
@app.errorhandler(Exception)
def handle_error(e):
    return server_error_response(e)

# Consistent error codes
class RetCode:
    SUCCESS = 0
    DATA_ERROR = 100
    AUTHENTICATION_ERROR = 109
    EXCEPTION_ERROR = 500
```

#### 2. Frontend Error Processing
```typescript
// Global error handler in request interceptor
const errorHandler = (error) => {
  // Handle different error types
  // Show user-friendly messages
  // Redirect to login if needed
};
```

### Frontend Service Layer Architecture

#### 1. Service Registration Pattern
```typescript
// Register services with unified request handler
const kbService = registerServer<keyof typeof methods>(methods, request);

// Usage: kbService.createKb(params)
```

#### 2. Hook-Based State Management
```typescript
// React Query hooks for data fetching
export const useFetchKnowledgeList = () => {
  const { data, isFetching } = useQuery({
    queryKey: ['fetchKnowledgeList'],
    queryFn: async () => {
      const { data } = await kbService.getList();
      return data?.data?.kbs ?? [];
    },
  });
  return { list: data, loading: isFetching };
};
```

### Backend App Organization

#### 1. Dynamic Blueprint Registration
```python
# Automatic app discovery and registration
def register_page(page_path):
    page.manager = Blueprint(page_name, module_name)
    app.register_blueprint(page.manager, url_prefix=url_prefix)
    
# URL prefix pattern: /v1/{app_name}
```

#### 2. Database Connection Management
```python
@app.before_request
def _db_connect():
    if DB.is_closed():
        DB.connect()

@app.teardown_request  
def _db_close(exc):
    if not DB.is_closed():
        DB.close()
```

### Key Integration Patterns

#### 1. Request Validation
```python
# Decorator-based validation
@validate_request("kb_id", "name", "description")
@not_allowed_parameters("id", "tenant_id")
def update_kb():
    # Function implementation
```

#### 2. CORS Configuration
```python
CORS(app, supports_credentials=True, 
     resources={r"/*": {
         "origins": ["http://localhost:8000", "http://localhost:9380"],
         "methods": ["GET", "POST", "PUT", "DELETE", "OPTIONS"]
     }})
```

#### 3. Frontend Request Interceptors
```typescript
// Snake case conversion for API compatibility
const convertedData = convertTheKeysOfTheObjectToSnake(requestData);

// Authorization header injection
headers: { Authorization: `Bearer ${token}` }
```

### Development Best Practices

#### 1. API Versioning
- All APIs use `/v1` prefix for future compatibility
- Module-based organization for clear separation of concerns

#### 2. Type Safety
- TypeScript interfaces for all API responses
- Consistent error code definitions
- Proper type inference in React hooks

#### 3. Performance Optimization
- React Query for caching and background updates
- Debounced search operations
- Infinite scroll for large datasets

#### 4. Development Tools
- Swagger UI available at `/apidocs/` for API documentation
- Health check endpoint at `/health`
- Comprehensive logging for debugging

---
> Source: [zalan159/taskme](https://github.com/zalan159/taskme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
