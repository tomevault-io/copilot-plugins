## mcp-cli-wrappers

> - **Type**: CLI Tool Collection (Monolithic)

# CLI Wrappers - MCP Tool Collection

## Overview

- **Type**: CLI Tool Collection (Monolithic)
- **Stack**: Python 3.10+ with `uv` (PEP 723 script execution)
- **Purpose**: Progressive-disclosure CLI wrappers for Model Context Protocol (MCP) tools
- **Architecture**: 13 self-contained, single-file Python scripts
- **Location**: `~/cli-wrappers/`

This CLAUDE.md is the authoritative source for development guidelines.

---

## MANDATORY: Before Using Any Tool

**ALWAYS run these commands first:**
```bash
uv run TOOL.py --help    # Quick overview
uv run TOOL.py list      # See all functions
```

---

## CLI Quick Reference

| Tool | Purpose | When to Use |
|------|---------|-------------|
| **crawl4ai** | Web scraping | Scrape pages, extract content |
| **firecrawl** | Advanced scraping | JS-heavy sites, structured extraction |
| **ref** | Doc search | Find API/framework documentation |
| **deepwiki** | Repo docs | Understand GitHub repos, ask questions |
| **gitmcp** | GitHub docs | Fetch llms.txt, READMEs from repos |
| **semly** | Code analysis | Outline code, find patterns |
| **repomix** | Pack codebases | Prepare code for AI analysis |
| **github** | GitHub API | Issues, PRs, file operations |
| **sequential-thinking** | Reasoning | Complex problems, design decisions |
| **claude-flow** | Orchestration | Multi-agent workflows |
| **flow-nexus** | Cloud execution | Sandboxed code execution |

**Full reference**: See `.claude/CLI_REFERENCE.md`

---

## Universal Development Rules

### Code Quality (MUST)

- **MUST** use PEP 723 headers for uv script execution
- **MUST** return valid JSON from all functions
- **MUST** validate arguments before execution
- **MUST** check API keys and credentials exist before MCP calls
- **MUST** follow 4-level progressive disclosure pattern
- **MUST NOT** add external CLI dependencies beyond `httpx`, `pydantic`, `click`, `rich`
- **MUST NOT** execute shell commands with `subprocess` without validation

### Best Practices (SHOULD)

- **SHOULD** use `rich` library for formatted output
- **SHOULD** provide clear error messages with recovery steps
- **SHOULD** include timeout limits on network requests (30s default)
- **SHOULD** support `--format json|text|table` output options
- **SHOULD** cache API responses when reasonable (5-minute TTL)
- **SHOULD** document all environment variables required
- **SHOULD** include helpful examples in `FUNCTION_EXAMPLES`

### Anti-Patterns (MUST NOT)

- **MUST NOT** use `subprocess.run()` with `shell=True`
- **MUST NOT** store credentials in code (use environment variables)
- **MUST NOT** execute user input directly
- **MUST NOT** suppress errors silently (always log/display)
- **MUST NOT** block on long-running operations (use async when possible)
- **MUST NOT** skip argument validation before MCP calls

---

## 4-Level Progressive Disclosure Pattern

All wrappers implement this pattern for token efficiency (60-70% savings):

### Level 1: `--help`
```bash
uv run WRAPPER.py --help
```
- Quick overview (10-15 lines)
- Available functions listed
- Usage examples
- Pointer to next level
- **Tokens**: ~30

### Level 2: `list` and `info FUNCTION`
```bash
uv run WRAPPER.py list                    # All functions
uv run WRAPPER.py info FUNCTION_NAME      # Detailed docs
```
- Full function signature
- All parameters with types
- Return type and format
- Related functions
- **Tokens**: ~150

### Level 3: `example FUNCTION`
```bash
uv run WRAPPER.py example FUNCTION_NAME
```
- 2-3 real working examples
- Success and error cases
- Realistic sample data
- Copy-paste ready
- **Tokens**: ~200

### Level 4: `FUNCTION --help`
```bash
uv run WRAPPER.py FUNCTION_NAME --help
```
- Complete argparse reference
- All OPTIONS with detailed explanations
- USAGE section
- ERROR CASES with recovery hints
- **Tokens**: ~500

---

## Project Structure

### Core Wrappers (MCP Tools)
- **`crawl4ai.py`** → Web scraping and crawling
- **`firecrawl.py`** → Advanced web data extraction
- **`ref.py`** → Documentation search
- **`repomix.py`** → Repository analysis and packing
- **`deepwiki.py`** → GitHub wiki documentation
- **`github.py`** → GitHub repository operations
- **`semly.py`** → Code search and analysis
- **`exa.py`** → Web search with AI
- **`gitmcp.py`** → GitHub docs via SSE (gitmcp.io)
- **`sequential-thinking.py`** → Multi-step reasoning
- **`claude-flow.py`** → Agentic workflow orchestration
- **`flow-nexus-cli-wrapper.py`** → Cloud orchestration

