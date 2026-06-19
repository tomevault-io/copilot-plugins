## mairu

> A dynamic context and memory storage system for coding agents. Provides native hybrid retrieval (vector + full-text + app-side re-ranking) backed by Meilisearch with local or remote embeddings. Exposes two interfaces: CLI and REST API (dashboard).

# mairu

A dynamic context and memory storage system for coding agents. Provides native hybrid retrieval (vector + full-text + app-side re-ranking) backed by Meilisearch with local or remote embeddings. Exposes two interfaces: CLI and REST API (dashboard).

## Tech Stack

- **Runtime:** Go 1.25+
- **Database:** Meilisearch 1.12+ (Docker)
- **Search:** Native hybrid — vector cosine similarity + full-text + app-side re-ranking (importance, recency decay)
- **Embeddings:** fastembed (local, `fast-all-MiniLM-L6-v2`, 384 dims) or OpenAI-compatible API
- **Frontend:** Svelte 5 + Vite
- **Testing:** Go testing (`go test`)
- **Linting:** `golangci-lint` (fallback: `go vet`)

## Setup

```bash
docker compose up -d    # start Meilisearch
bun install
bun --cwd mairu/ui install
cp .env.example .env    # fill in MEILI_URL, GEMINI_API_KEY, EMBEDDING_PROVIDER
make setup              # create Meilisearch indexes (destructive — drops and recreates)
```

## Commands

| Command | Description |
|---|---|
| `docker compose up -d` | Start Meilisearch container |
| `docker compose down` | Stop Meilisearch container |
| `make build` | Compile Go binary to `bin/mairu` |
| `make test` | Run Go tests |
| `make lint` | Run Go lint checks |
| `make clean` | Remove `mairu/bin/` |
| `make setup` | Init/reset Meilisearch indexes |
| `make dashboard` | Start context server (API) + Svelte dev UI |
| `bun run dashboard:dev` | Start Svelte dev server on port 5173 |
| `bun run dashboard:build` | Build Svelte UI |

### Evaluation

```bash
./mairu/bin/mairu eval:retrieval --dataset ./llmeval/sample_dataset.json --topK 5 --verbose true
./mairu/bin/mairu eval:retrieval --dataset ./llmeval/sample_dataset.json --topK 5 --fail-below-mrr 0.8 --fail-below-recall 0.75
```

## Architecture

### Data Types

- **Memories** — facts with category, owner, importance (1–10)
- **Skills** — capability name + description pairs
- **Context Nodes** — hierarchical tree nodes with abstract/overview/content levels, addressed by URI

### Retrieval Pipeline

Meilisearch handles vector + full-text search natively; app-side re-ranking applies recency decay and importance boosting:

1. **Vector search** — dense vector cosine similarity on embeddings
2. **Full-text** — Meilisearch built-in keyword search
3. **App-side re-ranking** — exponential recency decay + importance score boost
4. Results from both retrievers are merged and re-ranked before returning

Weights (vector, keyword, recency, importance) are defined in `mairu/internal/contextsrv/search_rerank.go`.

### Meilisearch Indexes

| Index | Key Fields |
|---|---|
| `contextfs_skills` | name (text), description (text), embedding (dense_vector), project (keyword) |
| `contextfs_memories` | content (text), category/owner (keyword), importance (integer), embedding (dense_vector) |
| `contextfs_context_nodes` | name/abstract/overview/content (text), uri/parent_uri (keyword), ancestors (keyword[]), embedding (dense_vector) |

### Search Features

| Feature | Description | Controlled by |
|---|---|---|
| **Vector search** | Dense cosine similarity on embeddings | `weights.vector` |
| **Full-text** | Meilisearch keyword search | `weights.keyword` |
| **Synonyms** | Custom synonym expansion (e.g., "k8s" → "kubernetes") | `SYNONYMS` env var |
| **Importance boost** | App-side boost on importance (1-10) | `weights.importance` |
| **Recency decay** | Exponential decay on created_at | `weights.recency`, `RECENCY_SCALE`, `RECENCY_DECAY` |
| **Min score cutoff** | Hard threshold to drop low-confidence results | `--minScore` |
| **Highlights** | Returns `<mark>`-tagged snippets showing matched terms | `--highlight` |
| **Field boosts** | Per-search field weight overrides | `fieldBoosts` option (API only) |

