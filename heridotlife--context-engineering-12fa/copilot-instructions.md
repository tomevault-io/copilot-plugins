## context-engineering-12fa

> > You are an agentic AI using the **12 Factor Agents** specification + Context Engineering extensions.

# Gemini

> You are an agentic AI using the **12 Factor Agents** specification + Context Engineering extensions.
> Load and follow all operational rules from `core/AGENTIC_GUIDE.md`.
> Treat that file as your system instruction source.

---

## Quick Start

1. **Load master spec**: `core/AGENTIC_GUIDE.md`
2. **Select persona**: Use `personas/PERSONA_CATALOG.md`
3. **Initialize session**: Create `SESSION_LOG.md` from `templates/SESSION_LOG.md`
4. **Execute task**: Follow Execution Footer protocol

---

## What is This?

This project implements [HumanLayer's 12 Factor Agents](https://github.com/humanlayer/12-factor-agents) with extensions:

- **12 Factor Agents** (HumanLayer) - Production-ready LLM application principles
- **Context Engineering** (Factor 3) - Active context window management
- **Rich Persona System** (Factor 10 Extended) - 88 specialized agent personas
- **Multi-Agent Orchestration** (Factor 10) - Supervisor pattern with context handoffs

---

## File Structure

```
core/AGENTIC_GUIDE.md              # Master specification (start here)
├── 12-factor-agents/              # Original 12FA reference
├── implementations/               # Our 12FA implementations
│   ├── factor-03-context-window/  # Context engineering
│   ├── factor-05-state-management/# SESSION_LOG system
│   ├── factor-09-error-handling/  # Error compaction
│   └── factor-10-focused-agents/  # Multi-agent patterns
├── personas/                      # 88 specialized personas
│   ├── PERSONA_CATALOG.md        # Selection guide
│   └── agents/                    # Persona files
├── templates/                     # Reusable templates
│   └── SESSION_LOG.md            # Session scratchpad
└── schemas/                       # Machine-readable specs
```

---

## Binding Contract

This file exists to bind the Gemini AI to the shared behavior contract defined in `core/AGENTIC_GUIDE.md`.

**For other platforms:**
- Claude: `CLAUDE.md` → Load `core/AGENTIC_GUIDE.md`
- Copilot: `COPILOT.md` → Load `core/AGENTIC_GUIDE.md`

---

**spec_version**: 0.4.0
**base_framework**: HumanLayer 12 Factor Agents
**license**: MIT

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/heridotlife)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/heridotlife)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
