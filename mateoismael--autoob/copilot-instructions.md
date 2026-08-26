## autoob

> This document provides comprehensive guidance for AI assistants working with the SEACE Scraper codebase.

# CLAUDE.md - AI Assistant Guide for SEACE Scraper

This document provides comprehensive guidance for AI assistants working with the SEACE Scraper codebase.

## Project Overview

**SEACE Scraper** (Automatizador de Búsqueda SEACE) is a full-stack web application that automates the discovery of Peruvian government procurement contracts. It scrapes the SEACE portal (Sistema Electrónico de Contrataciones del Estado) to find technology and goods/services contracts under 8 UIT (Unit of Tax Reference) and provides a time-based color-coded alert system for deadline tracking.

### Key Features
- Automated search for small government contracts (< 8 UIT)
- Color-coded time alerts: 🔴 Red (<24h), 🟡 Yellow (24-72h), 🟢 Green (>72h)
- Real-time web scraping with Playwright
- Modern React UI with responsive design
- RESTful API backend

### Project Status
- **Current State:** Fully functional with Playwright scraper enabled; uses public SEACE search (no login required)
- **Language:** Primarily Spanish documentation and UI
- **Environment:** Development-ready with complete backend setup, not production-deployed
- **Note:** Network connectivity required for scraping; may not work in sandboxed environments

---

## Architecture

### Tech Stack

**Backend (Python)**
- **Framework:** FastAPI 0.121.2 (modern async web framework)
- **Server:** Uvicorn 0.38.0 (ASGI server, port 8002)
- **Scraping:** Playwright 1.56.0 (headless browser automation)
- **Validation:** Pydantic 2.12.4 (type-safe data models)
- **Config:** python-dotenv 1.0.1 (environment variables)
- **Python Version:** 3.9+

**Frontend (TypeScript/React)**
- **Framework:** React 19.1.0 (latest with new features)
- **Language:** TypeScript 5.8.3 (strict mode enabled)
- **Build Tool:** Vite 7.0.4 (fast HMR, port 5173)
- **Styling:** Tailwind CSS 4.1.11 (utility-first CSS)
- **Linting:** ESLint 9.30.1 with TypeScript support
- **Node Version:** 18+

### System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    User Browser (localhost:5173)             │
└────────────────────────────┬────────────────────────────────┘
                             │
                    HTTP GET Request
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│              React Frontend (Vite Dev Server)                │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  App.tsx - Main Component                            │  │
│  │  - State management (useState hooks)                 │  │
│  │  - API calls to backend                              │  │
│  │  - Table rendering with color coding                 │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────────┬────────────────────────────────┘
                             │
                    fetch("/api/contrataciones")
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│          FastAPI Backend (Uvicorn on port 8002)              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  main.py - API Endpoints                             │  │
│  │  - CORS middleware (allows localhost:5173)           │  │
│  │  - GET /api/contrataciones                           │  │
│  │  - Currently returns mock data                       │  │
│  └──────────────────┬───────────────────────────────────┘  │
│                     │ (commented out)                       │
│                     ▼                                        │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  scraper.py - Playwright Web Scraper                 │  │
│  │  - SEACEScraper class                                │  │
│  │  - extraer_contrataciones() method                   │  │
│  │  - calcular_estado_tiempo() for alerts              │  │
│  │  - Headless Chrome automation                        │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────────┬────────────────────────────────┘
                             │
                    Scrapes data from
                             │
                             ▼
                  SEACE Portal (prod6.seace.gob.pe)
