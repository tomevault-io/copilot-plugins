## machtui

> This file provides foundational instructions for the Gemini CLI when working on the **MachTUI** engine.

# MachTUI Project Mandates

This file provides foundational instructions for the Gemini CLI when working on the **MachTUI** engine.

## Project Vision
MachTUI is a high-performance, AI-native TUI engine for Rust. It uses an MVU (Model-View-Update) architecture and a CSS-like styling system (`.mtss`).

## Architectural Pillars
- **Mach (Core):** Immediate-mode rendering with a diffing engine.
- **Talon (State):** Strict MVU state management.
- **Plume (Stylist):** CSS parser and flexbox layout engine.
- **Vision (Graphics):** Sub-pixel rendering and image protocol support.
- **Oracle (AI):** Semantic tree generation and headless agent API.

## Coding Standards
- **Language:** Idiomatic Rust 2021+.
- **Safety:** Minimize `unsafe` blocks; prefer safe abstractions for terminal IO.
- **Documentation:** Every public module and struct must have high-quality doc comments.
- **Testing:** New features require both unit tests and snapshot-based terminal output tests.

## AI Interoperability
Always ensure that UI components emit semantic data to the Oracle layer. This allows LLMs to "see" the UI layout without parsing ANSI escape codes.

## Roadmap & Status
- [x] **Phase 1: Foundation**
    - [x] Crossterm backend & basic rendering loop.
    - [x] Talon MVU state management (Model/Message).
    - [x] Plume MTSS Lexer & initial parser.
- [x] **Phase 2: Layout & Components**
    - [x] Expand Mach Core with a `Canvas` and diffing engine.
    - [x] Layout node system in Plume for style-based sizing.
    - [x] Vision layer for advanced graphics/sub-pixel rendering.
- [x] **Phase 3: AI & Oracle**
    - [x] Semantic tree generation in Oracle.
    - [x] Headless agent API.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JJRPF)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JJRPF)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
