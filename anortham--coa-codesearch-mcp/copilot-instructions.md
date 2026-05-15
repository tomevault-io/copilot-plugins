## coa-codesearch-mcp

> **SEMANTIC SEARCH IS A CORE FEATURE - NEVER SUGGEST REMOVING IT**

# CodeSearch MCP Server - Developer Guide

## ⚠️ NON-NEGOTIABLE REQUIREMENTS

**SEMANTIC SEARCH IS A CORE FEATURE - NEVER SUGGEST REMOVING IT**
- 3-tier search architecture: SQLite → Lucene → Semantic (vec0/HNSW)
- Semantic search is the tentpole feature - focus on making it FAST, not removing it
- If performance is an issue, optimize the implementation, don't remove functionality

## 🎯 Quick Reference

Lucene.NET-powered code search with julie-codesearch native type extraction. Local workspace indexing with cross-platform support.

**Version**: 4b32d04 | **Status**: Production Ready | **Performance**: 117 files/sec | **Framework**: julie-codesearch Rust CLI (25 languages)

### Core Tools (14 available) - **Now with Smart Defaults!** 🎯

```bash
# Essential Search
text_search         # Full-text code search with CamelCase tokenization
symbol_search       # Find classes, methods, interfaces by name
goto_definition     # Jump to exact symbol definitions
find_references     # Find ALL usages (critical before refactoring)

# File Discovery
search_files        # 🆕 Unified file/directory search (replaces file_search + directory_search)
recent_files        # Recent modifications (great for context)

# Advanced Search
line_search         # Line-by-line search (replaces grep/rg)
search_and_replace  # Bulk find & replace with preview + fuzzy matching
trace_call_path     # Hierarchical call chain analysis with semantic bridging

# Refactoring
smart_refactor      # AST-aware symbol renaming (byte-offset precision)

# Code Editing
edit_lines          # 🆕 Unified line editing (insert/replace/delete - replaces 3 tools)

# Semantic Analysis
get_symbols_overview # Extract all symbols from files (classes, methods, etc.)
find_patterns       # Detect code patterns and quality issues

# System
index_workspace     # Build/update search index (REQUIRED FIRST)
```

**Note:** Tools now have smart defaults - most calls need only 1-2 parameters!
**Replaced:** `insert_at_line`, `replace_lines`, `delete_lines` → use `edit_lines`
**Replaced:** `file_search`, `directory_search` → use `search_files`
**Removed:** `batch_operations` (obsolete with MCP parallel tool calls), `similar_files` (never implemented), `SearchType` parameter (use `SearchMode` instead)

## 🚨 Development Workflow

**Code Changes:**

1. Exit Claude Code completely
2. `dotnet build -c Debug` (Debug mode recommended)
3. Restart Claude Code
4. ⚠️ **Testing before restart shows OLD CODE**

**Never run:** `dotnet run -- stdio` (creates orphaned processes)

## 📝 Tool Parameter Documentation

**IMPORTANT: XML Comments are Primary, [Description] is Fallback**

When documenting tool parameters:
- **XML `/// <summary>` comments**: Primary source for MCP tool descriptions - this is what Claude sees
- **`[Description("...")]` attribute**: Fallback only if XML comments are missing
- **Always document in BOTH places** for consistency, but XML comments take precedence

Example:
```csharp
/// <summary>
/// The refactoring operation to perform.
/// Valid operations: rename_symbol, extract_to_file, move_symbol_to_file, extract_interface
/// </summary>
[Description("The refactoring operation to perform: rename_symbol, extract_to_file, move_symbol_to_file, extract_interface")]
public required string Operation { get; set; }
```

## 🔍 Usage Essentials - **Minimal Parameters!** ✨

**Always start with:**

```bash
# Index with just the path (or omit for current directory)
mcp__codesearch__index_workspace --workspacePath "."
```

**Common patterns (now with smart defaults):**