```

---

## File Structure

```
autoob/
├── .env.example              # Template for credentials (DO NOT commit .env)
├── .gitignore               # Comprehensive ignore rules (Python, Node, secrets)
├── README.md                # Main documentation (Spanish, 204 lines)
├── QUICKSTART.md            # Quick start guide (Spanish, 101 lines)
├── CLAUDE.md                # This file - AI assistant guide
├── package-lock.json        # Root npm lock file
│
├── backend/                 # Python FastAPI backend
│   ├── main.py              # FastAPI server (118 lines) ⭐ Entry point
│   ├── scraper.py           # Web scraping logic (175 lines) ⭐ Core functionality
│   ├── models.py            # Pydantic models (20 lines)
│   ├── requirements.txt     # Python dependencies
│   ├── .gitignore          # Backend-specific ignores
│   └── venv/               # Python virtual environment (gitignored)
│
└── frontend/                # React TypeScript frontend
    ├── src/
    │   ├── App.tsx          # Main React component (232 lines) ⭐ UI logic
    │   ├── main.tsx         # React DOM entry point (10 lines)
    │   ├── index.css        # Tailwind CSS imports (1 line)
    │   └── vite-env.d.ts    # Vite type definitions
    ├── public/              # Static assets
    ├── node_modules/        # npm dependencies (gitignored)
    ├── package.json         # npm configuration
    ├── package-lock.json    # npm lock file
    ├── vite.config.ts       # Vite build configuration
    ├── tsconfig.json        # TypeScript base config
    ├── tsconfig.app.json    # TypeScript app config (strict mode)
    ├── tsconfig.node.json   # TypeScript node config
    ├── eslint.config.js     # ESLint flat config
    ├── index.html           # HTML entry point
    └── README.md            # Frontend setup guide (Vite template)
```

### Key Files Explained

#### Backend Files

**`backend/main.py`** (118 lines)
- FastAPI application setup with OpenAPI/Swagger documentation
- CORS configuration allowing `localhost:5173`, `localhost:3000`, `127.0.0.1:5173`
- Three endpoints:
  - `GET /` - API info and endpoint listing
  - `GET /health` - Health check with timestamp
  - `GET /api/contrataciones` - Main endpoint (currently returns mock data)
- Windows + Python 3.13 compatibility fix: `asyncio.WindowsSelectorEventLoopPolicy()`
- Runs on port 8002 (configurable in line 118)

**`backend/scraper.py`** (175 lines)
- `SEACEScraper` class for web scraping
- Key methods:
  - `calcular_estado_tiempo(fecha_limite: str) -> str`: Calculates alert state (rojo/amarillo/verde)
  - `extraer_contrataciones() -> List[Contratacion]`: Extracts contract data from SEACE HTML
  - `scrape() -> List[Contratacion]`: Orchestrates Playwright browser automation
- Uses CSS selectors to parse SEACE portal HTML
- **Currently commented out** in main.py due to Playwright Windows compatibility issues

**`backend/models.py`** (20 lines)
- Pydantic models for type safety and validation:
  - `Contratacion`: Single contract with 8 fields
  - `ContratacionResponse`: API response wrapper with metadata
- Uses `Literal["rojo", "amarillo", "verde"]` for enum-like type safety

#### Frontend Files

**`frontend/src/App.tsx`** (232 lines)
- Main React component with all application logic
- State management using hooks:
  - `contrataciones`: Array of contracts
  - `loading`: Boolean for loading state
  - `error`: Error message string or null
  - `lastUpdate`: Timestamp of last fetch
- TypeScript features:
  - Literal types: `type EstadoTiempo = typeof ESTADOS_TIEMPO[number]`
  - `satisfies` operator for type-safe constants (line 57)
  - Strict null checks and type annotations
- UI sections:
  - Header with title
  - Control panel with "Actualizar" button and metadata
  - Error message display
  - Loading spinner
  - Data table with 6 columns
  - Empty state with SVG icon
- Color coding using Tailwind classes mapped to alert states
- Time formatting: converts hours to minutes/hours/days

**`frontend/vite.config.ts`** (8 lines)
- Vite configuration with two plugins:
  1. `@vitejs/plugin-react` - React Fast Refresh support
  2. `@tailwindcss/vite` - Tailwind CSS v4 integration

**`frontend/tsconfig.app.json`**
- Strict TypeScript configuration:
  - `noUnusedLocals: true` - Error on unused variables
  - `noUnusedParameters: true` - Error on unused parameters
  - `noFallthroughCasesInSwitch: true` - Prevent switch fallthrough bugs
  - `noUncheckedSideEffectImports: true` - Catch problematic imports

---

## Development Workflows

### Initial Setup

**Backend Setup**
```bash
cd backend

# Create virtual environment
python -m venv venv

# Activate virtual environment
# On Windows:
venv\Scripts\activate
# On Linux/Mac:
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Install Playwright browsers (required for scraping)
playwright install chromium

# Create .env file in root (not in backend/)
cd ..
cp .env.example .env
# Edit .env with SEACE credentials
```

**Frontend Setup**
```bash
cd frontend

# Install dependencies
npm install

