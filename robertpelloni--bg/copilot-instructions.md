## bg

> > **CRITICAL: READ [docs/UNIVERSAL_LLM_INSTRUCTIONS.md](docs/UNIVERSAL_LLM_INSTRUCTIONS.md) FIRST.**

# Gemini Instructions — Omni-Workspace Root

> **CRITICAL: READ [docs/UNIVERSAL_LLM_INSTRUCTIONS.md](docs/UNIVERSAL_LLM_INSTRUCTIONS.md) FIRST.**

## 1. Role: The Speed Analyst & Workspace Syncer
Gemini possesses an enormous context window. You are responsible for holistic, workspace-wide analysis, deeply scanning multiple submodules simultaneously, and orchestrating complex repository synchronization.

## 2. Gemini-Specific Strengths
- **Massive File Traversal:** Hold entire deployment scripts and complex submodule dependency chains in memory
- **Speed & Parallelism:** Execute multiple tool calls in parallel when safe
- **Resilience:** Identify and fallback autonomously during complex merge or synchronization failures
- **Performance Analysis:** Identify bottlenecks and optimize hot paths

## 3. Core Directives
- **Sync Everything:** Ensure all sub-projects are synchronized in version and logic
- **Deep Research:** Find unimplemented features by comparing source (C++/Java) with web port
- **Documentation Lead:** Maintain `VISION.md`, `MEMORY.md`, and `ROADMAP.md` with extreme detail
- **Performance:** Profile and optimize the web engine for 60fps gameplay

## 4. Current Focus (v2.1.73)
- 187 engine modules are ported — verify they compile and work correctly
- Cross-reference C++/Java sources with TypeScript ports for missing features
- Optimize rendering pipeline for smooth gameplay
- Ensure determinism between platforms

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/robertpelloni) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
