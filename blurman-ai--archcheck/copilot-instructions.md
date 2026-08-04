## archcheck

> Guide for coding agents working in the `archcheck` repository.

# AGENTS.md

Guide for coding agents working in the `archcheck` repository.

> For Claude Code, the primary source of instructions is [CLAUDE.md](CLAUDE.md).
> This file is a short agent-neutral summary; when details diverge,
> priority goes to the canonical docs it references.

## Language and working style

- Reply to the user in Russian, warmly and personally.
- Write concisely, to the point, without unnecessary chatter.
- Source priority on conflict:
  - **what is actually shipped** → [CHANGELOG.md](CHANGELOG.md) (authoritative on versions);
  - **product framing and architecture** → [docs/architecture-spec.md](docs/architecture-spec.md);
  - then [docs/MVP.md](docs/MVP.md), then [README.md](README.md).

## Project status

- **v0.1 in active development.** The repository is implemented and builds: there are
  `src/`, `include/archcheck/`, `tests/`, `CMakeLists.txt`, CI on GitHub Actions.
- The core pipeline works: fast preprocessor scan → include graph → default rules
  → reporters. The binary ships SF.7/8/9 + Lakos (cycles, god-headers, chain length),
  baseline, drift, and diff. The exact shipped set is in [CHANGELOG.md](CHANGELOG.md).
- The product name is **locked**: `archcheck` (binary `archcheck`). The working
  directory stays `cpparch` for path stability.

## What this product is

`archcheck` is a **CI-first CLI** for enforcing architectural invariants on C++ projects.

The tool:

- works preprocessor-only, without `compile_commands.json` (include rules);
  reading `compile_commands.json` for semantic rules is a v0.2 plan (libclang);
- builds the include dependency graph (by AST — later, v0.2);
- applies a set of default rules; YAML-described module rules are parsed
  and validated, but not yet enforced (v0.2);
- reports violations as `file:line` (column — future semantic backend);
- exits with a non-zero code on violations.

The positioning is deliberate:

- **not** "ArchUnit for C++";
- but **Lakos physical design + C++ Core Guidelines SF.\* checks in CI**.

Every default rule carries attribution (Core Guidelines / Lakos / Martin).
Preserve this framing in docs, messages, and feature descriptions.

## What it does NOT do

- not a linter (clang-tidy);
- not a bug finder (PVS/Coverity/cppcheck);
- not a formatter (clang-format);
- not an include optimizer (IWYU);
- not a GUI, not a web dashboard, not an IDE extension.

If a new idea contradicts this list — stop and check whether it is needed at all.

## Mandatory reading before non-trivial work

- [CLAUDE.md](CLAUDE.md) — working principles and commands;
- [docs/architecture-spec.md](docs/architecture-spec.md) — the main source on the product;
- [docs/MVP.md](docs/MVP.md) — the boundaries of the MVP;
- [docs/code_style.md](docs/code_style.md) — C++20 style (**the single source of truth on style**);
- [docs/code_quality.md](docs/code_quality.md) — anti-slop constraints and thresholds;
- [docs/dev/git_workflow.md](docs/dev/git_workflow.md) — the git process;
- [backlog/README.md](backlog/README.md) — the task lifecycle.
- [docs/openwiki/index.md](docs/openwiki/index.md) — agent navigation map (source-backed file/test/fixture pointers per rule/feature). **Derived** — never trust it over code/tests/CHANGELOG; update affected pages in the same PR. Health check: `/openwiki-check`.

Before editing an existing file, read it in full.

## Architecture

Pipeline:

```text
scan → graph → rules → report
```

Subsystems under `src/` (existing):

- `config/` — YAML loader → `Config`;
- `scan/` — `include_scanner` (fast, preprocessor-only, shipped) and
  `clang_scanner` (libclang, semantic rules — v0.2);
- `graph/` — DAG, cycles, levelization, CCD/ACD/NCCD;
- `rules/` — one rule = one class = one file, grouped by source;
- `report/` — text/json reporters (sarif — later).

Key invariants:

- The `include_scanner` / `clang_scanner` split is **deliberate**. Don't collapse it without discussion.
- **One rule = one class = one file.** Registration via a static table;
  adding a rule must not touch existing rule files (OCP).
- Don't build a "generic rule engine".

## Technology constraints