# No additional configuration needed
```

### Running the Application

**Start Backend** (Terminal 1)
```bash
cd backend
source venv/bin/activate  # or venv\Scripts\activate on Windows
python main.py
```
- Backend runs on `http://localhost:8002`
- Auto-reload enabled (changes trigger restart)
- OpenAPI docs available at `http://localhost:8002/docs`

**Start Frontend** (Terminal 2)
```bash
cd frontend
npm run dev
```
- Frontend runs on `http://localhost:5173`
- Vite HMR provides instant updates
- Access UI at `http://localhost:5173`

### Common Development Tasks

#### Adding a New API Endpoint

1. Define Pydantic model in `backend/models.py` if needed
2. Add route in `backend/main.py` using `@app.get()` or `@app.post()`
3. Use `response_model` parameter for type safety
4. Handle errors with `HTTPException`
5. Test endpoint at `http://localhost:8002/docs` (Swagger UI)

Example:
```python
from fastapi import FastAPI, HTTPException
from models import YourModel

@app.get("/api/your-endpoint", response_model=YourModel)
async def your_endpoint():
    try:
        # Your logic here
        return {"key": "value"}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

#### Modifying the Frontend UI

1. Edit `frontend/src/App.tsx`
2. Use TypeScript types for all variables and function parameters
3. Follow existing patterns:
   - Use `const` for all variables
   - Use arrow functions for components and handlers
   - Use Tailwind utility classes for styling (no custom CSS)
4. Test in browser at `http://localhost:5173`
5. Check for TypeScript errors: `npm run build` (runs `tsc -b`)
6. Check for linting issues: `npm run lint`

#### Adding a New Dependency

**Backend:**
```bash
cd backend
source venv/bin/activate
pip install package-name
pip freeze > requirements.txt  # Update requirements
```

**Frontend:**
```bash
cd frontend
npm install package-name
# package.json and package-lock.json auto-update
```

#### Testing the Playwright Scraper

The scraper is now **enabled and functional**. To test it:

1. Ensure virtual environment is activated: `source venv/bin/activate`
2. Verify dependencies installed: `pip install -r requirements.txt`
3. Install Playwright browsers: `python -m playwright install chromium`
4. Run test script: `python test_scraper.py`
5. Or start full backend: `python main.py`
6. Check that `.env` file exists in project root (not required for public search, but good to have)

**Note:** The scraper uses the public SEACE portal at `https://prod6.seace.gob.pe/buscador-publico/` which doesn't require authentication. The credentials in `.env` are not actively used by the current implementation.

---

## Code Patterns and Conventions

### Backend (Python)

**Code Style**
- Follow PEP 8 style guide
- Use type hints for all function parameters and return values
- Use async/await for I/O-bound operations
- Use Pydantic models for all data structures

**Example from `main.py`:**
```python
@app.get("/api/contrataciones", response_model=ContratacionResponse)
def obtener_contrataciones():
    """Docstring explaining what this endpoint does"""
    try:
        # Logic here
        return ContratacionResponse(...)
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

**Error Handling**
- Always wrap risky code in try/except blocks
- Use `HTTPException` for API errors with appropriate status codes
- Log errors using `print()` or `traceback.print_exc()` (no logging framework currently)
- Return user-friendly error messages

**Data Models**
- Use Pydantic `BaseModel` for all data structures
- Use `Literal` types for enums (e.g., `estado_tiempo: Literal["rojo", "amarillo", "verde"]`)
- Use `datetime` objects with timezone awareness (UTC)
- Optional fields use `Optional[Type]` or `Type | None`

### Frontend (TypeScript/React)

**TypeScript Patterns**
- Use `interface` for object shapes (e.g., `interface Contratacion`)
- Use `type` for unions and complex types (e.g., `type EstadoTiempo = ...`)
- Use `as const` for readonly literal arrays (line 4)
- Use `satisfies` operator for type-safe constant objects (line 57)
- Never use `any` type (strict mode prevents this)

**React Patterns**
- Use functional components with hooks (no class components)
- Use `useState` for local state (no Redux/Zustand currently)
- Use `async/await` for API calls inside try/catch blocks
- Use TypeScript generics with hooks: `useState<Type>(initialValue)`

**Naming Conventions**
- Components: PascalCase (e.g., `App`)
- Functions: camelCase (e.g., `fetchContrataciones`)
- Constants: UPPER_SNAKE_CASE (e.g., `ESTADOS_TIEMPO`)
- Interfaces: PascalCase (e.g., `Contratacion`)
- State variables: camelCase (e.g., `contrataciones`, `loading`)

**Styling with Tailwind**
- Use utility classes directly in JSX
- Follow mobile-first responsive design
- Use semantic color names: `bg-gray-50`, `text-blue-600`
- Group related utilities: spacing, then colors, then effects
- Use Tailwind's default spacing scale (4, 6, 8, 12, etc.)

**Example from `App.tsx`:**
```typescript
const [contrataciones, setContrataciones] = useState<Contratacion[]>([]);

