## matryoshka

> This project uses OpenWolf for context management. Read and follow .wolf/OPENWOLF.md every session. Check .wolf/cerebrum.md before generating code. Check .wolf/anatomy.md before reading files.

# OpenWolf

@.wolf/OPENWOLF.md

This project uses OpenWolf for context management. Read and follow .wolf/OPENWOLF.md every session. Check .wolf/cerebrum.md before generating code. Check .wolf/anatomy.md before reading files.


# Claude Code Guidelines for Matryoshka RLM

## What This Project Does

**Matryoshka RLM** (Recursive Language Model) processes documents 100x larger than an LLM context window without RAG or chunking. The LLM emits **Nucleus commands** (constrained S-expressions), which are parsed, type-checked, and executed by the **Lattice logic engine**. Results are stored server-side in SQLite — the LLM sees only descriptive handle stubs derived from the command (`$grep_error: Array(1000) [preview...]`), achieving **97%+ token savings**.

```
User Query → LLM Reasons → Nucleus S-expression
                            → Parser validates → Lattice Engine executes
                            → Results stored in SQLite
                            → LLM sees handle stub only
```

## CRITICAL: No Hardcoding

**DO NOT hardcode specific use cases into the prompts or code.**

This is a GENERAL-PURPOSE document analysis tool. When writing prompts or examples:
- Use GENERIC patterns, not domain-specific ones
- Say "data" not "sales data", "values" not "currency values"
- Let the LLM discover the actual data format from the document

## Architecture Overview

### Core Pipeline
- **RLM Entry** (`src/rlm.ts`): Main CLI, document loading, LLM loop orchestration
- **FSM Engine** (`src/fsm/engine.ts`): Generic finite state machine (repl-sandbox)
- **RLM States** (`src/fsm/rlm-states.ts`): QueryLLM → ParseResponse → Execute → Verify → TermOrContinue
- **Adapters** (`src/adapters/`): Model-specific prompting (nucleus, qwen, deepseek)

### Lattice Engine (the core)
- **Parser** (`src/logic/lc-parser.ts`): S-expression lexer → tokens → AST
- **Solver** (`src/logic/lc-solver.ts`): Term evaluator using miniKanren for relational reasoning
- **Type Inference** (`src/logic/type-inference.ts`): Type checking before execution
- **Search** (`src/logic/bm25.ts`, `semantic.ts`, `rrf.ts`): BM25, TF-IDF, Reciprocal Rank Fusion
- **Q-value Reranker** (`src/logic/qvalue.ts`): Learns across turns

### Handle System (97% token savings)
- **SessionDB** (`src/persistence/session-db.ts`): In-memory SQLite with FTS5
- **HandleRegistry** (`src/persistence/handle-registry.ts`): Create/manage handle references
- **HandleOps** (`src/persistence/handle-ops.ts`): Server-side filter/map/count/sum on handles
- **HandleSession** (`src/engine/handle-session.ts`): Wraps NucleusEngine + handle storage

### Nucleus Engine
- **NucleusEngine** (`src/engine/nucleus-engine.ts`): Standalone command executor, bindings, cross-query state
- **Bindings**: `RESULTS` (latest array), `_1`/`_2`/... (per-turn), `_fn_name` (synthesized fns)

### Synthesis (Barliman-style)
- **Coordinator** (`src/synthesis/coordinator.ts`): Routes to regex/extractor/relational synthesizers
- **Evalo DSL** (`src/synthesis/evalo/`): Type-checked extraction language
- **miniKanren** (`src/minikanren/`): Relational programming engine for constraint solving
- LLM provides CONSTRAINTS (input/output examples), NOT code — synthesizer builds programs

### Code Intelligence
- **Tree-sitter** (`src/treesitter/`): Symbol extraction (functions, classes, methods) for .ts/.js/.py/.go/.md
- **Symbol Graph** (`src/graph/`): Knowledge graph — callers, callees, ancestors, descendants, implementations

### Entry Points / Adapters
- `src/lattice-mcp-server.ts` — MCP server for Claude Code (handle-based)
- `src/mcp-server.ts` — MCP server with full LLM orchestration
- `src/tool/adapters/pipe.ts` — JSON subprocess control (stdin/stdout)
- `src/tool/adapters/http.ts` — REST API with session lifecycle
- `src/tool/adapters/claude-code.ts` — Auto-registers as MCP tools

### Session Management
- Session timeout: **10 minutes** inactivity (configurable in `lattice-mcp-server.ts`)
- Timer resets on every query
- Single session per MCP instance
- Max document: 50MB, max handles: 200 (LRU eviction), max grep matches: 10,000

## Key Principle: Barliman-Style Synthesis

