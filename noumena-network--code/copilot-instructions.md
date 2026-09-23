## code

> We want to build a first-class open-source AI coding assistant: a terminal-native agent that edits code, runs commands, and helps carry multi-step development workflows. This repository is the standalone public source export of `code/` from Noumena's internal monorepo.

# Noumena Code: What We're Building

We want to build a first-class open-source AI coding assistant: a terminal-native agent that edits code, runs commands, and helps carry multi-step development workflows. This repository is the standalone public source export of `code/` from Noumena's internal monorepo.

Your goal is to keep the public CLI correct, buildable, and safe for external contributors and users who clone it and immediately ask their agents to work on it.

This means: no hidden dependencies, no "agents will figure it out" assumptions, no silent fallbacks, and no changes that break `git clone && bun install && bun run build`.

**Elegant minimalism** isn't just fewer lines; it's disciplined intent plus impeccable execution.

## Repository Contract

- This repo is **public / OSS**. Do not commit secrets, internal hosts, hardcoded IPs, or unreleased model identifiers.
- Do not add internal-only features without a clear public bypass or build-time gate.
- The default build path must remain `bun install && bun run build`. Any new required step must be explicit in `README.md`.
- Keep surfaces small: one supported path per use-case unless we are intentionally replacing the old one.

## Execution Environment

- This repo uses **Git**, not Sapling. Use `git` for history, branches, and pull requests.
- The supported build tooling is **Bun + Rust/Cargo**. Do not assume Python, Docker, Kubernetes, or Buck2 are available.
- Local builds produce a single-file native `ncode` binary under `.tmp/packages/`.
- Tests are run with `bun test <file>` or `bun run test` (isolated per-file).

## Agent Protocol (Failure Prevention)

A lot of people clone this repo and immediately have their agent install things. These rules prevent that from breaking the repo.

### Session Start (Always)

- Read `AGENTS.md`, then `CLAUDE.md` (if present).
- Confirm repo root with `git rev-parse --show-toplevel`.
- Confirm working copy state with `git status --short`.

### Mode Gates (Hard)

**No-Edits Mode**
- Trigger: user says "do not make edits/changes", "review only", or "planning/brainstorming".
- In this mode: do not modify tracked files, do not install deps, do not run destructive Git ops, and do not change state; only read/inspect/analyze.
- Exit only when the user explicitly authorizes execution ("proceed", "implement", "make the changes", "do it").

**Execution Mode**
- Default when the user asks to implement/fix/build.
- If the user says "just X", do X immediately with minimal narration.

### Scope Lock (Before Editing)

- Before editing: list the exact files you will modify.
- Do not touch out-of-scope files; stable/working code is read-only unless explicitly told otherwise.
- For ports/refactors: preserve semantics by default; call out intentional semantic deltas and get approval.
- After fixing a bug pattern: search for other occurrences (prefer `rg`) and fix them in-scope.

### Don't Guess (Ever)

- Never guess at environment state, config values, or file contents. Verify via files, diffs, logs, or commands.
- Don't assume hidden tools exist. The supported toolchain is Bun, Cargo, and `git`.

### Git Safety (High Severity)

- This is a Git repo. Do not run Sapling commands (`sl`, `sl status`, `sl diff`, etc.).
- Do not run `git reset --hard`, `git checkout -- .`, `git rebase -i`, or destructive history edits without explicit approval.
- Never `git push --force` to `main`.
- Create pull requests for non-trivial changes; avoid pushing directly to `main` unless explicitly authorized.

### Dependency / Install Discipline (Critical for OSS)

- **Do not install system packages, global npm/bun/cargo packages, or OS tooling as a fix.** If the repo genuinely needs a new dependency, add it to `package.json` or `Cargo.toml` and document why.
- **Do not modify the user's shell config** (`.bashrc`, `.zshrc`, etc.) to make the build work.
- **Do not rely on `sudo` or root privileges.** The build must work in a normal user clone.
- If a build fails because a tool is missing, check `README.md` and `OSS_BUILD.md` first. If it's not documented, treat it as a missing repo setup step and ask before adding it.
- If `bun install` does not provide a needed Node dependency, add it as a normal dependency/devDependency in `package.json`, not as a side-effect command.