### Tooling & Documentation
- **`generate-all-wrappers.py`** → Master introspection script
- **`README.md`** → Quick start and overview
- **`INDEX.md`** → Complete tool index
- **`ref.md`** → Ref.py documentation
- **`exa-cli.md`** → Exa.py documentation
- **`.claude/`** → Claude Code configuration
- **`CLAUDE.md`** → This file

---

## Core Commands & Workflows

### Quick Start (After `cd ~/cli-wrappers/`)

#### Explore Any Wrapper (4 Levels)
```bash
# Level 1: Quick overview
uv run WRAPPER.py --help

# Level 2: See all functions
uv run WRAPPER.py list

# Level 2: Detailed function docs
uv run WRAPPER.py info FUNCTION_NAME

# Level 3: Working examples
uv run WRAPPER.py example FUNCTION_NAME

# Level 4: Complete reference
uv run WRAPPER.py FUNCTION_NAME --help
```

### Common Tasks

#### Scrape a Website
```bash
# Quick scrape with crawl4ai
uv run crawl4ai.py scrape https://example.com

# Advanced scraping with firecrawl
uv run firecrawl.py scrape https://example.com

# See options
uv run firecrawl.py scrape --help
```

#### Search Documentation
```bash
# Ref.py (fastest for docs)
uv run ref.py search "query"

# Exa (web search)
uv run exa.py search "machine learning"

# Semly (code search)
uv run semly.py outline /path/to/code

# GitMCP (GitHub repo docs via SSE)
uv run gitmcp.py fetch_generic_documentation --owner jlowin --repo fastmcp
uv run gitmcp.py search_generic_documentation --owner anthropics --repo claude-code --query "hooks"
```

#### GitHub Operations
```bash
# List issues
uv run github.py info list_issues

# Create PR
uv run github.py info create_pull_request

# Push files
uv run github.py info push_files
```

#### Analyze Repository
```bash
# Pack codebase
uv run repomix.py pack_codebase ~/my-project

# Generate Claude skill
uv run repomix.py generate_skill --output skill.md
```

#### Complex Problem Solving
```bash
# Use sequential thinking
uv run sequential-thinking.py sequentialthinking "your complex question"
```

---

## Required Environment Variables

### GitHub Operations
```bash
export GITHUB_TOKEN="ghp_..."              # GitHub Personal Access Token
export GITHUB_REPO="owner/repo"            # Target repository
```

### Web Scraping & Search
```bash
export CRAWL4AI_API_KEY="..."              # Crawl4AI (if API key required)
export FIRECRAWL_API_KEY="..."             # Firecrawl API key
export EXA_API_KEY="..."                   # Exa AI API key
```

### Documentation
```bash
export REF_API_KEY="ref-..."               # Ref Tools API key
```

### Optional: Model Context Protocol
```bash
export ANTHROPIC_API_KEY="sk-..."          # For Claude API calls
export MCP_SERVERS_CONFIG="~/.mcp.json"    # MCP server configuration
```

---

## Security Guidelines

### Credentials Management
- **NEVER** commit API keys, tokens, or secrets to git
- Always use environment variables (loaded with `python-dotenv`)
- Check `.gitignore` includes `.env`, `.env.local`
- Use CI/CD secrets for automated workflows

### Safe Operations
- **GitHub**: Confirm before pushing files or creating PRs to main/master
- **Credentials**: Always validate API keys exist before execution
- **File Operations**: Never use `subprocess` with shell=True
- **Input Validation**: Sanitize all user inputs before MCP calls

### Audit Trail
- Log API calls for debugging (use `--verbose` when available)
- Include timestamps in output
- Save important results to files, not just stdout

---

## Testing & Quality Assurance

### Pre-Commit Checks
```bash
# Verify all wrappers work
for wrapper in *.py; do
  echo "Testing $wrapper..."
  timeout 10 uv run "$wrapper" --help >/dev/null 2>&1 || echo "FAILED: $wrapper"
done
```

### Manual Testing
```bash
# Test each level of a wrapper
uv run WRAPPER.py --help                   # Level 1
uv run WRAPPER.py list                     # Level 2
uv run WRAPPER.py info FUNCTION            # Level 2
uv run WRAPPER.py example FUNCTION         # Level 3
uv run WRAPPER.py FUNCTION --help          # Level 4
```

