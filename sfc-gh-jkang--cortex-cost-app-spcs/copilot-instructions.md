## cortex-cost-app-spcs

> **PROJECT**: Advanced Cortex usage analytics dashboard with React 19 + TypeScript frontend and Flask backend, designed for SPCS deployment.

<!-- markdown -->
# Snowflake Cortex Usage Analytics Dashboard - Cursor AI Rules

## SPCS React + Flask Application Template

**PROJECT**: Advanced Cortex usage analytics dashboard with React 19 + TypeScript frontend and Flask backend, designed for SPCS deployment.

**REFERENCE**: Based on proven SPCS patterns from https://github.com/sfc-gh-ujagtap/sun_valley_spcs

## Current Project Structure

```
cortex-usage-dashboard/
├── README.md                    # Comprehensive 1800+ line documentation
├── server.py                    # Flask server with Cortex analytics API endpoints
├── package.json                 # React 19 + latest dependencies
├── pyproject.toml              # Python dependencies with uv
├── uv.lock                     # Locked Python dependencies
├── tsconfig.json               # TypeScript configuration
├── Dockerfile                  # Multi-stage Docker build (React + Flask)
├── deploy.sh                   # 🚀 ONE-COMMAND deployment script with --update mode
├── docker-compose.yml          # Local development setup
├── env.example                 # Environment template
├── test-local-container.sh     # Container testing script
├── src/                        # React application source
│   ├── App.tsx                 # Main app with dashboard layout
│   ├── components/
│   │   ├── Dashboard.tsx       # Main analytics dashboard with Recharts
│   │   ├── ErrorBoundary.tsx   # Error handling component
│   │   └── ThemeSelector.tsx   # Theme switching component
│   ├── contexts/
│   │   └── ThemeContext.tsx    # Theme management context
│   ├── types/
│   │   └── index.ts           # TypeScript interfaces
│   └── themes.css             # Theme-aware styling
├── scripts/                   # Database setup scripts
│   ├── create_app_role.sql    # Creates APP_SPCS_ROLE with permissions
│   └── setup_database.sql     # Database setup for Cortex analytics
├── snowflake/                 # SPCS deployment files
│   ├── deploy.sql            # Service deployment with resource specs
│   ├── manage_service.sql    # Service management commands
│   ├── setup_image_repo.sql  # Image repository configuration
│   ├── snowflake_utils.py    # Snowflake connection utilities
│   └── snowflake_queries.py  # Cortex analytics queries
└── test/                     # Testing utilities
    ├── mock-spcs/token       # Mock SPCS token for local testing
    └── nginx.conf           # nginx configuration for container testing
```

### Core Architecture Principles

1. **Use Flat Project Structure** 
   - ✅ Use: Root-level React app with single `package.json`
   - Pattern: `src/`, `public/`, `build/`, `server.py` all at project root
   - Flask serves both API routes AND static React build files
   - Beware of `manifest.json` causing Content Security Policy errors in SPCS

2. **Port Strategy - Always 3002**
   - Use port 3002 consistently across ALL environments
   - Local development, Docker, SPCS service spec - all port 3002

3. **TypeScript Over JavaScript**
   - For React frontend, prefer TypeScript: convert `.js` to `.tsx`
   - Add `tsconfig.json` from https://github.com/sfc-gh-ujagtap/sun_valley_spcs

## Building Phase

### Flask Server Configuration

**Current Implementation** (`server.py`):
- ✅ **Snowpark Session Management** with dual authentication (SPCS OAuth + Local credentials)
- ✅ **Cortex Analytics API** endpoints for dashboard data
- ✅ **Enhanced Error Handling** with VPN connection checks
- ✅ **CORS enabled** for cross-origin requests
- ✅ **Static file serving** for React build files
- ✅ **Health monitoring** with detailed connection status

