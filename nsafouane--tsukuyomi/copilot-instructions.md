## tsukuyomi

> **Purpose:** Define the absolute engineering philosophy and coding standards for the Tsukuyomi Engine MVP v0.1.

# 📜 Tsukuyomi MVP v0.1 - Project Rules & Standards

**Purpose:** Define the absolute engineering philosophy and coding standards for the Tsukuyomi Engine MVP v0.1.

## 1. 🎯 MVP v0.1 Philosophy: "Unified Codebase"

### 1.1 Core Principle
The engine is a singular, unified platform. "Simulation Mode" and "Standalone Mode" are merely different interfaces for the **exact same underlying `BaseAgent` logic**.

*   **No Version Fragmentation:** There is no "V2" or "V3". All logic upgrades and improvements belong in the core **MVP v0.1**. Any feature that cannot be integrated into the core event loop must be deleted, not isolated.
*   **Engine vs Testing Scenarios:** The engine code (`core/`) natively drives the world. It includes the `FateEngine` and the `DramaDirector` to dynamically guide narrative flow, manage events, and produce complete stories for the agents to live inside. However, you must **NEVER hardcode specific testing scenario data** (e.g., "If agent=Davis, say hello") inside the core engine.
*   **Isolating Experiments:** We use scenarios (e.g., the marketplace or angry-men) strictly as isolated experiments to test our engine improvements. All specific names, dialogue, testing states, and environment layouts belong exclusively inside the `experiments/` scripts, not the `core/`.

## 2. 🗂️ Coding Standards & Structure

### 2.1 File Structure
*   `/tsukuyomi/core/`: The absolute source of truth. Contains `BaseAgent`, unified interfaces (`core/emotion/`, `core/memory/`, `core/belief/`), and server logic.
*   `/tsukuyomi/brain/`: Simulation/gRPC specific implementations (must inherit from `core/`).
*   `/tsukuyomi/agent/`: Standalone/Async specific implementations (must inherit from `core/`).
*   `/tsukuyomi/proto/`: gRPC definitions.
*   `/tsukuyomi/dev-artifacts/`: **MANDATORY DIRECTORY** for all specifications, analysis reports, roadmaps, and non-engine developmental markdown files.
*   `/docs/`: Dedicated folder for official project documentation, component documentation, user manuals, and the published architecture.
*   `/tests/`: Dedicated folder for **all** test files. No test scripts should exist outside of this directory.

### 2.2 Style Guide & Quality Enforcements
*   **File Naming:** Strictly Python `snake_case` globally (e.g., `agent_brain.py`).
*   **Class Naming:** Strictly `PascalCase`. No acronym prefixes unless absolutely necessary.
*   **Size Constraints:** No file shall exceed **700 lines of code**. If a file grows past this limit, extract logic into sub-modules immediately.
*   **ID Generation:** Never use `random.randint()` for identity generation. Use `uuid.uuid4()` globally.

### 2.3 Production Readiness
*   **Lifecycle Management:** All agents and engine components MUST implement strict `start()`, `pause()`, `resume()`, and `cleanup()` interfaces. 
*   **Safety & Backoff:** The engine must gracefully degrade, circuit break on LLM timeouts, and cleanly close network channels to prevent memory leaks.
*   **Validation:** All input from clients must be treated as untrusted and validated (sanitization).

## 3. 🚫 Prohibited Practices

*   **Duck-Typing Architecture:** Do not use mixins or dynamic runtime attachment (e.g., `enhance_agent()`) to add features to agents. All components (memory, emotion) must be statically defined via composition on the `BaseAgent`.
*   **Unapproved Dependencies:** Do not add database connectors (like `sqlalchemy` or `asyncpg`) or heavy framework libraries until explicitly approved for the MVP architecture.
*   **Scripting:** Do not edit experiments to fix bugs that should be in the core engine. Fix the root cause in `/tsukuyomi/core/`.

---

**Last Updated:** 2026-02-21 (MVP v0.1 Foundation)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nsafouane)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nsafouane)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