### Continuous Validation
- Test with `uv run` (not direct python execution)
- Verify JSON output validity: `uv run WRAPPER.py FUNC | jq .`
- Check error handling: `uv run WRAPPER.py FUNC with_invalid_args`
- Validate timeouts: Commands should complete within 30-60 seconds

---

## Quick Find Commands

### Search Wrappers by Functionality
```bash
# Find scraping tools
rg -l "scrape|crawl" *.py

# Find GitHub-related wrappers
rg -l "github|github.py" *.py

# Find search/documentation tools
rg -l "search|documentation|wiki" *.py

# Find orchestration tools
rg -l "swarm|agent|workflow" *.py
```

### Inspect Wrapper Structure
```bash
# See available functions in any wrapper
uv run WRAPPER.py list

# Get detailed docs for a function
uv run WRAPPER.py info FUNCTION_NAME

# View function examples
uv run WRAPPER.py example FUNCTION_NAME
```

### Integration & Piping
```bash
# Pipe JSON output to jq for filtering
uv run firecrawl.py search "query" | jq '.results[] | {title, url}'

# Chain multiple tools
uv run github.py list_issues | jq '.[0].url' | xargs -I {} uv run github.py create_issue

# Save results to file
uv run repomix.py pack_codebase ~/project > codebase.txt
```

---

## Available Tools & Permissions

### All Tools Available
- ✅ Read any Python wrapper file
- ✅ Read/write documentation (README.md, INDEX.md, etc.)
- ✅ Execute `uv run` commands for testing
- ✅ Generate new wrappers (using generate-all-wrappers.py pattern)
- ✅ Modify docstrings and help text
- ✅ Update FUNCTION_INFO, FUNCTION_EXAMPLES
- ❌ Edit `.env` files (ask first)
- ❌ Modify PEP 723 script headers without testing
- ❌ Change core argument parsing logic without manual testing

### Execution Permissions
- ✅ Read wrappers and documentation
- ✅ Execute any wrapper with `uv run`
- ✅ Test with valid arguments (no destructive operations)
- ⚠️ GitHub operations require explicit confirmation (interactive prompts)
- ⚠️ File push operations require credential validation first
- ❌ Force operations (git force push) require explicit user approval

---

## Common Gotchas

### PEP 723 Headers
- Wrappers use `#!/usr/bin/env -S uv run` shebang
- Must include `# /// script` comment block
- Dependencies listed under `# dependencies = [...]`
- Always test after modifying script metadata

### JSON Output
- All functions return `json` format by default
- Use `--format text|table` for human-readable output
- Piping to `jq` requires valid JSON (test with `jq .` first)
- Empty results should return `{"success": false, "data": []}` not error

### API Keys & Credentials
- Must export environment variables BEFORE calling wrappers
- Missing keys cause silent failures (validate with `--verbose`)
- GitHub token needs repo, issues, pull_request scopes
- API rate limits vary by provider (check docs before automation)

### Network Timeouts
- Default 30-second timeout on all HTTP requests
- Some operations may need 60-second timeouts (adjust with `--timeout`)
- Retry logic not implemented (user responsibility for resilience)

### Output Formats
- `--format json` → Machine-readable (default)
- `--format text` → Human-readable (for terminals)
- `--format table` → ASCII tables (when applicable)
- Always pipe JSON through `jq` for validation

---

## Git Workflow

### Before Committing Changes
1. Test all modified wrappers: `uv run WRAPPER.py list`
2. Run documentation generation: `python generate-all-wrappers.py`
3. Update INDEX.md and README.md with new features
4. Commit with descriptive message: `git add . && git commit -m "feat: add new wrapper"`

### Commit Message Format
- `feat:` New wrapper or major feature
- `fix:` Bug fix or improvement
- `docs:` Documentation update
- `refactor:` Code reorganization
- `chore:` Script generation or tooling

### Branch Strategy
- Create feature branches from master: `git checkout -b feature/new-wrapper`
- Push when ready: `git push origin feature/new-wrapper`
- Create PR with full test results
- Squash on merge

---

## Special Files

### `generate-all-wrappers.py`
Master script that introspects all MCPs and generates wrappers. Used for:
- Adding new MCPs to the collection
- Regenerating wrappers from definitions
- Creating consistent structure across all tools

**Usage**: `uv run generate-all-wrappers.py`

### `.claude/settings.json`
Hooks and automation configuration for Claude Code:
- PreToolUse hooks (validation before execution)
- PostToolUse hooks (formatting and testing)
- Custom commands setup