**API Endpoints**:
- `/api/health` - Health check with environment detection
- `/api/overall-costs` - Credits usage over time grouped by service
- `/api/costs-by-user` - Usage by user with totals and latest activity
- `/api/summary` - Aggregated totals, unique users, service breakdown, date range
- `/api/all-data` - Optimized bundle returning overall_costs, costs_by_user, summary + cache_info
- `/api/cache-info` - Cache timeout and last Snowflake pull time (no refresh)
- `/api/debug` - Environment and configuration diagnostics
- `/api/debug-users` - Unique user analysis for troubleshooting
- `/api/test-table` - Simple connectivity/schema probe

**Flask Integration Pattern** (CRITICAL):
```python
import sys
import os
sys.path.append(os.path.join(os.path.dirname(__file__), 'snowflake'))
from snowflake_utils import get_snowflake_session, is_running_in_spcs

from flask import Flask, jsonify, send_from_directory
import logging
import traceback

app = Flask(__name__, static_folder='build', static_url_path='')
logger = logging.getLogger(__name__)

def get_snowpark_session():
    """Get Snowpark session with enhanced logging"""
    try:
        logger.info("🔗 Attempting to create Snowpark session...")
        
        # Log environment variables (without sensitive values)
        env_vars = ['SNOWFLAKE_ACCOUNT', 'SNOWFLAKE_WAREHOUSE', 'SNOWFLAKE_DATABASE', 'SNOWFLAKE_SCHEMA', 'SNOWFLAKE_ROLE']
        missing_vars = []
        for var in env_vars:
            value = os.environ.get(var, 'NOT SET')
            logger.info(f"   {var}: {'SET' if value != 'NOT SET' else 'NOT SET'}")
            if value == 'NOT SET':
                missing_vars.append(var)
        
        # Check for required environment variables
        if missing_vars and not is_running_in_spcs():
            logger.error(f"❌ Missing required environment variables for local connection: {', '.join(missing_vars)}")
            logger.error("   Please set these environment variables or run in SPCS environment")
            return None
        
        session = get_snowflake_session()
        if session is None:
            logger.error("❌ get_snowflake_session() returned None")
            return None
            
        logger.info("✅ Snowpark session created successfully")
        return session
    except ConnectionError as e:
        # Re-raise ConnectionError to preserve VPN message
        logger.error(f"❌ Connection failed: {str(e)}")
        raise
    except Exception as e:
        logger.error(f"❌ Failed to create Snowpark session: {str(e)}")
        logger.error(f"   Traceback: {traceback.format_exc()}")
        return None

def pull_data_snowflake(query):
    """Execute query using Snowpark session"""
    session = get_snowpark_session()
    if session is None:
        return []
    
    try:
        result = session.sql(query).collect()
        logger.info(f"✅ Query executed successfully, returned {len(result)} rows")
        return result
    except Exception as e:
        logger.error(f"❌ Query execution failed: {str(e)}")
        return []

@app.route('/api/health')
def health():
    try:
        session = get_snowpark_session()
        if session is None:
            return jsonify({'status': 'ERROR', 'message': 'Cannot connect to Snowflake'}), 500
        
        # Test connection with simple query
        result = session.sql("SELECT CURRENT_VERSION()").collect()
        return jsonify({
            'status': 'OK', 
            'snowflake_connected': True,
            'environment': 'SPCS' if is_running_in_spcs() else 'Local'
        })
    except ConnectionError as e:
        return jsonify({
            'status': 'ERROR', 
            'message': 'Connection failed - Please ensure you are connected to VPN',
            'error': str(e)
        }), 500
    except Exception as e:
        return jsonify({
            'status': 'ERROR', 
            'message': 'Health check failed',
            'error': str(e)
        }), 500

@app.route('/api/example-data')
def example_data():
    try:
        query = "SELECT CURRENT_TIMESTAMP() as timestamp, CURRENT_VERSION() as version"
        data = pull_data_snowflake(query)
        return jsonify({'data': [row.as_dict() for row in data]})
    except Exception as e:
        logger.error(f"Error in example_data: {str(e)}")
        return jsonify({'error': 'Failed to fetch data'}), 500

# Static React build and routing
@app.route('/')
def index():
    return send_from_directory('build', 'index.html')

@app.route('/<path:path>')
def static_proxy(path):
    full_path = os.path.join('build', path)
    if os.path.isfile(full_path):
        return send_from_directory('build', path)
    return send_from_directory('build', 'index.html')

if __name__ == '__main__':
    port = int(os.environ.get('PORT', '3002'))
    print(f"Available via {'SPCS endpoint' if is_running_in_spcs() else f'http://0.0.0.0:{port}'}")
    app.run(host='0.0.0.0', port=port)
```

