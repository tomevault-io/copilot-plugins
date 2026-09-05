## mcp-cheat-sheet

> Concise reference for all MCP servers: Loki, Postgres, GitHub, Filesystem, and custom indexers


# 🔧 MCP Servers Cheat Sheet

**Quick reference for all available MCP tools. Use MCP servers FIRST before shell commands.**

---

## 🔍 **Loki MCP** - Log Queries (100x faster than docker logs)

### Critical Rules
- ⚠️ **Time format**: ONLY RFC3339 (`2025-10-29T10:00:00Z`) - NOT "2h ago" or Unix timestamps
- ⚠️ **Best practice**: Omit `start`/`end` for recent logs, use `limit` parameter instead
- ⚠️ **Performance**: Use specific service names, avoid wildcards when possible

### Tools
- `mcp_loki_loki_query(query, start?, end?, limit?)`
- `mcp_loki_loki_label_names()` - List available labels
- `mcp_loki_loki_label_values(label)` - Get values for label

### Examples
```logql
# Recent errors (RECOMMENDED - no time calc needed)
{service="ws_engine"} |= "ERROR"  # Use limit=100

# With time range (if needed)
{service="ws_engine"} |= "ERROR"  # start="2025-10-29T10:00:00Z"

# Multi-service
{service=~"ws_engine|calendar_service|trade_engine"}

# JSON field filter
{service="api_gateway"} | json | status_code >= 500
```

### Key Services
- HA: `ws_engine` (containers: `v2-ws_engine-1`, `v2-ws_engine-2`)
- Core: `trade_engine`, `signal_engine`, `calendar_service`, `api_gateway`, `db_service`

### Environment Identification

**Dev vs Prod Architecture**:
- **Development**: Dev VM (`192.168.0.103`) - `dev.nexus-trade.top`
- **Production**: Prod VM (`192.168.0.102`) - `prod.nexus-trade.top`
- **Loki Instances**: Separate instances per environment (same labels currently)

**Which Instance to Query?**

**Query DEV logs when**:
- Working on `dev` branch
- Testing locally
- User reports issue on `dev.nexus-trade.top`
- Debugging development features

**Query PROD logs when**:
- User reports production issue
- Issue on `prod.nexus-trade.top`
- Production monitoring/alerts
- Post-deployment verification

**Query Patterns**:

```logql
# Dev environment (default when working locally)
{service="ws_engine"} |= "ERROR"

# Prod environment (specify in query context)
# Note: Currently both use same service labels
# Query prod VM Loki directly or specify in context
{service="ws_engine"} |= "ERROR"
```

**Current Limitation**:
Both dev and prod use same service labels (`service="ws_engine"`).
To query specific environment:
1. Identify from context (domain, branch, user report)
2. Query appropriate Loki instance
3. Future: Add `environment` label to differentiate

**Multi-Environment Queries**:
```logql
# Query both (if needed)
{service="ws_engine"} |= "ERROR"
# Then filter by timestamp/context to identify environment
```

---

## 🗄️ **Postgres MCP** - Database Queries

### Tool
- `mcp_postgres_query(sql)` - Read-only SQL queries (BECAUSE ALL MIGRATIONS MADE DIRECTLY THROUGH DB SERVICE VIA ALEMBIC)

### Examples
```sql
-- Check recent data
SELECT COUNT(*) FROM candles WHERE created_at > NOW() - INTERVAL '1 hour';

-- Validate schema
SELECT table_name FROM information_schema.tables WHERE table_schema = 'public';

-- Performance check
SELECT schemaname, tablename, pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) 
FROM pg_tables WHERE schemaname = 'public';
```

### Connection
- Database: `nexus_v2`
- User: `postgres`
- Port: `5435` (MCP auto-configured)

---

## 📁 **Filesystem MCP** - File Operations

### Tools
- `mcp_filesystem_read_text_file(path, head?, tail?)`
- `mcp_filesystem_write_file(path, content)`
- `mcp_filesystem_list_directory(path)`
- `mcp_filesystem_directory_tree(path)`
- `mcp_filesystem_search_files(path, pattern, excludePatterns?)`
- `mcp_filesystem_edit_file(path, edits[], dryRun?)`

