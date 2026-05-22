## debugging

> description: Systematic debugging approaches with minimal tool calls

---
description: Systematic debugging approaches with minimal tool calls
globs: 
  - "**/*.ts"
  - "**/*.js"
  - "**/*.py"
  - "**/*.java"
  - "**/*.go"
  - "**/*.rb"
  - "**/*.php"
  - "**/*.cs"
  - "**/*.cpp"
alwaysApply: false
---

# Efficient Debugging Strategies

## 🎯 Debugging Philosophy

### The 5-Step Debug Protocol
```
1. REPRODUCE → Confirm the issue exists
2. ISOLATE → Find the smallest failing case
3. DIAGNOSE → Identify root cause
4. FIX → Apply minimal solution
5. VERIFY → Ensure fix works
```

## 🔍 Initial Assessment

### Quick Triage (1 Tool Call)
```bash
# Get complete context in one call
echo "=== Debug Context ===" && \
pwd && \
echo -e "\n--- Error State ---" && \
tail -50 error.log 2>/dev/null || echo "No error log" && \
echo -e "\n--- Recent Changes ---" && \
git diff --stat HEAD~1 2>/dev/null || echo "No git" && \
echo -e "\n--- Running Processes ---" && \
ps aux | grep -E "(node|python|java)" | grep -v grep | head -5 && \
echo -e "\n--- Test Status ---" && \
npm test -- --no-coverage 2>&1 | tail -20 || echo "No tests"
```

## 🐛 Common Bug Patterns

### Type 1: Null/Undefined Errors
```javascript
// SYMPTOM: "Cannot read property 'x' of undefined"

// ❌ BAD DEBUG: Random changes
console.log(user);
console.log(user.profile);
console.log(user.profile.name);

// ✅ GOOD DEBUG: Systematic check
console.log({
  hasUser: !!user,
  hasProfile: !!user?.profile,
  profileKeys: user?.profile ? Object.keys(user.profile) : 'none',
  nameValue: user?.profile?.name
});

// FIX PATTERN:
const name = user?.profile?.name ?? 'Default';
```

### Type 2: Async/Promise Issues
```javascript
// SYMPTOM: "Promise pending" or race conditions

// ✅ DIAGNOSTIC PATTERN:
console.log({
  stage: 'before-await',
  timestamp: Date.now()
});

const result = await operation();

console.log({
  stage: 'after-await',
  timestamp: Date.now(),
  hasResult: !!result,
  resultType: typeof result
});

// FIX PATTERNS:
// 1. Missing await
const data = await fetchData();

// 2. Race condition
const [result1, result2] = await Promise.all([op1(), op2()]);

// 3. Error handling
try {
  const data = await riskyOperation();
} catch (error) {
  console.error('Operation failed:', error.message);
}
```

### Type 3: State Management Issues
```javascript
// SYMPTOM: "State not updating" or "Stale values"

// ✅ STATE DEBUG HELPER:
function debugState(label, state) {
  console.log(`[${label}]`, {
    state: JSON.stringify(state, null, 2),
    type: typeof state,
    isArray: Array.isArray(state),
    timestamp: new Date().toISOString()
  });
}

// USE:
debugState('Before update', currentState);
updateState(newValue);
debugState('After update', currentState);
```

## 🔧 Debugging Tools

### Universal Debug Logger
```javascript
// Add to any file for instant debugging
const DEBUG = {
  log: (label, data) => {
    console.log(`\n🔍 [${label}]`, {
      data,
      stack: new Error().stack.split('\n')[2],
      time: new Date().toISOString()
    });
  },
  
  checkpoint: (name) => {
    console.log(`✓ Checkpoint: ${name} at ${new Date().toISOString()}`);
  },
  
  measure: async (label, fn) => {
    const start = performance.now();
    try {
      const result = await fn();
      console.log(`⏱️ ${label}: ${(performance.now() - start).toFixed(2)}ms`);
      return result;
    } catch (error) {
      console.log(`❌ ${label} failed after ${(performance.now() - start).toFixed(2)}ms`);
      throw error;
    }
  }
};

// Usage:
DEBUG.checkpoint('Starting process');
const data = await DEBUG.measure('Database query', () => db.query(sql));
DEBUG.log('Query result', data);
```

### Error Boundary Pattern
```javascript
// Wrap suspicious code
function safeTry(operation, fallback = null) {
  try {
    return operation();
  } catch (error) {
    console.error('Safe operation failed:', {
      error: error.message,
      stack: error.stack,
      operation: operation.toString()
    });
    return fallback;
  }
}

// Usage:
const config = safeTry(() => JSON.parse(configString), {});
```

## 📊 Systematic Debugging

