## smokescreen

> **SMOKESCREEN** is a "Technical Incident Theatre" simulation suite designed for Platform and DevOps engineers. It generates hyper-technical, convincing simulations of catastrophic system failures, complete with real-time cross-device synchronization and authentic hardware aesthetics to provide high-fidelity "performance cover" during unnecessary meetings.

# GEMINI.md - SMOKESCREEN Project Context

## 1. Project Objective
**SMOKESCREEN** is a "Technical Incident Theatre" simulation suite designed for Platform and DevOps engineers. It generates hyper-technical, convincing simulations of catastrophic system failures, complete with real-time cross-device synchronization and authentic hardware aesthetics to provide high-fidelity "performance cover" during unnecessary meetings.

**Source:** [GitHub Repository](https://github.com/DrizzlyOwl/smokescreen)
**Copyright:** DrizzlyOwl

## 2. Technical Stack
- **Frontend:** React 19 (TypeScript)
- **Build Tool:** Vite
- **Sync:** P2P WebRTC via PeerJS (DataChannels)
- **Styling:** SCSS (BEM architecture, modular per-component styles, CSS Variable API).
- **Audio:** Web Audio API (procedural synthesis, graph-based architecture, global mute orchestration).
- **Simulation:** UI Avatars (Identities), Gemini 1.5 Flash (AI Intelligence).

## 3. Core Features (v5.5 - "Orchestrator Edition")
- **Centralized State Management**: Unified handling of global app state, incident parameters, and UI preferences via a robust context-based orchestrator.
- **Deep URL Synchronization**: Complete application state (Threat Level, Stack, Theme, Active Panes, Eco/Debug modes) is serialized into the URL for instant sharing and session persistence.
- **Dynamic Boot Sequencing**: Hardware-accurate BIOS sequence that visually loads modules based on active URL parameters.
- **Intelligent War Room**: 
    - **Smart Unread System**: Messages tracked via IntersectionObserver with a 3-second "dwell" requirement.
    - **Visual Grouping**: Slack-fidelity message grouping and indentation.
    - **Interactive Bios**: Clicking a user reveals their technical specialization (SRE, DBA, etc.).
    - **Dynamic Identity**: Initial-based avatars seeded via user names for consistent identity tracking.
- **Authentic CRT Simulation**: Theme-aware warm-up expansions and chaotic instability effects.
- **Global Audio Extract (Mute)**: Single-point control for all procedural hums, fan noises, and notification pings.

## 4. Operational Controls
- **Double-ESC / `resolve`**: Resolve incident state and clear alerts.
- **Command Line**: `declare`, `resolve`, `warroom`, `p0`, `aws`, `amber`, `debug`, etc.
- **URL Control**: Append `?sev=P0&stack=GCP&theme=amber&panes=chat,logs,map` to pre-configure the theatre.

## 5. Architectural Standards
- **BEM (Block Element Modifier)**: All UI components must use semantic BEM classes in dedicated SCSS files.
- **Component Isolation**: No atomic utilities or global layout classes; components are self-contained.
- **Strict Typing**: Zero `any` usage in core state handlers; all P2P payloads are interface-guaranteed.

---
*Last Updated: 30 March 2026 | Orchestrator Edition*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DrizzlyOwl)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DrizzlyOwl)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
