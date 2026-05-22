## efficiency

> description: Minimizes tool calls and maximizes AI agent efficiency

---
description: Minimizes tool calls and maximizes AI agent efficiency
globs: ["**/*"]
alwaysApply: true
---

# Efficiency Optimization Rules

## 🎯 Core Principle: Every Tool Call Counts

### Tool Call Hierarchy
```
1. NO CALL > Think first, can you answer without tools?
2. ONE CALL > Batch operations into single calls
3. FEW CALLS > Strategic sequence when necessary
4. MANY CALLS > Only for complex research tasks
```

## 📊 Efficiency Metrics

### Track Your Performance
```
EFFICIENCY_SCORE:
- 0 tool calls for knowledge-based answers = 100%
- 1 tool call for simple lookups = 90%
- 2-3 calls for standard tasks = 80%
- 4-5 calls for complex tasks = 70%
- 6+ calls = Review if necessary
```

## 🚀 Zero-Call Strategies

### Answer Directly When Possible
```javascript
// USER: "How do I create a React component?"
// ❌ BAD: Search for React component syntax
// ✅ GOOD: Provide answer from knowledge

// USER: "What's the syntax for Python list comprehension?"
// ❌ BAD: Search Python documentation
// ✅ GOOD: Show syntax directly with examples
```

### Knowledge-First Approach
```
Before ANY tool call, ask:
1. Do I already know this?
2. Is this stable information?
3. Can I provide value without searching?

If YES to all → Answer directly
```

## 💡 Single-Call Patterns

### File Operations
```bash
# ❌ BAD: Multiple calls
cat package.json
ls src/
grep -r "TODO" .

# ✅ GOOD: Single call
cat package.json && echo "---" && ls -la src/ && echo "---" && grep -r "TODO" . | head -20
```

### Code Analysis
```javascript
// ❌ BAD: Separate reads
// Read file1.js
// Read file2.js  
// Read file3.js

// ✅ GOOD: Batch read
const files = ['file1.js', 'file2.js', 'file3.js'];
const contents = await Promise.all(
  files.map(f => fs.readFile(f, 'utf8'))
);
```

### Information Gathering
```bash
# ❌ BAD: Sequential discovery
pwd
git status
git branch
npm list

# ✅ GOOD: Combined context
echo "=== Project Status ===" && \
pwd && \
git status -sb && \
git branch --show-current && \
npm list --depth=0 2>/dev/null | grep -E "^(├|└)" | head -10
```

## 🎪 Batching Strategies

### File Creation
```bash
# ❌ BAD: Individual creates
touch src/index.js
touch src/utils.js
touch src/config.js

# ✅ GOOD: Batch creation
mkdir -p src && touch src/{index,utils,config}.js
```

### Multi-File Updates
```javascript
// ❌ BAD: Update files one by one
updateFile('config.js', configContent);
updateFile('index.js', indexContent);
updateFile('utils.js', utilsContent);

// ✅ GOOD: Batch update
const updates = [
  { file: 'config.js', content: configContent },
  { file: 'index.js', content: indexContent },
  { file: 'utils.js', content: utilsContent }
];
await Promise.all(updates.map(u => 
  fs.writeFile(u.file, u.content)
));
```

## 🔄 Smart Caching

### Remember Previous Results
```javascript
// Store expensive computations
CACHED_RESULTS = {
  projectStructure: null,
  dependencies: null,
  configuration: null
};

// Use cache when available
if (!CACHED_RESULTS.projectStructure) {
  CACHED_RESULTS.projectStructure = await analyzeProject();
}
return CACHED_RESULTS.projectStructure;
```

### Context Preservation
```
PRESERVE_BETWEEN_CALLS:
- File contents already read
- Command outputs already seen
- Discovered patterns
- Validated information
```

## 📉 Tool Call Reduction Patterns

### Search Optimization
```
# ❌ BAD: Multiple searches
search: "React hooks"
search: "React useState"
search: "React useEffect"

# ✅ GOOD: Comprehensive search
search: "React hooks useState useEffect patterns"
```

### Progressive Enhancement
```javascript
// Start simple, enhance only if needed
// Level 1: Try with existing knowledge
// Level 2: Single targeted search
// Level 3: Deep dive only if critical
```

## ⚡ Performance Patterns

### Lazy Loading
```javascript
// ❌ BAD: Load everything upfront
const allFiles = await loadAllProjectFiles();

// ✅ GOOD: Load only what's needed
const getFile = async (path) => {
  if (!cache[path]) {
    cache[path] = await loadFile(path);
  }
  return cache[path];
};
```

### Early Termination
```bash
# ❌ BAD: Process everything
find . -name "*.js" | xargs grep "pattern"

# ✅ GOOD: Stop when found
find . -name "*.js" -exec grep -l "pattern" {} \; | head -1
```

## 🎯 Decision Trees

### Should I Use a Tool?
```
Is it a fact question?
├─ YES → Is it current/changing info?
│   ├─ YES → Use tool (1 call)
│   └─ NO → Answer from knowledge
└─ NO → Is it a task?
    ├─ YES → Can I batch operations?
    │   ├─ YES → Single batched call
    │   └─ NO → Minimize calls
    └─ NO → Provide guidance
```

### Tool Selection
```
Need information?
├─ Web search → Current events, prices, news
├─ Read file → Specific file content
├─ Terminal → System state, file operations
└─ Analysis → Complex calculations only
```

## 📊 Efficiency Anti-Patterns

### Avoid These
```
❌ Searching for basic syntax
❌ Multiple calls for related info
❌ Reading files already in context
❌ Unnecessary state checks
❌ Redundant validations
❌ Over-verification
```

### Embrace These
```
✅ Batch related operations
✅ Cache expensive results
✅ Use knowledge first
✅ Combine commands
✅ Trust previous results
✅ Fail fast
```

## 💾 Resource Conservation

### API Token Optimization
```javascript
// ❌ BAD: Wasteful
for (const item of items) {
  await processItem(item); // API call each
}

// ✅ GOOD: Efficient
const batchSize = 100;
for (let i = 0; i < items.length; i += batchSize) {
  const batch = items.slice(i, i + batchSize);
  await processBatch(batch); // Single API call
}
```

### Memory Efficiency
```javascript
// ❌ BAD: Load entire file
const fullContent = await fs.readFile('large.csv');
const lines = fullContent.split('\n');

// ✅ GOOD: Stream processing
const stream = fs.createReadStream('large.csv');
const rl = readline.createInterface({ input: stream });
for await (const line of rl) {
  // Process line by line
}
```

## 🏃 Speed Optimizations

### Parallel vs Sequential
```javascript
// ❌ BAD: Sequential (slow)
const a = await fetchA();
const b = await fetchB();
const c = await fetchC();

// ✅ GOOD: Parallel (fast)
const [a, b, c] = await Promise.all([
  fetchA(),
  fetchB(),
  fetchC()
]);
```

### Short-Circuit Evaluation
```javascript
// Stop as soon as condition met
const result = await findFirst(items, async (item) => {
  return await checkCondition(item);
});
```

## 📈 Efficiency Report Card

### Rate Your Session
```
EXCELLENT (A+): 0-1 tool calls
- Answered from knowledge
- Batched if needed

GOOD (B): 2-3 tool calls
- Minimal exploration
- Efficient execution

NEEDS WORK (C): 4-5 tool calls
- Some redundancy
- Could batch better

INEFFICIENT (D): 6+ tool calls
- Excessive searching
- Poor batching
```

Remember: The best tool call is the one you don't make!

---
> Source: [DVC2/cursor_prompts](https://github.com/DVC2/cursor_prompts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
