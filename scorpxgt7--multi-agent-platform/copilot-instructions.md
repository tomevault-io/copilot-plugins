## multi-agent-platform

> >


# Nexus Multi-Agent Platform

This project is a React-based multi-agent orchestration platform. The canonical
base is `nexus-agent-platform.jsx`. Treat `multi_agent_automation_platform.jsx`
as an alternate UI reference, not the execution source of truth.

## Phase 1 baseline

- Keep the platform runnable without any external model provider.
- Use a local orchestration runtime so the UI, state flow, and team handoff
  model can be tested before backend integration.
- Remove provider-specific assumptions from the frontend.

## Architecture

The platform uses a 3-tier hierarchy:

1. Manager
2. Supervisor
3. Specialist agents

The pipeline is sequential by default:

1. Manager defines the operational plan
2. Supervisor converts the plan into agent directives
3. Agents execute one by one with prior outputs as context
4. Supervisor synthesizes the work
5. Manager produces the final report

## Real-use direction

When moving beyond Phase 1:

- Add a backend provider adapter instead of calling any model API directly from
  the browser.
- Preserve agent editing, logs, statuses, and final-output rendering as core
  product features.
- Add approval checkpoints before external execution or delivery.
- Add saved runs, presets, and export support.

## Working rules

- Read `nexus-agent-platform.jsx` before modifying execution logic.
- Keep orchestration logic separate from presentation concerns when refactoring.
- Prefer one canonical platform over two drifting implementations.
- If a change is only visual, avoid changing pipeline behavior at the same time.
- Use ASCII by default and avoid corrupted punctuation or copied glyphs.

## File roles

- `nexus-agent-platform.jsx`
  Canonical execution shell and Phase 1 runtime.
- `multi_agent_automation_platform.jsx`
  UI reference for future layout ideas and workflow visualization patterns.

## Phase 2 target

Phase 2 should introduce:

1. backend-backed provider adapters
2. persisted sessions and presets
3. per-stage approvals and reruns
4. stronger domain presets
5. modular code structure

---
> Source: [scorpxgt7/Multi-Agent-Platform](https://github.com/scorpxgt7/Multi-Agent-Platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
