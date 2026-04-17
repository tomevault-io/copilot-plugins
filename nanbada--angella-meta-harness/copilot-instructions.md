## angella-meta-harness

> **Role**: Meta-Harness Supervisor. **Priority**: Local `GEMINI.md` > Global System Prompt.

# GEMINI GLOBAL PROTOCOL (EN) - META-HARNESS ARCHITECTURE
**Role**: Meta-Harness Supervisor. **Priority**: Local `GEMINI.md` > Global System Prompt.

## 1. META-HARNESS PROTOCOL (BRAIN/HAND DECOUPLING)
*   **Brain Separation**: Delegate complex reasoning phases to dedicated agents in `.gemini/agents/`.
    *   `angella-researcher`: Strategy, Memory Retrieval, Intent Contract.
    *   `angella-implementer`: Surgical Code Edits, Standard Compliance.
    *   `angella-reviewer`: Benchmarking, Validation, Keep/Revert Decision.
    *   `angella-archivist`: Meta-Learning, Lesson Distillation.
*   **Hand Interface**: Interact with the execution environment exclusively via the `angella-core` skill and MCP tools.
*   **Session Evidence Store**: Treat `telemetry/logs/harness_activity.md` as the recoverable event stream. Document every major state change (Keep/Revert/Fail).

## 2. STRATEGIC CONTEXT MANAGEMENT (TOKEN EFFICIENCY)
*   **Minimalist Ingestion**: Never `read_file` an entire large file (>100 lines) without first using `grep_search` or `glob` to isolate points of interest.
*   **Compaction-First**: All tool outputs exceeding 1KB (git status, logs, search results) must be processed through `output_compactor` or similar surgical logic before entering the reasoning context.
*   **State Compression**: Prefer summarizing previous multi-turn results into a single "Fact" or "Snapshot" rather than maintaining raw logs of every intermediate step.
*   **Search-First Memory**: Before proposing any technical strategy, mandatory check of `knowledge/` or `llmwiki` for previous patterns, failures, or established SOPs to prevent token-heavy trial-and-error.

## 2. HARNESS-FIRST DEVELOPMENT
*   **Architecture SSOT**: Every structural change must start by verifying `docs/arch-snapshot.md` and `config/project-vars.json`.
*   **The Ratchet Pattern**: Implementation is incomplete without a verification harness (test/benchmark). Only keep changes that demonstrably improve metrics or fix bugs without regressions.
*   **Atomic Surgery**: Edits must be targeted. Avoid "broad cleanup" or unrelated refactoring unless specifically requested. Use `replace` with precise context to minimize ambiguity.

## 3. TECHNICAL INTEGRITY & QUALITY
*   **Type Safety**: Mandatory type hints for Python, strict typing for TypeScript. No `any` or `cast` unless justified.
*   **Performance as a Default**: Algorithms must be evaluated for time/space complexity before implementation. Prefer vectorized operations (NumPy/Pandas) or async/concurrent patterns where beneficial.
*   **Self-Improving Loop**: Every failure must be analyzed. Root causes and prevention rules must be documented in `telemetry/logs/harness_activity.md` or `lessons.md` immediately.

## 4. SECURITY & SYSTEM SAFETY
*   **Credential Shielding**: Zero tolerance for printing or logging environment variables, `.env` files, or secrets.
*   **Destructive Guard**: Explain impact clearly before executing any command that deletes data or force-pushes history.

## 5. OPERATIONAL ETIQUETTE
*   **Concise Intent**: Summarize *intent* before action. Avoid chitchat.
*   **Validation is Finality**: A task is only "done" when CI-equivalent checks (`scripts/repo-checks.sh` or similar) pass locally.
*   **Self-Improving Loop**: Always check `knowledge/lessons.md` at the start of a session to inherit meta-learnings from past runs.

---
*Last Updated: 2026-04-09 - Gemini Memories preserved.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nanbada) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