const fetchContrataciones = async () => {
  setLoading(true);
  try {
    const response = await fetch("http://localhost:8002/api/contrataciones");
    const data: ContratacionResponse = await response.json();
    setContrataciones(data.contrataciones);
  } catch (err) {
    setError(err instanceof Error ? err.message : "Error desconocido");
  } finally {
    setLoading(false);
  }
};
```

---

## API Contract

### GET `/api/contrataciones`

**Description:** Fetches current government contracts for technology and goods/services in Lima.

**Request:**
```http
GET http://localhost:8002/api/contrataciones
Accept: application/json
```

**Response:** `200 OK`
```json
{
  "contrataciones": [
    {
      "codigo": "CM-117-2025-CPMP",
      "descripcion": "SERVICIO DE RENOVACIÓN E INSTALACIÓN DE ALFOMBRAS...",
      "entidad": "CAJA DE PENSIONES MILITAR - POLICIAL",
      "fecha_limite": "2025-11-19T15:00:00+00:00",
      "tiempo_restante_horas": 18.0,
      "estado_tiempo": "rojo",
      "url_detalle": "https://prod6.seace.gob.pe/buscador-publico/contrataciones/30529",
      "fecha_publicacion": "2025-11-13T10:00:00+00:00"
    }
  ],
  "total": 1,
  "timestamp": "2025-11-14T10:00:00+00:00"
}
```

**Field Descriptions:**
- `codigo`: Unique contract identifier (string)
- `descripcion`: Contract description/title (string)
- `entidad`: Government entity/agency (string)
- `fecha_limite`: Deadline for submissions (ISO 8601 datetime with timezone)
- `tiempo_restante_horas`: Hours remaining until deadline (float)
- `estado_tiempo`: Alert state - `"rojo"` (<24h), `"amarillo"` (24-72h), `"verde"` (>72h)
- `url_detalle`: Direct link to contract details on SEACE portal (string)
- `fecha_publicacion`: Publication date (ISO 8601 datetime or null)

**Error Response:** `500 Internal Server Error`
```json
{
  "detail": "Error al obtener contrataciones: [error message]"
}
```

### Other Endpoints

**GET `/`** - API information and available endpoints
**GET `/health`** - Health check returning `{"status": "ok", "timestamp": "..."}`

---

## Testing

**Current Status:** No testing framework is currently implemented.

### Recommended Testing Approach

**Backend Testing (not implemented)**
- Use `pytest` for unit and integration tests
- Use `httpx` or `TestClient` from FastAPI for API endpoint testing
- Use `pytest-asyncio` for async tests
- Use `pytest-playwright` for end-to-end scraper tests
- Test files: `test_main.py`, `test_scraper.py`, `test_models.py`

**Frontend Testing (not implemented)**
- Use `vitest` (Vite's test runner) for unit tests
- Use `@testing-library/react` for component testing
- Use `@testing-library/user-event` for user interaction simulation
- Test files: `App.test.tsx`, `utils.test.ts`

**Manual Testing Checklist**
1. Start both backend and frontend servers
2. Open browser at `http://localhost:5173`
3. Click "Actualizar" button
4. Verify loading spinner appears
5. Verify contracts display in table
6. Verify color coding matches time states
7. Verify "Ver detalle" links open SEACE portal in new tab
8. Check browser console for errors (should be none)
9. Test backend health endpoint: `http://localhost:8002/health`
10. Test API docs: `http://localhost:8002/docs`

---

## Known Issues and Limitations

### Current Known Issues

1. **Scraper Now Enabled - No Login Required**
   - **Status:** ✅ RESOLVED - Scraper is now enabled and functional
   - **Discovery:** SEACE has a public search portal that doesn't require authentication
   - **URL:** `https://prod6.seace.gob.pe/buscador-publico/`
   - **Location:** `backend/main.py` lines 14, 59-68 (now active)
   - **Note:** Network connectivity required; may fail in sandboxed/restricted environments
   - **Testing:** Use `python test_scraper.py` to verify functionality

