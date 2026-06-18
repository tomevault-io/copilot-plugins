## arxiv-cli

> This skill teaches AI agents (Copilot, Cursor) how to:

# arxiv-cli — Agent Skill File

**Skill:** `arxiv-cli`  
**Version:** 1.0.0  
**Platforms:** GitHub Copilot (VS Code), Cursor IDE  
**Install:** `pip install arxiv-cli` or `pip install git+https://github.com/codes1gn/arxiv-cli`

---

## What This Skill Teaches

This skill teaches AI agents (Copilot, Cursor) how to:
1. Search arXiv papers from the terminal
2. Retrieve paper details by ID
3. Download PDFs locally
4. Parse JSON output for further processing
5. Build research workflows combining multiple commands

---

## Activation

After installing, the CLI is available as the `arxiv` command.

**Verify installation:**
```bash
arxiv --version
# Output: arxiv-cli 1.0.0
```

---

## Command Reference

### 1. `arxiv search` — Search papers by query

```bash
# Basic search (human-readable output)
arxiv search "attention mechanism"

# Limit results
arxiv search "transformer language model" --limit 5

# JSON output (PREFERRED for agent use)
arxiv search "attention mechanism" --limit 10 --json

# Filter by author
arxiv search "language model" --author "Vaswani" --limit 5 --json

# Filter by category
arxiv search "reinforcement learning" --category cs.LG --limit 20 --json

# Show full abstracts
arxiv search "BERT" --abstract
```

**JSON output shape:**
```json
[
  {
    "id": "1706.03762",
    "title": "Attention Is All You Need",
    "authors": ["Ashish Vaswani", "Noam Shazeer", "Niki Parmar"],
    "published": "2017-06-12",
    "updated": "2023-08-02",
    "summary": "The dominant sequence transduction models are based on complex recurrent or convolutional neural networks...",
    "categories": ["cs.CL", "cs.LG"],
    "pdf_url": "https://arxiv.org/pdf/1706.03762",
    "abs_url": "https://arxiv.org/abs/1706.03762"
  }
]
```

---

### 2. `arxiv recent` — Recent papers in a category

```bash
# Last 7 days in cs.LG (default)
arxiv recent cs.LG

# Last 3 days, 10 results
arxiv recent cs.AI --days 3 --limit 10

# JSON output
arxiv recent cs.CV --days 7 --limit 20 --json
```

**Common categories:**
| Code | Field |
|------|-------|
| `cs.LG` | Machine Learning |
| `cs.AI` | Artificial Intelligence |
| `cs.CL` | Computation and Language (NLP) |
| `cs.CV` | Computer Vision |
| `cs.RO` | Robotics |
| `quant-ph` | Quantum Physics |
| `math.OC` | Optimization and Control |
| `stat.ML` | Statistics: Machine Learning |

---

### 3. `arxiv get` — Get paper by ID

```bash
# Get a specific paper (human output with abstract)
arxiv get 1706.03762

# JSON output (PREFERRED for agent use)
arxiv get 1706.03762 --json

# Also works with version suffix
arxiv get 2310.12567v2 --json
```

---

### 4. `arxiv download` — Download PDF

```bash
# Download to current directory
arxiv download 1706.03762

# Download to specific directory
arxiv download 1706.03762 --out ./papers/

# Example output:
# Downloading: Attention Is All You Need...
#   ID  : 1706.03762
#   Dest: ./papers/1706.03762.pdf
#   ✅  Saved to ./papers/1706.03762.pdf
```

---

## Agent Workflows

### Workflow 1: Literature Review Search

```bash
# Step 1: Search for papers (always use --json for agents)
arxiv search "diffusion models image generation" --limit 10 --json > /tmp/papers.json

# Step 2: Parse the JSON output — each entry has id, title, authors, summary, pdf_url
# Step 3: Select papers of interest by their id field
# Step 4: Get full details of a specific paper
arxiv get 2006.11239 --json

# Step 5: Download the most relevant paper
arxiv download 2006.11239 --out ./papers/
```

### Workflow 2: Track Recent Developments

```bash
# Monitor a field daily
arxiv recent cs.LG --days 1 --limit 50 --json

# Monitor multiple fields
arxiv recent cs.CL --days 7 --json
arxiv recent cs.CV --days 7 --json
```

