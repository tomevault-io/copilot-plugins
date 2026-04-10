## architectmind

> **Deployment:** [https://architect-mind.vercel.app/](https://architect-mind.vercel.app/)

# ArchitectMind: System Design Visualizer

**Deployment:** [https://architect-mind.vercel.app/](https://architect-mind.vercel.app/)

ArchitectMind is a web-based tool for visualizing and analyzing system architectures. It provides an interactive canvas to build infrastructure diagrams and a backend that validates the topology's semantic correctness.

## Project Structure

- `api/`: Go-based API using the Gin framework.
  - `_cmd/main.go`: Entry point, CORS configuration, and route definitions.
  - `handler/`: API request handlers and 38+ validation rules (e.g., `check_availability.go`).
  - `model/`: Data structures for nodes, edges, and protocols.
- `frontend/`: React-based visualizer using Vite and React Flow.
  - `src/components/Canvas.tsx`: Main diagramming area with Undo, Merge, and Duplicate.
  - `src/components/TabBar.tsx`: Multi-tab management.
  - `src/api/`: Frontend clients for backend communication.
  - `src/utils/`: Export utilities for Excalidraw, Image, Mermaid, and PDF.

## Tech Stack

- **Backend:** Go 1.25+, Gin.
- **Frontend:** React 19, Vite, React Flow, TypeScript.

## Development Conventions

- **Backend:**
  - Define new architecture components in `model/topology.go`.
  - Validation rules are modular: add `check_*.go` in `handler/` and call from `topology_handler.go`.
  - Always add unit tests for new rules in `check_*_test.go`.
- **Frontend:**
  - Use TypeScript for all components and utilities.
  - Custom node styling should be managed in `src/nodes/ArchitectureNode.tsx` and `nodeConfig.ts`.
  - Keep types in sync between `frontend/src/types/` and `api/model/`.
  - New export formats go into `src/utils/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/june-in-exile) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
