## ncs-gpu

> ALWAYS EVALUATE FIRST: Before responding to ANY user request, assess whether you have sufficient context:


---
description: 
globs: 
alwaysApply: true
---
# Memory Tool Usage Guide

## ⚠️ CRITICAL: MEMORY TOOLS ARE MANDATORY ⚠️

ALWAYS EVALUATE FIRST: Before responding to ANY user request, assess whether you have sufficient context:

1. **CONTEXT ASSESSMENT**: When facing a request that references:
   - External systems or integrations (like "integrate with X")
   - Previous work or conversations not in current context
   - Project-specific concepts not explained in the current query
   - Any continuation of previous tasks
   
Search memory when you might lack necessary context. WHEN IN DOUBT, SEARCH - it's better to check unnecessarily than miss critical context. Only skip searching when the query is completely self-contained.

KEY TRIGGER PHRASES requiring immediate `search_keys` or `search_memory`:
   - "we need to integrate with..."
   - References to previous work ("I added X, now we need to...")
   - Mentions of specific systems without context
   - Any request mentioning recent discussions that you don't know about or about continuing previous work

2. **LAST ACTION**: Store memory of significant interactions using `store_memory`. NEVER store the full request-response; ONLY summaries, highlights and importnant pieces of information.

## Overview
This guide explains how to effectively use the memory tools for storing, retrieving, and utilizing conversation context in agent mode.

### Execution Directives (delta)
- Inherits from `rules/global-rules.md` (Instruction hierarchy & Global execution directives); this section only lists deltas specific to Memory Tools.
- Delta specifics:
  - Search when context might be missing; stop early once you can act (early stop).
  - Prefer update over create to avoid duplicates; check existing memories first.
  - Never store full conversations or secrets/PII; store concise summaries with key facts only.
  - Include `Title`, `Tags`, `CorpusNames`, and clear `Content`; set `UserTriggered=true` only if the user explicitly asked you to store.
  - Cite evidence (`file:line`) inside stored content when applicable.
  - Respect tool budget ≤ 2 for search/store per small task; if exceeded, briefly report rationale.

### Tool Mapping (this repo)
- Search memories: `mcp2_search_nodes` (semantic search in the knowledge graph).
- Open specific memories/nodes: `mcp2_open_nodes`.
- Read entire graph: `mcp2_read_graph`.
- Create/Update/Delete memory: `create_memory` (Action: `create` | `update` | `delete`; requires exact `CorpusNames`, `Title`, `Content`; optional `Tags`, `Id`; `UserTriggered` flag).
- Advanced graph ops: `mcp2_add_observations`, `mcp2_create_entities`, `mcp2_create_relations`, `mcp2_delete_*` when needed.

## Memory Operations

### 1. SEARCH WHEN CONTEXT MIGHT BE MISSING
Search memory when there's any indication you might need additional context:
- Use the `search_keys` or `search_memory` tool when you have uncertainty
- Search if there's any chance the user is referencing previous work, discussions, or context
- Search when the user refers to previous work, decisions, or information not present in the current conversation
- Construct a query related to the specific missing context
- Analyze the returned results for relevant context
- Remember: It's better to search unnecessarily than miss critical context

### 2. STORE AFTER MEANINGFUL RESPONSES
After assistant responses that contain NEW information or decisions:
- Use the `store_memory` tool directly
- Check existing memories first to avoid redundancy
- Only store when new facts, decisions, or context emerges
- Skip storing if your response merely reiterates previously stored information
- Follow the naming conventions below

## IMPORTANT Memory Content Guidelines

### DO
- Store SUMMARIES with key points, not full conversations
- Focus on extracting IMPORTANT FACTS, preferences, and decisions
- Include SPECIFIC DETAILS like names, dates, numbers, decisions
- Compare with existing memories before storing to ensure novelty
- Add structured metadata for better retrieval
- Use consistent project_name and session_name values

### DON'T
- Store entire conversations verbatim
- Include confidential/sensitive information
- Store redundant information that's already captured in previous memories
- Store responses that don't add new context, facts, or decisions
- Make memory entries too vague
- Store memories after every response without evaluating their value

## IMPORTANT Naming Conventions

### Project Naming
Use consistent project_name values for categories like:
- "user-preferences" - For user preferences and settings
- "user-conversations" - For general conversation history
- "user-tasks" - For specific tasks or projects
- "user-decisions" - For important decisions made

### Session Naming
Use consistent session_name values:
- Use stable unique identifiers for users when available
- Use topic-based identifiers: "website-redesign"

### Sequence Numbering
- Use sequential numbers for ordering within a session

## Search Strategies

