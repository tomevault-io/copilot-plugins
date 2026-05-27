## mcp-google-doc

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
npm install          # install dependencies
npm run build        # compile TypeScript → build/
npm start            # run the MCP server
npm run dev          # watch mode — recompiles on change
```

Entry point: `build/server.js`. Requires env vars or local `credentials.json` + `token.json` for Google OAuth.

## Architecture

### Current State
Single-file MCP server (`src/server.ts`) exposing Google Docs/Drive CRUD operations over the Model Context Protocol via stdio. Auth: OAuth 2.0 via `@google-cloud/local-auth`; tokens cached in `token.json`.

**Tools:** `list-docs`, `get-doc`, `create-doc`, `update-doc`, `search-docs`, `delete-doc`
**Resources:** `googledocs://list`, `googledocs://{docId}`
**Prompts:** `create-doc-template`, `analyze-doc`

### Target Architecture: Vector RAG Support Tool

This server is evolving into an internal support tool for ~10 support agents. Agents ask natural-language questions; the server retrieves relevant content from Google Docs and returns grounded answers.

Full-text search was ruled out — it breaks on synonyms, paraphrases, and misspellings. Vector/semantic search operates on *meaning*, so "change my card" correctly matches a doc that says "update payment method".

```
INDEXING (periodic background job):
  Google Drive Docs → chunk (src/chunker.ts)
                    → embed via AWS Bedrock Titan (src/bedrock.ts)
                    → store in Amazon OpenSearch Serverless (src/opensearch.ts)

QUERY (per agent question):
  question → embed → k-NN search (top 5 chunks)
           → Claude claude-sonnet-4-6 grounded answer (src/answerer.ts)
           → return answer + source doc citations
```

Key modules (to be built):

| File | Role |
|------|------|
| `src/chunker.ts` | Splits doc text into ~500-token chunks with 50-token overlap |
| `src/bedrock.ts` | AWS Bedrock Titan embeddings client (`amazon.titan-embed-text-v2:0`) |
| `src/opensearch.ts` | OpenSearch Serverless vector index client |
| `src/indexer.ts` | Full + incremental sync pipeline |
| `src/retriever.ts` | Embed query + k-NN search |
| `src/answerer.ts` | RAG prompt builder + Claude answer |
| `src/secrets.ts` | AWS Secrets Manager loader (production) |

New MCP tools to be added to `src/server.ts`: `sync-docs`, `query-docs`, `sync-status`, `health`

AWS services: Bedrock (embeddings), OpenSearch Serverless (vector store), Secrets Manager (OAuth credentials at runtime), EC2 (hosting), CloudWatch (metrics).

**Required env vars (production):**
```
OPENSEARCH_ENDPOINT=https://...
AWS_REGION=us-east-1
GOOGLE_CREDENTIALS_SECRET_ARN=arn:aws:secretsmanager:...
GOOGLE_TOKEN_SECRET_ARN=arn:aws:secretsmanager:...
ANTHROPIC_API_KEY=...
```

## Hard Rules

- **Never commit `credentials.json` or `token.json`** — OAuth secrets, already in `.gitignore`. In production, load from AWS Secrets Manager via `src/secrets.ts`.
- **Create a PR after every completed task** before starting the next one.

## Connecting to Claude

**Claude Desktop** (`%APPDATA%\Claude\claude_desktop_config.json`):
```json
{
  "mcpServers": {
    "googledocs": {
      "command": "node",
      "args": ["C:/absolute/path/to/build/server.js"]
    }
  }
}
```

**Claude Code CLI:**
```bash
claude mcp add --transport stdio google-docs node /absolute/path/to/build/server.js
```

---
> Source: [ophydami/MCP-Google-Doc](https://github.com/ophydami/MCP-Google-Doc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