### When to Use
- ✅ Read/write config files (docker-compose, .env, etc.)
- ✅ Directory exploration
- ✅ File search and bulk operations
- ❌ NOT for log queries (use Loki MCP instead)

---

## 🐙 **GitHub MCP** - Repository Management

### Tools
- `mcp_github_list_issues(owner, repo, state?, labels?)`
- `mcp_github_update_issue(owner, repo, issue_number, title?, body?, state?)`
- `mcp_github_add_issue_comment(owner, repo, issue_number, body)`
- `mcp_github_list_pull_requests(owner, repo, state?, base?)`
- `mcp_github_get_file_contents(owner, repo, path, branch?)`
- `mcp_github_search_repositories(query, page?, perPage?)`

### Use Cases
- Create issues from RCA findings
- Update PR descriptions with implementation details
- Track technical debt and compliance issues
- Link code changes to GitHub Issues

---

## 🔬 **Repo Indexer MCP** - Code Search & Analysis

### Tools
- `mcp_repo-indexer_search_codebase(query, file_type?, limit?)` - Semantic code search
- `mcp_repo-indexer_get_code_complexity_report(file_pattern?)` - Complexity metrics
- `mcp_repo-indexer_get_dependency_graph(max_depth?)` - Architecture mapping
- `mcp_repo-indexer_get_performance_metrics()` - Performance analysis

### When to Use
- Find similar implementations or patterns
- Understand code complexity before refactoring
- Map service dependencies
- Identify technical debt hotspots

---

## 📚 **Docs Indexer MCP** - Documentation Search

### Tools
- `mcp_docs-indexer_search_documentation(query, category?, limit?)` - Search docs
- `mcp_docs-indexer_get_api_endpoints_catalog()` - List all API endpoints
- `mcp_docs-indexer_get_cross_references()` - Doc health check

### When to Use
- Find existing documentation before writing new docs
- Understand API endpoints and specifications
- Validate documentation completeness

---

## ⚙️ **Config Indexer MCP** - Configuration Analysis

### Tools
- `mcp_config-indexer_search_configuration(query, config_type?, limit?)` - Search configs
- `mcp_config-indexer_analyze_config_dependencies(config_path?)` - Dependency analysis
- `mcp_config-indexer_get_security_configuration()` - Security validation
- `mcp_config-indexer_get_performance_optimization()` - Optimization recommendations

### Config Types
- `docker` - docker-compose files
- `yaml` - YAML configs
- `json` - JSON configs
- `env` - Environment files

---

## 🎯 Quick Decision Tree

**Need logs?** → Use Loki MCP (NOT docker logs)
**Need database data?** → Use Postgres MCP (NOT docker exec psql)
**Need to find code?** → Use Repo Indexer MCP (NOT grep for semantics)
**Need documentation?** → Use Docs Indexer MCP
**Need to manage files?** → Use Filesystem MCP (NOT cat/echo/sed)
**Need GitHub operations?** → Use GitHub MCP (NOT gh CLI)
**Need config analysis?** → Use Config Indexer MCP

---

## ⚡ Performance Tips

1. **Loki**: Omit time params for recent queries, use `limit` instead of time calculations
2. **Postgres**: Use indexes in WHERE clauses, avoid SELECT *
3. **Repo Indexer**: Use specific file_type filters to narrow search
4. **Filesystem**: Use excludePatterns to skip node_modules, .git, etc.
5. **GitHub**: Use pagination for large result sets

---

## 🚫 Common Mistakes

- ❌ Using "2h ago" in Loki queries (use RFC3339 or omit)
- ❌ Using docker logs instead of Loki MCP (100x slower)
- ❌ Using grep for semantic code search (use Repo Indexer)
- ❌ Creating new docs without searching Docs Indexer first
- ❌ Using shell commands for file operations (use Filesystem MCP)

---

## 📖 Detailed Documentation

For complex scenarios, refer to:
- Loki queries: `v2/docs/loki-rca-queries.md`
- MCP deployment: `v2/docs/plans/LOKI-MCP-QUICKSTART.md`

---
> Source: [john-markowsky/PM-Bot](https://github.com/john-markowsky/PM-Bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
