## enhanced-mem-vector-rag

> This project, "Enhanced Mem-Vector RAG," aims to build a sophisticated, **personal AI Agent system** focused on **advanced memory/knowledge management**, **software development assistance** (specifically targeting AI/ML, Python, AWS, Next.js), **research**, **learning**, and managing personal/work information.

# Enhanced Mem-Vector RAG - Claude Code Development Guide (v4 - Final Comprehensive Architecture)

## 🧠 Project Overview & Goal

This project, "Enhanced Mem-Vector RAG," aims to build a sophisticated, **personal AI Agent system** focused on **advanced memory/knowledge management**, **software development assistance** (specifically targeting AI/ML, Python, AWS, Next.js), **research**, **learning**, and managing personal/work information.

**Core Use Case:** Create a persistent, shared memory and knowledge fabric accessible by both the user and multiple AI agents. The system will assist with software development, research report generation, learning, and managing personal/work information, primarily interacting via **Claude Code** and other tools leveraging the **Model Context Protocol (MCP)**.

## 🏛️ Core Technologies & Final Stack Choices

- **Development Environment:** Claude Code with integrated MCP Servers.
- **Agent Orchestration:** LangGraph (Supervisor-Worker pattern).
- **Agent Framework:** LangChain (Agent logic, planning, tool integration).
- **Data Indexing/Connection & RAG Pipeline Framework:** **LlamaIndex** (Central framework for connecting LLMs to data sources, building RAG pipelines, indexing data into vector/graph stores, and orchestrating complex queries across them).
- **Long-Term Memory (LTM - Semantic/Episodic):**
  - **Vector Store:** Qdrant (High-performance, filtering, hybrid search).
  - **Memory Management Interface:** Mem0 (Open Source, self-hosted; intelligent interface primarily for vector memory, adds scoring, personalization. Works alongside LlamaIndex).
- **Procedural Memory (Workflows/Tasks):**
  - **Graph Database:** Neo4j.
  - **Interface/Framework:** Graphiti (Runs _on_ Neo4j for temporally-aware knowledge graphs). LlamaIndex `Neo4jGraphStore` can also be used for indexing/querying graph data.
- **Structured Data/Metadata:** Supabase (PostgreSQL) (User profiles, project metadata, relational data).
- **Knowledge Representation:** OWL (Ontologies managed in Git), RDF (Semantic annotations via `n10s` in Neo4j).
- **Embedding Generation:** Locally via models like OpenAI `text-embedding-3-small` / Cohere `embed-english-v3`.
- **Embedding Tooling:** FastEmbed (For efficient embedding during ingestion).
- **Retrieval Strategy:** Hybrid RAG (Qdrant vector + BM25 keyword) + KAG (Neo4j/Graphiti graph context) + Tavily Web Search fusion. **LlamaIndex orchestrates retrieval.**
- **Ingestion Pipeline:** LlamaIndex data loaders, Crawl4AI, Firecrawl, Scrapy, Playwright, Official APIs (Reddit, X, etc.), GitHub Webhooks, IMAP. Originals stored in AWS S3 / Supabase Storage.
- **MCP Server Framework:** **FastMCP (gofastmcp.com)** - The framework used to build the custom Python MCP server for this project.
- **UI/Interaction (Initial):** Primarily via MCP-enabled tools (Claude Code, etc.). Chainlit is a secondary option for direct interaction later, leveraging LlamaIndex integration.
- **Deployment (Initial):** Local execution. MCP server via `stdio`. Components (Qdrant, Neo4j, Supabase) via Docker.
- **Observability:** Grafana + Prometheus stack.
- **Versioning:** Git (Code/Ontology), Neo4j CDC (Graph Data).
- **Rules/Constraints:** Neo4j APOC triggers/constraints.
- **Package Management:** `uv` for Python.
- **Security:** Env Vars/Secrets Mgmt, OAuth 2.1/PAT for MCP, RBAC mapping.

_(Alternatives Considered & Rejected/Deferred: Zep, Memonto, A-MEM, Letta, Memoripy (for LTM); Memgraph (for Graph DB); Cache-Augmented Generation (CAG); External Rule Engines like Jena/RDFox)_.

## 🧰 Available MCP Servers (for Claude Code)

Claude Code has access to the following _external_ MCP servers:

- `arxiv-mcp-server`
- `browser`
- `context7`
- `desktop-commander`
- `firecrawl`
- `git`
- `github`
- `playwright`
- `repomix`
- `sequential-thinking`
- `supabase`
- `tavily`
- `time`
- **`memory`**: This is the **custom MCP server we are building** using the **FastMCP framework**. Its functions (`/search.hybrid`, `/graph.query`, `/memory.*`, `/rules.validate`, `/ingest.*`) interact with _our_ backend components (Qdrant/Mem0, Neo4j/Graphiti, Supabase, LlamaIndex-powered retrieval, Ingestion triggers).

