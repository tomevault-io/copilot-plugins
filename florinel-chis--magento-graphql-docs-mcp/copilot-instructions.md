## magento-graphql-docs-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MCP (Model Context Protocol) server that provides tools to search and retrieve Magento 2 GraphQL API documentation. It parses 350+ local markdown files with YAML frontmatter, extracts GraphQL schema elements, indexes content in SQLite with FTS5, and exposes 8 tools via FastMCP over STDIO.

## Documentation Source

### Required: Adobe Commerce GraphQL Documentation

The MCP server requires access to the official Adobe Commerce GraphQL documentation markdown files from the [AdobeDocs/commerce-webapi](https://github.com/AdobeDocs/commerce-webapi) repository.

**Clone the documentation repository:**

```bash
git clone https://github.com/AdobeDocs/commerce-webapi.git
```

The GraphQL documentation is located at: `commerce-webapi/src/pages/graphql/`

**This directory contains:**
- 350+ markdown files with YAML frontmatter
- Structured in categories: schema/, tutorials/, develop/, usage/, payment-methods/
- Total code examples: 963 blocks (GraphQL, JSON, PHP, Bash)
- GraphQL schema elements: 51 (queries, mutations, types, interfaces)

### Configuration Options

The server looks for documentation in this priority order (implemented in `config.py:get_docs_path()`):

1. **Environment Variable** (highest priority)
   ```bash
   export MAGENTO_GRAPHQL_DOCS_PATH="/path/to/commerce-webapi/src/pages/graphql"
   ```
   - Validated on startup - will error if path doesn't exist
   - Use absolute paths for best results

2. **Symlink in Project** (`./data/` directory - recommended for development)
   ```bash
   cd magento-graphql-docs-mcp
   ln -s /path/to/commerce-webapi/src/pages/graphql data
   ```
   - Checks if `data/` exists and contains `.md` files
   - Can be symlink or actual directory with markdown files
   - Resolved to absolute path automatically

3. **Sibling Directory** (automatic detection)
   ```
   projects/
   ├── magento-graphql-docs-mcp/  (this server)
   └── commerce-webapi/
       └── src/pages/graphql/     (documentation)
   ```
   - Automatically detected if commerce-webapi is cloned as sibling

**Important**: All paths are validated on startup. If no valid documentation is found, the server will fail with a helpful error message explaining the three setup options.

### Verification

Before running the server, verify documentation access:

```bash
# Check environment variable (if set)
echo $MAGENTO_GRAPHQL_DOCS_PATH
ls -la $MAGENTO_GRAPHQL_DOCS_PATH

# Check symlink (if used)
ls -la data/

# Expected structure:
# - index.md (GraphQL overview)
# - release-notes.md
# - schema/ (285 files - queries, mutations, types)
# - tutorials/ (13 files - checkout, orders, etc.)
# - develop/ (8 files - extending GraphQL)
# - usage/ (10 files - using GraphQL API)
# - payment-methods/ (11 files)
```

### Database Location

The SQLite database is stored at `~/.mcp/magento-graphql-docs/database.db` by default.

Override with:
```bash
export MAGENTO_GRAPHQL_DOCS_DB_PATH="/custom/path/database.db"
```

The database is automatically created on first run and only re-indexed when markdown files change (based on mtime).

## Core Architecture

### Three-Layer System

1. **Parser Layer** (`magento_graphql_docs_mcp/parser.py`)
   - Parses markdown files with YAML frontmatter using `python-frontmatter`
   - Extracts document metadata (title, description, keywords from frontmatter)
   - Extracts categories from file paths (e.g., "schema/products/queries" → category="schema", subcategory="products")
   - Extracts markdown structure (headers, code blocks)
   - Detects GraphQL elements in code blocks (queries, mutations, types, interfaces)
   - Builds searchable text combining all available text
   - Uses Pydantic models: `Document`, `CodeBlock`, `GraphQLElement`

2. **Ingestion Layer** (`magento_graphql_docs_mcp/ingest.py`)
   - File modification time tracking (only re-parse if files change)
   - Creates SQLite schema with 4 tables: documents, code_blocks, graphql_elements, metadata
   - FTS5 indexes on: documents.searchable_text and graphql_elements.searchable_text (trigram tokenization)
   - Bulk inserts for performance (350 documents, 963 code blocks, 51 GraphQL elements)
   - Clears existing data before re-ingestion (not incremental)

3. **Server Layer** (`magento_graphql_docs_mcp/server.py`)
   - FastMCP server with lifespan manager that triggers ingestion on startup
   - Error handling in lifespan: catches FileNotFoundError with helpful setup instructions
   - Exposes 8 tools over STDIO
   - Returns formatted markdown responses
   - Returns top K results for searches (configurable via MAGENTO_GRAPHQL_DOCS_TOP_K env var, default: 5)
   - Uses configurable constants for code preview lengths and field limits

### MCP Tools (8 Total)

1. **search_documentation**: FTS search on documents with category/subcategory/content_type filters
2. **get_document**: Direct lookup by file_path, returns full markdown content
3. **search_graphql_elements**: FTS search on graphql_elements with element_type filter
4. **get_element_details**: Lookup element by name with source document and code examples
5. **list_categories**: Hierarchical category tree with document counts
6. **get_tutorial**: Sequential tutorial steps from tutorials/ directory
7. **search_examples**: Search code_blocks by content and language
8. **get_related_documents**: Find related docs by category and keywords

## Development Commands

### Installation
```bash
# Development install
cd magento-graphql-docs-mcp
pip install -e .
```

### Running the Server
```bash
# Run MCP server (parses docs on first run or if files modified)
magento-graphql-docs-mcp

# Or via Python module
python3 -m magento_graphql_docs_mcp.server
```

### Testing

**Important**: All tests must be run from the project root directory:

```bash
# Navigate to project root first
cd magento-graphql-docs-mcp

# Test markdown parser
python3 tests/verify_parser.py

# Test database ingestion
python3 tests/verify_db.py

# Test MCP server and all 8 tools
python3 tests/verify_server.py

# Run performance benchmarks
python3 tests/benchmark_performance.py
```

Running tests from other directories will cause import errors.

### Configuration
```bash
# Set custom documentation path
export MAGENTO_GRAPHQL_DOCS_PATH=/path/to/commerce-webapi/src/pages/graphql

# Set custom database path
export MAGENTO_GRAPHQL_DOCS_DB_PATH=/custom/path/database.db

# Run server
magento-graphql-docs-mcp
```

## Key Implementation Details

### Frontmatter Parsing
Uses `python-frontmatter` library to parse YAML metadata from markdown files:
```yaml
---
title: products query
description: Describes how to construct and use the Catalog Service products query
keywords:
  - GraphQL
  - Services
---
```

Fallback: If no frontmatter title, extracts first `#` header from markdown.

### Category Extraction from File Paths
Pattern: `category/subcategory/file.md`
- `schema/products/queries/products.md` → category="schema", subcategory="products"
- `develop/exceptions.md` → category="develop", subcategory=None
- `index.md` → category="root", subcategory=None

Content type determined by category:
- "schema" → "schema"
- "tutorials" → "tutorial"
- "develop", "usage" → "guide"

### GraphQL Element Detection
Regex patterns in `parser.py:extract_graphql_elements()`:
- `query\s+(\w+)` → Detects GraphQL queries
- `mutation\s+(\w+)` → Detects mutations
- `type\s+(\w+)` → Detects types
- `interface\s+(\w+)` → Detects interfaces

Extracts:
- Element name
- Fields (via regex on field patterns)
- Parameters (via `$(\w+):` pattern)
- Builds searchable text

### Database Design
```sql
-- Documents (350 rows)
documents (id, file_path, title, description, keywords_json, category,
          subcategory, content_type, searchable_text, headers_json,
          last_modified, content_md)
  + FTS5 index on searchable_text
  + Indexes on category, subcategory, content_type

-- Code blocks (963 rows)
code_blocks (id, document_id, language, code, context, line_number)
  + Indexes on document_id, language

-- GraphQL elements (51 rows)
graphql_elements (id, document_id, element_type, name, fields_json,
                 parameters_json, return_type, description, searchable_text)
  + FTS5 index on searchable_text
  + Indexes on document_id, element_type, name

-- Metadata (1 row)
metadata (key, docs_directory_mtime, total_files, ingestion_time)
```

### Performance Characteristics

**Actual Benchmark Results** (run `tests/benchmark_performance.py`)

*Benchmarked: November 2024, Apple Silicon M1/M2, 350 documents*

- **Startup Time**: 0.87s (when data unchanged) | 3-5s (first run or files changed)
- **Search Time**: 5.5ms average (FTS5 direct: 0.7ms) - **18x faster than 100ms target**
- **Document Retrieval**: 8.2ms (direct lookup: 0.3ms)
- **GraphQL Element Search**: 3.4ms
- **Parsing Time**: ~2-3 seconds for 350 markdown files
- **Ingestion Time**: ~1-2 seconds for database population
- **Re-ingestion**: Only happens if any markdown file mtime changes
- **Database Size**: ~30 MB for 350 documents

All performance targets exceeded: <5s startup ✓, <100ms searches ✓

**Note**: Performance will vary based on hardware. Run `python3 tests/benchmark_performance.py` to measure on your system.

### Searchable Text Building
Combines:
1. Title
2. Description (from frontmatter)
3. Keywords (from frontmatter)
4. All headers
5. Content (with code blocks removed, markdown links stripped)

This enables comprehensive FTS search across all textual content.

### Code Block Context Extraction
For each code block, extracts 1-5 lines before the block as context. This provides surrounding explanation for better search results in `search_examples` tool.

### Tutorial Sequencing
Tutorial files ordered by file_path natural sorting:
- `tutorials/checkout/create-cart.md` (Step 1)
- `tutorials/checkout/add-product-to-cart.md` (Step 2)
- etc.

File path ordering provides sequential step ordering.

## Important Notes

- The server MUST have access to GraphQL documentation markdown files
- Database file requires write permissions in ~/.mcp/magento-graphql-docs/
- FTS5 is SQLite extension - requires SQLite 3.9.0+ (standard in Python 3.10+)
- Frontmatter parsing gracefully handles files without frontmatter
- GraphQL element extraction is best-effort (uses regex, not full parser)
- Multiple elements can have same name (e.g., "Query" appears in multiple docs)

## Configuration

All configuration in `config.py`:

### Path Configuration
- `DB_PATH`: Database location (default: ~/.mcp/magento-graphql-docs/database.db)
- `DOCS_PATH`: Documentation directory (auto-detected or from environment variable)
- Override via environment variables:
  - `MAGENTO_GRAPHQL_DOCS_DB_PATH`: Custom database location
  - `MAGENTO_GRAPHQL_DOCS_PATH`: Custom documentation path

### Performance Tuning (Environment Variables)
- `MAGENTO_GRAPHQL_DOCS_TOP_K`: Number of search results to return (default: 5)
- `MAGENTO_GRAPHQL_DOCS_MAX_FIELDS`: Max fields per GraphQL element (default: 20)
- `MAGENTO_GRAPHQL_DOCS_CODE_PREVIEW`: Max code preview length in chars (default: 400)

### Constants
- `SEARCH_RESULT_MULTIPLIER`: Fetch 2x results before filtering (hardcoded: 2)

## Extension Points

### Adding New Tools
1. Add `@mcp.tool()` decorated function in `server.py`
2. Use type hints with `Annotated[Type, Field(description=...)]` for parameter docs
3. Query database using `Database(DB_PATH)`
4. Format output as markdown string
5. Test with `verify_server.py`

### Improving GraphQL Element Extraction
To more accurately extract GraphQL elements:
1. Consider using a GraphQL parser library (e.g., `graphql-core`)
2. Parse code blocks marked as `graphql` language
3. Extract full AST for queries, mutations, types
4. Store more structured data (input types, output types, arguments)

### Supporting Multiple Documentation Versions
To support multiple Magento versions:
1. Add version parameter to config
2. Store multiple doc directories: `graphql-2.4.7/`, `graphql-2.4.8/`
3. Include version in database table or use separate databases
4. Add version parameter to tools

### Adding Full-Text Search on Code
To enable FTS search on code content (not just surrounding context):
1. Create FTS5 index on code_blocks.code
2. Update `search_examples` to use FTS instead of LIKE
3. Benefit: Faster, more relevant code search

## Comparison with Other MCP Implementations

### vs Gemini Docs MCP

| Aspect | Gemini Docs MCP | GraphQL Docs MCP |
|--------|----------------|------------------|
| Data Source | Web (HTML) | Local (Markdown) |
| Startup | 30-60s | 3-5s |
| Offline | ❌ | ✅ |
| Content | Unstructured | Structured + Narrative |
| Tools | 3 | 8 |

### vs Magento REST API MCP

| Aspect | REST API MCP | GraphQL Docs MCP |
|--------|-------------|------------------|
| Data Source | OpenAPI JSON | Markdown files |
| Content Type | API specs only | Schemas + Guides + Tutorials |
| Examples | None | 963 code blocks |
| Tutorials | ❌ | ✅ |
| Tools | 5 | 8 |

## Key Learnings from Implementation

1. **Frontmatter is Gold**: YAML frontmatter provides high-quality metadata
2. **File Path = Category**: Directory structure encodes valuable categorization
3. **Code Context Matters**: Surrounding text makes code examples searchable
4. **FTS5 is Fast**: Trigram tokenization + FTS5 = <100ms searches
5. **Regex is Good Enough**: Don't need full GraphQL parser for useful element extraction
6. **Markdown is Convenient**: Direct markdown content in DB allows full document retrieval
7. **Bulk Inserts Win**: Batch inserts much faster than row-by-row

## Project Dependencies

- Core: `fastmcp`, `sqlite-utils`, `pydantic`
- Parsing: `python-frontmatter`, `markdown-it-py`
- Dev: `mcp>=1.16.0` (for testing with MCP client)
- Python: 3.10+ (for modern type hints and SQLite FTS5)

## Common Pitfalls

1. **Missing Frontmatter**: Not all markdown files have frontmatter - use fallbacks
2. **GraphQL in Non-GraphQL Blocks**: Only parse code blocks with language="graphql"
3. **File Path Assumptions**: Don't assume all paths have subcategory level
4. **FTS Syntax**: FTS5 uses different syntax than LIKE queries
5. **JSON Serialization**: Must serialize lists to JSON for SQLite storage
6. **Context Extraction**: Index bounds checking when looking backwards for context

## Testing Strategy

Four-level verification:
1. **Parser Test** (`tests/verify_parser.py`): Ensures markdown → Pydantic models works
2. **Database Test** (`tests/verify_db.py`): Ensures ingestion → SQLite works + FTS search works
3. **Server Test** (`tests/verify_server.py`): Ensures all 8 MCP tools work via STDIO
4. **Performance Benchmark** (`tests/benchmark_performance.py`): Measures actual startup and query times

Run all four tests to verify complete system functionality and performance.

---
> Source: [florinel-chis/magento-graphql-docs-mcp](https://github.com/florinel-chis/magento-graphql-docs-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