```bash
# Search - just the query! (workspace defaults to current directory)
mcp__codesearch__text_search --query "class UserService"

# Verify types - just the symbol name!
mcp__codesearch__goto_definition --symbol "UserService"

# Check impact - just the symbol!
mcp__codesearch__find_references --symbol "UpdateUser"

# Find files - just the pattern! (defaults to files, current workspace)
mcp__codesearch__search_files --pattern "*.cs"
mcp__codesearch__search_files --pattern "**/*.csproj"

# Find directories - specify resourceType
mcp__codesearch__search_files --pattern "test*" --resourceType "directory"

# Edit code - operation + line + content
mcp__codesearch__edit_lines --filePath "User.cs" --operation "insert" --startLine 42 --content "// TODO"
mcp__codesearch__edit_lines --filePath "User.cs" --operation "replace" --startLine 10 --endLine 15 --content "refactored code"
mcp__codesearch__edit_lines --filePath "User.cs" --operation "delete" --startLine 20 --endLine 25
```

### ⚡ Before vs After

**Before (8+ parameters):**
```bash
text_search --workspacePath "." --query "UserService" --searchMode "auto" --searchType "standard" --caseSensitive false --maxTokens 8000 --noCache false --documentFindings false --autoDetectPatterns false
```

**After (1 parameter!):**
```bash
text_search --query "UserService"  # All other params have smart defaults!
```

## 🏗️ Architecture

**Index Storage**: `.coa/codesearch/indexes/{workspace-hash}/` (local per workspace)
- `lucene/` - Lucene.NET full-text search index
- `db/` - SQLite canonical symbol database (workspace.db)
- `vectors/` - HNSW semantic search index (julie-semantic)

**Logs**: `.coa/codesearch/logs/` (workspace-specific logging)

**Token Optimization**: Active via `BaseResponseBuilder<T>` with 40% safety budget
**Test Framework**: NUnit (427 tests, zero warnings)
**Framework**: Local project references for active development

### 3-Tier Search Architecture

**Tier 1: SQLite Exact Lookups** (~1ms)
- Symbol definitions (classes, methods, interfaces)
- Identifier usages (calls, references) via LSP-quality extraction
- Use: `goto_definition`, `find_references`, exact symbol queries

**Tier 2: Lucene Fuzzy Search** (~20ms)
- Full-text code search with CamelCase tokenization
- Smart scoring: type definitions boosted 10x, test files de-prioritized
- Use: `text_search`, `symbol_search`, fuzzy matching

**Tier 3: Semantic Search** (~47ms) ✅ **PRODUCTION READY**
- Vector similarity via sqlite-vec (vec0) + HNSW index
- 384-dimensional embeddings (bge-small-en-v1.5 model)
- Cross-language semantic code discovery
- Use: Finding conceptually similar code, semantic refactoring

### Semantic Search Pipeline

**Bulk Indexing** (~40.6s total):
1. julie-semantic generates embeddings with ONNX (~40s)
2. Writes f32 vectors as BLOBs to SQL (~0.05s)
3. C# copies BLOBs to vec0 for fast KNN search (~0.6s)

**Incremental Updates** (~0.6s per file):
1. julie-semantic update --write-db regenerates embeddings (~0.4s)
2. C# copies updated BLOBs to vec0 (~0.2s)
3. Semantic search stays current automatically

**Query Time**:
- Search queries converted to vectors via C# ONNX (instant, <10ms)
- vec0 KNN search finds top matches (~47ms for 5074 symbols)
- Results enriched with symbol metadata from SQLite

**Key Components**:
- `julie-semantic` (Rust): ONNX inference, HNSW indexing, BLOB storage
- `SqliteVecExtensionService.cs`: Loads vec0 extension for vector operations
- `SQLiteSymbolService.BulkGenerateEmbeddingsAsync`: Copies BLOBs to vec0
- `SQLiteSymbolService.SearchSymbolsSemanticAsync`: Executes KNN queries

## 🚀 Recent Improvements (v2.1.8+)

**🆕 Smart Defaults & Tool Consolidation** (Latest - v2.1.8)

- ✅ **Tool consolidation**: 19 tools → 14 tools (-26% reduction)
  - `edit_lines`: Replaces insert_at_line, replace_lines, delete_lines (3→1)
  - `search_files`: Replaces file_search, directory_search (2→1)
  - Removed `batch_operations` (obsolete with MCP parallel tool calls)
