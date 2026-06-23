## quick-troubleshooting-index

> Troubleshooting frontend and backend in the app

# Quick Troubleshooting Index - Sprint 10 Edition

## 🚨 EMERGENCY: Rapid Issue Resolution

Use this index to quickly find solutions to common problems encountered during Sprint 10.

### 🔥 HYDRATION ERRORS (Next.js/Chakra UI)

**Symptoms:**
- Console error: "Text content does not match server-rendered HTML"
- White flash on page load
- Theme switching not working
- Components showing wrong initial state

**Quick Fix (30 seconds):**
```bash
# 1. Clear Next.js cache
rm -rf .next && npm run dev

# 2. Apply mounted state pattern to problem component
const [mounted, setMounted] = useState(false);
useEffect(() => setMounted(true), []);
if (!mounted) return <div>Loading...</div>;
```

**Detailed Solution:** `frontend/.cursor/rules/nextjs-hydration-prevention.mdc`

---

### 🔥 BACKEND CIRCULAR IMPORTS

**Symptoms:**
- "ImportError: cannot import name" during startup
- uvicorn fails to start
- Router files causing import errors
- `from main import app` errors

**Quick Fix (1 minute):**
```python
# NEVER do this in routers:
from ..main import app  # ❌

# ALWAYS do this instead:
from ..dependencies import get_qdrant_client
def endpoint(client = Depends(get_qdrant_client)):  # ✅
```

**Detailed Solution:** `backend/.cursor/rules/fastapi-dependency-injection.mdc`

---

### 🔥 MCP BROWSER TOOLS NOT WORKING

**Symptoms:**
- `mcp_browser-tools_takeScreenshot` fails
- "Failed to discover browser connector server"
- "Chrome extension not connected"
- MCP commands timeout

**Quick Fix (2 minutes):**
```bash
# 1. Check Node.js version (must be ≥18)
node --version

# 2. Start server in separate terminal
npx @agentdeskai/browser-tools-server@latest

# 3. Verify server responds
curl http://localhost:3025/health

# 4. Test basic connectivity
mcp_browser-tools_wipeLogs
```

**Detailed Solution:** `.cursor/rules/mcp-browser-tools-setup.mdc`

---

### 🔥 API STATE MANAGEMENT ISSUES

**Symptoms:**
- Race conditions in useEffect
- Inconsistent loading states
- Manual API error handling
- Data not updating consistently

**Quick Fix (30 seconds):**
```tsx
// NEVER do this:
useEffect(() => {
  fetch('/api/data').then(...)  // ❌
}, []);

// ALWAYS do this:
const { data, isLoading, error } = useQuery({  // ✅
  queryKey: ['data'],
  queryFn: () => api.get('/api/data')
});
```

**Detailed Solution:** `frontend/.cursor/rules/react-query-api-integration.mdc`

---

### 🔥 QDRANT POINT ID VALIDATION ERRORS

**Symptoms:**
- "Invalid point ID format" errors
- Ingestion pipeline failures
- SHA256 hashes rejected as point IDs

**Quick Fix (10 seconds):**
```python
# NEVER use SHA256 as point ID:
point_id = sha256_hash  # ❌

# ALWAYS use UUID:
import uuid
point_id = str(uuid.uuid4())  # ✅
payload = {"file_hash": sha256_hash}  # Store hash in payload
```

**Detailed Solution:** `.cursor/rules/sprint-lessons-learned.mdc` → Lesson 5

---

## 🎯 DIAGNOSTIC FLOWCHART

```
Issue Category?
├── Frontend: Dark theme, hydration, React errors
│   ├── Hydration? → frontend/.cursor/rules/nextjs-hydration-prevention.mdc
│   └── API calls? → frontend/.cursor/rules/react-query-api-integration.mdc
├── Backend: Import errors, startup failures
│   └── Circular imports? → backend/.cursor/rules/fastapi-dependency-injection.mdc
├── MCP: Tool failures, connection issues
│   └── MCP not working? → .cursor/rules/mcp-browser-tools-setup.mdc
└── Database: Qdrant errors, point IDs
    └── Point validation? → .cursor/rules/sprint-lessons-learned.mdc
```

## ⚡ SEVERITY LEVELS

### 🚨 CRITICAL (Development Blocked)
1. **Hydration errors** preventing Next.js from loading
2. **Circular imports** preventing backend startup
3. **MCP failures** blocking debugging workflow

### ⚠️ HIGH (Feature Development Impacted)
1. **Manual API state** creating bugs and complexity
2. **Point ID validation** causing silent ingestion failures
3. **Theme inconsistencies** affecting user experience

### 📝 MEDIUM (Technical Debt)
1. Missing TypeScript types
2. Inconsistent error handling
3. Outdated dependencies

## 🚀 QUICK VERIFICATION COMMANDS

### Frontend Health Check
```bash
# Next.js build verification
npm run build && npm start
# Should start without hydration warnings

# React Query verification
grep -r "useEffect.*fetch" src/  # Should return no matches
grep -r "useState.*loading" src/  # Should be minimal
```

### Backend Health Check
```bash
# Circular import check
python -c "import backend.main; print('✅ Main imports OK')"

# Dependency injection verification
grep -r "from.*main import" backend/routers/  # Should return no matches
grep -r "Depends(" backend/routers/  # Should find dependency usage
```

### MCP Health Check
```bash
# 3-component verification
node --version  # ≥18
curl http://localhost:3025/health  # Should respond
mcp_browser-tools_wipeLogs  # Should succeed
```

---

**🔄 Update Protocol:** When new issues arise, add them to this index with:
1. **Symptoms** - How to recognize the problem
2. **Quick Fix** - 30-second to 2-minute solution
3. **Detailed Solution** - Link to comprehensive rule file

*This index prevents repeating the time-consuming debugging from Sprint 10.*

---
> Source: [rm2thaddeus/Pixel_Detective](https://github.com/rm2thaddeus/Pixel_Detective) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
