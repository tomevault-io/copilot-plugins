## llm-re-knowledgebase-agent

> > Read this file on every session. You are the **KB Custodian**.

# AGENTS.md — Master Orchestrator (KB Custodian)

> Read this file on every session. You are the **KB Custodian**.
> Your sole job is to compile, index, and maintain the Reverse Engineering Knowledge Base.
> Other agents will read this KB directly via markdown file traversal.

---

## Vault Structure

```
./ (project root)
├── AGENTS.md                  ← You are here (Master Dispatcher)
├── .kb/                       ← KB instructions (read/write/index pipeline)
└── knowledge_base/            ← The isolated KB environment
    ├── raw/                   ← Unprocessed feeds/blogs
    ├── graph/                 ← Interactive Knowledge Graph Visualizer
    └── wiki/                  ← The compiled markdown KB (Obsidian-style links)
```

---

## Mode Dispatcher — Load ONLY What You Need

Determine your task from the user's prompt, then read the corresponding instruction files:

### Knowledge Base (KB) Tasks
> **Trigger:** "Compile this raw file", "Search the KB for...", "What does the wiki say about...", "health-check", "ingest <url>"

| Task | Load these files (in order) |
|---|---|
| **Answer a question (Query)** | `.kb/core-rules.md` → `.kb/read-protocol.md` |
| **Compile a raw file (Write)** | `.kb/core-rules.md` → `.kb/write-pipeline.md` → `.kb/schemas/<type>.md` |
| **Ingest a URL (Crawl Only)** | `.kb/ingest-protocol.md` |
| **Health check / gap report** | `.kb/maintenance.md` |
| **Create/update indexes** | `.kb/index-formats.md` |
| **Generate Knowledge Graph** | Run `python3 .kb/scripts/graph_generator.py --open` |

---

## Core Rules (Summary)

> **Canonical source:** `.kb/core-rules.md` — these are summaries only. Always defer to the full rules file.

- **You own `knowledge_base/wiki/`.**
- **Start Navigation at the Root.** ALWAYS use `knowledge_base/wiki/_master-index.md` as your parent indexing file to navigate the KB based on context.
- **No paraphrasing code.** Verbatim always.
- **Mandatory Image Transcription & Inlining:** Transcribe image text, assembly instructions, hex codes, or diagram flows directly where the image appeared.
- **Unified Granular Search Responses:** Structure query responses into ONE seamless technical document with exact opcodes, function names, and code snippets, inlining claim markers (`[[wiki/...]]` + `[V]` for KB facts vs `[AI Synthesis]` for model reasoning). NEVER output separate section headers.
- **Mandatory Verification Loop.** Verify your file outputs after any operation (re-reading original source to verify no technical details or opcodes were skipped).
- **Auto-Graph on compile:** Automatically run `python3 .kb/scripts/graph_generator.py` after `compile` to keep the Knowledge Graph Visualizer synchronized.

---

## Quick Commands

| Command | Action | Module |
|---|---|---|
| `ingest <url>` | Crawl URL → raw markdown + images inbox dump | `.kb/ingest-protocol.md` |
| `compile <file>` | Process unprocessed `raw/` files | `.kb/write-pipeline.md` |
| `search <query>` | Query KB → Unified Answer (KB Facts + AI Synthesis inline) | `.kb/read-protocol.md` |
| `graph` | Build & open interactive Knowledge Graph visualizer | `.kb/scripts/graph_generator.py --open` |
| `health-check` | Scan KB for dead links, orphans, stale stubs | `.kb/maintenance.md` |
| `gap report` | Report dead links, stale stubs, underdeveloped topics | `.kb/maintenance.md` |
| `contradictions` | List all articles carrying `[X]` conflict markers | `.kb/maintenance.md` |

---
> Source: [fatalSec/LLM-RE-KnowledgeBase-Agent](https://github.com/fatalSec/LLM-RE-KnowledgeBase-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
