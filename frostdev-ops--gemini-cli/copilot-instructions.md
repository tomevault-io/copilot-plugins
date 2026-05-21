## 04-memory

> Use this rule to locate the persistent memory storage code (MemoryStore), understand the LanceDB backend and embedding generation via MCP, see how prompts are enhanced with context, and find semantic search/filtering logic.

# Gemini Rust Suite: Persistent Memory and Embeddings

Persistent, semantic memory is managed by the `gemini-memory` crate ([memory/README.md](mdc:memory/README.md)).

## Key Components:

*   **`MemoryStore` ([memory/src/store.rs](mdc:memory/src/store.rs))**: Main struct for interacting with the memory database.
*   **`Memory` Struct ([memory/src/memory.rs](mdc:memory/src/memory.rs))**: Represents a single memory item (key, value, timestamp, tags, metadata, vector embedding).
*   **LanceDB Integration:**
    *   Uses LanceDB ([https://lancedb.com/](mdc:https:/lancedb.com)) as the vector database for efficient storage and semantic search.
    *   Database typically located at `~/.local/share/gemini-suite/memory.db` ([memory/src/config.rs](mdc:memory/src/config.rs)).
    *   Schema defined using Apache Arrow ([memory/src/schema.rs](mdc:memory/src/schema.rs)), including a vector column.
    *   CRUD operations and vector search are implemented using the LanceDB Rust SDK.
*   **Embeddings via MCP:**
    *   Generating vector embeddings (numerical representations of text meaning) is required for semantic search.
    *   This is **delegated** to an external MCP server providing an `embed` tool.
    *   The `MemoryStore` requires an implementation of the `McpHostInterface` trait ([memory/src/broker.rs](mdc:memory/src/broker.rs)) to call this external tool.
    *   The primary embedding server used is the Python E5 server ([mcp_embedding_server/README.md](mdc:mcp_embedding_server/README.md)).
*   **Semantic Search:**
    *   The `get_semantically_similar` method ([memory/src/store.rs](mdc:memory/src/store.rs)) takes query text, calls the MCP `embed` tool to get the query vector, and then performs a vector similarity search in LanceDB.
    *   More advanced search combining semantic, keyword, tag, and time filters is available via `search_memories` ([memory/src/store.rs](mdc:memory/src/store.rs)).
*   **Prompt Enhancement:**
    *   The `enhance_prompt` function ([memory/src/broker.rs](mdc:memory/src/broker.rs)) automatically retrieves relevant memories based on a user prompt (using semantic search) and prepends them as context.
    *   Configurable via the `enable_memory_broker` flag/setting.
*   **Auto Memory:**
    *   Automatically extracts and saves key information from conversations.
    *   Configurable via the `enable_auto_memory` flag/setting. Requires analysis logic (potentially involving LLM calls) and uses the `add_memory` function.

---
> Source: [frostdev-ops/gemini-cli](https://github.com/frostdev-ops/gemini-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