- Language **C++20**; build **CMake (Ninja)**; dependencies via FetchContent.
- YAML: **`ryml`**. Tests: **Catch2 v3**. CI: GitHub Actions.
- libclang/libtooling — for the future semantic backend.
- **Dependencies — minimum.** No Boost, no heavy graph libraries:
  `unordered_map<NodeId, vector<NodeId>>` is sufficient.
- Distribution: a single static binary per platform + Docker image.

## Scope boundaries

Without an explicit request, do NOT add: AST-based rules beyond the plan, a plugin system,
visualization, automatic architecture inference, IDE integrations. These are explicitly out of the product.

## Quality and design rules

Principles: **YAGNI**, **Authority over opinion**, **Zero-config first**,
**Deterministic output**, **Boring tech**.

Practice:

- first check whether the task can be solved by deleting code;
- first look for an existing entity before creating a new one;
- don't add abstractions "for growth"; don't refactor code you don't understand;
- don't do a large refactor in the middle of a feature without separate agreement.

Hard thresholds are in [docs/code_quality.md](docs/code_quality.md) (function ≤ 30 lines,
class ≤ 300, parameters ≤ 4, nesting ≤ 3, ≤ 50 new lines per change without
tests/fixtures, ≤ 2 new files, ≤ 1 new class, 0 abstractions without a request).
Before pushing, run `lizard --CCN 15 --length 30 --arguments 5 --warnings_only src/ include/ tests/`.

## C++ code style

**The single source of truth is [docs/code_style.md](docs/code_style.md). Read it, don't duplicate it.**

The most common points (exact wording and rationale — in code_style.md):

- Allman braces, 2 spaces, lines ≤ 120, UTF-8 without BOM, Unix newlines.
- Namespaces `lower_snake_case`; types `PascalCase`; **methods and free functions
  `lowerCamelCase`**; locals/parameters `lowerCamelCase`.
- **Class fields — trailing underscore: `name_`** (not `_name`, not `m_name`).
  Fields of `struct` data-carriers — without an underscore.
- `constexpr` constants `kPascalCase`; macros `UPPER_SNAKE_CASE`.
- **For new code we don't prefix interfaces with `I`** — a pure-virtual base is
  an ordinary `class` (`Rule`, not `IRule`). (There is legacy `IRule` in the code — it is not a target
  for new code; a mass rename is out of scope for an ordinary task.)
- No `using namespace` in headers; headers self-contained; `#pragma once`;
  no anonymous namespaces in headers.
- Prefer `[[nodiscard]]`, `noexcept`, `string_view`/`span`, `optional`/`variant`,
  `ranges`, `concepts`, RAII, and the standard library.

## Rules and fixtures

A rule without fixtures does not exist. Every new rule must have
`fixtures/<rule>/pass/` and `fixtures/<rule>/fail_*/`. If a feature cannot be tested
through fixtures — it is not implemented.

Every default rule must have attribution (Core Guidelines / Lakos / Martin).
Without a source, it is not a default but an optional lower-priority check.

## Dogfooding

`archcheck` must pass its own rules in CI (no cycles, SF.7/8/9/21,
no god-headers, etc.). Any merge that breaks its own rules is unacceptable.

## CLI contracts

Exit codes (a contract — don't change without versioning):

- `0` — OK
- `1` — violations found
- `2` — config / parsing error
- `3` — internal error

## Workflow

Tasks live in `backlog/`, one `.md` per task: `new/` — filed,
`wip/` — in progress, `completed/` — done and turned into documentation
(sections "how it works / what controls it / what it relates to / diagnostics").
Don't mix multiple tasks into one file. See [backlog/README.md](backlog/README.md).

## Constraints on agent actions

- **Don't commit without an explicit request** (`/commit` or "make a commit"). Done — wait.
- **Builds can be run freely** — this is a tool project, verification through
  a real compilation is normal here. By default build **Debug**, not Release.
- Don't invent extra structure or abstractions without a confirmed need.

## A short check before considering a task done

- did you not expand scope beyond what was requested;
- did you not add an abstraction without clear benefit;
- can part of the new code be deleted;
- is there a testable path in fixtures/tests;
- does the change not contradict the Lakos/Core Guidelines framing;
- does the change not conflict with [docs/architecture-spec.md](docs/architecture-spec.md).

---
> Source: [blurman-ai/archcheck](https://github.com/blurman-ai/archcheck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