### Database Setup
- Look at user prompt to see if the app needs access to the existing database 
- Implement sample data if no database is provided in the prompt
- Keep any database setup in a single script file
- Use `CREATE IF NOT EXISTS` for idempotency

###Warehouse & SPCS Service Setup
- Look for the warehouse to use in the prompt
- If no warehouse is specified, create a new one and use it for creating and accessing data
- Create deploy.sql to create a service with inline specs including the warehouse used
- Create separate files for image repo setup, service setup and managing service under spcs folder
 
### Role Access
- Every app should provision a new account level role named specifically for the app
- This role should own the SPCS service, warehouse and sample dataset
- This role should also be granted any existing databases provided in the prompt
- SPCS service should access the data with the provisioned role
- This role should be granted to the current user

### Dual Authentication Setup in Flask Server for Data Access

**CRITICAL**: Use this proven 3-environment detection pattern for robust Snowflake connections:

```python
import os
import logging
from snowflake.snowpark.session import Session
from typing import Optional

logger = logging.getLogger(__name__)

def is_running_in_spcs():
    """
    Checks if the current environment is Snowpark Container Services (SPCS)
    by looking for the Snowflake session token file and verifying it's not a mock.
    """
    snowflake_token_path = "/snowflake/session/token"
    if not os.path.exists(snowflake_token_path):
        return False
    
    # Check if it's a mock token (for local container testing)
    try:
        with open(snowflake_token_path, "r") as f:
            token_content = f.read().strip()
            # Mock tokens contain "mock" in them
            if "mock" in token_content.lower():
                logger.info("Detected mock SPCS token - treating as local environment")
                return False
    except Exception:
        # If we can't read the token, assume it's real SPCS
        pass
    
    return True

def get_login_token():
    """Get the login token from the Snowflake session token file."""
    with open("/snowflake/session/token", "r") as f:
        return f.read()

def get_snowflake_session() -> Optional[Session]:
    """Create a Snowflake session using environment variables."""
    try:
        if is_running_in_spcs():
            logger.info("Detected SPCS environment - using token authentication")
            connection_parameters = {
                "host": os.getenv("SNOWFLAKE_HOST"),
                "account": os.getenv("SNOWFLAKE_ACCOUNT"),
                "token": get_login_token(),
                "authenticator": "oauth",
                "database": os.getenv("SNOWFLAKE_DATABASE"),
                "schema": os.getenv("SNOWFLAKE_SCHEMA", "PUBLIC"),
                "warehouse": os.getenv("SNOWFLAKE_WAREHOUSE", "COMPUTE_WH"),
            }
            session = Session.builder.configs(connection_parameters).create()
            logger.info("Successfully connected to Snowflake via SPCS token authentication")
        else:
            logger.info("Detected local environment - using credential authentication")
            connection_parameters = {
                "account": os.getenv("SNOWFLAKE_ACCOUNT"),
                "user": os.getenv("SNOWFLAKE_USER"),
                "password": os.getenv("SNOWFLAKE_PASSWORD"),
                "warehouse": os.getenv("SNOWFLAKE_WAREHOUSE", "COMPUTE_WH"),
                "database": os.getenv("SNOWFLAKE_DATABASE"),
                "schema": os.getenv("SNOWFLAKE_SCHEMA", "PUBLIC"),
            }
            session = Session.builder.configs(connection_parameters).create()
            logger.info("Successfully connected to Snowflake locally")
        return session
    except Exception as e:
        if is_running_in_spcs():
            logger.error(f"Error connecting to Snowflake in SPCS environment: {e}")
        else:
            logger.error(f"Error connecting to Snowflake in local environment: {e}")
            logger.error("🔒 LOCAL CONNECTION FAILED - Please ensure you are connected to VPN!")
            logger.error("   Common causes:")
            logger.error("   1. Not connected to corporate VPN")
            logger.error("   2. Invalid Snowflake credentials")
            logger.error("   3. Network connectivity issues")
            logger.error("   4. Snowflake account URL incorrect")
            raise ConnectionError(
                "Failed to connect to Snowflake from local environment. "
                "Please ensure you are connected to VPN and have valid credentials. "
                f"Original error: {str(e)}"
            )
        return None
```

