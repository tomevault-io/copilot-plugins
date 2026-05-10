## qwen-code-oai-proxy

> must first read the ./docs/README.md To undertsadn the goal of the porject . the qwen-code cli dir is ./qwen-code . its being added to git ignore to not push in github.

# Qwen Project Context for AI Interactions

## Project Overview
 must first read the ./docs/README.md To undertsadn the goal of the porject . the qwen-code cli dir is ./qwen-code . its being added to git ignore to not push in github. 
This project is a Qwen-to-OpenAI API proxy server that allows users to interact with Qwen's API using OpenAI-compatible endpoints. Key features include:
- Multi-account support with automatic rotation to handle request quotas
- QR code authentication flow
- Streaming response support
- Token counting and management
- Detailed debug logging capabilities

## Documentation Practices
- All new features should be documented in the `docs/` folder with dedicated markdown files
- Update existing documentation when modifying existing features
- Key user-facing changes should also be reflected in the main `README.md`
- Architecture and implementation details should be captured in `docs/codebase_analysis.md`

## Changelog Practices
- Maintain a high-level feature changelog in the commit messages
- For detailed technical changes, run these commands to analyze recent modifications:

```bash
# Current repository status
git status --porcelain

# Recent commits (last 10)
git log --oneline -10

# Detailed recent changes
git log --since="1 week ago" --pretty=format:"%h - %an, %ar : %s" --stat

# Recently changed files
git diff HEAD~5 --name-only | head -20

# New files added
git diff --name-status HEAD~10 | grep "^A" | head -15

# Deleted files
git diff --name-status HEAD~10 | grep "^D" | head -10

# Modified core files
git diff --name-status HEAD~10 | grep "^M" | grep -E "(package\.json|README|config|main|index|app)" | head -10
```

## Recent Project State Notes
- Documentation is generally well-maintained with dedicated files for major features
- Recent development has focused on multi-account management and streaming improvements
- The project has a good test suite with both simple and complex test scripts
- Configuration is handled through environment variables with example configurations provided
- Authentication has been enhanced with QR code support and account management commands

## Project Structure
- `src/` - Main source code for the proxy server
- `docs/` - Detailed documentation for features
- `authenticate.js` - CLI tool for account management
- `src/qwen/` - Qwen-specific API and authentication logic
- `src/utils/` - Utility functions like logging and token counting
- `qwen-code/`  this is the dir for the qwen-code cli souce code . read this when user asks to look into the source code of qwen cli . 

---
> Source: [aptdnfapt/qwen-code-oai-proxy](https://github.com/aptdnfapt/qwen-code-oai-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
