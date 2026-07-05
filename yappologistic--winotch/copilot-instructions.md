## winotch

> Winotch is a native Windows desktop notch: a centered, top-attached black overlay inspired by the MacBook notch. It shows time, date, battery, Wi-Fi, volume, and recent notifications with polished motion.

# Winotch Agent Guide

## Project
Winotch is a native Windows desktop notch: a centered, top-attached black overlay inspired by the MacBook notch. It shows time, date, battery, Wi-Fi, volume, and recent notifications with polished motion.

## Stack
- Use C# and WPF on `net8.0-windows` for direct Windows desktop integration, transparent always-on-top windows, and repeatable CLI build/run loops.
- Prefer Windows built-in APIs and .NET libraries before adding packages.
- Keep the app unpackaged unless a later feature needs MSIX capabilities.

## Commands
- Build: `dotnet build`
- Run: `dotnet run --project src/Winotch/Winotch.csproj`
- Test: `dotnet test`
- CI parity: `dotnet restore`, `dotnet build --no-restore -warnaserror`, `dotnet test --no-build --verbosity normal`

## Design
- Keep design tokens in one place and reuse them.
- Use Segoe UI Variable / Segoe UI first; it is the closest native Windows equivalent to Apple's San Francisco without bundling proprietary fonts.
- Keep the notch centered across resolution changes and monitor changes.
- Prefer small, focused components over speculative architecture.

## Git
- Do not commit build outputs, user settings, logs, packages, or local IDE files.
- Do not commit screenshots, videos, or locally published EXE folders.
- Do not commit QA screenshots, screen recordings, captured clipboard images, or temporary Codex/agent artifacts.
- Before GitHub publication, scan tracked files and history for local artifacts; rewrite history only when artifacts are actually reachable.
- Commit coherent milestones: project setup, core shell/UI, OS integrations, docs/verification.
- Keep GitHub docs source-only while alpha: no release workflows, tags, or published binaries unless explicitly requested.

## Privacy
- Document clipboard, notification, camera, and local settings behavior plainly before changing those surfaces.
- Do not add telemetry or network calls beyond user-provided calendar URLs without explicit approval.

---
> Source: [yappologistic/winotch](https://github.com/yappologistic/winotch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-05 -->