The LLM provides CONSTRAINTS (input/output examples), NOT code implementations.
The synthesizer builds programs automatically from examples.

## Using Nucleus for Large File Analysis

When you need to analyze files larger than ~500 lines, use the Nucleus tool instead of reading files directly. This saves 80%+ tokens.

### Recommended Workflow for Codebase Analysis
1. **Use Glob first** to discover all relevant files (e.g., `**/*.py`, `**/*.ts`)
2. **Read small files directly** (<300 lines) - Nucleus is overkill for these
3. **Use Nucleus only for large files** (>500 lines)
4. **Aggregate data across ALL files**, not just the largest one

This workflow ensures complete analysis. Using Nucleus alone misses:
- Small config/utility files with important details
- Multi-file patterns (imports, classes across files)
- File discovery and project structure

### When to Use Nucleus
- File is >500 lines
- You need multiple searches on the same file
- You're extracting or aggregating structured data
- Exploratory analysis (don't know what you're looking for)

### When NOT to Use
- File is <300 lines (just read it directly)
- You only need one search
- You need full document context/structure
- You haven't discovered files yet (use Glob first)

### Quick Start (Programmatic)
```typescript
import { PipeAdapter } from "./src/tool/adapters/pipe.ts";

const nucleus = new PipeAdapter();
await nucleus.executeCommand({ type: "load", filePath: "./large-file.txt" });

// Search - returns only matching lines
const result = await nucleus.executeCommand({
  type: "query",
  command: '(grep "pattern")'
});

// Chain operations - RESULTS persists
await nucleus.executeCommand({ type: "query", command: "(count RESULTS)" });
await nucleus.executeCommand({ type: "query", command: "(sum RESULTS)" });
```

### Handle System

Query results are stored server-side as handles with **descriptive names derived from the command** (e.g. `(grep "ERROR")` → `$grep_error`, `(list_symbols "function")` → `$list_symbols_function`). Repeated commands get numeric suffixes (`$grep_error_2`). You receive compact stubs instead of full data, saving 97%+ tokens. Use `lattice_expand` to inspect actual content when needed.

- **`lattice_expand`** - View full data from a handle (supports `limit`, `offset`, `format`)
- **`lattice_bindings`** - Show all active handles and their stubs
- **`lattice_reset`** - Clear all handles but keep the document loaded
- **`lattice_status`** - Session info, active handles, timeout

### Common Queries

#### Search
```scheme
(grep "pattern")                    ; Regex search
(bm25 "query terms" 10)            ; BM25 ranked keyword search
(semantic "query terms" 10)         ; TF-IDF cosine similarity search
(fuzzy_search "query" 10)          ; Fuzzy text search
(text_stats)                        ; Document statistics
(lines 10 20)                       ; Get specific line range (1-indexed)
```

#### Chunking (pre-slice a large document before mapping over it)
```scheme
(chunk_by_size 2000)                ; 2000-character slices
(chunk_by_lines 100)                ; 100-line slices
(chunk_by_regex "\\n\\n")           ; Split on blank lines; capture groups ignored
```
Canonical pattern for per-chunk semantic work:
```scheme
(map (chunk_by_lines 100)
     (lambda c (llm_query "summarize: {chunk}" (chunk c))))
```
Each chunk becomes the `{chunk}` placeholder in a sub-LLM call — the
paper's OOLONG-Pairs pattern applied to a single large document.

#### Sub-LLM (multi-turn — works with any MCP client)
```scheme
(llm_query "summarize this")                                    ; bare prompt
(llm_query "classify: {items}" (items RESULTS))                 ; with binding
(map RESULTS (lambda x (llm_query "tag: {item}" (item x))))     ; per-item, N suspensions
(llm_batch RESULTS (lambda x (llm_query "tag: {item}" (item x)))) ; per-item, ONE suspension
(map RESULTS (lambda x (llm_query "Rate: {name}" (name x) (body (get_symbol_body x)))))  ; symbols + body
(filter RESULTS (lambda x (match (llm_query "keep?: {item}" (item x)) "keep" 0)))
```
When `(llm_query ...)` is evaluated, execution suspends and returns a
`[LLM_QUERY_REQUEST id=...]` message. The MCP client responds via
`lattice_llm_respond` to resume execution. For queries with multiple
`llm_query` calls (e.g., inside `map`), each item triggers one suspension —
respond to each in turn until the final handle stub or scalar is returned.

