## architecture-patterns

> Miniverse combines **deterministic physics** (pure Python simulation rules) with **non-deterministic cognition** (LLM-driven agent behavior).

# Architecture Overview

## System Design
Miniverse combines **deterministic physics** (pure Python simulation rules) with **non-deterministic cognition** (LLM-driven agent behavior).

## Key Patterns
- **Dependency injection** - All dependencies passed as constructor arguments, no globals
- **Strategy pattern** - Pluggable backends via ABC/Protocol: `PersistenceStrategy`, `MemoryStrategy`, `Planner`, `Executor`, `ReflectionEngine`, `SimulationRules`
- **Pydantic everywhere** - All data structures are Pydantic models
- **Immutability** - Always `state.model_copy(deep=True)` before modifying
- **Async for I/O** - Use `await` for persistence, memory, LLM calls; `asyncio.gather()` for parallelism

## Codebase Organization
```
miniverse/
├── __init__.py              # Public API exports
├── config.py                # Environment variable configuration
├── orchestrator.py          # Main simulation loop
├── schemas.py               # All Pydantic models (WorldState, AgentProfile, etc.)
├── simulation_rules.py      # Deterministic physics interface
├── persistence.py           # Storage strategies (InMemory, JSON, Postgres)
├── memory.py                # Memory retrieval strategies
├── perception.py            # Builds agent observations
├── scenario.py              # JSON scenario loader
├── llm_calls.py             # World Engine LLM integration
├── llm_utils.py             # LLM retry logic with validation
├── logging_utils.py         # Color-coded output utilities
├── cognition/               # Agent cognition stack
│   ├── __init__.py
│   ├── scratchpad.py        # Working memory data structure
│   ├── planner.py           # Planning interface + implementations
│   ├── executor.py          # Action execution interface + implementations
│   ├── reflection.py        # Reflection interface + implementations
│   ├── llm.py               # LLM-backed cognition modules
│   ├── prompts.py           # Prompt templates library
│   ├── renderers.py         # Template rendering utilities
│   ├── context.py           # Prompt context builder
│   ├── cadence.py           # Planner/reflection scheduling
│   └── runtime.py           # AgentCognition bundles
└── environment/             # Environment tier implementations
    ├── __init__.py
    ├── schemas.py           # Graph/grid Pydantic schemas
    ├── graph.py             # Tier 1: Logical graph (rooms, networks)
    ├── grid.py              # Tier 2: Spatial grid (tile maps)
    └── helpers.py           # Occupancy, pathfinding utilities
```

## Environment Tiers
- **Tier 0**: Abstract metrics only (KPIs, no spatial semantics)
- **Tier 1**: Logical graph (rooms, networks, adjacency)
- **Tier 2**: Spatial grid (tile-based maps with collision)

---
> Source: [miniverse-ai/miniverse](https://github.com/miniverse-ai/miniverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
