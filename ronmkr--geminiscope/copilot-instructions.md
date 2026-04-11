## geminiscope

> Geminiscope is a high-performance Rust TUI dashboard for Gemini CLI, providing real-time observability into AI sessions, costs, and system health.

# Geminiscope: Project Roadmap & Capability Map

Geminiscope is a high-performance Rust TUI dashboard for Gemini CLI, providing real-time observability into AI sessions, costs, and system health.

## 🎯 Capability Mapping

| Feature | Status | Implementation Strategy |
| :--- | :--- | :--- |
| **Session Explorer** | ✅ Done | High-density custom renderer with syntax highlighting. |
| **Real-time Tracking** | ✅ Done | Integrated `notify` events with debounced background parser. |
| **Vim Bindings** | ✅ Done | `j/k` for navigation, `J/K` or `Alt+Arrows` for detail scrolling. |
| **Mouse Navigation** | ✅ Done | Clickable rail icons and mouse wheel support. |
| **Global Search** | ✅ Done | `/` to filter sessions by content across Chats and Tools. |
| **Multi-Mode Sorting**| ✅ Done | Sort projects by Date, Cost, Tokens, or Name. |
| **Cost Estimation** | ✅ Done | Pricing engine for Pro/Flash models + token math. |
| **Analytics (Stats)** | ✅ Done | Interactive Sparklines for token trends. |
| **Secret Scanning** | ✅ Done | Entropy-aware background scan of `chats/*.json`. |
| **Session Export** | ✅ Done | `e` key to export raw un-truncated JSON for large logs. |
| **Notification System**| ✅ Done | Immediate feedback banner for file operations. |
| **Help Modal** | ✅ Done | Press `?` or `h` for integrated documentation. |
| **Interactive Settings**| ✅ Done | Edit `settings.json` live via TUI with validation. |
| **Theme Engine** | ✅ Done | Multi-theme support through `themes.json`. |
| **Session Diffing** | ✅ Done | Compare two AI turns side-by-side using the `similar` crate. |

## 🛠️ Engineering Specs & Performance

### 1. Custom Render Pipeline
To eliminate the vertical padding issues inherent in standard markdown libraries, Geminiscope uses a specialized line-buffer renderer. This allows for:
- **Maximum Density**: Multiple blocks of content (JSON, Headers, Text) are collapsed into a single cohesive view.
- **Dynamic Syntax Highlighting**: Real-time parsing of JSON structures to provide color-coded keys and values without blocking the UI thread.
- **Visual Feedback**: Real-time scrollbars and empty-state placeholders.

### 2. High-Volume Data Strategy
AI logs can grow to several megabytes. Geminiscope maintains performance through:
- **Intelligent Truncation**: Strings exceeding 1000 characters are truncated in the UI preview to maintain 60FPS rendering.
- **Off-Thread Parsing**: All JSON deserialization happens outside the main event loop.
- **On-Demand Export**: Users can bypass UI limits by exporting the full raw payload to disk for external viewing.

### 3. Clean & Idiomatic Rust (v0.2.0+)
- **Modular Architecture**: Decoupled UI using the `ViewHandler` trait and specialized sub-handlers.
- **DRY Principles**: Centralized data formatting and search logic in `models.rs`.
- **Zero Warnings**: Strictly adheres to `#![warn(clippy::all, clippy::pedantic)]` with zero warnings.
- **Robust Error Handling**: Eliminated all unsafe `.unwrap()` calls in the core execution path.

## 🚀 Next Steps & Structural Improvements

### 1. Advanced Features
- [ ] **Remote MCP Dashboard**: Live health monitoring of remote MCP servers.
- [ ] **Interactive Timeline**: Scrub through AI history using a visual time-slider.

### 2. Security & Hardening (Completed)
- [x] **Secure Persistence**: 0600 permissions for `settings.json` and session exports.
- [x] **Atomic Saves**: Temp-file-and-rename for configuration to prevent corruption.
- [x] **Path Sanitization**: Canonicalization and HOME-directory restriction.
- [x] **Input Guardrails**: 256-char limits on all text inputs.
- [x] **Secret Redaction**: Masking identified secrets in the UI by default (`Ctrl-R` to toggle).

## 🛠️ Tech Stack Notes
- **Language**: Rust (Edition 2024)
- **TUI**: [ratatui](https://github.com/ratatui-org/ratatui)
- **Async**: [tokio](https://github.com/tokio-rs/tokio)
- **File Watching**: [notify](https://github.com/notify-rs/notify)
- **Security**: Shannon Entropy scanning for credentials.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ronmkr)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ronmkr)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
