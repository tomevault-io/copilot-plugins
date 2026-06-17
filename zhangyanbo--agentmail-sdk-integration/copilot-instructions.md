## agentmail-sdk-integration

> Integrate AgentMail into Python or Node.js applications with accurate API/SDK interfaces, input-output types, and data structures. Use when implementing inbox/message/thread/draft/domain/webhook/websocket/metrics/pod workflows, generating AgentMail client code, reviewing AgentMail integration correctness, or mapping business requirements to concrete AgentMail endpoints.


# AgentMail SDK Integration

## Overview

Use this skill to turn AgentMail requirements into production-ready Python or Node implementations.
Prefer generated references for endpoint contracts and types instead of guessing SDK signatures.

## Workflow

1. Identify language target (`Python` or `Node`) and integration mode (`SDK`, `REST`, `Webhook`, `WebSocket`).
2. Load `references/documentation-guide.md` for architecture and operational guidance.
3. Load language file:
   - Python: `references/python-sdk.md`
   - Node: `references/node-sdk.md`
4. Validate endpoint contract against `references/endpoint-matrix.md`.
5. Resolve field-level structure from `references/data-structures.md`.
6. Implement with idempotency (`client_id`) for create operations and signature verification for webhooks.

## Language Selection Rules

- Use `references/python-sdk.md` when generating Python code with `AgentMail` / `AsyncAgentMail`.
- Use `references/node-sdk.md` when generating TypeScript/Node code with `AgentMailClient`.
- Use `references/endpoint-matrix.md` for cross-language parity checks and unsupported SDK convenience paths.
- Use `references/documentation-guide.md` when designing flow-level behavior (send/receive loop, webhook lifecycle, deliverability, IMAP/SMTP fallback).

## Implementation Rules

1. Always include both `text` and `html` when sending messages, unless requirements explicitly constrain format.
2. Always use deterministic `client_id` for create operations (`inbox`, `webhook`, `pod`, etc.).
3. Always verify webhook signatures (`svix-id`, `svix-timestamp`, `svix-signature`) before trusting payload.
4. Always model reply/forward flows with `message_id` and `thread_id` to preserve conversation continuity.
5. Always handle `from_` compatibility in payload parsing (`from_` vs `from`).
6. Always treat list management APIs (`/lists`, `/pods/{pod_id}/lists`) as REST-first if SDK helper methods are not available.
7. Always confirm filtering/pagination inputs (`limit`, `page_token`, `before`, `after`, `labels`, `ascending`, `include_spam`, `include_blocked`) from reference before coding.

## Output Expectations

When asked to generate code, produce:

1. Runnable snippet(s) for the requested language.
2. Explicit method/interface used and parameter mapping.
3. Input/output type annotations aligned with references.
4. Notes for error handling, retries, and idempotency.
5. Event handling path if real-time behavior is involved (webhook or websocket).

## Refresh References

Run this script to re-sync docs and regenerate references from `docs.agentmail.to`:

```bash
python scripts/sync_agentmail_references.py --skill-dir .
```

## References

- `references/documentation-guide.md`: product-level behavior and best practices.
- `references/python-sdk.md`: Python interface map, IO types, and examples.
- `references/node-sdk.md`: Node interface map, IO types, and examples.
- `references/endpoint-matrix.md`: full endpoint contract matrix across protocols.
- `references/data-structures.md`: schema field inventory.

---
> Source: [Zhangyanbo/agentmail-sdk-integration](https://github.com/Zhangyanbo/agentmail-sdk-integration) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