**Key Features**:
- **3-Environment Detection**: Real SPCS, Local Container (mock token), Local Development
- **Mock Token Detection**: Identifies test containers vs. real SPCS deployment
- **Enhanced Error Handling**: Specific VPN connection guidance for local failures
- **Proper Session Management**: Uses Snowpark Session instead of raw connector
- **Environment-Specific Logging**: Clear indication of which authentication method is used

**Environment Variables Required**:
```bash
# For ALL environments
SNOWFLAKE_ACCOUNT=your-account.snowflakecomputing.com
SNOWFLAKE_DATABASE=your_database
SNOWFLAKE_SCHEMA=your_schema
SNOWFLAKE_WAREHOUSE=your_warehouse

# For LOCAL development only
SNOWFLAKE_USER=your_username
SNOWFLAKE_PASSWORD=your_password

# For SPCS only (automatically provided)
SNOWFLAKE_HOST=your-host.snowflakecomputing.com
# Token file: /snowflake/session/token
```

**Mock Token Setup for Local Container Testing**:
```bash
# Create test/mock-spcs/token with mock content
mkdir -p test/mock-spcs
echo "mock-token-for-local-testing" > test/mock-spcs/token

# Mount in docker-compose.yml or docker run
docker run -v $(pwd)/test/mock-spcs:/snowflake/session ...
```

### Essential Endpoints
- **Health check**: `@app.route('/api/health')` returns `jsonify({'status': 'OK'})`
- **Static files**: `app = Flask(__name__, static_folder='build', static_url_path='')`
- **React routing**: catch-all routes serving `send_from_directory('build', 'index.html')`
- ⚠️ **Never hardcode localhost** - use dynamic container detection for environment-specific behavior

### React Frontend Features

**Current Implementation**:
- ✅ **React 19** with TypeScript for type safety
- ✅ **Recharts 3.2.1** for professional data visualizations
- ✅ **Cortex Analytics Dashboard** with real-time data
- ✅ **Multi-theme support** (Light/Dark/Auto) with ThemeContext
- ✅ **Responsive design** with mobile-optimized charts
- ✅ **Error boundaries** for graceful error handling
- ✅ **Loading states** for all data fetching operations

**Chart Components**:
- 📊 **Service Breakdown** - Pie chart with intelligent text wrapping
- 📈 **Credits Usage Trend** - Area chart with formatted dates (Day of Week, Day Month Year)
- 🎯 **Interactive tooltips** with detailed information
- 📱 **Mobile-responsive** with optimized label sizing

**Key Features**:
- Smart date formatting without timezone/time display
- Intelligent text wrapping for long service names (no truncation)
- Dynamic X-axis spacing to prevent overcrowding
- TypeScript interfaces in `src/types/index.ts`
- Theme-aware CSS with `themes.css`

## Testing Phase
### Local Development
1. **Build process**: Ensure `npm run build` creates clean `build/` directory
2. **Test locally**: Verify React app loads with proper styling and functionality
3. **Database connection**: Test both local and mock data scenarios
4. **API endpoints**: Verify all `/api/*` routes work correctly

### Docker Testing
1. **Build locally**: `docker build --platform linux/amd64 -t app-name:latest .`
2. **Run container**: `docker run -p 3002:3002 app-name:latest`
3. **Verify health**: Access `/api/health` endpoint on container port 3002
4. **Test full app**: Verify React app loads and functions correctly

