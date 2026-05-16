## temporal-playground

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Polyglot playground for Temporal workflows across Go, TypeScript, Python, Java, .NET, and Ruby SDKs.

## Setup

All toolchain dependencies are managed by [mise](https://mise.jdx.dev/) (see `mise.toml`):

```sh
mise install
```

## Commands (via just)

```sh
just server              # Temporal dev server on localhost:7233
just server-ui           # same, with Web UI on port 8080
just versions            # print all installed tool versions
just history-export WID  # export workflow history JSON (optional: run-id)
just history-replay FILE # replay from saved history
just history-list        # list saved history files
```

Override namespace/address: `just --set namespace prod --set address host:7233 <recipe>`

## Scaffolding TypeScript Projects

Bootstrap from the [official samples repo](https://github.com/temporalio/samples-typescript):

```sh
just new-ts sample=hello-world          # defaults project name to sample name
just new-ts sample=saga name=my-saga    # explicit project name
```

If the directory already exists, the project name auto-increments (`schedules`, `schedules-2`, `schedules-3`, …).

Or let Claude pick the right sample based on what you want to build:

```
/new-ts <describe what you want to build>
```

Claude reads `ts/samples-catalog.md`, selects the best sample, scaffolds the project, and explains the code.

## Scaffolding Rust Projects

Generate a hello-world Temporal project (no official samples repo exists for Rust yet):

```sh
just new-rust                              # defaults to "hello-world"
just new-rust my-project                  # explicit project name
just new-rust my-project 0.1.0-alpha.1   # pin SDK version
just new-rust my-project 0.1.0-alpha.1 1.85.0  # pin Rust toolchain version too
```

If the directory already exists, the project name auto-increments. Each generated project includes its own `justfile`:

```sh
cd rust/hello-world
just worker   # start the Worker (separate terminal)
just start    # run the Workflow starter
just build    # cargo build
just check    # cargo clippy
```

Or let Claude generate and explain the project:

```
/new-rust <describe what you want to build>
```

Note: The Temporal Rust SDK is prerelease (alpha). See https://github.com/temporalio/sdk-core/blob/master/crates/sdk/README.md

## Structure

- `justfile` — all dev commands
- `mise.toml` — toolchain versions (Go, Node, Python, Java, .NET, Ruby, Rust, Temporal CLI, uv, pnpm, gradle)
- `histories/` — exported workflow history JSON files (git-ignored)
- `.cache/` — shallow clones of Temporal samples repos (git-ignored, auto-created)
- `ts/` — TypeScript projects (subdirectories git-ignored); `ts/samples-catalog.md` is tracked
- `rust/` — Rust projects (subdirectories git-ignored)
- `.claude/commands/new-ts.md` — `/new-ts` slash command prompt
- `.claude/commands/new-rust.md` — `/new-rust` slash command prompt
- `.claude/rust-project-gen.py` — helper script that generates Rust project files

## Adding a New Language

1. Create `{lang}/samples-catalog.md` from `temporalio/samples-{lang}`
2. Copy `.claude/commands/new-ts.md` → `.claude/commands/new-{lang}.md`, update lang references
3. Replace the `new-{lang}` stub in `justfile` — call `just _clone-samples {lang}`, copy the sample, run language-specific install (`uv sync` for Python, `go mod tidy` for Go, etc.)

---
> Source: [andrewspinks/temporal-playground](https://github.com/andrewspinks/temporal-playground) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
