## twine

> Twine is a macOS-first agent workspace. SwiftUI and AppKit present the UI; Rust owns the reusable application core. The terminal is the primary interaction surface.

# Twine

Twine is a macOS-first agent workspace. SwiftUI and AppKit present the UI; Rust owns the reusable application core. The terminal is the primary interaction surface.

Use the terms defined in [CONTEXT.md](CONTEXT.md) when naming folder, session, workflow, role, harness, agent, or trace concepts in code or UI.

## Architecture

- Put any new Rust crates at the workspace root when they need a separate package.
- Keep the Swift–Rust boundary explicit: commands go in, and state changes, trace events, and terminal bytes come out.
- Keep the three layers independent:
  - `twine-core` exposes a plain Rust API with no knowledge of FFI or UI. Every core feature is testable in Rust without the app.
  - `twine-bridge` only translates between the core API and the C ABI. It holds no domain logic or state.
  - Swift talks to `twine-core` only through `CoreClient`.
- Domain state (folders, sessions, workflows, traces, transcripts, config) lives in core. Presentation state (selected tabs, pane layouts, form prefills) lives in Swift.
- Model workflow types as ways to coordinate agents. The workflow graph is not a general DAG executor.

## Component instructions

- Run builds, formatting, linting, and tests through the root `Makefile`. `make` lists the targets.
- For Rust core work in `twine-core/`, read [twine-core/AGENTS.md](twine-core/AGENTS.md).
- For C ABI work in `twine-bridge/`, read [twine-bridge/AGENTS.md](twine-bridge/AGENTS.md).
- For Swift app work in `macOS/`, read [macOS/AGENTS.md](macOS/AGENTS.md).
- For changes spanning components, follow each component's file, and run `make fmt` and `make check` in place of the per-component targets. Give any new component its own `AGENTS.md` when it needs distinct instructions.

## Code style guidelines

- **Rust unit tests:** Unit tests must be inline in a `#[cfg(test)] mod tests` block in the same source file as the code they test.
- **Branches:** `main` accepts changes only through pull requests, and force pushes are blocked. Do your work on a local branch. Never push, open a pull request, or merge without the user's explicit approval.
- **Commits:** Use Linux kernel style (`scope: imperative command`). Write the subject as an
  instruction to edit the repository. A complete subject names both the codebase artifact and the
  edit applied to it, using a repository-edit verb such as `add`, `move`, `split`, `extract`,
  `replace`, `remove`, or `rename`. Keep the message as a short subject line. You can add at most one or
  two body lines if absolutely necessary.
  - **Examples of good commit messages:**
    - `api: add pagination middleware to collection endpoints`
    - `database: split customer addresses into normalized tables`
    - `web: extract checkout form into reusable component`
  - **Reasoning:** States the repository mutation directly, so the commit history reads as a
    sequence of concrete operations that transformed the project.

---
> Source: [aravind-n/twine](https://github.com/aravind-n/twine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
