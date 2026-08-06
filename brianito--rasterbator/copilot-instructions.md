## rasterbator

> Rasterbator is a private, offline, macOS-first desktop application for splitting one local image into a dimensionally accurate, multi-page A4 mosaic PDF. Version 1 exports the original image only, with configurable blank cutting margins, vector cutting guides, optional overlap, and page labels. The source aspect ratio is immutable in MVP: fitting may crop or add unused poster area, but it must never stretch or squash the image.

# Rasterbator Agent Guide

## Introduction

Rasterbator is a private, offline, macOS-first desktop application for splitting one local image into a dimensionally accurate, multi-page A4 mosaic PDF. Version 1 exports the original image only, with configurable blank cutting margins, vector cutting guides, optional overlap, and page labels. The source aspect ratio is immutable in MVP: fitting may crop or add unused poster area, but it must never stretch or squash the image.

The planned stack is Tauri 2, Rust, React, TypeScript, Vite, and Tailwind CSS. The Rust geometry engine is the canonical source of truth for both preview and PDF export. Do not duplicate output geometry in the frontend.

## Source of Truth and References

`AGENTS.md` is the source of truth for agent and contributor instructions. Supporting documents provide detailed requirements but do not override this guide. When an intentional decision changes these instructions, update `AGENTS.md` and every affected supporting document in the same change.

### Project specifications

Load only the context required for the current task. Search for the relevant section before reading a whole supporting document, and do not reread an unchanged document in the same session.

- **Scope or architecture changes:** read [PLAN.md](./PLAN.md), then every supporting document affected by the decision.
- **Rust geometry, validation, or page mapping:** read the relevant geometry and command sections in [PLAN.md](./PLAN.md) plus the Rust test layers in [TESTS.md](./TESTS.md).
- **Image decoding, rendering, or PDF export:** read the relevant processing/PDF sections in [PLAN.md](./PLAN.md), the export sequence in [USER_FLOW.md](./USER_FLOW.md), and the applicable integration-test sections in [TESTS.md](./TESTS.md).
- **React UI, styling, accessibility, or interaction:** read the relevant sections in [DESIGN.md](./DESIGN.md) and [USER_FLOW.md](./USER_FLOW.md), plus applicable frontend test sections in [TESTS.md](./TESTS.md).
- **Packaging, repository, documentation, or release mechanics:** read the files being changed and the release/manual-gate sections in [TESTS.md](./TESTS.md). Read product documents only when release scope or claims change.
- **Small isolated fixes:** inspect the affected code, its nearest tests, and only the specification section governing that behavior.

Reference documents:

- [Implementation plan](./PLAN.md) — product scope, architecture, geometry, milestones, and definition of done.
- [UI design plan](./DESIGN.md) — layout, components, styling, accessibility, motion, and design acceptance criteria.
- [User flow](./USER_FLOW.md) — complete import-to-export journey, failure paths, keyboard flow, and flow acceptance criteria.
- [Test strategy](./TESTS.md) — canonical automated test layers, fixtures, milestone coverage, CI gates, and manual release checks.

When supporting documents appear to conflict, use `PLAN.md` for product and technical scope, `DESIGN.md` for presentation, `USER_FLOW.md` for interaction sequencing, and `TESTS.md` for testing implementation. `AGENTS.md` takes precedence for agent and contributor behavior. Record unresolved conflicts before implementing assumptions.

### External documentation

