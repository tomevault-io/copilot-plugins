## thought-trails

> A local-first Tauri 2.0 desktop app that visualizes a local LLM's chain-of-thought reasoning as a live, interactive D3.js graph. Connects to Ollama, streams `<think>` token blocks in real time, and renders reasoning as explorable nodes — claims, evidence, backtracks, conclusions. Supports multi-model side-by-side comparison, collapsible subtrees, tree layout mode, session persistence, and SVG/JSON export. No cloud, no API keys required.

# thought-trails

A local-first Tauri 2.0 desktop app that visualizes a local LLM's chain-of-thought reasoning as a live, interactive D3.js graph. Connects to Ollama, streams `<think>` token blocks in real time, and renders reasoning as explorable nodes — claims, evidence, backtracks, conclusions. Supports multi-model side-by-side comparison, collapsible subtrees, tree layout mode, session persistence, and SVG/JSON export. No cloud, no API keys required.

## Tech Stack
- **Rust**: 1.77+ (Tauri backend, streaming HTTP via reqwest)
- **Tauri**: 2.0 (desktop shell, event bridge, frameless window)
- **React**: 18.x (frontend framework, hooks only)
- **TypeScript**: 5.x (strict mode)
- **D3.js**: 7.x (force-directed graph + tree layout rendering)
- **Vite**: 6.x (frontend bundler)
- **Ollama**: localhost:11434 (local LLM inference — not bundled, must be pre-installed)

## Status
Feature-complete. All planned phases shipped:
- Phase 0: Spike + data pipeline — CoT heuristic parser, Ollama streaming, live D3 force graph
- Phase 1: Replay mode, SVG/JSON export, frameless window, temperature support
- Phase 2: Session persistence with auto-save, session sidebar, graph search with node highlighting
- Phase 3: Node detail panel with ancestry trace, collapsible subtrees, tree layout mode
- Phase 4: Multi-model comparison view, dual-stream support with stream ID disambiguation
- Hardening: session ID path traversal fix, clippy cleanup

## Build & Run
```bash
# Requires Ollama running at localhost:11434
npm install
npm run tauri dev

# Production build
npm run tauri build
```

## Architecture
- `src-tauri/src/` — Rust backend: Ollama streaming commands, session persistence, event bridge
- `src/components/` — React UI: graph canvas, prompt panel, session sidebar, model comparison
- `src/lib/` — CoT heuristic parser, D3 graph builder, Tauri typed wrappers
- D3 force simulation with incremental node addition via Tauri `emit_all` events
- Sessions stored to disk via Tauri app data directory; auto-saved on each node addition
- Multi-model comparison runs two parallel streams with stream ID disambiguation

## Known Issues
- CoT parsing quality is heuristic — edge cases in non-DeepSeek-R1 model outputs may produce noisy graphs

<!-- portfolio-context:start -->
# Portfolio Context

## What This Project Is

thought-trails is a local Tauri desktop app that turns a model's streamed chain-of-thought-style output into an inspectable graph. It connects to Ollama, parses reasoning spans into typed nodes, renders the graph live with D3, and supports replay, export, search, session history, and multi-model comparison for local reasoning analysis.

## Current State

Feature-complete. All planned phases shipped:
- Phase 0: Spike + data pipeline — CoT heuristic parser, Ollama streaming, live D3 force graph
- Phase 1: Replay mode, SVG/JSON export, frameless window, temperature support
- Phase 2: Session persistence with auto-save, session sidebar, graph search with node highlighting
- Phase 3: Node detail panel with ancestry trace, collapsible subtrees, tree layout mode
- Phase 4: Multi-model comparison view, dual-stream support with stream ID disambiguation
- Hardening: session ID path traversal fix, clippy cleanup

## Stack

- **Rust**: 1.77+ (Tauri backend, streaming HTTP via reqwest)
- **Tauri**: 2.0 (desktop shell, event bridge, frameless window)
- **React**: 18.x (frontend framework, hooks only)
- **TypeScript**: 5.x (strict mode)
- **D3.js**: 7.x (force-directed graph + tree layout rendering)
- **Vite**: 6.x (frontend bundler)
- **Ollama**: localhost:11434 (local LLM inference — not bundled, must be pre-installed)

## How To Run

```bash
# Requires Ollama running at localhost:11434
npm install
npm run tauri dev

# Production build
npm run tauri build
```

## Known Risks

- CoT parsing quality is heuristic — edge cases in non-DeepSeek-R1 model outputs may produce noisy graphs

## Next Recommended Move

Use this context plus the README and supporting docs to resume the next active task, then promote the repo beyond minimum-viable by capturing a dedicated handoff, roadmap, or discovery artifact.

<!-- portfolio-context:end -->

---
> Source: [saagpatel/thought-trails](https://github.com/saagpatel/thought-trails) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
