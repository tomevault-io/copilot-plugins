## eposter

> This project is a long-term modular voice-controlled platform running on Linux (target: Raspberry Pi 4).

# AGENTS.md

## 0. Purpose

This project is a long-term modular voice-controlled platform running on Linux (target: Raspberry Pi 4).
It is fully offline-first. Internet usage is strictly forbidden unless explicitly approved by the project owner.

The system consists of:

* Backend (Flask + State Machine + Voice Engine)
* Frontend (Single-page HTML + Vanilla JS + Canvas)
* Voice Layer (Wake-word + STT)
* Storage Layer (JSON-based persistence)

This file defines strict architectural rules.
AI agents (Codex) must follow them.

---

# 1. Core Architectural Principles

## 1.1 Layer Separation (Mandatory)

The system is strictly divided into layers:

```
backend/
    api/
    states/
    voice/
    core/
    storage.py

frontend/
    templates/
    static/

data/
```

Layer responsibilities:

| Layer    | Responsibility          |
| -------- | ----------------------- |
| Voice    | Detect commands         |
| Core     | State machine + routing |
| States   | Domain logic per state  |
| Storage  | JSON persistence        |
| API      | HTTP interface          |
| Frontend | Pure rendering          |

No layer may leak logic into another.

---

# 2. State Architecture (Strict)

States are modular and dynamically loaded.

Structure:

```
backend/states/
    base/
        state.py
    map/
        state.py
    review/
        state.py
```

Each state MUST implement:

```python
class State:
    def enter(self): ...
    def exit(self): ...
    def handle_command(self, command: str): ...
    def get_view_model(self) -> dict: ...
```

Rules:

* `exit()` ALWAYS transitions to `base`
* States never talk directly to UI
* States never access Flask
* States never access Voice
* States may use storage.py
* State transitions are STRICT and voice-driven

The backend state manager controls transitions.

---

# 3. State Machine Rules

* Strict transition model
* No rollback
* No recovery system
* No database
* Persistence via JSON only
* After day verification → immediately saved

State Manager responsibilities:

* Load state dynamically
* Call exit() on current state
* Call enter() on new state
* Provide current state's view_model to API

---

# 4. View Model Pattern (Mandatory)

Frontend receives ONLY:

```
GET /api/state
```

Backend returns:

```json
{
    "state": "map",
    "view_model": { ... }
}
```

Frontend:

* Does NOT compute logic
* Does NOT change state
* Does NOT interpret commands
* Only renders view_model

All business logic is backend-only.

---

# 5. Voice Architecture

Wake word: "плакат"

Voice pipeline:

```
Wake Engine → STT → Command JSON mapping → Core → State
```

Rules:

* No NLP
* No AI interpretation
* Commands stored in JSON:

  ```
  data/commands.json
  {
      "следующий день": "NEXT_DAY",
      "карта": "GO_MAP"
  }
  ```
* No logging of recognized speech
* Wake word must always listen
* Voice runs in daemon thread

Voice generates events.
Core handles events.
States process commands.

UI never reacts to voice directly.

---

# 6. Storage Rules

Only JSON storage.

```
data/
    session.json
    commands.json
```

All persistence goes through:

```
backend/storage.py
```

No direct JSON access inside states.

No database allowed.

---

# 7. Frontend Rules (Strict)

Technology:

* HTML
* CSS
* Vanilla JS
* Canvas only (no WebGL)

Forbidden forever:

* React
* Vue
* Angular
* Svelte
* Tailwind
* Bootstrap
* jQuery
* Electron
* Any UI framework

Design rules:

* Modern minimalism
* Rounded corners
* Smooth gradients
* Large UI elements
* Heavy use of animations
* Smooth transitions
* Dark or modern palette
* Fullscreen only

Frontend must be:

* Single page (current mode)
* Architecturally ready for multi-page states in future

---

# 8. Networking Rules

Internet is technically available.

BUT:

* No external API usage
* No cloud services
* No telemetry
* No analytics
* No remote logging

Internet usage requires explicit approval.

---

# 9. Concurrency Rules

* Flask runs in main thread
* Voice runs in daemon thread
* Additional threads allowed only if architecturally justified
* Multiprocessing allowed if simplifies architecture
* No uncontrolled background workers

---

# 10. Logging

Currently disabled.

No voice logs.
No analytics.
No telemetry.

---

# 11. Extensibility Philosophy

The system must be:

* Modular
* Domain-oriented
* State-driven
* Easily extendable via new states
* Independent between modules

No plugin system required.

States are the only extension mechanism.

---

# 12. Codex Authority Rules

Codex MAY:

* Create new states
* Create new modules inside defined layers
* Extend functionality
* Improve internal implementation

Codex MUST NOT:

* Change architecture
* Change folder structure
* Merge layers
* Introduce frameworks
* Add database
* Introduce internet usage
* Refactor core architecture
* Replace state machine model

Architectural changes require explicit approval.

---

# 13. Forbidden Changes (Hard Ban)

The following are permanently forbidden unless owner explicitly overrides:

* Any frontend framework
* Any backend framework besides Flask
* Database integration
* Cloud integration
* Online APIs
* Logging user voice
* Telemetry
* Electron
* Docker
* Full rewrite patterns
* Converting to SPA framework
* Moving logic to frontend

---

# 14. Target Platform

Primary target:

* Raspberry Pi 4 (Linux)

Development:

* Linux
* WSL (Windows)

No Docker required.

---

# 15. Design Philosophy Summary

This is a:

* Voice-driven
* State-driven
* Backend-controlled
* UI-render-only
* Modular
* Long-term platform

All complexity belongs to backend.
Frontend only renders.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NQUAREx)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/NQUAREx)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