2. **No Data Persistence**
   - **Issue:** No database or file storage
   - **Impact:** Data lost on server restart; no historical tracking
   - **Future:** Consider PostgreSQL, SQLite, or MongoDB integration

3. **No Authentication**
   - **Issue:** API endpoints are public (no auth required)
   - **Impact:** Anyone can access the API
   - **Security:** SEACE credentials stored in `.env` (gitignored, but still sensitive)

4. **CORS Hardcoded**
   - **Location:** `backend/main.py` line 28
   - **Issue:** Only allows localhost origins
   - **Impact:** Cannot deploy frontend to different domain without backend changes

5. **Port Mismatch in Documentation**
   - **README says:** Backend runs on port 8000
   - **Actual port:** Backend runs on port 8002 (line 118 in `main.py`)
   - **Frontend expects:** Port 8002 (line 34 in `App.tsx`)
   - **Impact:** Minor documentation inconsistency

### Limitations

- **No scheduled execution:** Manual button click required
- **No notifications:** No push notifications or email alerts
- **No custom filters:** Search criteria hardcoded in scraper
- **No historical data:** Cannot view past contracts
- **Single-language UI:** Spanish only (not internationalized)
- **No mobile app:** Web-only interface
- **No rate limiting:** API can be overwhelmed with requests
- **No caching:** Every request triggers full scrape (if enabled)

---

## Environment Configuration

### Environment Variables

**File:** `.env` (in project root, not committed to git)

**Required Variables:**
```env
user=YOUR_RUC_NUMBER
pass=YOUR_SEACE_PASSWORD
```

**Notes:**
- `.env` is gitignored (see `.gitignore` line 5)
- Use `.env.example` as template
- These are SEACE portal credentials (sensitive!)
- Variables loaded by `python-dotenv` in `backend/main.py` line 17
- Accessed in code: `os.getenv("user")`, `os.getenv("pass")`

### Port Configuration

**Backend:** Port 8002 (configurable in `backend/main.py` line 118)
```python
uvicorn.run("main:app", host="0.0.0.0", port=8002, reload=True)
```

**Frontend:** Port 5173 (Vite default, configurable in `vite.config.ts`)
- Backend API URL hardcoded in `frontend/src/App.tsx` line 34: `http://localhost:8002`
- Update both if changing backend port

---

## Git Workflow

### Branch Strategy

**Current Branch:** `claude/claude-md-mhz13cdve1mhrlus-016dd1L1KtHwx7kVkJvp9G27`

**Important:** All development must happen on the designated Claude branch (starts with `claude/` and ends with session ID).

**Workflow:**
1. Make changes on the Claude branch
2. Commit with descriptive messages
3. Push to the Claude branch: `git push -u origin <branch-name>`
4. DO NOT push to main/master without explicit permission

### Commit Message Conventions

**Format:**
```
<type>: <subject>

<optional body>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `refactor`: Code refactoring (no functional changes)
- `style`: Code style/formatting
- `test`: Adding or updating tests
- `chore`: Build, dependencies, or tooling changes

**Examples:**
```
feat: add filtering by entity name

fix: correct time calculation for verde state

docs: update README with correct backend port

