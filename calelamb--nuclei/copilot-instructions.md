## nuclei

> Nuclei is a purpose-built desktop IDE for quantum computing. It combines a Monaco code editor with live circuit visualization, Bloch sphere animations, probability histograms, and an AI teaching assistant called Dirac (powered by Claude API). The primary audience is students taking their first quantum computing course.

# CLAUDE.md — Nuclei

## Project Overview

Nuclei is a purpose-built desktop IDE for quantum computing. It combines a Monaco code editor with live circuit visualization, Bloch sphere animations, probability histograms, and an AI teaching assistant called Dirac (powered by Claude API). The primary audience is students taking their first quantum computing course.

**This is a free, open-source tool. No monetization. People download the .dmg and code on it.**

## Tech Stack

| Layer | Technology | Notes |
|-------|-----------|-------|
| Desktop Shell | **Tauri 2.x** (Rust) | ~10MB binary, handles process management, file I/O, IPC |
| Frontend | **React 19 + TypeScript** | Vite for bundling, all UI lives here |
| State Management | **Zustand** | Minimal boilerplate, reactive updates across panels |
| Code Editor | **Monaco Editor** | VS Code's engine — syntax highlighting, autocomplete |
| Circuit Rendering | **D3.js** + custom SVG | Live circuit diagrams that update as you type |
| 3D Rendering | **Three.js** | Bloch sphere visualization |
| Charts | **D3.js or Recharts** | Probability histograms |
| Python Kernel | **Managed subprocess + WebSocket** | Executes Qiskit/Cirq/CUDA-Q code |
| AI Assistant | **Claude API** (Haiku for fast, Sonnet for complex) | Dirac — Claude API wrapper with tutor persona, system prompt, context injection, and tool definitions |
| Build | **Vite + Tauri CLI** | HMR in dev, .dmg packaging for release |
| Python Bundling | **conda-pack** (later) | So users don't need Python installed |

## Architecture

Three-layer architecture:

### Layer 1: Tauri Shell (Rust) — `src-tauri/`
- Spawns and manages the Python kernel process
- File system operations (project open/save/watch)
- IPC bridge — typed commands between frontend and backend
- Auto-updater, native menus, system tray

### Layer 2: Frontend (React + TypeScript) — `src/`
- Monaco Editor with Python + Qiskit/Cirq/CUDA-Q aware IntelliSense
- Circuit diagram renderer (D3.js SVG)
- Bloch sphere (Three.js)
- Probability histogram
- Dirac AI chat panel
- Panel layout system (resizable, rearrangeable)
- Zustand stores for circuit state, editor state, simulation results

### Layer 3: Python Kernel — `kernel/`
- WebSocket server that receives code from the frontend
- Framework adapters that convert Qiskit/Cirq/CUDA-Q circuits into a universal CircuitSnapshot format
- Simulation execution and result serialization
- Auto-detects framework from imports

## Key Data Structures

```typescript
// Sent from kernel to frontend on every code change (lightweight, no simulation)
interface CircuitSnapshot {
  framework: 'qiskit' | 'cirq' | 'cuda-q';
  qubit_count: number;
  classical_bit_count: number;
  depth: number;
  gates: Array<{
    type: string;          // 'H', 'CNOT', 'RZ', etc.
    targets: number[];     // qubit indices
    controls: number[];    // control qubit indices
    params: number[];      // rotation angles, etc.
    layer: number;         // depth position
  }>;
}

// Sent from kernel to frontend after explicit execution (Cmd+Enter)
interface SimulationResult {
  state_vector: Array<{ re: number; im: number }>;
  probabilities: Record<string, number>;
  measurements: Record<string, number>;
  bloch_coords: Array<{ x: number; y: number; z: number }>;
  execution_time_ms: number;
}
```

## Data Flow (Critical Path)

