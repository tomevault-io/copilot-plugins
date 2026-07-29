## deeprefine-skill

> This extension lets Gemini CLI use DeepRefine to refine and evolve Graphify / LLM-Wiki knowledge graphs.

# DeepRefine Skill for Gemini CLI

This extension lets Gemini CLI use DeepRefine to refine and evolve Graphify / LLM-Wiki knowledge graphs.

Use the `deeprefine` skill when the user asks to refine, diagnose, debug, improve, inspect, or query a Graphify-generated knowledge graph.

## Core behavior

1. Prefer the agent-native DeepRefine loop over the terminal-only `deeprefine refine` command unless the user explicitly requests CLI / FAISS mode.
2. Follow the same control flow as `DeepRefine.refine()` from DeepRefine.
3. Import Graphify memory with `deeprefine history sync-memory` when running the default queue workflow.
4. Process pending history entries one by one.
5. Use Graphify query results and k-hop expansion over `graphify-out/graph.json` as retrieval evidence.
6. Create and maintain a valid `loop_trace_<query_id>.json` before applying graph updates.
7. Review proposed `<refinement>...</refinement>` actions before applying them.
8. Apply graph updates only through `deeprefine apply`; do not hand-edit `graphify-out/graph.json`.
9. Store DeepRefine artifacts under `graphify-out/.deeprefine/`.
10. If graph artifacts are missing, ask the user to run `/graphify .` and `/graphify ./ --wiki` first.

## Commands

- `/deeprefine` — run the default agent-native pending-query workflow.
- `/deeprefine:review <query>` — review graph evidence for a single query without graph writes.
- `/deeprefine:apply <request>` — apply validated refinement actions only from a valid trace and refinement file.

When extension changes are installed or linked, restart Gemini CLI before checking `/extensions list` or `/commands list`.

---
> Source: [HKUST-KnowComp/DeepRefine-Skill](https://github.com/HKUST-KnowComp/DeepRefine-Skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