### Key Modules

| File | Role |
|---|---|
| `mairu/internal/contextsrv/meili.go` | Meilisearch integration, hybrid retrieval orchestration |
| `mairu/internal/contextsrv/service.go` | High-level API used by CLI |
| `mairu/internal/llm/openai_embedder.go` | OpenAI-compatible embedding calls |
| `mairu/internal/contextsrv/search_rerank.go` | Hybrid score blending and reranking weights |
| `mairu/internal/llm/router.go` | LLM-powered deduplication (CREATE / UPDATE / SKIP) |
| `mairu/internal/llm/ingestor.go` | Free-form text → structured context nodes |
| `mairu/internal/contextsrv/service_vibe.go` | LLM-driven free-text query planning and mutation planning |
| `mairu/cmd/mairu/main.go` | CLI entry point |
| `mairu/internal/web/server.go` | REST API for dashboard |
| `mairu/internal/eval/evaluate.go` | Evaluation harness entry point |
| `mairu/internal/daemon/daemon.go` | File watcher daemon: parallel processing, persistent cache, NL content assembly |
| `mairu/internal/ast/language_describer.go` | Pluggable interface for language-specific AST extraction + shared types/utilities |
| `mairu/internal/ast/typescript_describer.go` | TypeScript/JS implementation of LanguageDescriber (tree-sitter based) |
| `mairu/internal/ast/nl_describer.go` | AST-to-English engine: converts function bodies to numbered NL descriptions |
| `mairu/internal/ast/nl_enricher.go` | Post-enrichment pass: injects cross-function context into NL descriptions |

### AST Ingestion (Daemon)

The daemon watches a directory for TS/JS file changes and produces human-readable natural language descriptions of code via pure AST heuristics (no LLM calls).

**Architecture:** Single-pass AST walker behind a pluggable `LanguageDescriber` interface extracts symbols + edges + NL descriptions. A post-enrichment pass stitches cross-function references.

**Content field layout for file context nodes:**

| Field | Content |
|---|---|
| `abstract` | NL file summary — concise description of exported symbols and file purpose |
| `overview` | Compact graph notation — machine-readable symbol/edge listing for programmatic use |
| `content` | Full NL AST — statement-level English descriptions of every function/method body |

**NL generation** uses AST pattern matching to translate code constructs to English:
- Conditions: `x === null` → "`x` is null", `!x` → "`x` is falsy", `typeof x === "string"` → "`x` is a string"
- Control flow: if/else, for/while loops, try/catch, switch, throw — all described in plain English
- Cross-references: call edges enriched with callee context (e.g., "calls `validate` (which checks if input is falsy)")

**Performance features:**
- **Parallel processing** — configurable concurrency pool (default 8) for initial scan and batch changes
- **Persistent hash cache** — `.contextfs-cache.json` persists fingerprint/content/payload hashes so daemon restarts skip unchanged files
- **Triple-layer dedup** — file stat fingerprint → content SHA1 → payload SHA1 prevents unnecessary re-indexing

**Pluggable interface** — `LanguageDescriber` is designed for future language support. Currently only TypeScript/JS (via tree-sitter). To add a new language, implement the interface with `languageId`, `extensions`, and `extractFileGraph()`.

### Hierarchical Context (Tree Queries)

Context nodes store a materialized `ancestors` array. Tree operations:
- **Subtree**: filter `ancestors = nodeUri` finds all descendants
- **Path**: get node's ancestors array, then fetch the full chain by URI list

### LLM Deduplication

Before writing, `llmRouter` does a vector-only search. If cosine similarity ≥ 0.75, an LLM decides whether to CREATE, UPDATE, or SKIP the new entry.

## Configuration

Mairu relies on TOML configuration files instead of massive `.env` files.
A 5-tier configuration system determines settings: defaults -> user -> project -> env vars -> CLI flags.

View the fully resolved config layout:

```bash
mairu config list
mairu config get daemon.concurrency
```

You can initialize a project config or check health settings via:
```bash
mairu init --defaults
mairu doctor
```

# Agent Integration Instructions

To integrate OpenContextFS into Claude or Opencode using the CLI, refer to this section. You must use the terminal (`bash` tool) to invoke `mairu` (or compatibility alias `context-cli`).

**IMPORTANT**: Always use the `-P, --project <project>` flag when managing or searching memories/context so that information is correctly isolated by project.

### 1. Deterministic Retrieval (Recommended Default)
When starting a new session or debugging an issue, you MUST search memories and context nodes for existing constraints or decisions.
Prefer direct retrieval commands first so the agent can control scope and ranking behavior explicitly:

```bash
mairu memory search "authentication token validation rules" -k 5 -P my-project
mairu node search "authentication architecture" -k 5 -P my-project
mairu node ls "contextfs://my-project/backend/auth" -P my-project
```

### 2. Natural Language Storage (Recommended)
When you successfully complete a complex task, summarize the structural decisions and save them. `vibe-mutation` interprets your instructions and automatically updates/creates memories and nodes.
**Note:** Always pass `-y` to auto-approve mutations.

```bash
mairu vibe-mutation "remember that we switched from REST to gRPC for internal service calls" -P my-project -y
```

### 3. Natural-Language Retrieval
Use direct `memory search` and `node search` for all retrieval needs. Combine multiple targeted searches for broad or ambiguous questions.

### 4. Advanced/Precise Operations
Use direct commands when you need exact control over what is stored or retrieved.

**Memory Store:**
```bash
mairu memory store "In project X, we use Vitest instead of Jest for unit testing." -c observation -o agent -i 5 -P my-project
```

**Memory Search:**
```bash
mairu memory search "testing framework" -k 5 -P my-project

# With highlights
mairu memory search "authentication setup" -k 5 -P my-project --highlight
```

**Managing Context Nodes (Hierarchical Knowledge):**
```bash
mairu node store "contextfs://my-project/backend/auth" "Auth Module" "Uses JWT with RSA signatures." -P my-project
mairu node ls "contextfs://my-project/backend" -P my-project
```

Agents should proactively search memories and context nodes when beginning a task, and store important discoveries or user preferences as they work.

### 5. Bash History Auto-Logging (Built-in)
When you execute bash commands through the mairu agent, they are **automatically logged** to the searchable history. This includes:
- The command string
- Exit code (success/failure)
- Execution duration
- Output (stdout/stderr)

This happens transparently via the `HistoryLogger` interface in `mairu/internal/agent`. You can query this history anytime:

```bash
# Find similar commands that failed before
mairu history search "docker build failed" -P my-project

# See most frequently used commands
mairu history stats -P my-project
```

No manual action is required - the agent handles this automatically when using the `bash` tool.

### AI-Optimized GNU Tools
Agents are encouraged to use the `mairu` binary for token-dense, strictly parsable exploration:
- `mairu map [dir] -d 2` -> Fast, `.gitignore` aware, token-counted directory tree
- `mairu outline <file>` -> Emits imports and logic symbols (classes, functions) via AST
- `mairu peek <file> -s <symbol>` -> Smart, bracket-aware symbol extraction (no `sed`/`head` needed!)
- `mairu scan <regex> [dir] -C 1 -e .go -H -n 5` -> Token-budgeted regex search preventing context window blowouts
- `mairu sys` -> Quick system status and memory check
- `mairu info [dir]` -> Repository analytics (token sizes, file counts, extensions)
- `mairu env [file] -r` -> Smart env reader. Extracts keys, flags secrets (`is_secret: true`), and safely reveals non-sensitive config values (booleans, ports)

### Extended Namespaces (Code Analysis, Scraping & History)
Mairu commands are grouped into logical namespaces.