## Deployment Phase

### Multi-stage Dockerfile
```dockerfile
# Builder stage (React)
FROM node:22-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --legacy-peer-deps
COPY . .
RUN npm run build

# Production stage (Flask)
FROM python:3.13-slim AS production
RUN apt-get update && apt-get install -y --no-install-recommends dumb-init curl \
    && rm -rf /var/lib/apt/lists/*
RUN groupadd -g 1001 appgroup && useradd -u 1001 -g appgroup -m appuser

# Install uv
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/uv

WORKDIR /app

# Install Python deps with uv
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-cache --no-dev

# Copy server and static build
COPY --from=builder /app/build ./build/
COPY server.py ./server.py

USER appuser
EXPOSE 3002
ENV PORT=3002 PYTHONUNBUFFERED=1
ENTRYPOINT ["dumb-init", "--"]
CMD ["uv", "run", "gunicorn", "--bind", "0.0.0.0:3002", "server:app", "--workers", "2", "--threads", "4", "--timeout", "120"]
```

### Deployment Commands & Ingress URL Management

**CRITICAL DEPLOYMENT WORKFLOW**:

#### 🚀 First Deployment:
```bash
./deploy.sh
```
- Creates new service with NEW ingress URL
- Sets up complete infrastructure

#### 🔄 All Subsequent Deployments:
```bash
./deploy.sh --update
```
- ✅ **PRESERVES existing ingress URL** 
- Updates code without recreating service
- Maintains bookmarks and shared links

#### 📋 Available Deploy Modes:
- `./deploy.sh` or `./deploy.sh --spcs` - Full deployment (NEW URL)
- `./deploy.sh --update` - Update existing service (SAME URL) ⭐
- `./deploy.sh --local` - Local development mode

#### ⚠️ ALWAYS USE --update AFTER FIRST DEPLOYMENT:
- Without `--update`: Service gets DROP/CREATE → new ingress URL
- With `--update`: Service gets SUSPEND/RESUME → same ingress URL

#### 🛠️ Deployment Script Features:
- Single deployment script `deploy.sh` with multi-mode support
- Sets up app role (APP_SPCS_ROLE) and warehouse
- Configures sample Cortex analytics dataset
- Sets up image repository and builds/pushes Docker image
- Waits for endpoints to be available
- Provides complete management commands after deployment
- Uses `-c default` flag in all snowsql commands

Note: Ensure a `pyproject.toml` exists with at least:
```toml
[project]
name = "spcs-app"
version = "0.1.0"
description = "SPCS Flask React App"
requires-python = ">=3.13"
dependencies = [
    "flask>=3.1.2",
    "gunicorn>=23.0.0",
    "snowflake-connector-python>=3.17.4",
    "snowflake-snowpark-python>=1.39.0",
]

[build-system]
requires = ["uv_build>=0.8.17,<0.9.0"]
build-backend = "uv_build"


[tool.uv]
dev-dependencies = []
```

## Quick Development Commands

### 🚀 Deployment (MOST IMPORTANT):
See "CRITICAL DEPLOYMENT WORKFLOW" section above for detailed deployment instructions.

### 🛠️ Local Development:
```bash
# Install dependencies
npm install
uv sync

# Start development servers
npm start                    # React dev server (port 3000)
./deploy.sh --local         # Flask + React (port 3002)
npm run dev                 # Flask server only (port 3002)

# Build for production
npm run build               # Creates build/ directory

# Container testing
npm run docker:build        # Build Docker image
npm run docker:run          # Run container locally
./test-local-container.sh   # Full container test
```

