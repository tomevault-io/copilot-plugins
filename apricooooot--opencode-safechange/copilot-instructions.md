## opencode-safechange

> SafeChange is an OpenCode multi-agent runtime for risk-aware repository changes.

# SafeChange development guide

SafeChange is an OpenCode multi-agent runtime for risk-aware repository changes.

## Commands

- Run validation with `npm run check`.
- Run unit tests with `npm test`.

## Architecture

- `.opencode/agents/` contains the primary agent and two subagents.
- `.opencode/tools/` contains the structured report tool exposed to OpenCode.
- `runtime/` contains deterministic runtime primitives that can be tested without an LLM.
- `examples/` contains prompts that demonstrate the intended workflow.
- `benchmark/` contains deterministic impact-analysis fixtures and scoring.

Keep analysis read-only. File edits belong to the implementer-verifier and require
the primary agent to have entered APPLYING state.

Benchmark predictions must use repository-relative POSIX paths. Do not expose
the answer key to the agent being evaluated.

---
> Source: [Apricooooot/opencode-safechange](https://github.com/Apricooooot/opencode-safechange) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-02 -->