### Binary Search Debug
```javascript
// For "worked before, broken now" issues
// 1. Find last working commit
git bisect start
git bisect bad HEAD
git bisect good abc123  // last known good

// 2. Test each commit
npm test || git bisect bad
npm test && git bisect good

// 3. Find exact breaking change
git bisect reset
git show <bad-commit>
```

### Divide & Conquer
```javascript
// For complex flows
function debugFlow() {
  console.log('Step 1: Input validation');
  // If fails here, input issue
  
  console.log('Step 2: Data transformation');
  // If fails here, transformation issue
  
  console.log('Step 3: Business logic');
  // If fails here, logic issue
  
  console.log('Step 4: Output formatting');
  // If fails here, formatting issue
}
```

## 🎪 Performance Debugging

### Quick Performance Check
```javascript
// One-liner performance wrapper
const perf = (fn, label = 'Operation') => {
  const start = Date.now();
  const result = fn();
  console.log(`${label}: ${Date.now() - start}ms`);
  return result;
};

// Usage:
const data = perf(() => processLargeDataset(), 'Dataset processing');
```

### Memory Leak Detection
```javascript
// Memory snapshot helper
const memorySnapshot = (label) => {
  if (global.gc) global.gc();
  const usage = process.memoryUsage();
  console.log(`Memory [${label}]:`, {
    heapUsed: `${(usage.heapUsed / 1024 / 1024).toFixed(2)} MB`,
    external: `${(usage.external / 1024 / 1024).toFixed(2)} MB`,
    total: `${(usage.heapTotal / 1024 / 1024).toFixed(2)} MB`
  });
};

// Track memory growth
memorySnapshot('Before operation');
// ... do operation ...
memorySnapshot('After operation');
```

## 🔄 Debug Patterns by Error Type

### Network/API Errors
```bash
# Quick network debug (1 call)
curl -i -X GET "http://api.endpoint.com/path" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -w "\n\nTime: %{time_total}s\nHTTP Code: %{http_code}\n" \
  || echo "Connection failed"
```

### Database Errors
```sql
-- Quick DB debug
SELECT 
  'Connections' as metric, 
  count(*) as value 
FROM pg_stat_activity
UNION ALL
SELECT 
  'Slow queries', 
  count(*) 
FROM pg_stat_activity 
WHERE state = 'active' 
  AND query_start < now() - interval '5 seconds';
```

### File System Errors
```bash
# File system debug (1 call)
ls -la problematic/path/ 2>&1 && \
df -h . && \
echo "Permissions:" && \
stat -c "%a %n" problematic/path/* 2>/dev/null | head -10
```

## 💡 Smart Logging

### Conditional Debug Output
```javascript
// Only log when debugging
const DEBUG_MODE = process.env.DEBUG === 'true';

function debugLog(...args) {
  if (DEBUG_MODE) {
    console.log('[DEBUG]', new Date().toISOString(), ...args);
  }
}

// Strategic placement
debugLog('Entering function', { params });
// ... code ...
debugLog('Exiting function', { result });
```

### Structured Error Info
```javascript
// Enhanced error for debugging
class DebugError extends Error {
  constructor(message, context = {}) {
    super(message);
    this.name = 'DebugError';
    this.context = context;
    this.timestamp = new Date().toISOString();
    this.stack = new Error().stack;
  }
  
  toJSON() {
    return {
      name: this.name,
      message: this.message,
      context: this.context,
      timestamp: this.timestamp,
      stack: this.stack.split('\n').slice(0, 5)
    };
  }
}

// Usage:
throw new DebugError('User not found', {
  userId,
  searchParams,
  dbQuery: query.toString()
});
```

## 🚨 Debug Checklist

### Before Deep Diving
```
□ Is error message clear?
□ Can I reproduce consistently?
□ Did it work before?
□ What changed recently?
□ Is it environment-specific?
```

### Quick Fixes to Try
```
□ Clear cache/node_modules
□ Restart services
□ Check environment variables
□ Verify dependencies versions
□ Test in isolation
```

## 📈 Debug Efficiency

### Time Budget
```
5 min:  Read error, check obvious issues
10 min: Reproduce and isolate
15 min: Apply systematic debugging
20 min: Consider asking for help
30 min: Take break, return with fresh eyes
```

### Debug Complexity Levels
```
Level 1: Console.log at error point
Level 2: Binary search for issue
Level 3: Use debugger/breakpoints
Level 4: Add comprehensive logging
Level 5: Profile and analyze deeply
```

## 🎯 Golden Rules

1. **Reproduce before fixing**
2. **Change one thing at a time**
3. **Keep a debug log**
4. **Verify fixes work**
5. **Document the solution**

Remember: The best debugger is a systematic approach!

---
> Source: [DVC2/cursor_prompts](https://github.com/DVC2/cursor_prompts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
