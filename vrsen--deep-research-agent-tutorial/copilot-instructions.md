## deep-research-agent-tutorial

> **PURPOSE:** Quick reference for AI agents working with this Deep Research Agency Tutorial codebase.

# AI Agent Guidelines

**PURPOSE:** Quick reference for AI agents working with this Deep Research Agency Tutorial codebase.

## 🚨 CRITICAL PRINCIPLES

### Evidence-Based Development - NEVER ASSUME
- **ALWAYS examine actual code before making claims**
- **NEVER make assumptions about compatibility or architecture**
- **READ the actual imports and implementations**
- **TEST before declaring something broken**
- Agency Swarm v1.x inherits from OpenAI's Agents SDK - they are COMPATIBLE, not competing frameworks

### User Intent is Supreme
- **If you have even 1% doubt about user intent → STOP and ASK**
- User requirements override everything - no exceptions
- Never apply changes that are misaligned with user needs

### Documentation Separation
- **README.md = USERS** - Setup, usage, features (no AI agent instructions)
- **AGENTS.md = AI AGENTS** - Internal guidance, operational principles
- Never mix internal development details into user-facing documentation

### Cookbook Alignment First
- Always replicate existing patterns from `original_cookbook.ipynb` before innovating
- Preserve exact prompt text and model names from cookbook
- Challenge unused code: if not used + not in cookbook = delete
- Honor cookbook design decisions - we're building on top of it

## Project Context

**Deep Research Agency Tutorial** - Beginner-friendly Agency Swarm v1.x implementation of OpenAI's Deep Research cookbook.

**Two Patterns:**
1. **BasicResearchAgency** - Single agent with web search (simplest)
2. **DeepResearchAgency** - Multi-agent handoffs: Triage → [Clarifying] → Instruction → Research

## Core Values

- **Beginner-Friendly** - Non-coders can understand the structure
- **No Over-Engineering** - Simple, direct implementations
- **Self-Contained** - Each agency file works independently
- **Flat Structure** - Avoid nested complexity where possible

## Development Standards

### File Size Discipline
- Keep individual files under 200 lines max
- Long instructions go in separate `instructions.md` files

### Progress Validation
- Use `git status` and `git diff` to review ALL changes before committing
- Review your own progress - assume you made mistakes
- Question every function: is it used? is it in cookbook?

### Agency Structure
```
AgencyName/
├── agency.py              # Main agency with demos (imports individual agents)
├── utils.py              # Utilities (if needed)
└── AgentName/            # Individual agents in folders
    ├── files/            # Files for OpenAI upload (if needed)
    ├── schemas/          # OpenAPI schemas converted to tools (if needed)
    ├── tools/            # Tools imported by default (if needed)
    ├── __init__.py       # Package initialization
    ├── AgentName.py      # Main agent class file
    └── instructions.md   # Instruction document for the agent
```


## Quick Checklist
- [ ] Does this align with cookbook patterns?
- [ ] Is user intent clear?
- [ ] Are changes minimal and necessary?
- [ ] Will beginners understand this?
- [ ] Files under 200 lines?

---
> Source: [VRSEN/deep-research-agent-tutorial](https://github.com/VRSEN/deep-research-agent-tutorial) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