**Prefer `llm_batch` over `map`+`llm_query` for independent per-item
judgment tasks.** `llm_batch COLL (lambda x (llm_query ...))` is a
drop-in replacement with identical surface syntax. The solver collects
every interpolated prompt into a single array and fires ONE suspension
(`[LLM_BATCH_REQUEST id=... count=N]`) carrying all N prompts; the
client replies once with a JSON array of N responses via
`lattice_llm_batch_respond`. Benchmarked at ~92% round-trip reduction
on N=12 and ~99% on N=100 — the dominant cost in the map+llm_query
pattern is protocol overhead, not per-item work. Use `map`+`llm_query`
only when the per-item judgment needs to reference prior items or
when the lambda body is more complex than a single `llm_query`.

**Constrain responses with `(one_of …)` for classification tasks.**
Attach an enum to any `llm_query` / `llm_batch` to make downstream
`(filter …)` / `(count …)` reliable without having to re-normalize
free-text output:
```scheme
(llm_batch RESULTS
  (lambda x (llm_query "Rate: {item}" (item x)
                       (one_of "low" "medium" "high"))))
```
The solver appends a directive naming the allowed values to each
prompt, then validates every response case-insensitively and
canonicalizes matches to the declared spelling. Out-of-set responses
fail the query with a specific error naming the offending item so
you can retry or investigate. Without this, `(filter $res (lambda x
(match x "low" 0)))` silently drops "LOW", "  low\n", or "low —
simple function" because the raw LLM output drifts.

**Add `(calibrate)` to `llm_batch` for subjective-judgment tasks.**
```scheme
(llm_batch RESULTS
  (lambda x (llm_query "Rate: {item}" (item x)
                       (one_of "low" "medium" "high")
                       (calibrate))))
```
The marker tells the MCP bridge to prepend a directive to the batched
suspension request asking the model to scan all N prompts first and
establish a consistent relative scale before answering any. Useful
when ratings depend on the distribution of the corpus (complexity,
severity, importance) rather than being absolute.

**Data model in map/filter lambdas:** each item is converted to a string —
grep results → matched line text, symbol objects → symbol name, chunks →
chunk text. Inside a lambda, the parameter is this string value. Operations
like `get_symbol_body`, `find_references`, `llm_query` work inside lambdas.

**Client compatibility:**
- ✅ **All MCP clients** — uses multi-turn suspension protocol (no sampling needed).
- ✅ **Claude Desktop** — also uses native MCP sampling when available for lower latency.

#### Symbol Operations (.ts, .js, .py, .go, .md, etc.)
```scheme
(list_symbols)                      ; List all symbols (functions, classes, headings, etc.)
(list_symbols "function")           ; Filter by kind: "function", "class", "method", "interface", "type", "struct"
(get_symbol_body "funcName")        ; Get source code for a symbol
(get_symbol_body RESULTS)           ; Get source code from previous query result
(find_references "identifier")      ; Find all references to an identifier
```

#### Graph Operations (knowledge graph for code structure)
```scheme
(callers "funcName")                ; Who calls this function?
(callees "funcName")                ; What does this function call?
(ancestors "ClassName")             ; Inheritance chain (extends)
(descendants "ClassName")           ; All subclasses (transitive)
(implementations "InterfaceName")   ; Classes implementing this interface
(dependents "name")                 ; All transitive dependents
(dependents "name" 2)               ; Dependents within depth limit
(symbol_graph "name" 1)             ; Neighborhood subgraph around symbol
```

#### Graph Analysis (community detection & structural insights)
```scheme
(communities)                       ; Detect communities with cohesion scores
(community_of "name")               ; Which community does this symbol belong to?
(god_nodes)                         ; Top 10 most-connected nodes (hubs)
(god_nodes 5)                       ; Top N most-connected nodes
(surprising_connections)            ; Cross-community or low-confidence edges
(surprising_connections 5)          ; Top N surprising connections
(bridge_nodes)                      ; Nodes bridging different communities
(bridge_nodes 5)                    ; Top N bridge nodes
(suggest_questions)                 ; Questions the graph is uniquely positioned to answer
(graph_report)                      ; Full analysis (all of the above combined)
```

#### Collection Operations
```scheme
(count RESULTS)                     ; Count items
(sum RESULTS)                       ; Sum numeric values
(reduce RESULTS init fn)            ; Generic reduce
(map RESULTS (lambda (x) (match x "regex" 1)))   ; Extract/transform data
(filter RESULTS (lambda (x) (match x "pat" 0)))  ; Filter results
```

#### Predicates (for filter)
```scheme
(lambda (x) (match x "pattern" group))           ; Regex match predicate
(classify "line1" true "line2" false)             ; Build classifier from examples
```

#### String Operations
```scheme
(match str "pattern" 1)             ; Extract regex group
(replace str "from" "to")          ; Replace pattern in string
(split str "delim" index)          ; Split and get part at index
(parseInt str)                      ; Parse string to integer
(parseFloat str)                    ; Parse string to float
```