### Build Discipline (High Severity)

- The canonical build is `bun run build` (alias for `bun run build:external`).
- Do not delete `.tmp/`, `dist/`, or build outputs "just to be safe".
- Do not introduce a second durable build path for the same supported workflow.
- Native Rust N-API modules live under `native/` and are built by the Bun build scripts. Do not manually invoke `cargo build` at the root as the supported path.

### Validation Discipline

- Start with the smallest relevant validation:
  - `bun test <changed-file>`
  - `bun test src/path/to/file.test.ts`
- Run the full suite only when the task touches broad cross-cutting concerns.
- For build/package changes, run `bun run build` end-to-end and verify the binary path printed in the manifest.

### Output Completeness

- If asked for "full diff/log output", do not truncate.
- Provide complete copy/paste-ready commands (include `cd`, env vars, and flags).
- When giving build/test commands, prefer the Bun scripts in `package.json` over raw tool invocations.

### Security / Leak Prevention

- Do not add internal hostnames, IP addresses, private gateway URLs, or unreleased model identifiers to public code or tests.
- If a fixture or test needs a placeholder endpoint, use `.invalid` or `.test` domains or noumena-network/code-local paths.
- Internal-only model profiles should be commented out, not removed, if upstream sync requires them; mark them clearly as internal-only.

## Craftsmanship Rubric for Any Change

- Intent: Does this improve correctness, stability, performance, or clarity for OSS users?
- Uniqueness: Are we creating a second way to do something? If yes, why?
- Surface: Did we add a new public knob? Could it be expressed via existing config?
- Repro: Is config/provenance captured to rerun months later on a fresh clone?
- Elegance: Is the code visibly simpler afterward?

## Build Modes

Choose a build mode before building; the mode is baked into the binary.

| Mode | Command | Intended use |
| --- | --- | --- |
| `external` | `bun run build:external` | Default OSS build. Public-safe gates, Noumena OAuth, Noumena API keys, and BYOK. |
| `noumena` | `bun run build:noumena` | Noumena first-party/product build with Noumena compatibility features enabled. |
| `dev` | `bun run build:dev` | Contributor/debug build that enables development/internal capability gates. |
| `internal` | `bun run build:internal` | Internal compatibility spin for Noumena-controlled environments. |

Set the build mode at build time with `NCODE_USER_TYPE`:

```bash
# Default OSS build (public-safe gates).
NCODE_USER_TYPE=external bun run build

# Noumena first-party build (enables managed-model aliases, first-party UI
# surfaces, and other Noumena compatibility features).
NCODE_USER_TYPE=noumena bun run build
```

## Local Environment

- Repository root: `/mlstore/src/noumena-network/code` (when working in the internal mirror; external users use their own clone path).
- Required:
  - Node.js 18+
  - Bun 1.3.10+
  - Rust/Cargo
- Optional for tests: `tmux` (for PTY/tmux integration tests).

## Git Quick Commands

```bash
cd /mlstore/src/noumena-network/code
git rev-parse --show-toplevel
git status --short
git diff -- <paths>
git log --oneline -3
```

## Build And Validation Quick Commands

```bash
cd /mlstore/src/noumena-network/code
bun install
bun run build
bun test src/path/to/file.test.ts
bun run test
```

## What This Repo Is NOT

- It is not the full internal Noumena monorepo. Buck/Sapling/Mononoke/Eden integration, internal runbooks, and staging launchers are intentionally excluded.
- It is not a training/kernels repo. GPU training, kernels, and inference hot-path work belong in `noumena`.
- It is not a generic AI assistant framework. Keep changes scoped to the terminal coding assistant product.

Our ethos: ship a clean, buildable, public CLI that people can clone and work on without their agent breaking it.

---
> Source: [Noumena-Network/code](https://github.com/Noumena-Network/code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
