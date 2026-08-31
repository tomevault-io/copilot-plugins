## media-rag-pipeline

> This file provides guidance to Claude Code when working with this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

## Tech Stack

- **Backend**: FastAPI (Python 3.12)

## Quick Commands

## Dependencies

### Adding Dependencies
```bash
# Backend
cd backend && uv add <package>  # Auto-updates uv.lock
```

### Lock File
- `uv.lock` is auto-verified by pre-commit hooks
- Run `uv lock` after editing pyproject.toml manually

## Critical Rules

### Git Workflow
- **NEVER push directly to master** - always use feature branches
- **ALWAYS create a GitHub issue first** - every PR must be linked to an issue
  - Multiple PRs can reference the same issue
  - Use `Closes #<issue-number>` in PR description to auto-close on merge
- Create PR for all changes


## Documentation Guidelines

When creating or updating documentation:

1. **Table of Contents**: Every document must have a TOC at the top with anchor links
2. **Mermaid Diagrams**: Use [Mermaid.js](https://mermaid.js.org/) for all diagrams (no ASCII art)
3. **Cross-References**: Link to related documents where applicable
4. **Code Examples**: Include runnable examples where possible
5. **Keep Updated**: Update docs when code changes

Documentation index: See [docs/README.md](./docs/README.md)

## Additional Resources

---
> Source: [dremdem/media_rag_pipeline](https://github.com/dremdem/media_rag_pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