### `.claude/commands/*.md`
Specialized slash commands for common workflows:
- `/scrape` - Quick webpage scraping
- `/search` - Multi-tool documentation search
- `/github-pr` - GitHub PR workflow
- `/analyze-repo` - Repository analysis

---

## MCP Server Configuration

### Currently Wrapped MCPs
1. **GitHub MCP** - `github.py`
2. **Ref-Tools MCP** - `ref.py`
3. **Firecrawl MCP** - `firecrawl.py`
4. **Crawl4AI MCP** - `crawl4ai.py`
5. **DeepWiki MCP** - `deepwiki.py`
6. **Semly MCP** - `semly.py`
7. **Sequential Thinking MCP** - `sequential-thinking.py`
8. **Claude Flow MCP** - `claude-flow.py`
9. **Flow Nexus MCP** - `flow-nexus-cli-wrapper.py`

### Adding New MCPs
1. Update MCP_DEFINITIONS in `generate-all-wrappers.py`
2. Run generator: `uv run generate-all-wrappers.py`
3. Test new wrapper: `uv run NEW_WRAPPER.py list`
4. Document in INDEX.md
5. Commit changes

---

## Performance & Optimization

### Token Efficiency (Goal: 60-70% savings)
- Use progressive disclosure to avoid information overload
- Show Level 1 help first (~30 tokens)
- User requests Level 2 when needed (~150 tokens)
- Total: ~180 tokens vs 500 for monolithic help

### Execution Speed
- Fastest: `--help` (cached strings, ~100ms)
- Quick: `list` (function index lookup, ~200ms)
- Medium: `info FUNC` (docstring formatting, ~300ms)
- Network: Any function call (30-60s depending on API)

### Caching Strategy
- Cache API responses for 5 minutes when applicable
- Store tool discovery results locally
- Reuse parsed JSON between calls

---

## Documentation Standards

### Wrapper Documentation
- **QUICK_HELP**: 10-15 lines, main functions listed
- **FUNCTION_INFO**: Full signature, parameters, returns
- **FUNCTION_EXAMPLES**: 2-3 real working examples
- **Full reference**: Argparse help text (auto-generated)

### Code Comments
- Minimal comments (code should be self-explanatory)
- Only explain non-obvious logic
- Document API endpoint URLs
- Note timeout limits and retry policies

### README & INDEX
- Keep synchronized with actual wrapper count
- Update after adding/removing tools
- Include usage examples
- Provide troubleshooting steps

---

## Support & Debugging

### Getting Help (4 Levels)
```bash
# Quick overview
uv run WRAPPER.py --help

# See all functions
uv run WRAPPER.py list

# Detailed function docs
uv run WRAPPER.py info FUNCTION_NAME

# Working examples
uv run WRAPPER.py example FUNCTION_NAME

# Complete reference
uv run WRAPPER.py FUNCTION_NAME --help
```

### Common Issues

#### MCP Connection Failed
```bash
# Verify MCPs are running
claude mcp list

# Test connectivity
uv run crawl4ai.py list
```

#### Missing API Key
```bash
# Export environment variable
export REQUIRED_KEY="value"

# Verify it's set
echo $REQUIRED_KEY

# Retry the operation
uv run WRAPPER.py FUNCTION
```

#### JSON Output Invalid
```bash
# Validate with jq
uv run WRAPPER.py FUNCTION | jq .

# Show raw output
uv run WRAPPER.py FUNCTION --format text
```

#### Timeout Issues
```bash
# Increase timeout
uv run WRAPPER.py FUNCTION --timeout 60

# Check network
timeout 5 ping google.com
```

---

## Version & Status

- **Generated**: 2025-02-01
- **Python Version**: 3.8+
- **Total Wrappers**: 12
- **Status**: ✅ All wrappers functional and tested
- **Last Updated**: 2025-02-01

---

## Next Steps

1. **Explore the wrappers**:
   ```bash
   cd ~/cli-wrappers
   uv run crawl4ai.py --help
   ```

2. **Learn the 4-level system**:
   ```bash
   uv run github.py list
   uv run github.py info create_pull_request
   uv run github.py example create_pull_request
   ```

3. **Try practical tasks**:
   ```bash
   uv run firecrawl.py scrape https://example.com | jq .
   uv run repomix.py pack_codebase ~/my-project
   ```

4. **Set up Claude Code integration** (optional):
   - Copy `.claude/settings.json` for automated hooks
   - Create custom commands in `.claude/commands/`
   - Configure MCP servers in `.mcp.json`

---

**This CLAUDE.md is the authoritative source for development guidelines.**
**For specific tool documentation, see README.md and INDEX.md.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brunogama) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