## 🧭 Development Workflow & Memory Protocol

**🔄 Session Initialization (CRITICAL):**
_ALWAYS_ start every development session by refreshing your understanding of the project state:

1.  `memory.read_graph()` - Load the project's knowledge graph state from _our_ custom `memory` MCP server.
2.  `memory.search_nodes("Implementation Progress")` - Find the latest progress summary node.
3.  `memory.search_nodes("Recent Decisions")` - Review key architectural or implementation choice nodes.
4.  `git.git_status({"repo_path": "./"})` - Check the local Git working tree status.

**🧠 During Development (Continuous Memory Updates via _our_ `memory` MCP):**

- **Record Implementations:** As you write code (e.g., agent logic using LangChain, data loading using LlamaIndex, MCP endpoint using FastMCP):
  `memory.add_observations({"entityName": "ComponentName", "contents": ["Implemented LlamaIndex retriever for Qdrant", "Added FastMCP endpoint logic", "Wrote test for Graphiti query"]})`
- **Document Decisions:** When a choice is made (e.g., LlamaIndex query engine, Mem0 configuration):
  `memory.create_entities([{"name": "Decision - LlamaIndex Query Engine", "entityType": "Decision", "observations": ["Using KnowledgeGraphQueryEngine for Neo4j interaction via LlamaIndex."]}])`
  `memory.create_relations([{"from": "KAGRetriever", "relationType": "uses_decision", "to": "Decision - LlamaIndex Query Engine"}])`
- **Track Progress:** Update or create a daily progress entity:
  `memory.add_observations({"entityName": "Implementation Progress - YYYY-MM-DD", "contents": ["Integrated LlamaIndex Neo4jGraphStore", "Refactored Mem0 interface"]})` (Use `time.get_current_time()` for date). _Update today's entity if it exists._
- **Link Components:** Define relationships (dependencies, data flow, framework usage):
  `memory.create_relations([{"from": "IngestionPipeline", "relationType": "uses_framework", "to": "LlamaIndex"}])`
  `memory.create_relations([{"from": "HybridRAGRetriever", "relationType": "uses_framework", "to": "LlamaIndex"}])`
  `memory.create_relations([{"from": "HybridRAGRetriever", "relationType": "uses_interface", "to": "Mem0"}])`

**🧹 Memory Hygiene:**

- **Consolidate:** Merge related observations.
- **Update, Don't Duplicate:** Modify existing entities (especially progress).
- **Prune (Carefully):** Use `memory.delete_entities` only for confirmed obsolete items.

**⚠️ On Context Reset/Error:** _Immediately_ re-run the Session Initialization steps.

## 💻 Coding Standards & Practices

- **Language:** Python 3.11+
- **Package Manager:** `uv` (`uv pip install ...`, `uv run ...`).
- **Formatting/Linting:** Black, Ruff (configure in `pyproject.toml`). Enforce via pre-commit hooks.
- **Type Hinting:** Mandatory (`typing` module).
- **Docstrings:** Google-style (for all modules, classes, functions).
- **Error Handling:** Robust `try/except`, custom exceptions, structured logging.
- **Modularity & Design:** SOLID principles. Design for testability and reusability.
- **Asynchronicity:** `asyncio` (`async`/`await`) for I/O (APIs, DBs, scraping).
- **Configuration:** `.env` files (`python-dotenv`) for secrets/config. **NO SECRETS IN GIT.**
- **Logging:** Standard `logging` module configured for structured JSON output.
- **File Size:** Aim for < 400 LoC per file.
- **Naming:** Lowercase hyphenated dirs/files (`ingestion-connectors/`); `snake_case` vars/funcs; `PascalCase` classes.

## 🐙 Git Workflow

- **Branching:** `feat/`, `fix/`, `docs/`, `chore/` prefixes. Use `git.git_create_branch`, `git.git_checkout`.
- **Commits:** Conventional Commits (`<type>(<scope>): <description>`). Use `git.git_add`, `git.git_commit`.
  - Scopes: `agent`, `ingestion`, `memory`, `graph`, `vector`, `config`, `ci`, `docs`, `mcp`, `ui`, `llama_index`, `fastembed`, `mem0`, `graphiti`, `fastmcp`, `chainlit`, `qdrant`, `neo4j`, `supabase`, `langchain`, `langgraph`, etc.
- **Pull Requests (GitHub):** Target `main`. Conventional Commit titles. Use `github.create_pull_request`.

