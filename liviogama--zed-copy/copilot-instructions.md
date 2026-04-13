## zed-copy

> The workspace is rooted in `crates/`, which holds Rust crates such as `gpui` (UI toolkit), `editor`, `project`, `workspace`, and the entry-point `zed`. Shared assets live under `assets/`, while documentation and how-to guides are in `docs/`. `extensions/` contains sample and test extensions that showcase editor integrations. Automation and release scripts sit in `script/`, and configuration for tooling (e.g., Nix, Docker) is grouped under `tooling/`, `nix/`, and top-level manifests like `Cargo.toml` and `flake.nix`.

# Repository Guidelines

## Project Structure & Module Organization
The workspace is rooted in `crates/`, which holds Rust crates such as `gpui` (UI toolkit), `editor`, `project`, `workspace`, and the entry-point `zed`. Shared assets live under `assets/`, while documentation and how-to guides are in `docs/`. `extensions/` contains sample and test extensions that showcase editor integrations. Automation and release scripts sit in `script/`, and configuration for tooling (e.g., Nix, Docker) is grouped under `tooling/`, `nix/`, and top-level manifests like `Cargo.toml` and `flake.nix`.

## Build, Test, and Development Commands
- Never run `cargo run` or `cargo build`
- At the end of each action run `cargo check` and fix every errors

## Coding Style & Naming Conventions
Rust code is formatted with `cargo fmt --all` (Rustfmt defaults, 4-space indentation). Use `snake_case` for modules and functions, `UpperCamelCase` for types, and `SCREAMING_SNAKE_CASE` for constants. Prefer explicit imports within each crate and keep public APIs documented with `///` comments. When touching scripts, follow the existing shell style (set `set -euo pipefail` and use lowercase filenames).

## Testing Guidelines
Dont do any test

## Commit & Pull Request Guidelines
Follow the existing conventional subject format: `<scope>: <imperative summary>` (e.g., `feat: add split diff viewer`) and reference GitHub issues with `(#12345)` when applicable. Squash incidental fixups before pushing. Pull requests should describe the problem, outline the solution, list verification steps (`cargo test`, `script/clippy`, etc.), and attach before/after screenshots or recordings for UI-facing changes. Ensure CI passes and request reviews early if architectural guidance is needed.

## Security & Configuration Tips
When working on collaboration features, provision the local services described in `docs/src/development/local-collaboration.md` and avoid committing secrets. Keep `livekit.yaml` and other environment-specific files out of version control or encrypted via the team’s approved tooling.



📜 Rules for Converting egui_version → gpui
We work mostly in crates/diff_viewer/.

	1.	Project Structure Awareness
	•	The project of reference is located under:
crates/diff_viewer/egui_version/


	•	This crate is the reference implementation.
	•	The goal is to reimplement it using gpui (Zed’s UI framework).

	2.	Reference-Driven Conversion
	•	Treat the egui_version as the single source of truth.
	•	Do not invent new features, abstractions, or designs.
	•	Conversion must be structural and faithful, preserving the original logic and flow.
	3.	What to Convert
	•	Only convert the UI layer and related code that uses egui.
	•	Core logic (diff computation, models, data structures) should be reused as-is.
	•	Replace egui UI calls with the equivalent gpui components.
	4.	What to Avoid
	•	❌ Do not create new APIs, abstractions, or utilities unless strictly necessary for gpui.
	•	❌ Do not simplify by removing features present in egui_version.
	•	❌ Do not add speculative features or UI elements not in the reference.
	5.	Mapping Principles
	•	egui layouts (horizontal, vertical, Grid, etc.) → gpui::Flex, gpui::Split, etc.
	•	egui text rendering → gpui::Text, gpui::TextEditor.
	•	egui shapes (Painter) → gpui::Canvas.
	•	Input handling (mouse, keyboard) → routed through gpui’s event system.
	6.	Consistency & Fidelity
	•	Match the look and behavior of egui_version as closely as possible.
	•	Ensure the diff visualization (side-by-side alignment, highlights, connectors) behaves identically.
	7.	Focus Areas for Conversion
	•	Layout (side-by-side editors).
	•	Diff highlighting.
	•	Connectors/shapes between left & right panels.
	•	Scroll & input handling.
	8.	Output Expectation
	•	The resulting code should be a gpui-based version of egui_version, not a redesign.
	•	If uncertain about a gpui mapping, keep the structure from egui_version and use placeholders, with a clear note for follow-up.

⸻

⚡ In short:
“Clone the design and behavior from crates/diff_viewer/egui_version, but swap out egui APIs for gpui equivalents. No invention, no extras — just faithful conversion.”

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LivioGama) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