refactor: extract time formatting to utility function
```

### Files to NEVER Commit

**Sensitive:**
- `.env` (credentials)
- `*.pem`, `*.key` (certificates)
- Any file with passwords or API keys

**Generated:**
- `backend/venv/` (Python virtual environment)
- `frontend/node_modules/` (npm dependencies)
- `frontend/dist/` (build output)
- `__pycache__/`, `*.pyc` (Python bytecode)

**IDE:**
- `.vscode/`, `.idea/` (editor settings)
- `*.swp`, `*.swo` (Vim swap files)

All these are already in `.gitignore` (107 lines total).

---

## Troubleshooting Guide

### Backend Issues

**Problem: `ModuleNotFoundError: No module named 'fastapi'`**
- **Cause:** Virtual environment not activated or dependencies not installed
- **Fix:**
  ```bash
  cd backend
  source venv/bin/activate  # or venv\Scripts\activate on Windows
  pip install -r requirements.txt
  ```

**Problem: `Playwright not installed`**
- **Cause:** Playwright browsers not downloaded
- **Fix:** `playwright install chromium`

**Problem: Backend starts but API returns 500 errors**
- **Check:** Backend console logs for detailed error messages
- **Check:** `.env` file exists in project root (not in `backend/`)
- **Check:** Credentials in `.env` are correct

**Problem: CORS errors in browser console**
- **Cause:** Frontend origin not in CORS whitelist
- **Fix:** Add origin to `allow_origins` list in `backend/main.py` line 28

### Frontend Issues

**Problem: `Cannot find module` or TypeScript errors**
- **Cause:** Dependencies not installed or type definitions missing
- **Fix:**
  ```bash
  cd frontend
  rm -rf node_modules package-lock.json
  npm install
  ```

**Problem: Blank white screen in browser**
- **Check:** Browser console for errors (F12)
- **Check:** Backend is running on port 8002
- **Check:** Network tab shows 200 OK response from API
- **Fix:** Try hard refresh (Ctrl+Shift+R or Cmd+Shift+R)

**Problem: Build fails with TypeScript errors**
- **Cause:** Type errors in code (strict mode enabled)
- **Fix:** Review error messages and add proper type annotations
- **Example:** Change `const x = null` to `const x: string | null = null`

**Problem: Tailwind styles not working**
- **Cause:** Tailwind not properly configured or build failed
- **Fix:**
  - Check `index.css` imports `@import "tailwindcss"`
  - Check `vite.config.ts` includes `@tailwindcss/vite` plugin
  - Restart dev server: `npm run dev`

---

## Performance Considerations

### Backend Performance

**Current:**
- Synchronous scraping (blocks request until complete)
- No caching mechanism
- No request throttling

**Recommendations:**
- Implement background task queue (Celery, RQ, or FastAPI BackgroundTasks)
- Add caching layer (Redis) for contract data (e.g., 5-minute TTL)
- Add rate limiting (slowapi or custom middleware)
- Use async Playwright API for better performance

### Frontend Performance

**Current:**
- Single component (no code splitting)
- No memo or optimization hooks
- Fetches all data on button click (no pagination)

**Recommendations:**
- Use `React.memo()` for table rows if dataset grows
- Implement virtual scrolling for large datasets (react-window)
- Add pagination or infinite scroll
- Consider `useMemo()` for expensive calculations
- Use loading skeletons instead of spinner for better UX

---

## Future Improvements

From `README.md` (lines 194-200), planned enhancements:

1. **Custom Filters from Frontend**
   - Allow users to select entity, category, amount range
   - Add date range picker for publication/deadline dates

2. **Push Notifications**
   - Browser notifications when new contracts appear
   - Email alerts for specific criteria
   - Webhook integration for external systems

3. **Historical Data Storage**
   - Database integration (PostgreSQL recommended)
   - Archive contracts after deadline passes
   - Analytics dashboard for trends

4. **Scheduled Automatic Execution**
   - Cron job or task scheduler integration
   - Configurable scraping intervals (e.g., every 30 minutes)
   - Background processing

5. **Improved Scraper Selectors**
   - More robust CSS selectors that handle SEACE updates
   - Error recovery for missing fields
   - Validation of extracted data

6. **Authentication System**
   - User accounts for personalized filters
   - Saved searches and favorites
   - Multi-user support

---

## Quick Reference

### Most Important Files

| File | Lines | Purpose |
|------|-------|---------|
| `backend/main.py` | 118 | FastAPI server and endpoints |
| `backend/scraper.py` | 175 | Playwright scraper (currently disabled) |
| `frontend/src/App.tsx` | 232 | React UI component |
| `backend/models.py` | 20 | Pydantic data models |

### Development Servers

| Component | URL | Port | Command |
|-----------|-----|------|---------|
| Frontend | http://localhost:5173 | 5173 | `npm run dev` |
| Backend API | http://localhost:8002 | 8002 | `python main.py` |
| API Docs | http://localhost:8002/docs | 8002 | (auto-generated) |
| Health Check | http://localhost:8002/health | 8002 | (endpoint) |

### Key Commands

```bash
# Backend setup
cd backend
python -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows
pip install -r requirements.txt
playwright install chromium

# Frontend setup
cd frontend
npm install

# Run development servers
cd backend && python main.py              # Terminal 1
cd frontend && npm run dev                # Terminal 2

