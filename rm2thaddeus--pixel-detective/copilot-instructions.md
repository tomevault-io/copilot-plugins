## sprint-lessons-learned

> This rule captures the **most expensive mistakes** from Sprint 10 that must **NEVER be repeated**.

# Sprint 10 Critical Lessons Learned - Master Reference

## 🚨 CRITICAL FAILURES & SOLUTIONS FROM SPRINT 10

This rule captures the **most expensive mistakes** from Sprint 10 that must **NEVER be repeated**.

### **🔥 LESSON 1: Hydration Errors Are Project Killers**

**Problem:** Recurring Next.js hydration errors with Chakra UI dark mode consumed **days** of debugging time.

**Root Cause:** Server/client HTML mismatches from theme system and browser-only APIs.

**✅ PREVENTION PROTOCOL:**
```bash
# BEFORE any theme or client-side work:
1. Set up proper ColorModeScript in <head>
2. Use mounted state pattern for client-only components  
3. Test in production build: npm run build && npm start
4. NEVER use window/localStorage in render phase
```

**🚫 FORBIDDEN:** Any theme changes without hydration testing.

---

### **🔥 LESSON 2: Backend Circular Imports Crash Everything**

**Problem:** Circular imports between `main.py` and routers caused backend startup failures.

**Root Cause:** Routers importing from main.py to access shared state.

**✅ PREVENTION PROTOCOL:**
```python
# ALWAYS use dependency injection pattern:
# 1. Create dependencies.py with AppState class
# 2. Use @asynccontextmanager lifespan in main.py
# 3. Routers import dependencies, NEVER main.py
# 4. Use Depends() for all shared resources
```

**🚫 FORBIDDEN:** Any router importing from main.py or app.state access.

---

### **🔥 LESSON 3: MCP Browser Tools Setup Is Complex**

**Problem:** Browser Tools MCP failures wasted hours on "simple" debugging tasks.

**Root Cause:** 3-component setup (Node.js v18+, server on 3025, Chrome extension) with silent failures.

**✅ PREVENTION PROTOCOL:**
```bash
# BEFORE using Browser Tools MCP:
1. Verify Node.js ≥18: node --version
2. Start server: npx @agentdeskai/browser-tools-server@latest
3. Install Chrome extension in developer mode
4. Test basic MCP: mcp_browser-tools_wipeLogs
5. Verify full functionality: mcp_browser-tools_takeScreenshot
```

**🚫 FORBIDDEN:** Using Browser Tools MCP without 3-component verification.

---

### **🔥 LESSON 4: Manual API State Management Creates Bugs**

**Problem:** Complex manual `useEffect` + `useState` patterns for API calls introduced race conditions and inconsistent state.

**Root Cause:** Not using React Query for server state management.

**✅ PREVENTION PROTOCOL:**
```tsx
// ALWAYS use React Query for server state:
// 1. All API calls through useQuery/useMutation
// 2. Centralized error handling with query client
// 3. Automatic caching and background refetching
// 4. Consistent loading/error states
```

**🚫 FORBIDDEN:** Manual useEffect for API calls when React Query is available.

---

### **🔥 LESSON 5: Point ID Validation Causes Silent Failures**

**Problem:** Qdrant rejected SHA256 hashes as point IDs, causing ingestion pipeline failures.

**Root Cause:** Qdrant only accepts UUIDs or unsigned integers as point IDs.

**✅ PREVENTION PROTOCOL:**
```python
# ALWAYS use UUID for Qdrant point IDs:
point_id = str(uuid.uuid4())  # ✅ Valid
# Store SHA256 in payload for deduplication:
payload = {"file_hash": sha256_hash}  # ✅ For deduplication
```

**🚫 FORBIDDEN:** Using SHA256 hashes directly as Qdrant point IDs.

---

## 🎯 **MANDATORY PRE-WORK CHECKLISTS**

### **Before Frontend Theme Work:**
- [ ] ColorModeScript properly placed in layout.tsx `<head>`
- [ ] Mounted state pattern ready for client-only components
- [ ] suppressHydrationWarning only on `<html>` tag
- [ ] Production build test environment available

### **Before Backend Development:**
- [ ] Dependencies.py module exists with AppState class
- [ ] Lifespan manager configured in main.py
- [ ] Router files use Depends(), not direct imports
- [ ] Circular import detection tool configured

### **Before MCP Usage:**
- [ ] Node.js version ≥18 verified
- [ ] MCP-specific setup requirements documented
- [ ] Browser Tools server setup if needed
- [ ] Fallback strategies for MCP failures

### **Before API Integration:**
- [ ] React Query installed and configured
- [ ] Query client with error handling setup
- [ ] API client with interceptors configured
- [ ] Error boundary components ready

### **Before Vector Database Work:**
- [ ] Point ID generation strategy using UUIDs
- [ ] Payload structure for metadata storage
- [ ] Deduplication logic using hashes in payload
- [ ] Qdrant schema validation

---

## 🚀 **EMERGENCY DEBUGGING PROTOCOLS**

### **Hydration Error Recovery:**
```bash
1. Clear Next.js cache: rm -rf .next
2. Check browser console for specific mismatch
3. Apply mounted state pattern to problematic component
4. Test in production build
5. If persistent: Dynamic import with ssr: false
```

### **Backend Startup Failure Recovery:**
```bash
1. Check for circular imports: python -m py_compile main.py
2. Verify dependencies.py exists and is importable
3. Check lifespan function is properly async
4. Test router imports independently
5. Use dependency injection everywhere
```

### **MCP Connection Failure Recovery:**
```bash
1. Verify Node.js version: node --version ≥18
2. Check server status: curl localhost:3025/health
3. Reinstall Chrome extension if needed
4. Test basic MCP connectivity first
5. Check specific MCP server requirements
```

---

## 📊 **SUCCESS METRICS TO PREVENT REGRESSION**

### **Development Speed Indicators:**
- **Hydration Issues:** Zero hydration errors in console
- **Backend Startup:** Clean uvicorn startup logs
- **MCP Reliability:** First-try MCP tool success rate >90%
- **API Integration:** No manual useEffect for server state
- **Database Operations:** No point ID validation errors

### **Quality Gates:**
- [ ] **Build Success:** npm run build passes without warnings
- [ ] **Startup Clean:** Backend starts without circular import errors
- [ ] **MCP Ready:** All required MCP components verified before use
- [ ] **Query Consistency:** All server state managed by React Query
- [ ] **Database Validation:** All point IDs are valid UUIDs

---

## 🔄 **CONTINUOUS IMPROVEMENT PROTOCOL**

### **After Each Sprint:**
1. **Document New Failures:** Add patterns that caused delays
2. **Update Prevention Checklists:** Include new verification steps
3. **Refine Setup Protocols:** Streamline complex setup processes
4. **Test Emergency Procedures:** Verify recovery protocols work
5. **Share Knowledge:** Update team on critical patterns

### **Before Each New Feature:**
1. **Review Relevant Lessons:** Check applicable failure patterns
2. **Run Prevention Checklists:** Verify setup requirements
3. **Test Integration Points:** Validate critical dependencies
4. **Prepare Rollback Plan:** Document recovery procedures

---

**⚠️ CRITICAL REMINDER:** These lessons represent **real development time lost**. Following these protocols prevents repeating the same expensive mistakes.

**Status:** LIVING DOCUMENT - Update after each sprint with new critical lessons.

---

*Sprint 10 taught us these lessons the hard way. Don't repeat these mistakes.*

---
> Source: [rm2thaddeus/Pixel_Detective](https://github.com/rm2thaddeus/Pixel_Detective) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
