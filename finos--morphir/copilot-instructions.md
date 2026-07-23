## morphir

> This file provides instructions for Claude Code and other AI assistants working on the Morphir Go project.

# Claude Code Agent Instructions

This file provides instructions for Claude Code and other AI assistants working on the Morphir Go project.

For complete development guidelines, coding standards, and contribution practices, please refer to [AGENTS.md](AGENTS.md).

## ⚠️ CRITICAL: No AI Co-Authors in Commits

**DO NOT add Claude or any AI assistant as a commit co-author under any circumstances.**

This project uses EasyCLA (Easy Contributor License Agreement) for FINOS compliance. Adding AI co-authors:
- ❌ Breaks the CLA verification process
- ❌ Blocks pull requests from being merged
- ❌ Violates FINOS contribution requirements

**NEVER include lines like:**
```
Co-Authored-By: Claude <noreply@anthropic.com>
🤖 Generated with Claude Code
```

Only the human developer should be listed as the author/co-author. See [AGENTS.md](AGENTS.md#commit-authorship-for-ai-assistants) for details.

## Quick Reference

- **Functional Programming First**: This codebase follows functional programming principles
- **TDD/BDD**: Write tests before implementation
- **No AI Co-Authors**: See critical warning above - this breaks EasyCLA
- **Morphir Alignment**: Maintain compatibility with Morphir IR specification

See [AGENTS.md](AGENTS.md) for detailed guidelines.

---
> Source: [finos/morphir](https://github.com/finos/morphir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
