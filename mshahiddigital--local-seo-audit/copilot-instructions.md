## local-seo-audit

> The Local SEO Audit System uses 6 specialized Claude Code agents for parallel audit execution. Agents are defined in `.claude/agents/` and auto-delegated based on task matching or invoked explicitly via `@agent-name`.

# Agent Team — Local SEO Audit System

The Local SEO Audit System uses 6 specialized Claude Code agents for parallel audit execution. Agents are defined in `.claude/agents/` and auto-delegated based on task matching or invoked explicitly via `@agent-name`.

## Architecture

```
┌──────────────────────────────────────────────────────────┐
│                   ORCHESTRATOR (Claude)                    │
│                                                           │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐        │
│  │  Technical   │ │   Content   │ │   Keyword   │        │
│  │  Analyst     │ │   Analyst   │ │   Analyst   │        │
│  └─────────────┘ └─────────────┘ └─────────────┘        │
│                                                           │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐        │
│  │ Local SEO   │ │AI Visibility│ │  Off-Page   │        │
│  │  Analyst    │ │  Analyst    │ │   Analyst   │        │
│  └─────────────┘ └─────────────┘ └─────────────┘        │
└──────────────────────────────────────────────────────────┘
```

## Agent Definitions

All agent files live in `.claude/agents/` with YAML frontmatter (name, description, tools, model, maxTurns).

### 1. Technical Analyst — `@technical-analyst`
- **File**: `.claude/agents/technical-analyst.md`
- **Model**: sonnet | **Max Turns**: 50
- **Phases**: 2 (Technical SEO), 10 (Speed/CWV), 19 (Accessibility)
- **Tools**: Read, Write, Edit, Glob, Grep, Bash, WebFetch, WebSearch
- **Output**: `technical-findings.md`, `speed-findings.md`, `accessibility-findings.md`

### 2. Content Analyst — `@content-analyst`
- **File**: `.claude/agents/content-analyst.md`
- **Model**: sonnet | **Max Turns**: 50
- **Phases**: 4 (Content Audit), 5 (Content Gaps), 7 (Topical Gaps), 8 (Topical Authority)
- **Tools**: Read, Write, Edit, Glob, Grep, Bash, WebFetch, WebSearch
- **Output**: `content-inventory.md`, `content-gaps.md`, `topical-gaps.md`, `topical-authority.md`

### 3. Keyword Analyst — `@keyword-analyst`
- **File**: `.claude/agents/keyword-analyst.md`
- **Model**: sonnet | **Max Turns**: 50
- **Phases**: 3 (On-Page SEO), 6 (Keyword Gaps), 9 (Entity Audit)
- **Tools**: Read, Write, Edit, Glob, Grep, Bash, WebFetch, WebSearch
- **Output**: `onpage-findings.md`, `keyword-gaps.md`, `entity-findings.md`

### 4. Local SEO Analyst — `@local-seo-analyst`
- **File**: `.claude/agents/local-seo-analyst.md`
- **Model**: sonnet | **Max Turns**: 50
- **Phases**: 11 (Local SEO & GBP), 15 (Reputation & Reviews), 21 (Multi-Location)
- **Tools**: Read, Write, Edit, Glob, Grep, Bash, WebFetch, WebSearch
- **Output**: `local-findings.md`, `reputation-findings.md`, `multi-location-findings.md`

### 5. AI Visibility Analyst — `@ai-visibility-analyst`
- **File**: `.claude/agents/ai-visibility-analyst.md`
- **Model**: sonnet | **Max Turns**: 50
- **Phases**: 14 (AI SEO & GEO), 16 (Brand SERP), 18 (Voice Search)
- **Tools**: Read, Write, Edit, Glob, Grep, Bash, WebFetch, WebSearch
- **Output**: `ai-seo-findings.md`, `brand-serp-findings.md`, `voice-findings.md`

### 6. Off-Page Analyst — `@offpage-analyst`
- **File**: `.claude/agents/offpage-analyst.md`
- **Model**: sonnet | **Max Turns**: 50
- **Phases**: 12 (Backlinks), 13 (Social Media), 17 (UX & CRO)
- **Tools**: Read, Write, Edit, Glob, Grep, Bash, WebFetch, WebSearch
- **Output**: `backlink-findings.md`, `social-findings.md`, `cro-findings.md`

## Wave Execution with Agents

During a full 21-phase audit, agents are spawned in parallel per wave:

| Wave | Agents Spawned | Phases Executed |
|---|---|---|
| 2 — Technical | `@technical-analyst`, `@keyword-analyst`, `@content-analyst` | 2, 3, 4, 10 |
| 3 — Gap Analysis | `@keyword-analyst`, `@content-analyst` | 5, 6, 7 |
| 4 — Local + Authority | `@keyword-analyst`, `@local-seo-analyst`, `@offpage-analyst` | 9, 11, 12, 13 |
| 5 — Advanced | `@content-analyst`, `@ai-visibility-analyst`, `@local-seo-analyst`, `@offpage-analyst`, `@technical-analyst` | 8, 14, 15, 16, 17, 18, 19, 21 |

Waves 1 (Foundation), 6 (Scoring), and 7 (Final Output) run sequentially without agents.

## Inter-Agent Data Flow

All agents read from and write to `{AUDIT_DIR}/`. The orchestrator ensures wave ordering so downstream agents can read upstream outputs.

| Producer Agent | Output File | Consumer Agents |
|---|---|---|
| Orchestrator | `intake-data.md`, `competitor-profiles.md` | All agents |
| `@technical-analyst` | `technical-findings.md` | `@ai-visibility-analyst`, scoring |
| `@content-analyst` | `content-inventory.md` | `@keyword-analyst`, `@ai-visibility-analyst` |
| `@local-seo-analyst` | `local-findings.md` | `@ai-visibility-analyst`, scoring |
| `@offpage-analyst` | `backlink-findings.md` | `@ai-visibility-analyst`, scoring |

## Invocation

Agents are invoked via the Agent tool during full audit execution:
```
Agent tool → subagent_type not needed (Claude Code auto-delegates based on .claude/agents/ descriptions)
```

Or explicitly referenced in prompts: "Run Phase 2 Technical SEO audit for {business} at {url}" → auto-delegates to `@technical-analyst` based on description match.

---
> Source: [mshahiddigital/local-seo-audit](https://github.com/mshahiddigital/local-seo-audit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