- ✅ **Smart defaults**: All tools now have optional `workspacePath` (defaults to current workspace)
- ✅ **Minimal usage**: Most tools need only 1-2 parameters instead of 8+
- ✅ **Parameter descriptions**: All defaults clearly documented in descriptions
- ✅ **Zero breaking changes**: All existing code continues to work
- ✅ **Semantic bridging**: `trace_call_path` uses embeddings for cross-language tracing (confidence ≥ 0.7)

**SmartRefactorTool - AST-Aware Symbol Renaming** (v2.1.8)

- ✅ Byte-offset replacement using SQLite identifiers table (AST-validated positions)
- ✅ ReferenceResolverService for LSP-quality reference finding
- ✅ Last-to-first replacement algorithm preserves byte offsets during replacement
- ✅ Dry run mode by default for safe preview before applying changes
- ✅ Token-optimized responses with insights and next actions
- ✅ Operations supported: `rename_symbol` (more coming: extract_function, inline_variable)

**Fuzzy Matching Mode for SearchAndReplaceTool** (Latest)

- ✅ Google's DiffMatchPatch fuzzy matching (battle-tested since 2006)
- ✅ Configurable threshold (0.0-1.0) and distance (100-10000 chars)
- ✅ Finds ALL fuzzy matches across files (not just first occurrence)
- ✅ Handles typos, spacing variations, and imperfect matches
- ✅ Infinite loop protection with forced progress guarantee
- ✅ Example: Finds `getUserData()`, `getUserDat()` (typo), `getUserData ()` (spacing)

**SmartQueryPreprocessor Integration**

- ✅ Comprehensive test suite: 35 unit tests with full coverage
- ✅ Intelligent query routing: Auto-detects Symbol/Pattern/Standard modes
- ✅ Wildcard validation: Shared utility prevents invalid Lucene queries
- ✅ Multi-field optimization: Routes queries to optimal indexed fields

**CodeAnalyzer Consistency Audit**

- ✅ Fixed 3 critical inconsistencies in SmartSnippetService, GoToDefinitionTool, SymbolSearchTool
- ✅ Unified dependency injection: All tools use single configured CodeAnalyzer instance
- ✅ Consistent tokenization: `preserveCase: false, splitCamelCase: true` across system

**Framework Integration**

- ✅ Local project references: Active development with live framework changes
- ✅ JSON Schema fix: Only properties with explicit `[Required]` attribute or `required` modifier are required
- ✅ Smart defaults work correctly: Value types with defaults (bool, int, float) no longer incorrectly marked as required
- ✅ All 427 tests passing: Full compatibility with framework improvements
- ✅ Zero regressions: Maintains production stability during development

## ⚠️ Common Pitfalls

1. **Missing index**: Always run `index_workspace` first
2. **Field access**: Use `hit.Fields["name"]` not `hit.Document.Get()`
3. **Method assumptions**: Verify signatures with `goto_definition`
4. **Testing changes**: Must restart Claude Code after building
5. **Type extraction**: Check `type_info` field structure

## 🛠️ Code Patterns

```csharp
// ✅ Path Resolution
_pathResolver.GetIndexPath(workspacePath)

// ✅ Lucene Operations
await _indexService.SearchAsync(...)

// ✅ Response Building
return new AIOptimizedResponse<T> { Data = new AIResponseData<T>(...) }
```

## 🧪 Testing

```bash
# All tests (427 total)
dotnet test

# Specific tool tests
dotnet test --filter "SymbolSearchToolTests"

# Health check
mcp__codesearch__recent_files --workspacePath "."
```

## 📚 Related

- **COA MCP Framework**: Core MCP framework (v2.1.32)
- **Goldfish MCP**: Session/memory management
- **julie-codesearch**: Rust CLI for native tree-sitter type extraction (25 languages)
- **julie-semantic**: Rust CLI for ONNX embeddings and HNSW semantic search

---

_Updated: 2025-10-09 - Updated documentation to reflect julie-codesearch Rust CLI (25 language support), removed obsolete TreeSitter.DotNet references, expanded TypeExtraction.Languages config to all 25 supported languages_

---
> Source: [anortham/coa-codesearch-mcp](https://github.com/anortham/coa-codesearch-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
