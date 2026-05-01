## shrimp2

> After **any app change** (UI, API, runtime, storage, or subagent logic), you must validate the chat experience by exercising the chat APIs end-to-end before considering the task done.

# AGENTS

## Mandatory Post-Change Chat API Validation
After **any app change** (UI, API, runtime, storage, or subagent logic), you must validate the chat experience by exercising the chat APIs end-to-end before considering the task done.

Scope rule:
1. Run message/API validation only when the change can affect chat/runtime behavior (API, orchestration, model interaction, context, storage, subagents, streaming).
2. Do **not** run message-based checks for changes that do not need them (for example, purely visual/UI-only adjustments with no chat logic impact).

Prompt quality rule:
1. Test messages must be realistic and behavior-oriented, aimed at validating the feature that was changed.
2. Avoid meaningless probe strings (for example `CODEX-1314124124`) unless that exact pattern is required by the feature under test.
3. Prefer prompts that require the model/subagent to actually perform the new or modified behavior so the test verifies real functionality.

Required checks:
1. Send a user message via `POST /api/omni/messages` and confirm success.
2. Read recent timeline via `GET /api/omni/messages` and confirm the new message appears.
3. Trigger and observe subagent flow:
   - create a run (`POST /api/subagents/trigger`),
   - verify run state via `GET /api/subagents/runs`,
   - if `devMode` is enabled, inspect internal events via `GET /api/subagents/:id/events`.
4. Verify the app receives updates from `GET /api/omni/stream` (SSE) for message/subagent updates.

If any check fails, report the failure clearly and do not mark the change complete.

---
> Source: [j0taaa/shrimp2](https://github.com/j0taaa/shrimp2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
