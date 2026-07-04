## code-brain

> `.ai/` owns repo-local agent context, memory, generated metadata, trust, and runtime tooling.

# Code Brain Agent Contract

`.ai/` owns repo-local agent context, memory, generated metadata, trust, and runtime tooling.

## Response

- Match the user's language unless they request otherwise.
- Keep self-initiated progress/output under 10 words.
- Keep answers concise by default.
- Expand beyond these defaults only for explicit detail, severe error/risk, or required question.
- No progress narration or next-step outro; continue until done, blocked, or approval is required.

## Branches

- Work on `develop` by default.
- Never commit or push `main` unless the user explicitly requests main.
- Do not create branches unless asked.
- After a requested main merge, return to `develop`.

## Loop

1. Understand the request.
2. Inspect local rules and current git state.
3. Locate code with Code Brain before broad shell search.
4. Before editing existing files, read exact target slices with hashline.
5. Preserve unrelated user changes.
6. Make the smallest coherent change.
7. Verify before claiming success.
8. Record durable decisions/todos when useful.

## Planning

- Plan first only when scope or impact is unclear.
- Single-file fixes and explicit user-scoped work can proceed directly.
- If new risk appears, stop only when approval or a decision is required.

## Search Routing

- Code discovery: MCP `code_query` first, `context_pack` next.
- Call graph: `code_graph_callers`, `code_graph_callees`, `code_graph_symbol`.
- Before editing existing files, read exact target slices: `code_read_hashline`; CLI fallback: `.ai/bin/ai code read-hashline PATH --start START --end END`.
- Long output or broad search: MCP `sandbox_execute` or `.ai/bin/ai exec run -- ...`.
- Direct broad shell `grep -r`, `rg .`, `find`, `tree`, `ack`, `ag`, `git grep` is last resort only when Code Brain is unavailable/stale.

## Memory

- `record_decision`: durable user or architecture decisions, one concise item.
- `record_todo`: deferred work or explicit future tasks.
- `close_todo`: completed/obsolete todos.
- `append_session_note`: important milestones.
- Working tree beats memory when they disagree.

## Subagents

- Use read-only agents for broad discovery.
- Use plan/review agents for independent analysis.
- Keep edit ownership non-overlapping.
- Reserve multi-agent fan-out (workflows) for big items: architecture, security, broad audits, large/risky refactors. Keep small/local changes solo, verified by tests + direct check. Estimate cost and get approval before a large fan-out; do not auto-fan-out small work.

## Security

- Do not read, edit, print, or commit real secrets.
- No auth, billing, destructive DB, deployment, package, or prod-secret changes without explicit approval.
- CI is read-only; write commands must be rejected before worker contact.
- Hooks/MCP hot paths must not call the network.
- Tracked source must not contain plaintext secrets.
- Redact MCP, diagnostics, and external-channel output.

## Generated Files

- Scratch: `.ai/tmp/`.
- Durable outputs: `.ai/outputs/`.
- Do not hand-edit owned/derived files unless the task requires it: `.ai/generated/manifest.json`, `.ai/cache/**`, vendor/build/lock files.

## Verify

- Use the closest test/check first.
- Select tests by changed files and likely impact; skip unrelated suites.
- Run full `make test` only when broad shared contracts changed or the user asks.
- Typical root checks: `make lint`, targeted pytest, `make doctor`.
- Never report success without verification.

---
> Source: [ezBuilder/code-brain](https://github.com/ezBuilder/code-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
