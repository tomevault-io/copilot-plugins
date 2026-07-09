## animalavd

> - Root docs: README.md

# Repository Guidelines

## Project Structure
- Root docs: README.md
- Scheduler: scheds/rust/scx_lavd/src/ (userspace) and scheds/rust/scx_lavd/src/bpf/ (BPF scheduler)
- Hyprland daemon: scheds/rust/scx_lavd/examples/anima_hypr_daemon.rs
- Systemd/services: services/
- Shared Rust workspace: rust/ and lib/

## Build / Format / Docs
- cargo build --release
- cargo fmt
- cargo fmt -- --check

## Coding Conventions
- Format with cargo fmt before committing
- Rust: snake_case for identifiers, PasicalCase for types
- Keep BPF patches minimal and local to scheduler logic
- When editing, match the existing code style in the file

## Verification
- For scheduler-related changes, describe how you validated behavior
- For BPF changes, consider verifier and runtime impact
- Run relevant workspace checks before opening a PR

## Commits & PRs
- Use concise, present-tense commit messages explaining behavior change
- Keep commits focused on one concern
- PRs should summarize the change, rationale, affected paths, and how it was tested

## SOP: MCP Tool Routing for Codebase Exploration

> Always route through the native Droid equivalents listed in the Deferred toolbox
> before falling back to their corresponding MCP hooks. `Indexing first, searching,
> then reading full files only as needed` keeps exploration cheap.

### 1. Torrent of the whole tree

*Prefer* `LS` over `jcodemunch list_watchers` for one-shot directory listing.
*MCP hook* `jcodemunch list_content` shines when you have already
indexed a repo and need to search across **both** code and docs (TOML, Markdown,
Dockerfile) simultaneously — for example, pinpointing where a scheduling
constant is documented in `README.md` and referenced in `src/bpf/` at the same
time.

### 2. Exact symbol search

*Try* `Grep` first with type filter `type=rust` or `case_insensitive=true`
and `context`/`line_numbers` on.
*MCP hook* `jcodemunch search_units` beats `Grep` when symbols cross
file boundaries or when you expect to also surface documentation sections and
doc comments — searching `LAT_CRI` across code + adjacent Markdowns is a typical
case.

### 3. Outline / signature drill-down

*Use* `Glob` and `Read` with a hand-picked line range.
*MCP hook* `jcodemunch get_outline` + `get_unit_context` returns
token-budgeted symbol capsules — better than big-file `Read` when you need a
summary of every struct or inline function in `main.bpf.c` without reading
the whole object.

### 4. Symbol-level context (callers / callees / references)

*Start* with `Grep` and cursor the surrounding block with `context`.
*MCP hooks* `jcodemunch find_references` / `get_blast_radius` /
`get_call_hierarchy` win when the symbol appears in ten or more files
(danger zone for missing places to patch during a refactor). For example,
tracing which call sites actually depend on the BPF map update in
`anima.bpf.c` is easier via blast radius than multiple greps across crates.

### 5. Dead code and refactoring

*No native equivalents.* Use these MCP hooks directly:
`jcodemunch get_dead_code_v2` + `find_similar_symbols` to spot duplicate
LAVD helper code that could be hoisted into `lib/`; `check_safe` before
renaming lat-calc constants; `plan_refactoring` to produce concrete
edit blocks you can feed into the file patches.

### 6. Structural anti-pattern scan

*No native equivalents.* `jcodemunch find_implementations` for locating
polyfilles of ticket-function interfaces, `search_ast` for language-agnostic
scans (empty catch blocks, deeply nested loops) across `.bpf.c` Rust and
BPF C alike when global static-code violations need hunting down.

### 7. Runtime diagnostics and profiling

*No native equivalents.* Use `jcodemunch import_runtime_signal` /
`find_hot_paths` when tuning BPF scheduler overhead on real workloads;
`get_symbol_complexity` / `get_repo_health` as a quick triage before a larger
rewrite.

### Rules of thumb

1. **Always start with `LS / Grep / Read`.** Spin up `jcodemunch` only when
   the question crosses file or language boundaries, needs a hierarchical
   view, or is a "structural smell" / "refactor" query.
2. **Never install `jcodemunch`just for a "deeper search" click** — first
   exhaust native tooling.
3. **Feed MCP context to patches.** Wrap `get_symbol_source` /
   `find_references` output into every file edit so downstream reviewers see the
   provenance chain.
4. **Alias usage by intent.** Before each MCP hit, add one line:
   _"why `_hook_` > native: <one-line reason>."_ This keeps your SOP honest
   and explainable to new contributors.

---
> Source: [animaios/animalavd](https://github.com/animaios/animalavd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