**Bash Command History (`mairu history`)**
Agents can query the developer's bash history to understand previous commands, outputs, and workflows:
- `mairu history search "test fail"` -> Semantically search past bash commands and their outputs.
- `mairu history stats` -> Show the most frequently run bash commands.
- `mairu history feedback <id> -r 10` -> Apply reinforcement learning feedback to a command execution.
- `mairu history import --from ~/.zsh_history --project my-project` -> Backfill shell history (zsh or bash) into the searchable store. Every entry is run through the `internal/redact` pipeline before persistence — Layer 1 (known-token regex), Layer 2 (arg/flag heuristics), Layer 3 (entropy), Layer 4 (credential-handling tool denylist), Layer 5 (damage cap). Entries whose damage cap triggers are dropped entirely. Dedup is by sha256 of the redacted command within a single import run. Supports `--dry-run` for preview and `--format zsh|bash` to override filename-based detection.

**Real-time shell integration (zsh, bash, fish):**
Run `mairu ingestd run &` once per session (or under launchd/systemd), then add the appropriate init line to your shell rc file:

- **zsh (`~/.zshrc`):** `eval "$(mairu shell init zsh)"` — uses `add-zsh-hook` with `preexec`/`precmd` and `$EPOCHREALTIME` for sub-ms timing.
- **bash (`~/.bashrc`):** `eval "$(mairu shell init bash)"` — uses a `DEBUG` trap plus `PROMPT_COMMAND`. Requires bash 5.0+ for `$EPOCHREALTIME`; older bash silently reports `duration_ms=0`.
- **fish (`~/.config/fish/config.fish`):** `mairu shell init fish | source` — uses `fish_preexec` / `fish_postexec` event handlers and the built-in `$CMD_DURATION` (already in milliseconds).

Every command you run is captured by the hook, sent non-blockingly over a Unix socket (`$MAIRU_INGEST_SOCK`, default `~/.mairu/ingest.sock`), redacted through `internal/redact`, and stored in the same searchable bash-history as imports and agent-logged commands. Metadata only — command + exit code + duration + cwd. Set `MAIRU_NO_HOOK=1` to suspend capture without editing your rc file. If the daemon isn't running, the client fails silently — the shell prompt never blocks or errors.

**Opt-in output capture (per-command):**
For the commands where you *do* want the stdout/stderr remembered (build logs, test failures, deployment traces), prefix them with `mairu capture`:

```bash
mairu capture -- make deploy
mairu capture --max-output 16384 -- npm test
mairu capture --project my-project -- go test ./...
```

The command runs exactly as if you'd typed it directly — stdout/stderr stream live to your terminal, stdin is wired through, the exit code is propagated. In parallel, a buffered copy of the combined output (default cap: 64 KiB) is sent to ingestd alongside command metadata and run through `internal/redact.KindText` before persistence. If the output is >50% redacted (damage cap), the stored output is replaced with `[REDACTED:damage_cap]` but the command row is kept — so you can search for `"make deploy"` and see that it ran even when the payload was too hollow to keep. `mairu capture` is opt-in per-command precisely because command output is the highest-leakage surface in shell history; it is deliberately NOT wired into the always-on shell hook. Interactive TTY commands (`vim`, `htop`, `less`) don't currently work under `mairu capture` — run them directly.

**Automatic Logging:** When using the mairu agent (via `mairu tui`, `mairu web`, or headless mode), all bash commands executed by the agent are **automatically stored** in mairu history through the `HistoryLogger` interface. This includes command, exit code, duration, and output - making your command history fully searchable for future sessions without any manual intervention.

**Manual Logging (when NOT using mairu agent):**
If you're operating outside the mairu agent context (e.g., as a standalone agent), you must manually preserve important commands:

```bash
# Store complex or important commands as memories
mairu memory store "Command: go test ./... -race | Result: All passed | Purpose: Race condition verification" -c command -o agent -i 6 -P my-project

# Store debugging commands that solved issues
mairu memory store "Debug fix: lsof -i :8080 | kill -9 <pid> | Frees blocked port" -c debugging -o agent -i 8 -P my-project
```

Log commands manually when: they took time to figure out, solved a tricky problem, or will likely be reused. Use format: `Command: <cmd> | Result: <summary> | Purpose: <why>`

