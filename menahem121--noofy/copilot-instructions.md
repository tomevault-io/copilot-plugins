## noofy

> This project is a local AI workflow app for Linux, Windows, and macOS. It should make powerful AI workflows usable by beginners: a user opens a simple dashboard, presses clear controls, and the app runs the local AI workflow in the background.

# Agent Entry Point

This project is a local AI workflow app for Linux, Windows, and macOS. It should make powerful AI workflows usable by beginners: a user opens a simple dashboard, presses clear controls, and the app runs the local AI workflow in the background.

The app is desktop-first, privacy-focused, and powered first by ComfyUI. ComfyUI should be treated as the first execution engine, not as the public contract of the app.

## Core Rule

The frontend must call the app backend API. It must not call ComfyUI directly.

The backend owns the `EngineAdapter` contract and translates app workflow requests into the active engine implementation. The first adapter is `ComfyUIEngineAdapter`; future adapters may target platform-native inference paths.

## Documentation Index

- [README.md](README.md): public project introduction and high-level vision.
- [DESIGN_SYSTEM.md](DESIGN_SYSTEM.md): frontend visual direction, component rules, UI copy guidance, and interaction principles.
- [docs/README.md](docs/README.md): compact docs index and status map.
- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md): stack, process boundaries, and major architecture decisions.
- [docs/ENGINE_CONTRACT.md](docs/ENGINE_CONTRACT.md): app-owned engine operations and job lifecycle.
- [docs/WORKFLOW_PACKAGES.md](docs/WORKFLOW_PACKAGES.md): workflow package, required model, input binding, output, and dashboard schema concepts.
- [docs/DASHBOARD_ARCHITECTURE.md](docs/DASHBOARD_ARCHITECTURE.md): workflow import, dashboard authoring, canvas run view, user state, and dashboard assets.
- [docs/MODEL_RESOLUTION_AND_DOWNLOADS.md](docs/MODEL_RESOLUTION_AND_DOWNLOADS.md): Noofy Models folder, Hugging Face/Civitai API keys, staged import preview, provider resolver, download transactions, background download job, and session TTL.
- [docs/RUNTIME_ISOLATION_ARCHITECTURE.md](docs/RUNTIME_ISOLATION_ARCHITECTURE.md): accepted runtime isolation architecture for community workflows, workflow capsules, runner processes, and trust boundaries.
- [docs/NOOFY_VERIFIED_PUBLISHING.md](docs/NOOFY_VERIFIED_PUBLISHING.md): Noofy Verified publishing, signing, validation gates, and revocation process.
- [docs/OS_SANDBOXING_FEASIBILITY.md](docs/OS_SANDBOXING_FEASIBILITY.md): OS sandboxing feasibility and product-claim boundaries for community workflow execution.
- [docs/MEMORY_GOVERNOR.md](docs/MEMORY_GOVERNOR.md): v1 Memory Governor strategy for RAM/VRAM estimates, runner co-residence, eviction, retry, UI states, diagnostics, and remaining hardware-validation limits.
- [docs/MANAGED_COMFYUI_SIDECAR.md](docs/MANAGED_COMFYUI_SIDECAR.md): v1 requirement for app-managed ComfyUI startup, isolation, logs, health, and shutdown.
- [docs/PACKAGED_RUNTIME.md](docs/PACKAGED_RUNTIME.md): packaged Python and bundled uv runtime artifact layout, manifest, verification, and CI release gate.
- [docs/COMFYUI_UPDATES.md](docs/COMFYUI_UPDATES.md): user-managed upstream ComfyUI update flow, local validation, activation, and rollback safety.
- [docs/FEEDBACK_TESTING_MONITORING.md](docs/FEEDBACK_TESTING_MONITORING.md): diagnostics, automated tests, progress feedback, logs, and monitoring direction.
- [docs/MODEL_COMPATIBILITY_PLAN.md](docs/MODEL_COMPATIBILITY_PLAN.md): active future plan for model identity scanning and compatible LoRA discovery.

## Working Guidance

When adding frontend features, preserve the backend API boundary.

When adding or changing frontend UI, follow [DESIGN_SYSTEM.md](DESIGN_SYSTEM.md) for visual direction, component behavior, interaction states, and beginner-friendly copy.

When adding engine behavior, implement it through an `EngineAdapter`.

External ComfyUI URLs are development mode only. The v1 product must use an app-managed, isolated ComfyUI sidecar so users do not manually launch ComfyUI or install its Python dependencies.

Community workflows from the internet are a first-class product direction. When adding community workflow behavior, follow the accepted runtime isolation architecture: install custom nodes and Python dependencies only into isolated dependency environments and runner workspaces, never into the trusted core runtime.

The trusted backend process must never import community custom node modules or execute custom node setup code. Custom node imports, compatibility checks, and smoke tests must happen only inside isolated runner processes.

Unverified community workflows may be prepared automatically only when Noofy can resolve them into isolated workflow capsules. Noofy protects the app from dependency conflicts and broken installs, but it must not claim arbitrary community Python code is safe or trustworthy.

Workflow model validation must use the active `EngineAdapter`. Do not validate required models by reading a hardcoded local ComfyUI source `models` folder.

When adding backend behavior, add structured diagnostics for success, failure, and important state transitions so the UI and future agents can understand what happened.

When adding or changing meaningful behavior, add automated tests for the success path and the likely failure path. Also add or update feedback/monitoring surfaces when the behavior affects workflow execution, model validation, ComfyUI communication, sidecar lifecycle, progress, errors, or user-visible state.

When adding workflow behavior, prefer workflow packages and dashboard schema over hardcoded single-purpose flows.

When in doubt during early MVP work, keep the implementation small: start with one text-to-image workflow that proves the architecture. This is not a long-term product limit.

When adapting large files from `third_party/comfyui/`, do not manually rewrite big chunks. If most of a file is needed, copy it with a terminal command into the app-owned codebase, rename it clearly, and then make focused edits from there. Keep ComfyUI-derived files identifiable and listed for maintenance reasons.

### Testing

Prefer focused tests for normal changes. Run only the tests that cover the area you modified.

For focused backend tests, use:
backend/.venv/bin/python -m pytest <test-path>

Use the full repo-level test suite only when the change is broad, low-level, or risky enough that it could affect multiple parts of the app.

```bash
make test

---
> Source: [menahem121/Noofy](https://github.com/menahem121/Noofy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
