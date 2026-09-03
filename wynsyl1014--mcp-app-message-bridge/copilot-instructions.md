## mcp-app-message-bridge

> This repository is written primarily for an AI/agent that has been asked to adapt the pattern into a real project.

# AGENTS.md

This repository is written primarily for an AI/agent that has been asked to adapt the pattern into a real project.

## Mission

Build a bridge that lets an external system create a real user-role turn inside an MCP Apps-compatible conversation while preserving a clean split between:

- authoritative backend state,
- model-visible but chat-invisible context,
- visible conversation text,
- widget-only metadata.

Do **not** turn this repository into a universal wake scheduler or memory framework. Those are owner-specific policies.

## Protocol invariants

Preserve these unless the owner explicitly chooses a different design:

1. **Backend is authoritative.** The widget is a door, not the source of truth.
2. **Reserve before dispatch.** At most one event should own the door at a time.
3. **Context first, message second.** Call `ui/update-model-context` before `ui/message` for the same event.
4. **Remember overwrite semantics.** Repeated context updates replace the previous View-provided model context; do not stage many events in context at once.
5. **Do not confuse `_meta` with model context.** Tool-result `_meta` is widget-only. Use `structuredContent`/`content` when the model should see tool results, and `ui/update-model-context` when the View should add model context without a visible message.
6. **Route explicitly when routing matters.** If correct handling depends on a specific custom MCP/plugin, include an explicit route hint such as `@{{PLUGIN_HANDLE}}` in the visible trigger. This is a reliability heuristic, not an MCP protocol requirement.
7. **Do not dispatch merely because the UI mounted.** Mounting/opening the listener should be side-effect-light. Start polling/listening deliberately.
8. **Make dispatch idempotent.** Every external event needs a stable `eventId`; retries must not create duplicate turns.
9. **Release failed reservations.** If context or message delivery fails, return the event to a retryable state.
10. **Version UI resource URIs on breaking UI changes.** Hosts may cache UI resources.
11. **Do not put secrets in `_meta`, `structuredContent`, visible prompts, or browser code.** Authorization remains server-side.
12. **Feature-detect host capabilities.** Prefer standard MCP Apps methods over product-name branching.

## Required owner-specific inputs

Before finishing an integration, resolve these values from the owner/project:

```text
{{PLUGIN_HANDLE}}          # Name/handle that should route the turn to the correct MCP/plugin
{{BACKEND_ADAPTER}}        # DB/API implementation for reserve/deliver/release
{{EXTERNAL_CHANNEL}}       # PWA, Telegram, Discord, web app, etc.
{{MODEL_CONTEXT_BUILDER}}  # Hidden-to-chat context for each event
{{VISIBLE_MESSAGE_BUILDER}}# What appears as the user-role turn
{{OUTBOUND_ROUTE}}         # Optional tool/path for AI replies back to the external channel
{{WAKE_POLICY}}            # Optional; fully project-specific
{{SESSION_BINDING}}        # How events are scoped to the intended conversation/listener
```

If these are genuinely ambiguous and cannot be inferred from the codebase, ask the owner. Do not invent personal wake behavior.

## Files to read

- `AI_IMPLEMENTATION_GUIDE.md`: adaptation procedure.
- `ARCHITECTURE.md`: boundaries and data flow.
- `docs/context-channels.md`: visibility semantics.
- `docs/event-lifecycle.md`: minimum reliable state machine.
- `docs/pitfalls.md`: failures observed in real integrations.
- `reference/src/`: copyable TypeScript skeleton.

## What not to copy blindly

The examples contain sample route text and sample wake text only to demonstrate where policy belongs. Replace them. A wake prompt is part of the owner's relationship/product design, not part of the bridge protocol.

---
> Source: [wynsyl1014/mcp-app-message-bridge](https://github.com/wynsyl1014/mcp-app-message-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
