## gowork-local-llm

> Use the local `gowork` CLI (backed by MLX + Qwen3.5-4B) to offload token-heavy tasks (file summarization, data cleaning, text extraction, format conversion, bulk file processing) to a local LLM. Prefer `gowork --role summarize` which bundles the right model and flags automatically; fall back to `gowork --no-tools` if roles aren't configured. Invoke this skill whenever you need to process large files, summarize documents, clean messy data, extract structured info from unstructured text, or do any repetitive text transformation that would consume significant Claude tokens. Even for seemingly simple file reads — if the file is large (>200 lines) and the user just needs a summary or key points, use gowork instead of reading the entire file yourself.


# gowork Local LLM Offloader

Use the local `gowork` command to delegate token-intensive work to a free local model (MLX + Qwen3.5-4B-OptiQ-4bit), saving Claude API tokens for higher-value reasoning.

## Backend Architecture

```
gowork CLI (Rust, 30ms startup)
    │
    ▼
mlx_lm.server (:8080)
    │  OpenAI-compatible API
    ▼
Qwen3.5-4B-OptiQ-4bit (MLX, Apple Silicon optimized)
```

- **Backend**: `mlx_lm.server` running on `http://localhost:8080/v1`
- **Model**: `mlx-community/Qwen3.5-4B-OptiQ-4bit` (~3.4GB memory)
- **Hardware**: Apple M5 32GB, MLX framework (比 Ollama 快 20-50%)
- **Thinking mode**: 已关闭 (enable_thinking=false)，节省时间和内存

## When to Use gowork vs Claude

**Use gowork for** (mechanical, high-token tasks):
- Summarizing large files or documents
- Cleaning/formatting messy data (CSV, JSON, logs)
- Extracting specific fields from unstructured text
- Bulk text transformations across multiple files
- Getting a quick overview of a file's contents before deciding what to do
- Translating or reformatting text
- Generating boilerplate from templates

**Keep in Claude** (reasoning-heavy tasks):
- Architecture decisions, code review, debugging
- Multi-step planning that requires context from the conversation
- Tasks where quality matters more than token savings
- Anything that needs access to Claude's tools (file editing, git, etc.)

## Key Flags (Optimized for Preprocessing)

### Preferred: Use `--role summarize`

The user has a `summarize` role preset configured that already bundles `--no-tools` + the 4B model. **Always prefer this**:

```bash
# Simplest form — role takes care of --no-tools + model selection
gowork --role summarize --file big.go -p "list all functions"
gowork --role summarize -p "extract errors from this log"
```

For heavy code analysis (MoE 30B model), use `--role code`:
```bash
gowork --role code --file complex_module.rs -p "find bugs in this code"
```

For cloud fallback (when local is busy):
```bash
gowork --role deepseek --file data.json -p "clean and restructure"
```

Run `gowork --list-roles` to see all available roles on the user's system.

### Fallback: Manual `--no-tools`

If the user has no role config yet, fall back to `--no-tools` explicitly:

```bash
# Manual form (equivalent to --role summarize in role-aware setups)
gowork --no-tools -p "your prompt"
gowork --no-tools --file path/to/file.go -p "list all functions"

# Pipe stdin
cat file.txt | gowork --no-tools -p "summarize in 3 bullet points"
grep "^func " file.go | gowork --no-tools -p "list function names"

# Batch process multiple files (single process, reuses connection)
gowork --no-tools --batch "src/*.go" -p "one-line summary: {}"

# Cache results (skip re-processing unchanged files)
gowork --no-tools --cache --file data.csv -p "list column names"
```

### Full flag reference

| Flag | Purpose |
|------|---------|
| `-p "prompt"` | One-shot mode, runs prompt and exits |
| `--role <name>` | **Preferred.** Apply a named preset (`summarize`/`code`/`deepseek`/...). Bundles `--no-tools` + model + backend |
| `--list-roles` | Show all configured roles |
| `--no-tools` | Skip 8 tool definitions, 35% faster (implied by `--role summarize`) |
| `--file path` | Read file and prepend content to prompt |
| `--batch "glob"` | Process multiple files. `{}` = content placeholder |
| `--cache` | Cache by file path + mtime + prompt |
| `--stats` | Show token savings statistics |
| `--stats-reset` | Reset all statistics |
| `-m model` | Override model |
| `--base-url` | Override API URL |

## Direct curl Alternative (Zero Overhead)

For maximum speed, bypass gowork entirely and call MLX API directly:

```bash
curl -s http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d "{\"model\":\"mlx-community/Qwen3.5-4B-OptiQ-4bit\",\"messages\":[{\"role\":\"user\",\"content\":\"$(cat file.txt | head -100 | sed 's/"/\\"/g; s/$/\\n/' | tr -d '\n')summarize in 3 bullet points\"}],\"temperature\":0.1,\"max_tokens\":512}" \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['choices'][0]['message']['content'])"
```

Use curl when: you need absolute minimum latency, or gowork binary is unavailable.
Use gowork when: cleaner syntax matters, or you need --batch/--cache features.

## Environment Configuration

The following environment variables should already be set in the user's `~/.zshrc`:

```bash
export GOWORK_BASE_URL="http://localhost:8080/v1"
export GOWORK_MODEL="mlx-community/Qwen3.5-4B-OptiQ-4bit"
```

If gowork fails with "connection refused", the MLX server may not be running. Tell the user:
```bash
mlx-start   # alias to start mlx_lm.server
```

## Speed Optimization

### 原则一：先筛再喂（减少输入）

