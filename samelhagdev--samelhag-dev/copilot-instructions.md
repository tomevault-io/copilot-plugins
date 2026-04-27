## samelhag-dev

> ﻿# SamElhagPersonalSite Development Guidelines

﻿# SamElhagPersonalSite Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-02-20

## Active Technologies
- JavaScript (ES2020+) + C# / .NET 10 (Blazor Server) + Browser Canvas API (`CanvasRenderingContext2D`, `ImageData`, (001-cfd-simulation-performance)
- N/A — all state is in-memory JavaScript arrays within the simulation loop (001-cfd-simulation-performance)
- C# 13 / .NET 10 — Blazor Server (Interactive Server render mode) + MudBlazor 8.x — `MudPaper`, `MudText`, `MudChip`, `MudChipSet`, `MudIcon`; Bootstrap 5.x grid utilities for responsive wrapper (002-about-conveyor)
- N/A — all milestone data is static, code-managed in `About.razor @code` block (002-about-conveyor)
- C# 13 / .NET 10 + MudBlazor 8.x (existing), MailKit (new — industry-standard .NET email library, replaces obsolete `System.Net.Mail.SmtpClient`) (004-contact-email-send)
- N/A — no database; emails are delivered directly via SMTP (004-contact-email-send)

- [e.g., Python 3.11, Swift 5.9, Rust 1.75 or NEEDS CLARIFICATION] + [e.g., FastAPI, UIKit, LLVM or NEEDS CLARIFICATION] (001-cfd-simulation-performance)

## Project Structure

```text
backend/
frontend/
tests/
```

## Commands

cd src; pytest; ruff check .

## Code Style

[e.g., Python 3.11, Swift 5.9, Rust 1.75 or NEEDS CLARIFICATION]: Follow standard conventions

## Recent Changes
- 004-contact-email-send: Added C# 13 / .NET 10 + MudBlazor 8.x (existing), MailKit (new — industry-standard .NET email library, replaces obsolete `System.Net.Mail.SmtpClient`)
- 002-about-conveyor: Added C# 13 / .NET 10 — Blazor Server (Interactive Server render mode) + MudBlazor 8.x — `MudPaper`, `MudText`, `MudChip`, `MudChipSet`, `MudIcon`; Bootstrap 5.x grid utilities for responsive wrapper
- 001-cfd-simulation-performance: Added JavaScript (ES2020+) + C# / .NET 10 (Blazor Server) + Browser Canvas API (`CanvasRenderingContext2D`, `ImageData`,


<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SamElhagDev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
