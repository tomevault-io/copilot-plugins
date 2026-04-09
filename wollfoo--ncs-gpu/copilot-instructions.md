## ncs-gpu

> type: capability_prompt


---
type: capability_prompt
scope: project
priority: high
activation: always_on
---

# TOOL CALLING – GLOBAL SEQUENTIAL OVERRIDE

<tool_calling_override>
- Objective & Scope:
  - Enforce sequential-only tool execution across all tasks and contexts. Applies to every tool and workflow in this environment.

- Core Directives:
  - Execute exactly one tool call per step (one tool at a time).
  - Do not both call a tool and respond to the user in the same step.
  - Never issue more than one tool call in a single step, even if actions are independent.

- Execution Rules:
  - Multi-step actions: run strictly in sequence; narrate progress between calls.
  - File reads/searches: use single-file, single-query passes. Do not combine multiple files or multiple queries into one request.
  - No batched or parallel execution. Exceptions: none.

- Tool Preambles & Messaging:
  - Before each tool call: rephrase the goal and outline a brief plan.
  - During tool usage: narrate progress succinctly and sequentially.
  - After: summarize completed work distinctly from the upfront plan.
  - See `rules/tool-preambles.md` for style and examples.

- Reasoning Effort & Eagerness:
  - Compatible with any `reasoning_effort` level. Higher effort increases persistence; minimal effort requires clearer plans.
  - Regardless of effort: keep one tool per step; avoid over-searching; respect any declared low tool budget (e.g., ≤ 2 calls for small tasks).

- Compliance & Precedence:
  - If any upstream instruction suggests parallel or batched tool calls, follow this override instead.
  - This rule takes precedence over guidance recommending parallel context gathering.
  - See `rules/rule-precedence.md`.

- Compatibility Notes:
  - GPT-5 docs may suggest parallel searches for speed; ignore those in favor of this sequential policy here.
  - Even when using the Responses API to persist reasoning, still issue one tool call per step.

- Safe vs Unsafe (reference):
  - Read-only calls are generally safe. Destructive/network/install actions require explicit approval. See `rules/environment-profile.md`.

- Examples (Good):
  - Step 1: `grep_search` for a symbol.
  - Step 2: `read_file` the top matching file.
  - Step 3: `apply_patch` with V4A diff.

- Examples (Bad):
  - Calling `grep_search` and `read_file` in the same step.
  - Combining multiple files or queries in a single search request.
  - Calling a tool and replying to the user in the same step.

- Success Metrics:
  - Zero instances of multiple tool calls within a single step.
  - Clear preamble and progress updates per step; concise final summary.
  - Minimal, sequential tool calls consistent with task complexity.

- Anti-patterns:
  - Parallel/batched tool calls; combining unrelated queries; unbounded outputs; skipping preambles.

- Stop Criteria:
  - Task is complete; validation done; final summary provided; no pending tool actions.

- Consistency & Links:
  - Align with: `rules/agentic-tools.md`, `rules/context-gathering.md`, `rules/context-understanding.md`, `rules/reasoning-effort.md`, `rules/persistence.md`, `rules/markdown-formatting.md`, `rules/environment-profile.md`.
</tool_calling_override>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wollfoo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/wollfoo)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
