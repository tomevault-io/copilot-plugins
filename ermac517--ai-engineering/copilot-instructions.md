## ai-engineering

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A personal learning repo for an AI Engineering course, organized as Jupyter notebooks in three parts
(Part 1 = LLM/RAG/Gradio fundamentals, Part 2 = agents, Part 3 = MCP). There is no package, no test
suite, no linter, and no build step — the deliverable of each lesson is a runnable notebook.
Commits follow `Week N Day M - Topic`.

## Environment

Two virtualenvs exist at the repo root (both gitignored):

- `ai_env` — **Python 3.14.3, the current default.** Jupyter kernel display name `ai_env (3.14.3.final.0)`.
- `ai_env_3_12` — Python 3.12.10, kept because three notebooks still pin a 3.12 kernel:
  `rag.ipynb`, `digital-twin.ipynb`, `digital-twin-arch4-full-RAG.ipynb`.

```bash
source ai_env/bin/activate
jupyter lab                 # or: jupyter notebook
```

Both envs already have `openai`, `openai-agents`, `mcp`, `chromadb`, `gradio`, `litellm`, `ddgs`,
`trafilatura`, `plotly`, `scikit-learn`. There is no root `requirements.txt`; notebooks install
extras inline (`!pip install ddgs trafilatura -q`).

Secrets live in a root `.env` (gitignored): `OPENAI_API_KEY`, `PUSHOVER_USER`, `PUSHOVER_TOKEN`.
Every notebook's first cell calls `load_dotenv()` and raises if `OPENAI_API_KEY` is missing.

Part 3 needs external binaries on PATH: `npx`/`node` (for `@modelcontextprotocol/server-filesystem`)
and `uvx` (for `mcp-server-fetch`).

## Notebook conventions

Cells are grouped under `### Step N:` markdown headings. Step 1 is always imports + `load_dotenv()` +
`client = OpenAI(api_key=OPENAI_API_KEY)`, and most notebooks define `MODEL = "gpt-4.1-mini"`
(evals use `JUDGE_MODEL = "gpt-4.1"`). Follow this shape when adding a lesson.

## Two parallel agent stacks

The repo deliberately builds the same capabilities twice — once by hand, once with the SDK. Know
which one a notebook is in before editing.

**Hand-rolled tool calling** (Part 1 `tool-calling.ipynb`, `digital-twin*`, Part 2 `research-agent.ipynb`,
`research-agent-evals.ipynb`, and `digital-twin/app.py`):

- `tools = []`, then one JSON-schema dict per function appended as `{"type": "function", "function": {...}}`.
- `handle_tool_call(tool_calls)` dispatches on `function_name` via if/elif and returns a list of
  `{"role": "tool", "content": ..., "tool_call_id": ...}` messages.
- The agentic loop is `while message.tool_calls:` — append the assistant message, extend with tool
  results, re-call the API — always with an iteration/message cap to prevent runaway loops.

**OpenAI Agents SDK** (Part 2 `research-agent-sdk.ipynb`, `article-writer.ipynb`, all of Part 3):
`from agents import Agent, Runner, function_tool, handoff, trace`, `await Runner.run(agent, input=..., max_turns=N)`.
`article-writer.ipynb` is the orchestrator/handoff example (research + image agents as tools, three
writer personas as handoff targets).

## RAG pipeline (Part 1)

`rag.ipynb` and the `digital-twin` line share one pipeline: `split_text_into_chunks()` (character
chunking that walks the cut backward to a paragraph → newline → sentence → space boundary, only past
the halfway point) → `client.embeddings.create(model="text-embedding-3-small")` → ChromaDB. The
collection is always emptied before re-adding (`collection.delete(collection.get()["ids"])`) so cells
are idempotent. Retrieval concatenates the top-3 documents and appends them to the system message as
`"\n\nContext:\n" + context` rather than to the user turn.

Persistent Chroma dirs (`chroma_db/`, `chroma_db3/`, `chroma_db_twin/`) are gitignored generated
artifacts — rebuild them by re-running the notebook, don't treat them as source.

## The digital twin progression

`digital-twin-arch1` → `arch4` are progressive versions of the same app (dynamic context injection →
basic tool calling → proper tool calling → full RAG); `digital-twin.ipynb` is the consolidated final
version. When asked to change "the digital twin", edit the consolidated notebook and
`digital-twin/app.py`, not the numbered archX snapshots — those are frozen teaching steps.

`digital-twin/` is the Hugging Face Space deployment bundle (`app.py`, `requirements.txt`,
`mcruz.jpeg` avatar). `app.py` is a flattened copy of the notebook with two deliberate differences:
it uses `os.getenv` without `load_dotenv` (Space secrets) and an **in-memory** `chromadb.Client()`
instead of `PersistentClient`, so the index is rebuilt on every Space boot. Keep the two in sync
when changing prompts, tools, or documents. `my-first-space/` is the trivial Gradio Space used to
learn deployment.

## MCP notebooks (Part 3)

Servers are launched as `async with MCPServerStdio(name=..., params=..., client_session_timeout_seconds=60) as server:`
and passed to `Agent(..., mcp_servers=[...])`; the agent must be constructed *inside* the context
manager. `SANDBOX_DIR = os.path.abspath("secret_project_007")` — the filesystem server needs an
absolute allowed directory, and the sandbox dirs (`secret_project_006/`, `secret_project_007/`) are
gitignored and created plus seeded with sample files by the notebook itself.

`mcp-filesystem.ipynb` keeps a running `# Errors to debug:` comment block above successive versions of
`SUMMARIZER_AGENT_PROMPT`, each version fixing one observed agent failure. Preserve that history when
editing the prompt — the iteration is the lesson.

## Evals

`research-agent-evals.ipynb` is the LLM-as-judge pattern: a binary TRUE/FALSE judge prompt scored with
`temperature=0, max_tokens=1`, run across a fixed `TOPICS` list, with agent stdout suppressed via
`contextlib.redirect_stdout`. "Evals Run 1" and "Evals Run 2" differ only in the agent's system
prompt — the notebook exists to compare failure rates between prompt revisions, so keep both runs.

---
> Source: [Ermac517/ai-engineering](https://github.com/Ermac517/ai-engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
