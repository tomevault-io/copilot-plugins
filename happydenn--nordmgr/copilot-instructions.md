## nordmgr

> This file serves as the source of truth for the `nordmgr` project context, architectural decisions, and operational constraints.

# Gemini Context & Ground Rules

This file serves as the source of truth for the `nordmgr` project context, architectural decisions, and operational constraints.

## 1. Project Goal
Build a self-hosted, single-binary web application for generating NordVPN WireGuard (NordLynx) configurations. It allows users to browse servers, check latency, and generate configs without needing the official NordVPN app.

## 2. Core Constraints & Decisions

### Architecture
*   **Monolith**: The application MUST be a single Go binary with the React frontend embedded (using `go:embed`).
*   **No Database**: State (like the user's Private Key) should be stored client-side (LocalStorage) or strictly in-memory if needed. No SQL/SQLite unless explicitly requested.

### Functionality
*   **Latency Testing**:
    *   **MUST** use the system's `ping` command via `os/exec`.
    *   **DO NOT** use native Go ICMP libraries that require `root` or `CAP_NET_RAW`.
    *   Target platforms: Linux VMs, OpenWRT routers (MIPS/ARM).
*   **Authentication**:
    *   **NO** NordVPN Login flow. Do not attempt to reverse engineer the login API.
    *   **Manual Key**: The user provides their WireGuard Private Key manually in the UI.
*   **Data Source**:
    *   Fetch server data from public/undocumented NordVPN API endpoints.
    *   Aggregate data (Country -> City -> Server Load/IP) on the backend.

### Tech Stack
*   **Backend**: Go (Standard `net/http` or minimal router like `chi`).
*   **Frontend**: React + Vite + TailwindCSS.

## 3. Operational Rules for the Agent

1.  **NO UNRELATED EDITS (CRITICAL)**:
    *   **NEVER** touch code unrelated to the current specific task.
    *   **NEVER** "cleanup" or "refactor" files unless explicitly asked.
    *   If fixing `A`, verify `B`, `C`, `D` are untouched.
    *   Preserve existing comments and structure in files you *do* edit.
    *   **VERIFY ALL EDITS**: After every modification, double-check that the change is correct and that no unexpected or unrelated lines were modified elsewhere in the file.

2.  **PLAN & WAIT (NON-NEGOTIABLE)**:
    *   **ALWAYS** present a detailed plan before taking ANY action.
    *   **BRIEFLY DESCRIBE** the exact next steps you intend to take.
    *   **STOP** immediately after presenting the plan/description.
    *   **WAIT** for explicit user approval to proceed with the proposed actions.
    *   **NEVER** assume approval or skip this step, regardless of how small the task is.

3.  **Convention First**: Follow the directory structure defined in `PLAN.md`.

3.  **No External Runtime Deps**:
    *   Do not add features that require `root` privileges.
    *   Do not add external runtime dependencies (like Redis, Postgres, or Nginx).

4.  **Testing**:
    *   When implementing the Pinger, mock the `exec.Command` interface to allow testing without actual network calls.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/happydenn)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/happydenn)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