#### Type Coercion
```scheme
(parseDate str)                     ; Parse date string to ISO format
(parseCurrency str)                 ; Parse currency string to number
(parseNumber str)                   ; Parse numeric string with separators
(coerce term "type")                ; Coerce to: date/currency/number/boolean/string
```

#### Synthesis
```scheme
(synthesize (example "in1" out1) (example "in2" out2) ...)  ; Synthesize function from examples
```

#### Variables
```scheme
RESULTS                             ; Last array result (auto-bound)
_1, _2, _3, ...                    ; Results from turn N (auto-bound)
context                             ; Raw document content
```

Note: Descriptive handle stubs (e.g. `$grep_error`, `$filter`, `$map`) are for `lattice_expand` only — do NOT reference them inside Nucleus queries. Use `RESULTS` (latest array) or `_1`, `_2`, `_3` (per-turn) to chain queries.

### Multi-Signal Search Pipeline
```scheme
;; Fuse multiple search signals using Reciprocal Rank Fusion
(fuse (grep "ERROR") (bm25 "error handling") (semantic "failure"))

;; Remove false positives with gravity dampening
(dampen (bm25 "database error") "database error")

;; Q-value learning reranker (learns across turns)
(rerank (fuse (grep "ERROR") (bm25 "error")))

;; Full pipeline
(rerank (dampen (fuse (grep "ERROR") (bm25 "error") (semantic "failure")) "error"))
```

### Example Workflows

#### Symbol Workflow
```
1. (list_symbols "function")         → $list_symbols_function: Array(15) [preview]
2. (get_symbol_body "myFunction")    → Returns source code directly
3. (find_references "myFunction")    → $find_references_myfunction: Array(8) [references]
```

#### Graph Workflow
```
1. (callers "handleRequest")         → $callers_handlerequest: Array(3) [who calls it]
2. (callees "handleRequest")         → $callees_handlerequest: Array(5) [what it calls]
3. (ancestors "MyService")           → $ancestors_myservice: [BaseService, EventEmitter]
4. (symbol_graph "handleRequest" 2)  → Subgraph: 12 nodes, 15 edges
5. (communities)                     → $communities: [{id: 0, nodes: [...], cohesion: 0.67}, ...]
6. (graph_report)                    → Full analysis: god nodes, surprises, bridges, questions
```

#### Markdown Workflow
```
1. (list_symbols)                    → $list_symbols: Array(12) [# Intro, ## Setup, ...]
2. (grep "## Installation")         → Find specific section content
```

### Memory Pad (lattice_memo)

Store arbitrary context as handle-backed memos for token-efficient roundtripping.
Instead of carrying full context in every message, store it server-side and reference by stub.

```
# Store context — no lattice_load required
lattice_memo content="<file summary or analysis>" label="auth module architecture"
→ $memo_auth_module_architecture: "auth module architecture" (2.1KB, 50 lines)

# Continue working with just the stub (~15 tokens instead of ~500)
# Pull back full content only when needed:
lattice_expand $memo_auth_module_architecture
→ Full text

# Memos persist across document loads
lattice_load ./other-file.ts    # $memo_auth_module_architecture survives this
```

**IMPORTANT — Maintain a memo index**: After stashing memos, include a brief index
in your response so you know what's available on future turns without calling
`lattice_bindings`. Example:

```
Stashed memos:
- $memo_auth_module_architecture — key types, middleware chain, session flow
- $memo_perf_bottleneck_analysis — hot paths, allocation sites, recommendations
```
Memo names are derived from the label you pass to `lattice_memo` (repeated
labels get numeric suffixes like `$memo_auth_2`). Labels without slug content
fall back to `$memo`, `$memo_2`, …

Keep the index to one line per memo (handle + label + what's inside). Update it
when you add or delete memos. This costs ~10 tokens per memo but saves a tool call
every time you need to decide what to expand.

**Token savings**: ~93% over a 30-message session with 3 source files stashed as memos.
Traditional roundtripping: 836K tokens. Memo-based: 57K tokens.

**Session timeout**: 10 min inactivity (configurable via `LATTICE_TIMEOUT_MS` env var).
Every tool call resets the timer.

### HTTP Server Option
```bash
# Start server
npx tsx src/tool/adapters/http.ts --port 3456

# Load document
curl -X POST http://localhost:3456/load -d '{"filePath":"./file.txt"}'

# Query
curl -X POST http://localhost:3456/query -d '{"command":"(grep \"ERROR\")"}'
```

---
> Source: [yogthos/Matryoshka](https://github.com/yogthos/Matryoshka) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