1. User types code in Monaco Editor
2. On change (300ms debounce), frontend sends code to Python kernel via WebSocket
3. Kernel parses code, detects framework, builds circuit object
4. Kernel extracts CircuitSnapshot (gate list, qubit count — NO simulation) and returns JSON
5. Frontend renders circuit diagram from snapshot in real time
6. User presses Cmd+Enter → kernel runs full simulation → returns SimulationResult
7. Frontend updates Bloch sphere + histogram panels

## Project Structure

```
nuclei/
├── src-tauri/                   # Rust backend
│   ├── src/
│   │   ├── main.rs              # Entry point, window setup
│   │   ├── commands/            # Tauri IPC commands
│   │   │   ├── kernel.rs        # Python process management
│   │   │   ├── filesystem.rs    # File I/O operations
│   │   │   └── settings.rs      # User preferences
│   │   └── kernel/
│   │       └── manager.rs       # Kernel lifecycle management
│   ├── Cargo.toml
│   └── tauri.conf.json
├── src/                         # React frontend
│   ├── components/
│   │   ├── editor/              # Monaco wrapper
│   │   ├── circuit/             # Circuit diagram renderer
│   │   ├── bloch/               # Bloch sphere (Three.js)
│   │   ├── histogram/           # Probability charts
│   │   ├── dirac/               # AI assistant panel
│   │   ├── terminal/            # Output terminal
│   │   └── layout/              # Panel system & drag-drop
│   ├── stores/                  # Zustand state stores
│   │   ├── circuitStore.ts
│   │   ├── editorStore.ts
│   │   └── simulationStore.ts
│   ├── hooks/                   # Custom React hooks
│   ├── types/                   # TypeScript type definitions
│   └── App.tsx
├── kernel/                      # Python kernel
│   ├── server.py                # WebSocket server entry point
│   ├── executor.py              # Code execution engine
│   ├── adapters/
│   │   ├── base.py              # Abstract adapter interface
│   │   ├── qiskit_adapter.py
│   │   ├── cirq_adapter.py
│   │   └── cudaq_adapter.py
│   ├── models/
│   │   ├── snapshot.py          # CircuitSnapshot dataclass
│   │   └── result.py            # SimulationResult dataclass
│   └── requirements.txt
├── package.json
├── tsconfig.json
├── vite.config.ts
└── README.md
```

## Framework Abstraction

Each quantum framework has an adapter in `kernel/adapters/` that converts framework-specific circuit objects into the universal CircuitSnapshot format. The kernel auto-detects which framework the user is using by analyzing imports.

| Framework | Circuit Object | Simulator | Adapter |
|-----------|---------------|-----------|---------|
| Qiskit | QuantumCircuit | AerSimulator | qiskit_adapter.py |
| Cirq | cirq.Circuit | cirq.Simulator | cirq_adapter.py |
| CUDA-Q | cudaq.kernel | cudaq.sample() | cudaq_adapter.py |

## Gate Mapping (Universal Registry)

| Canonical | Qiskit | Cirq | CUDA-Q |
|-----------|--------|------|--------|
| H | qc.h(q) | cirq.H(q) | h(q) |
| CNOT | qc.cx(c, t) | cirq.CNOT(c, t) | cx(c, t) |
| RZ | qc.rz(θ, q) | cirq.rz(θ)(q) | rz(θ, q) |
| Measure | qc.measure(q, c) | cirq.measure(q) | mz(q) |
| Toffoli | qc.ccx(c1, c2, t) | cirq.TOFFOLI(c1,c2,t) | x.ctrl(c1, c2, t) |

## UI Layout

Default four-panel layout:
- **Left panel (60%):** Monaco code editor
- **Right panel (40%):** Circuit diagram (top) + Bloch sphere (bottom), stacked
- **Bottom panel (collapsible):** Tabs — Terminal, Histogram, Dirac chat
- **Status bar:** Framework indicator, qubit count, circuit depth, sim status

All panels resizable and rearrangeable. Layout persists across sessions.

## Visual Design