**Code Analysis (`mairu analyze`)**
- `mairu analyze diff` -> Analyzes blast radius of current git changes.
- `mairu analyze graph` -> Analyzes the codebase graph to build project understanding.

**Web Scraping (`mairu scrape`)**
Agents can extract documentation or read web sources using LLM-powered scrapers:
- `mairu scrape web <url>` -> Fetch, summarize, and store as context node.
- `mairu scrape smart <url> --prompt "..."` -> Extract structured data via LLM.
- `mairu scrape search <query>` -> Search the web and extract structured data.
- `mairu scrape multi <url1> <url2>` -> Scrape multiple URLs concurrently.
- `mairu scrape depth <url> -d 2` -> Crawl up to depth 2 and extract.
- `mairu scrape omni <urls...>` -> Scrape and merge results into a single summary.
- `mairu scrape script <url>` -> Auto-generates a Go `goquery` scraper script for a given URL.

### CLI Command Reference
Agents should be aware of the exact CLI structure. Below are the primary namespaces and their subcommands:

<details>
<summary>Memory Commands (`mairu memory`)</summary>

```
ContextFS memory operations

Usage:
  mairu memory [command]

Available Commands:
  add         Alias for memory store
  delete      Delete memory
  feedback    Apply reinforcement learning feedback to a memory (reward 1-10)
  list        List memories
  search      Search memories
  store       Store a memory
  update      Update a memory

Flags:
  -h, --help             help for memory
  -P, --project string   Project name

Global Flags:
      --debug           Enable debug logging
  -o, --output string   Output format: table, json, plain (default "table")
      --quiet           Only output results, no status messages
      --verbose         Show extra details (timing, weights, query plan)

Use "mairu memory [command] --help" for more information about a command.

```
</details>

<details>
<summary>Scrape Commands (`mairu scrape`)</summary>

```
Web scraping and content extraction tools

Usage:
  mairu scrape [command]

Available Commands:
  depth        Fetch a URL, discover relevant links up to depth K, and extract data concurrently
  multi-scrape Fetch multiple URLs concurrently and extract structured data using LLM
  omni-scrape  Fetch multiple URLs concurrently and merge extracted data into a single summary
  script       Generate a Go scraping script using goquery tailored for a specific URL
  search       Search web for query and extract structured data from top results using LLM
  smart        Fetch a URL and extract structured data using LLM based on prompt
  web          Fetch a URL, extract content, summarize, and store as a context node

Flags:
  -h, --help   help for scrape

Global Flags:
      --debug           Enable debug logging
  -o, --output string   Output format: table, json, plain (default "table")
      --quiet           Only output results, no status messages
      --verbose         Show extra details (timing, weights, query plan)

Use "mairu scrape [command] --help" for more information about a command.

```
</details>

<details>
<summary>Analyze Commands (`mairu analyze`)</summary>

```
Analyze codebase graphs and diffs

Usage:
  mairu analyze [command]

Available Commands:
  diff        Analyze the current git diff against the codebase graph to determine blast radius
  graph       Analyze the AST graph to generate execution flows and functional clusters (skills)

Flags:
  -h, --help   help for analyze

Global Flags:
      --debug           Enable debug logging
  -o, --output string   Output format: table, json, plain (default "table")
      --quiet           Only output results, no status messages
      --verbose         Show extra details (timing, weights, query plan)

Use "mairu analyze [command] --help" for more information about a command.

```
</details>

<details>
<summary>History Commands (`mairu history`)</summary>

```
Manage and search your bash command history

Usage:
  mairu history [command]

Available Commands:
  feedback    Apply reinforcement learning feedback to a bash history item (reward 1-10)
  search      Semantically search past bash commands and outputs
  stats       Show the most frequently run bash commands

Flags:
  -h, --help   help for history

Global Flags:
      --debug           Enable debug logging
  -o, --output string   Output format: table, json, plain (default "table")
      --quiet           Only output results, no status messages
      --verbose         Show extra details (timing, weights, query plan)

Use "mairu history [command] --help" for more information about a command.

```
</details>

Use `mairu <command> --help` to discover exact flags for any specific command.

---
> Source: [enekos/mairu](https://github.com/enekos/mairu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