### 🔍 SPCS Management Commands:
```bash
# Get service endpoint
snow sql -q "SHOW ENDPOINTS IN SERVICE DB.SCHEMA.SERVICE;" --connection default

# Check service status  
snow sql -q "SELECT SYSTEM\$GET_SERVICE_STATUS('DB.SCHEMA.SERVICE');" --connection default

# View service logs
snow sql -q "SELECT SYSTEM\$GET_SERVICE_LOGS('DB.SCHEMA.SERVICE', '0', 'sales-analytics-app', 100);" --connection default

# Suspend/Resume service
snow sql -q "ALTER SERVICE DB.SCHEMA.SERVICE SUSPEND;" --connection default
snow sql -q "ALTER SERVICE DB.SCHEMA.SERVICE RESUME;" --connection default
```

## 🚨 **CRITICAL RULES - DO NOT VIOLATE**

### **NEVER DELETE .env FILES**
- ❌ **NEVER** run `rm .env` or `delete_file .env` 
- ❌ **NEVER** automatically delete environment files
- ✅ **ALWAYS** ask user before touching .env files
- ✅ **REMEMBER**: .env files are in .gitignore for a reason
- ✅ **UNDERSTAND**: .env files contain user's working credentials
- 📝 **RATIONALE**: Users need their .env files for local development

### **File Deletion Policy**
- ❌ **NEVER** auto-delete user configuration files
- ❌ **NEVER** run `rm` commands without explicit user request
- ✅ **ALWAYS** ask permission before deleting ANY file
- ✅ **PREFER** adding to .gitignore over deleting

### **Commit and Push Policy (Cursor MUST Ask First)**
- ❌ **NEVER** auto-run `git add`, `git commit`, or `git push`.
- ❌ **NEVER** enable or rely on any auto-commit/auto-push features.
- ✅ **ALWAYS** ask for explicit user approval before any commit or push.
- ✅ Before committing, present:
  - A concise summary of changed files and key edits
  - A proposed commit message
  - A confirmation prompt: “Proceed with commit?”
- ✅ Before pushing, explicitly confirm: branch, remote, and whether CI will trigger.
- ✅ Default to a single cohesive commit per logical change unless user instructs otherwise.
- ❌ **DO NOT** amend, squash, or rewrite history without user approval.
- ❌ **DO NOT** change Git config, bypass hooks, or force-push without approval.
- ✅ If a script includes Git operations, pause and request confirmation before executing them.

## Recent Updates & Best Practices

### ✅ **React 19 + Latest Dependencies** (Updated 2024):
- React 19.1.1 with latest TypeScript 5.9.2
- Recharts 3.2.1 with built-in TypeScript definitions (no @types/recharts needed)
- ESLint 9.36.0 with modern flat config and typescript-eslint 8.44.1
- Vite 7.1.7 build system (migrated from deprecated react-scripts)
- All dependencies at latest versions with zero deprecation warnings

### ✅ **Modern Build System**:
- Production builds use Rolldown (`rolldown.config.js`) via `npm run build`
- Vite is used for dev/preview (`npm start`, `vite preview`)
- Multi-stage Docker build: Node.js 22 builder → Python 3.13 production
- Build time: ~5s local, ~20s Docker (vs previous 8+ minutes)
- Zero npm deprecation warnings in Docker builds

### ✅ **Dual Deployment Support (SPCS + Native App)**:
- SPCS deployment: `./deploy.sh` (first time) and `./deploy.sh --update` (subsequent)
- Native App deployment: `./deploy-native-app.sh` with `--skip-docker` and `--force` options
- Native App requires ACCOUNTADMIN or equivalent application package privileges
- README includes full Native App architecture, permissions, and management commands

### ✅ **Enhanced Error Handling**:
- VPN connection checks for local development
- Graceful handling of `None` Snowpark sessions
- User-friendly error messages in UI

### ✅ **Chart Improvements**:
- Intelligent text wrapping (no truncation)
- Smart date formatting (Day of Week, Day Month Year)
- Dynamic X-axis spacing for readability
- Mobile-responsive design

### ✅ **Deployment Workflow**:
- `--update` flag preserves ingress URLs
- Complete management commands provided post-deployment
- Comprehensive README.md with 1800+ lines of documentation

---
> Source: [sfc-gh-jkang/cortex-cost-app-spcs](https://github.com/sfc-gh-jkang/cortex-cost-app-spcs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