- [Tauri 2](https://v2.tauri.app/)
- [React](https://react.dev/)
- [TypeScript](https://www.typescriptlang.org/docs/)
- [Vite](https://vite.dev/guide/)
- [Tailwind CSS](https://tailwindcss.com/docs)
- [Rust `image` crate](https://docs.rs/image/)
- [Rust `printpdf` crate](https://docs.rs/printpdf/)
- [Rust `pdf-writer` crate](https://docs.rs/pdf-writer/)
- [Motion for React](https://motion.dev/docs/react)
- [Remix Icon React](https://github.com/Remix-Design/RemixIcon)

## Workflow

1. **Review the specifications**
   - Identify the relevant milestone and acceptance criteria.
   - Keep MVP work limited to macOS, A4, offline processing, and original-image output.
   - Update the documents when an intentional product or architecture decision changes.

2. **Work milestone by milestone**
   - Complete Milestone 0 technical spikes before committing to a PDF crate.
   - Build and test the Rust geometry engine before reproducing its results in the UI.
   - Implement import and settings before preview, then implement export and hardening.
   - Do not begin deferred effects, other paper sizes, or other platforms during MVP work.

3. **Implement in small vertical slices**
   - Define or update canonical Rust types and validation.
   - Add geometry/processing tests before wiring Tauri commands.
   - Expose a small, typed command boundary.
   - Mirror or generate TypeScript contracts without copying geometry formulas.
   - Add UI behavior, translated English strings, validation, loading, and error states.

4. **Validate every change**
   - Follow the test layers, fixture policy, milestone coverage, and gates in [TESTS.md](./TESTS.md).
   - Add or update tests at the narrowest useful layer for every behavior change and regression fix; do not duplicate canonical geometry formulas in test helpers or TypeScript.
   - Run Rust formatting, linting, and tests: `cargo fmt --check`, `cargo clippy`, and `cargo test`.
   - Run frontend formatting, linting, type checking, and tests using the package scripts established during scaffolding.
   - Check keyboard operation, focus visibility, reduced motion, and non-color status cues.
   - Confirm preview data comes from the same Rust geometry used by export.
   - For PDF changes, verify structure and rasterized output, including page size, margin and guide coordinates, page order, seams, cancellation, atomic replacement, and temporary-file cleanup.
   - Report checks that were run and any checks that could not be run.

5. **Perform physical acceptance checks where required**
   - Print at Actual size / 100%.
   - Measure the A4 page, retained tile, margin, and guide positions.
   - Cut and assemble the 2 × 2 fixture to confirm continuity and page ordering.
   - Record test conditions and results; automated checks do not replace this release gate.

6. **Keep the repository clean**
   - Commit lockfiles, intentional fixtures, and tests.
   - Do not commit personal source images, generated PDFs, temporary renders, logs, secrets, or build output.
   - Keep processing logic out of React components and arbitrary shell access out of Tauri capabilities.
   - Treat image files, settings, and paths as untrusted input and enforce resource/page limits in Rust.

## TODO

This checklist reflects the repository state at the time this guide was created.

### Done

- [x] Define version 1 goals, scope, exclusions, and confirmed product decisions.
- [x] Select the proposed Tauri 2 + Rust + React/TypeScript architecture.
- [x] Specify geometry, rendering, PDF, security, performance, and testing requirements.
- [x] Define the desktop UI, visual language, motion, accessibility, and responsive behavior.
- [x] Document the primary, returning-user, keyboard, cancellation, and failure flows.
- [x] Define implementation milestones and version 1 acceptance criteria.
- [x] Add this contributor/agent workflow.
- [x] Define the automated test strategy and testing source-of-truth rules.

### Next — Milestone 0: technical spikes

- [x] Initialize version control and add an appropriate `.gitignore`.
- [x] Scaffold Tauri 2, React, TypeScript, Vite, and Tailwind CSS.
- [x] Establish strict TypeScript, Rust formatting/linting, and frontend quality scripts.
- [x] Confirm PNG, JPEG, and WebP decoding plus EXIF orientation handling.
- [x] Spike one-page and 2 × 2 A4 mosaic PDF output.
- [x] Add vector cutting guides at a known physical margin.
- [x] Compare `printpdf` with `pdf-writer` and document the selected crate.
- [ ] Verify PDF dimensions in Preview, Chrome, and Acrobat Reader.
- [x] Benchmark sequential rendering at 150 and 300 DPI.
- [ ] Print and measure the 2 × 2 technical-spike fixture.

### Remaining MVP

- [x] Implement canonical project settings, validation, units, layout, fitting, alignment, overlap, and tile mapping in Rust.
- [ ] Add Rust unit tests, deterministic fixtures, golden-image tests, and PDF integration tests.
- [ ] Implement constrained Tauri commands, capabilities, stable errors, progress events, cancellation, and stale-preview protection.
- [x] Build image import/drop handling, metadata, remembered non-image preferences, and the English translation catalog.
- [x] Build poster, paper, fit, cutting, assembly, output, validation, and page-limit controls.
- [x] Build poster and sheet previews with canonical geometry, zoom/pan, labels, boundaries, and accessible overlays.
- [ ] Implement sequential PDF export with blank margins, vector guides, labels, overlap, metadata, and atomic output replacement.
- [ ] Add export progress, cancellation, success/failure states, and Reveal in Finder.
- [ ] Enforce file, decoded-image, memory, disk, and 100/400-page safeguards.
- [ ] Complete keyboard, screen-reader, reduced-motion, reduced-transparency, contrast, and minimum-window checks.
- [ ] Add CI for formatting, linting, tests, and macOS builds.
- [ ] Add app icons and reproducible unsigned local macOS packaging.
- [ ] Pass the final physical 2 × 2 cut-and-assemble acceptance test.

### Deferred until after MVP

- [ ] Portuguese localization toggle.
- [ ] Windows and Linux releases.
- [ ] Additional/custom paper sizes.
- [ ] Grayscale, halftone, dithering, alternate shapes, and color effects.
- [ ] Advanced glue flaps, borderless mode, page-image export, and project files.
- [ ] Signed/notarized installers, distribution, and automatic updates.

---
> Source: [BrianIto/rasterbator](https://github.com/BrianIto/rasterbator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