### Workflow 3: Find Papers by Author

```bash
# Find recent papers by a specific author
arxiv search "deep learning" --author "Hinton" --limit 10 --json
arxiv search "" --author "LeCun" --limit 20 --json
```

### Workflow 4: Download Multiple Papers

```bash
# For each paper ID from a previous search:
arxiv download 1706.03762 --out ./papers/
arxiv download 1810.04805 --out ./papers/
arxiv download 2005.14165 --out ./papers/
```

---

## Agent Rules

> **CRITICAL RULES FOR AI AGENTS:**

1. **Always use `--json` flag** in agent workflows — never parse human-readable table output
2. **Parse the JSON array** — each element is a paper object with the schema above
3. **Check for empty results** — `[]` means no papers found, handle gracefully
4. **Use `id` field for follow-up** — the `id` field (e.g., `"1706.03762"`) works with `get` and `download`
5. **`pdf_url` field is direct download link** — agents can also use it with curl/wget if needed
6. **Non-zero exit code = error** — check stderr for error messages
7. **Rate limit**: arXiv requests ≤3 requests/second; the CLI handles this automatically — do NOT add sleep() between commands
8. **No API key required** — arXiv is fully open access

---

## Installation

### For Copilot (VS Code)

```bash
pip install arxiv-cli
# Or from source:
pip install git+https://github.com/codes1gn/arxiv-cli
```

Add to Copilot instructions (`.github/copilot-instructions.md`):
```markdown
## arxiv-cli

Use `arxiv search`, `arxiv recent`, `arxiv get`, `arxiv download` commands.
Always use `--json` flag for machine-readable output.
See SKILL.md in ~/.copilot/skills/arxiv-cli/ for full command reference.
```

### For Cursor IDE

```bash
pip install arxiv-cli
```

Add to `.cursor/rules/arxiv-cli.mdc`:
```markdown
---
description: Use arxiv-cli for searching and downloading arXiv papers
globs: ["**/*.py", "**/*.md"]
alwaysApply: false
---

Use the `arxiv` CLI tool for paper search. Always pass `--json` for structured output.
Commands: `arxiv search`, `arxiv recent`, `arxiv get`, `arxiv download`
Full reference: SKILL.md in ~/.copilot/skills/arxiv-cli/
```

---

## Error Handling

| Error | Cause | Fix |
|-------|-------|-----|
| `command not found: arxiv` | Not installed or not in PATH | `pip install arxiv-cli` |
| `Error querying arXiv: ...` | Network/API error | Check internet connection; retry |
| `Paper not found: <id>` | Invalid arXiv ID | Verify the ID format (e.g., `1706.03762`) |
| `Download failed: ...` | PDF unavailable or network error | Try again or check `abs_url` |
| `[]` (empty JSON) | No results matching query | Broaden the search query |

---

## Memory

arxiv-cli logs searches and downloads to:
```
~/.copilot/skills/arxiv-cli/data/user-memory.jsonl
```

Memory entry format:
```json
{"timestamp": "2025-05-29T12:00:00Z", "type": "search", "query": "attention mechanism", "result_count": 10}
{"timestamp": "2025-05-29T12:01:00Z", "type": "download", "paper_id": "1706.03762", "title": "Attention Is All You Need", "out_path": "./papers/1706.03762.pdf"}
```

---

## Scenario Coverage

| # | Scenario | What It Tests |
|---|----------|--------------|
| 01 | `search-basic` | Basic search returns results |
| 02 | `search-json` | `--json` flag returns valid JSON array |
| 03 | `search-author` | `--author` filter works |
| 04 | `search-category` | `--category` filter works |
| 05 | `search-limit` | `--limit N` respected |
| 06 | `recent-category` | `arxiv recent cs.LG` returns results |
| 07 | `recent-json` | recent command `--json` output |
| 08 | `get-by-id` | `arxiv get <id>` returns correct paper |
| 09 | `get-json` | get command `--json` output schema |
| 10 | `download-pdf` | PDF download to directory |
| 11 | `agent-workflow` | Multi-step search→get→download workflow |
| 12 | `skill-commands` | SKILL.md documents all 4 commands |
| 13 | `skill-json-rules` | SKILL.md has agent rules section |

---
> Source: [codes1gn/arxiv-cli](https://github.com/codes1gn/arxiv-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