## 🛠️ MCP Tool & Library Usage Guidelines

- **`memory` (Our Custom FastMCP Server):** **Central Hub.** Use constantly for state, progress, decisions. Its functions map to _our_ backend (Qdrant/Mem0, Neo4j/Graphiti, Supabase, LlamaIndex-powered retrieval, Ingestion triggers).
- **`context7`:** **Mandatory** before using external libraries/frameworks (LangChain, LangGraph, Qdrant client, Neo4j driver, **LlamaIndex**, FastEmbed, Mem0 SDK, Graphiti client, Chainlit, FastMCP, etc.) to get up-to-date docs and usage patterns.
- **`sequential-thinking`:** For complex logic design, algorithm planning, debugging _before_ coding.
- **`tavily`:** Quick web searches during development or for the RAG pipeline.
- **`firecrawl`:** Deeper web research, structured scraping for ingestion.
- **`git` / `github`:** Local vs. Remote repo interactions.
- **`browser` / `playwright`:** Simple vs. Advanced browser automation (likely for ingestion tasks).
- **`repomix`:** Understanding code structure (ours or external).
- **`supabase`:** Managing the PostgreSQL schema/data.
- **`arxiv-mcp-server`:** Specific arXiv interaction.
- **LlamaIndex / FastEmbed / Mem0 SDK / Graphiti Client / Chainlit / FastMCP:** These are **Python libraries/frameworks**, not external MCP servers (unless we build wrappers later). Use them directly in the Python code after checking docs with `context7`. Import and call their functions/classes as needed within agents, ingestion scripts, the MCP server implementation, or a potential Chainlit UI. **LlamaIndex is key for both ingestion (loaders, indexing) and retrieval (query engines, RAG modules). FastMCP is the framework for building our `memory` server.**

## 🏛️ Key Architectural Roles Clarification

