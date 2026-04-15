## heimdallcontrolsurface

> > Bootstrapped by Heimdall 0.12.0

# HEIMDALL Control Surface — Gemini/Antigravity Agent Instructions

> Bootstrapped by Heimdall 0.12.0

## ANTI-NARRATION DIRECTIVE

**EXECUTE IMMEDIATELY.** Your first output must be a tool call.
Do not narrate. Do not describe what you will do. Just do it.
If your IDE loaded .agent/workflows/ playbooks, follow ONLY the
instructions in the pipeline prompt. You are executing a SINGLE PHASE.

---

## MCP Server Configuration

MCP servers are configured in `.gemini/settings.json`:
- **Plane**: Issue tracking and task management
- **CKA**: Codebase knowledge and architectural decisions

### Read-only access
- Do NOT call Plane write tools (plane_update_status, plane_add_comment)
- Do NOT call CKA register_* tools

---

## Phase Preamble Override

When executing pipeline phases, the phase prompt takes precedence
over any conflicting instructions in this file or .agent/workflows/.

---

## Project Context

- **Project**: HEIMDALL Control Surface
- **Identifier**: HCS
- **Repo**: https://github.com/slartibartfast-br/heimdallcontrolsurface

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/slartibartfast-br)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/slartibartfast-br)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