# Build for production
cd frontend && npm run build              # Creates dist/ folder

# Code quality
cd frontend && npm run lint               # ESLint check
cd frontend && npm run build              # TypeScript type check
```

### TypeScript Types Reference

```typescript
// Core types from App.tsx
type EstadoTiempo = "rojo" | "amarillo" | "verde";

interface Contratacion {
  codigo: string;
  descripcion: string;
  entidad: string;
  fecha_limite: string;
  tiempo_restante_horas: number;
  estado_tiempo: EstadoTiempo;
  url_detalle: string;
}

interface ContratacionResponse {
  contrataciones: Contratacion[];
  total: number;
  timestamp: string;
}
```

---

## AI Assistant Guidelines

### When Working on This Codebase

1. **Always Read Before Writing**
   - Read existing files before modifying
   - Understand current patterns and conventions
   - Match existing code style

2. **Respect Type Safety**
   - Never use `any` type in TypeScript
   - Add type annotations for all function parameters and return values
   - Use Pydantic models for all Python data structures

3. **Test Locally**
   - Verify both backend and frontend run without errors
   - Check browser console for warnings
   - Test API endpoints using Swagger UI at `/docs`

4. **Update Documentation**
   - Update this CLAUDE.md if adding new patterns or conventions
   - Update README.md for user-facing changes
   - Add docstrings to Python functions
   - Add JSDoc comments for complex TypeScript functions

5. **Handle Errors Gracefully**
   - Always wrap risky operations in try/catch
   - Provide user-friendly error messages
   - Log errors for debugging

6. **Follow Git Workflow**
   - Work on designated Claude branch
   - Write descriptive commit messages
   - Never commit sensitive files

7. **Consider Performance**
   - Avoid blocking operations in backend
   - Use React optimization hooks for heavy rendering
   - Consider caching strategies

8. **Maintain Spanish UI**
   - Keep UI text in Spanish (consistent with existing design)
   - Use Spanish for user-facing messages
   - Code comments can be in English or Spanish

### Common Tasks for AI Assistants

**Task: Add a new filter to the scraper**
1. Modify `scraper.py` to accept filter parameter
2. Update `get_contrataciones()` function signature
3. Update `main.py` endpoint to accept query parameters
4. Update `App.tsx` to send filter in API request
5. Add UI controls for filter in `App.tsx`
6. Update API documentation in this file

**Task: Fix a bug**
1. Reproduce the bug locally
2. Identify root cause in code
3. Write a fix following existing patterns
4. Test the fix thoroughly
5. Commit with `fix:` prefix
6. Document the fix if it addresses a known issue

**Task: Add a new feature**
1. Check "Future Improvements" section for alignment
2. Plan the implementation (backend → API → frontend)
3. Update data models if needed
4. Implement backend logic first
5. Test backend endpoint in Swagger UI
6. Implement frontend integration
7. Test end-to-end workflow
8. Update documentation

---

## Additional Resources

### External Documentation

- **FastAPI:** https://fastapi.tiangolo.com/
- **Playwright:** https://playwright.dev/python/
- **React:** https://react.dev/
- **TypeScript:** https://www.typescriptlang.org/docs/
- **Tailwind CSS:** https://tailwindcss.com/docs
- **Vite:** https://vitejs.dev/guide/
- **Pydantic:** https://docs.pydantic.dev/

### SEACE Portal

- **Public Search:** https://prod6.seace.gob.pe/buscador-publico/
- **Documentation:** Limited official API documentation available

---

## Summary

This codebase is a modern, well-structured full-stack application with:
- ✅ Clean separation of concerns (backend/frontend)
- ✅ Type safety (TypeScript + Pydantic)
- ✅ Modern tooling (Vite, FastAPI, React 19)
- ✅ Comprehensive documentation
- ⚠️ Currently using mock data (Playwright scraper disabled)
- ❌ No testing framework
- ❌ No production deployment setup

**Total Application Code:** ~545 lines (excluding config and dependencies)
**Development Status:** Functional prototype ready for enhancement
**Primary Language:** Spanish (UI and docs)
**Code Quality:** High (strict TypeScript, type hints, linting)

---

**Document Version:** 1.0
**Last Updated:** 2025-11-14
**Maintained By:** AI Assistants working on SEACE Scraper

---
> Source: [mateoismael/autoob](https://github.com/mateoismael/autoob) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