- **LangChain:** Provides the _building blocks_ for agents (LLM wrappers, prompts, parsers, basic memory, tools, planning).
- **LangGraph:** _Orchestrates_ complex agent workflows using LangChain components (cycles, state, multi-agent - Supervisor/Specialist/Critic roles). Uses Plan-Execute/ReAct/Graphiti planners. Includes reflection loops.
- **LlamaIndex:** The primary framework for **building and orchestrating the RAG pipeline**. It connects data sources (files, APIs, DBs like Qdrant/Neo4j via `Neo4jGraphStore`), handles indexing (into Qdrant, Neo4j), provides various query engines (vector, graph, hybrid), and integrates with LLMs for generation. Used heavily in both ingestion and retrieval logic within agents.
- **Mem0:** An intelligent _interface_ primarily for the LTM vector store (Qdrant), adding scoring/personalization. LlamaIndex can work _with_ Mem0, potentially using Mem0 as a specialized vector retriever within a larger LlamaIndex pipeline.
- **Graphiti:** Provides the _framework and query capabilities_ for procedural/temporal knowledge stored _in_ Neo4j. LlamaIndex can query this graph data (potentially via Graphiti's API or directly using Cypher via `Neo4jGraphStore`).
- **FastMCP:** The _framework_ used to build our custom `memory` MCP server, defining the API contract (`/search.hybrid`, `/graph.query`, `/memory.*`, `/rules.validate`, `/ingest.*`) for external tools (like Claude Code) to interact with our system's backend components (which internally use LlamaIndex, Mem0, Graphiti, etc.).
- **Chainlit:** A potential **UI framework** for direct user interaction, integrating well with LlamaIndex for displaying RAG pipeline results and intermediate steps.
- **Qdrant:** Stores vector embeddings for semantic search (LTM). Supports hybrid search.
- **Neo4j:** Stores graph data for procedural memory and structured knowledge representation (OWL/RDF via `n10s`). Queried via Graphiti/LlamaIndex/Cypher. Uses APOC for rules.
- **Supabase (PostgreSQL):** Stores relational metadata, user profiles, potentially configuration or agent state. Also used for storing original ingested documents (alongside S3).
- **FastEmbed:** Used during the ingestion pipeline for efficient generation of vector embeddings before storage in Qdrant.

## 🚀 Essential Combined Workflows (Examples)

**1. Implement LlamaIndex Retriever for Hybrid Search:**
`memory.read_graph()` -> `context7.resolve_library_id({"libraryName": "llama-index"})` -> `context7.get_library_docs(...)` (Focus on Retrievers, Query Engines, Qdrant integration) -> `context7.resolve_library_id({"libraryName": "qdrant-client"})` -> `context7.get_library_docs(...)` -> `sequential-thinking.plan_feature({"feature": "LlamaIndex Hybrid Retriever for Qdrant+BM25"})` -> Write Python code (`retrievers/hybrid_retriever.py`) using `llama_index.core` and Qdrant client -> Write tests -> `uv run tests/...` -> `git.git_add(...)` -> `git.git_commit({"message": "feat(llama_index): implement hybrid retriever"})` -> `memory.add_observations({"entityName": "HybridRAGRetriever", "contents": ["Implemented using LlamaIndex query fusion"]})` -> `github.create_pull_request(...)`

**2. Agent Uses LlamaIndex for KAG Query:**
`memory.read_graph()` -> Agent logic (LangChain) needs graph info -> Agent uses LlamaIndex `KnowledgeGraphQueryEngine` (configured with `Neo4jGraphStore`) -> LlamaIndex translates natural language/query to Cypher (or uses predefined queries) -> LlamaIndex queries Neo4j -> Neo4j returns results -> LlamaIndex processes results -> Agent receives structured info. (Record relevant observations in memory).

**3. Setup Chainlit UI (If/When Implemented):**
`memory.read_graph()` -> `context7.resolve_library_id({"libraryName": "chainlit"})` -> `context7.get_library_docs(...)` -> `context7.resolve_library_id({"libraryName": "llama-index"})` -> `context7.get_library_docs(...)` (Focus on Chainlit callbacks) -> `sequential-thinking.plan_feature({"feature": "Basic Chainlit UI for RAG interaction"})` -> Write Chainlit app code (`ui/app.py`) integrating LlamaIndex query engine and callback handler -> Test locally -> `git.git_add(...)` -> `git.git_commit({"message": "feat(ui): add initial Chainlit app with LlamaIndex integration"})` -> `memory.add_observations(...)` -> `github.create_pull_request(...)`

**4. Implement FastMCP Endpoint (`/ingest.url`):**
`memory.read_graph()` -> `context7.resolve_library_id({"libraryName": "fastmcp"})` -> `context7.get_library_docs(...)` -> `context7.resolve_library_id({"libraryName": "llama-index"})` -> `context7.get_library_docs(...)` (Relevant ingestion/loader parts) -> `context7.resolve_library_id({"libraryName": "fastembed"})` -> `context7.get_library_docs(...)` -> `sequential-thinking.plan_feature({"feature": "FastMCP /ingest.url endpoint"})` -> Write Python code in FastMCP server project (`mcp_server/endpoints/ingest.py`) defining the endpoint, calling appropriate LlamaIndex loaders/pipeline, using FastEmbed, and storing results in Qdrant/Neo4j/Supabase -> Write tests -> `uv run tests/...` -> `git.git_add(...)` -> `git.git_commit({"message": "feat(fastmcp): implement /ingest.url endpoint"})` -> `memory.add_observations(...)` -> `github.create_pull_request(...)`

## 🧑‍💻 Development Context & Preferences (User Profile)

- **Goal:** Build a personal SOTA AI agent system while learning; aiming for AI/ML Engineer roles (Anthropic, OpenAI, etc.).
- **Expertise Areas (Target for System):** AI/ML, Python, AWS, Next.js, TailwindCSS, Shadcn-UI, LLMs, Quantization, Fine-Tuning, NLP, AI Agents, MCP.
- **Current Projects:** This system (`enhanced-mem-vector-rag`), `ai-powered-development-prompts` repo, deploying FastAPI/Next.js app to AWS.
- **Preferences:** Follow OpenAI Prompting Guide principles, value visual aids/diagrams/clear docs, SOLID principles, Conventional Commits, modularity, cost-consciousness, local-first dev.

---

**‼️ IMPORTANT REMINDERS:**

1.  **MEMORY IS KEY:** Start with `memory.read_graph()`. Update `memory` constantly.
2.  **PLAN -> DOCS -> CODE:** Use `sequential-thinking` and `context7` _before_ implementation.
3.  **TEST:** Write tests for all logic.
4.  **COMMIT:** Conventional Commits, frequently.
5.  **STANDARDS:** Follow PEP 8, typing, SOLID, etc.
6.  **MCP vs. LIBS:** Use MCP servers for external tools/our API. Use Python libraries (LlamaIndex, Mem0 SDK, FastEmbed, Graphiti client, Chainlit, FastMCP) directly in code. **LlamaIndex is central to data handling. FastMCP builds our `memory` server.**
7.  **Local First:** Initial implementation focuses on local execution (`stdio` MCP, Dockerized backends). Cloud deployment (Cloudflare, AWS) is a future step.

This guide reflects the final, detailed architecture including all clarifications. Use it rigorously.

---
> Source: [BjornMelin/enhanced-mem-vector-rag](https://github.com/BjornMelin/enhanced-mem-vector-rag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