- **Dark theme** by default (navy background #0F1B2D, teal accents #00B4D8)
- Light theme option
- **Code font:** JetBrains Mono
- **UI font:** Inter
- **Quantum elements:** Teal (#00B4D8)
- **Dirac AI elements:** Purple (#7B2D8E)
- 60fps animation target for Bloch sphere

## Dirac (AI Assistant)

Dirac is a Claude API wrapper with a quantum computing tutor persona. Named after Paul Dirac. There is no custom model — Dirac is Claude (Haiku for fast interactions, Sonnet for complex reasoning and tool use) called through the Anthropic API with a system prompt, context injection pipeline, and tool definitions that shape its behavior.

**What makes Dirac "Dirac":**
- A system prompt that establishes the tutor persona (patient, beginner-friendly, named Dirac)
- Context injection: the user's current code, CircuitSnapshot, SimulationResult, errors, and framework are injected into each API call
- Tool definitions passed to Claude's tool use API that let Dirac act on the IDE
- Student model data (Phase 5+) injected into the system prompt so Dirac adapts to skill level

**Capabilities (via system prompt + tools):**
- Context-aware help (sees current code, circuit, simulation results)
- Concept explanations calibrated for beginners
- Code generation and insertion
- Error diagnosis with plain-English explanations
- Gate explorer (explain any gate with matrix + Bloch sphere interpretation)
- Exercise mode (generate challenges, verify solutions)

**Tools (passed to Claude's tool use API):**
- `insert_code` — insert code at cursor or replace selection
- `run_simulation` — execute current circuit
- `highlight_gate` — highlight a gate in the visualization
- `step_to` — advance step-through to a specific gate
- `create_exercise` — generate a quantum exercise
- `verify_solution` — check student's answer

**Model selection:** Haiku for fast Q&A and ghost completions. Sonnet for tool use, code generation, reasoning mode, and complex explanations. Selection logic lives in the frontend — route based on detected intent.

## Development Phases

### Phase 1: Foundation (Weeks 1–4)
Editor → kernel (Qiskit + Cirq) → circuit visualization → histogram → basic Dirac chat

### Phase 2: Visualization & Polish (Weeks 5–8)
Bloch sphere, CUDA-Q adapter, file operations, themes, context-aware Dirac v2, PlatformBridge abstraction

### Phase 3: Dirac Full Integration (Weeks 9–12)
Tool use (insert code, run sim), gate explorer, step-through, exercises, onboarding, error diagnosis

### Phase 4: Web Version & Community (Post-launch)
Browser version (Pyodide), learning paths, circuit sharing/export, plugin system, Python bundling

### Phase 5: Inline AI Editor + Intelligent Dirac (Weeks 13–18)
Ghost completions, Cmd+K inline edit, Dirac memory & student model, reasoning mode, smart code actions, multi-file projects

### Phase 6: UX Polish, Learning Identity & Zero-to-Quantum (Weeks 19–24)
Zero-knowledge onboarding (Quantum Playground), progressive disclosure UI (beginner→advanced modes), micro-interactions & animation system, command palette, visual identity refinement, accessibility (WCAG 2.1 AA), performance optimization

### Phase 7: Real Hardware, Community & Scale (Ongoing)
Real quantum hardware integration (IBM/Google/IonQ), community gallery & profiles, capstone projects, concept map, educator/classroom tools, plugin marketplace, localization, CI/CD for macOS/Windows/Linux

## Commands

```bash
# Dev
npm run tauri dev          # Start dev server with hot reload

# Build
npm run tauri build        # Build .dmg for distribution

# Kernel (standalone testing)
cd kernel && python server.py   # Start kernel WebSocket server

# Test
npm test                   # Frontend tests
cargo test                 # Rust backend tests
cd kernel && pytest        # Kernel tests
```

## Conventions

- All TypeScript, no plain JS
- Functional React components with hooks only
- Zustand for all shared state — no prop drilling
- All kernel ↔ frontend communication via WebSocket JSON messages
- Python kernel code uses type hints and dataclasses
- Rust code follows standard Tauri patterns
- Commit messages: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/calelamb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