输入量直接决定速度。不要把整个文件丢给模型，先用 grep/head/awk 筛出关键内容：

```bash
# 慢 (18s)：把 500 行全喂给模型
gowork --no-tools --file bigfile.go -p "summarize"

# 快 (5s)：先 grep 关键行
grep "^func " file.go | gowork --no-tools -p "list functions with descriptions"

# 快 (3s)：只喂文件头部
head -50 file.go | gowork --no-tools -p "what does this file do? one sentence"
```

### 原则二：限制输出长度

```bash
# 慢：模型自由发挥
gowork --no-tools -p "总结这个文件"

# 快：限制输出
gowork --no-tools -p "一句话总结"
gowork --no-tools -p "列出函数名，每行一个，不要解释"
```

### 原则三：用 --cache 避免重复计算

```bash
# 第一次：调模型 ~5s
gowork --no-tools --cache --file config.rs -p "list functions"

# 第二次：命中缓存 ~0.1s（文件没改过）
gowork --no-tools --cache --file config.rs -p "list functions"
```

Cache 按 file path + modification time + prompt 做 key，文件修改后自动失效。

### 原则四：用 --batch 批量处理

```bash
# 一个进程处理所有文件，复用连接
gowork --no-tools --batch "src/*.rs" -p "one-line summary: {}"

# 带缓存的批量处理
gowork --no-tools --cache --batch "docs/*.md" -p "extract title and summary: {}"
```

### 不要做的事

- **不要省略 `--no-tools`** — 预处理不需要工具，省掉 tool 定义能快 35%
- **不要开多个 mlx_lm.server 实例** — 浪费内存，抢 GPU
- **不要把整个大文件直接喂** — 先 grep/head 筛选

## Prompting Tips for 4B Model

The local model is a 4B parameter model — fast and lightweight but not as strong as Claude:

- **Short and direct**: one clear instruction, no multi-step chains
- **Concrete**: "list the column names" not "analyze the data structure"
- **Formatted simply**: plain text, bullet points, or simple JSON only
- **Single-task**: break complex work into multiple calls
- **Output-constrained**: always specify output format and length limit

## Common Patterns

All examples below use `--role summarize` (preferred). If the user has no role config, replace `--role summarize` with `--no-tools`.

### File overview
```bash
gowork --role summarize --file unknown_file.py -p "what does this code do? one sentence"
```

### Extract function list
```bash
grep "^func " file.go | gowork --role summarize -p "list function names, one per line"
```

### Summarize large file
```bash
head -100 report.md | gowork --role summarize -p "summarize in 3 bullet points, under 50 words"
```

### Clean CSV data
```bash
head -100 messy.csv | gowork --role summarize -p "output only rows where all fields are non-empty, CSV format"
```

### Extract structured data
```bash
gowork --role summarize --file email.txt -p "extract: sender, date, subject. key: value format, 3 lines only"
```

### Batch file summaries
```bash
gowork --role summarize --cache --batch "src/**/*.go" -p "one-line summary: {}"
```

### Log analysis
```bash
grep "ERROR" app.log | head -50 | gowork --role summarize -p "group by error type, list counts"
```

### Heavy code analysis (use code role, 30B MoE)
```bash
gowork --role code --file complex_module.rs -p "find potential bugs and explain"
```

## Workflow: gowork + Claude

1. **grep/head** filters raw data
2. **gowork --role summarize** summarizes/cleans with the 4B model
3. **Claude** reasons about the concise result

Example: "what's wrong with my dataset?"
```bash
head -50 dataset.csv | gowork --role summarize -p "list column names, data types, obvious issues. under 100 words"
```
Claude reads the ~100 word summary instead of 10,000 lines of CSV.

## Error Handling

| 错误 | 原因 | 解决 |
|------|------|------|
| connection refused | MLX server 没启动 | 让用户执行 `mlx-start` |
| Empty output | prompt 太复杂 | 简化 prompt，拆成多步 |
| Garbled output | 输入太长 | 先 `head -n` 截断 |
| Timeout | 大文件处理慢 | 分块处理 |

## Token Savings Statistics

gowork has built-in token usage tracking. Every one-shot and batch call automatically records:
- Input/output tokens sent to the local model
- Estimated Claude tokens saved
- Cache hits
- Response time

### Viewing Stats

After using gowork, show the user their savings:
```bash
gowork --stats
```

Example output:
```
gowork Token Stats
─────────────────────────────────
Total calls:          2
Cache hits:           1
Local input tokens:   827
Local output tokens:  966
Claude tokens saved:  1.3K
Estimated cost saved: $0.0038
Avg response time:    12794ms
Total time:           25.6s
```

The "Claude tokens saved" is the key metric — it counts all input tokens that Claude would have consumed if it read the raw files directly.

### Resetting Stats
```bash
gowork --stats-reset
```

### When to Show Stats to the User

After completing a batch of preprocessing work, or when the user asks about token usage, run `gowork --stats` and share the results. This helps the user see the concrete value of the local model pipeline.

## Token Savings Reference

| Task | Without gowork | With gowork | Savings |
|------|---------------|-------------|---------|
| Summarize 500-line file | ~7000 tokens | ~500 tokens | **93%** |
| Extract functions (1957 lines) | 18550 tokens (超限!) | ~80 tokens | **99%** |
| Scan 10 files overview | ~10000 tokens | ~1000 tokens | **90%** |
| Batch 5 files summary | ~15000 tokens | ~500 tokens | **97%** |

---
> Source: [go7th/gowork-local-llm](https://github.com/go7th/gowork-local-llm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
