## training-harnest

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Training Harness transforms AI-assisted development sessions into structured case studies and syncs them to NotebookLM for AI-powered learning. It provides Claude Code commands that document conversations as pedagogical materials.

**Core Pipeline:**
```
Development Session → /document-session → Case Study (Markdown)
                                              ↓
                                      /sync-to-notebook → NotebookLM Notebook
```

## Project Structure

```
.claude/
├── commands/           # Slash commands (document-session, sync-to-notebook, etc.)
├── agents/             # Subagent definitions (notebook-assembler)
└── skills/             # Natural language triggered skills
    └── dev-process-docs/  # Session documentation skill (legacy)

templates/
├── pedagogical/        # Case study templates
├── onboarding/         # NotebookLM learner guides
└── studio-prompts/     # NotebookLM Studio prompt templates (variants, artifacts, styles)

ai_docs/
├── features/           # Feature specifications (FT-XX-*.md)
├── learnings/          # Generated case studies (development-process-*.md)
├── prompts/            # Generated Studio prompts ({notebook}-{artifact}-prompt.md)
├── refs/               # External reference docs for Claude Code (MCP tools, Claude features)
├── architecture.md     # System design
└── roadmap.md          # Development plan

docs/                   # Project documentation (test results, guides, specs)
config/                 # MCP configuration and setup guides
```

### Documentation Locations

| Location | Purpose |
| -------- | ------- |
| `ai_docs/refs/` | External reference docs to help Claude Code use tools (MCP patterns, Claude Code features, third-party APIs) |
| `docs/` | Project documentation produced by or for Training Harness (test results, implementation guides) |
| `docs/notebooklm-studio-note.md` | Copy/paste content for NotebookLM onboarding (manual Studio note) |

## Key Commands

| Command | Purpose |
| ------- | ------- |
| `/implement-feature FT-XX-*.md` | Implement a feature with branch creation and context loading |
| `/commit-and-push` | Commit staged changes and push |
| `/review-code` | Pre-PR code review |

**Training Harness Commands (v0.1):**

| Command | Purpose |
| ------- | ------- |
| `/document-session` | Transform conversation into case study |
| `/sync-to-notebook` | Upload case studies to NotebookLM |
| `/document-session-to-notebook` | Combined documentation + sync |
| `/generate-studio-prompt` | Generate steering prompts for NotebookLM Studio artifacts |

## Development Workflow

### Implementing Features

1. Features are specified in `ai_docs/features/FT-XX-*.md`
2. Use `/implement-feature FT-XX-feature-name.md` to start
3. The command creates a branch, loads context from roadmap/architecture, and guides implementation
4. Update `ai_docs/features/INDEX.md` when completing features

### Feature Document Structure

Each feature spec contains:
- Description and acceptance criteria
- Technical dependencies (required prior features)
- Implementation steps
- Test plan (isolation and end-to-end)

### Architecture Decisions

Review `ai_docs/architecture.md` before making structural changes. Key decisions:
- **Commands over skills** for explicit invocation
- **Decoupled pipeline** (document and sync are separate operations)
- **One notebook per project** for cross-session learning
- **Incremental sync** via `.notebook-sync.json` tracking
- **No onboarding as source** — Sources are indexed for RAG; onboarding is a manual Studio note

## NotebookLM MCP Integration

The project uses `notebooklm-mcp` for NotebookLM operations.

### Tool Naming Convention

Use fully qualified names: `notebooklm-mcp:tool_name`

Common tools:
- `notebooklm-mcp:notebook_create`
- `notebooklm-mcp:notebook_add_text`
- `notebooklm-mcp:notebook_list`
- `notebooklm-mcp:notebook_query`

See `ai_docs/refs/mcp-tool-patterns.md` for complete reference.

### Authentication

If MCP operations fail with auth errors:
```bash
notebooklm-mcp-auth
```

## File Conventions

| Artifact | Pattern |
| -------- | ------- |
| Feature specs | `ai_docs/features/FT-XX-feature-name.md` |
| Case studies | `ai_docs/learnings/development-process-{N}.md` |
| Sync tracking | `ai_docs/learnings/.notebook-sync.json` |
| Studio prompts | `ai_docs/prompts/{notebook}-{artifact}-prompt.md` |

## Testing

No build step required. Changes to `.claude/` artifacts take effect immediately in Claude Code CLI.

For MCP testing, see `docs/mcp-tool-test-results.md` for verified tool behavior and response formats.

---
> Source: [hopchouinard/training-harnest](https://github.com/hopchouinard/training-harnest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
