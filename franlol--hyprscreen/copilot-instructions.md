## hyprscreen

> Hyprscreen is a Hyprland-only Rust desktop app for screenshots and screen recording.

# Hyprscreen Agent Guide

## Project

Hyprscreen is a Hyprland-only Rust desktop app for screenshots and screen recording.

## Global Rules

- Keep the main product flow inside one window.
- Extra windows are limited to the recording HUD and transient identification overlays (e.g. monitor identifier labels during `Monitor` selection).
- `Preview` is an internal state, not a top-level tab.
- `Back` from `Preview` clears the temporary result and returns to `Setup`.
- Keep preview actions consistent across screenshots and recordings.
- Do not allow hidden-HUD recording without a verified stop shortcut.
- Prefer minimal, direct changes over extra abstraction.

## Repo Map

- Product context: `docs/product.md`
- Architecture overview: `docs/architecture.md`
- UI scope: `docs/scopes/ui.md`
- Capture scope: `docs/scopes/capture.md`
- Recording scope: `docs/scopes/recording.md`
- Hyprland integration: `docs/scopes/hyprland.md`
- Config scope: `docs/scopes/config.md`
- CLI scope: `docs/scopes/cli.md`
- Packaging notes: `docs/scopes/packaging.md`

## ADRs

Read `docs/adr/` before changing core product behavior.

## Build Notes

- System dependencies are required for GTK4.
- External tools planned for v0.1: `grim`, `slurp`, `wf-recorder`, `wl-clipboard`.

---
> Source: [franlol/hyprscreen](https://github.com/franlol/hyprscreen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-24 -->