### Effective Query Construction
Form search queries with:
- Key topics from user's question
- Related concepts that might be in memory
- User-specific identifiers

### Semantic Key Search
Use `search_keys` when you need to find related memory keys:
- Provide a semantic query related to the topic
- Adjust topK for more or fewer results
- Lower minScore (e.g., 0.65) for broader matches

### Direct Key Retrieval
When you know the exact memory key, use `get_memory`:
- Format: "project-name_date_session-name_sequence"
- Example: "user-preferences_2025-04-15_user123_1"

## Advanced Usage

### Handling Multiple Results
When search returns multiple relevant entries:
- Compare similarity scores to prioritize
- Consider recency (sequence numbers/dates)
- Look for topic overlap with current query

### Metadata Usage
Use metadata to track:
- Importance of information
- Related topics for cross-referencing
- Categories for organizing memories
- Temporal information (expiration, relevance period)

### Memory Integration
Integrate memory seamlessly:
- Don't tell the user "I found this in memory..."
- Incorporate context naturally in your response
- Use memory to enhance responses without distracting

## Success Metrics
- 100% of tasks with missing context include ≥1 memory search before the final answer.
- ≤ 2 memory-tool calls (search/store) per small task; if exceeded, briefly report rationale.
- 0 storage of PII/secrets; redact sensitive data proactively (tokens, passwords, access keys, email/phone, addresses).
- 100% stored items include `Title`, `Tags`, `CorpusNames`, concise `Content`; `UserTriggered=true` only when the USER explicitly asks to store.
- ≤ 1 duplicate per 50 stored items; prefer `update` over `create` when semantically related memory exists.
- Evidence citations (`file:line`) present when referencing repository artifacts.
- Retrieval helpfulness: subsequent search hit-rate for stored items within the session ≥ 0.8.

## Anti-patterns
- Storing full conversations verbatim or raw logs.
- Saving secrets/tokens or PII; storing URLs with query strings containing identifiers.
- Creating multiple near-duplicates instead of updating existing memory.
- Vague, opinionated, or non-actionable memory content.
- Using `create_memory` without checking for existing related memories first.
- Setting `UserTriggered=true` without explicit user instruction.
- Omitting `CorpusNames`/`Tags`, making retrieval difficult.
- Ignoring tool budget and running long memory operations mid-critical path without rationale.

## Examples (Good/Bad)
- Good: After updating `rules/context-understanding.md` with a boilerplate delta, store a concise summary with file evidence.
  - Title: "Updated context-understanding rule — boilerplate delta"
  - Tags: ["rules", "context-understanding", "delta", "success-metrics"]
  - CorpusNames: ["/rules/"]
  - Content: 2–4 bullets summarizing what changed and why, with citations like `rules/context-understanding.md:34-65`.
- Bad: "Saved conversation transcript"; or storing raw token/secret; or duplicating an existing summary without new info.

## Consistency & Precedence
- Inherit instruction hierarchy from `rules/global-rules.md`. Sequential-only tool execution; low tool budget (≤ 2) for small tasks; evidence citation (`file:line`).
- Default `reasoning_effort` for this repo: high. You may switch to medium/minimal for latency-sensitive steps, but keep early-stop + escape-hatch and success metrics.
- Tool preambles: keep brief and structured — plan, action, summary — per `global-rules.md` guidance.

## Data Safety & Privacy
- Redaction-first: never store tokens, passwords, API keys, access URLs with embedded credentials, customer PII. Prefer summaries; if needed, store non-reversible identifiers (hash with salt) instead of raw.
- Consent gating: set `UserTriggered=true` only when the USER explicitly requests persistence.
- Minimization: store the least amount of data required to be useful; avoid copying user content unless necessary for retrieval.
- Retention hygiene: delete incorrect/outdated memories; prefer `update` over `create` for corrections.
- Multi-tenant hygiene: scope via `project_name` and `session_name` to avoid cross-user contamination.

## Triggers (Consolidated)
- Integration mentions ("integrate with ...").
- References to previous work ("I added X, now we need to ...").
- Mentions of specific systems without context.
- Requests that continue prior tasks or mention recent discussions.
- Handoffs from prior sessions ("continue from yesterday", "resume").
- Requests to summarize previous decisions or outputs.

## Example Workflow
  
  1. User asks a question
  2. EVALUATE if you might need additional context to answer properly
  3. IF IN DOUBT, SEARCH memory for relevant context using `search_keys` or `search_memory`
4. Process user request with available context
5. Formulate response
6. EVALUATE if response contains NEW information worth storing
7. IF YES, STORE key points from the interaction using `store_memory`
8. Return response to user

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wollfoo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
