## guru-maker

> Guru Maker is an investment memory layer shared by AI agent hosts.

# Guru Maker Repository Guide

## Product boundary

Guru Maker is an investment memory layer shared by AI agent hosts.
The first supported hosts are OpenAI Codex and Anthropic Claude Code. Host
agents own reasoning, research strategy, tool selection, delegation, file
editing, and permissions. Guru Maker owns only the Markdown record contract,
deterministic CLI operations, the project-local Guru Mode setting, GitHub-backed
Guru Packs, and the derived Decision Graph, plus minimal native-edit guardrails
around that state.

Do not add a fixed Head Manager, Portfolio Manager, Risk Manager, agent router,
workflow engine, permission system, OMS, broker adapter, Django service,
database, first-party MCP server, web viewer, background process, or Node
runtime.

## Canonical ownership

| Path | Owns |
| --- | --- |
| `src/` | The host-neutral Rust CLI and deterministic record, pack, and graph behavior. |
| `skills/guru-{research,memory,decision,train,packs}/` | The five public Agent Skills and their concise trigger surfaces. |
| `shared/gurumaker/` | Canonical shared Runtime bundle, wrappers, and progressively loaded references; never a public Skill. |
| `.agents/skills/` | Repository-local development guidance for Codex; never shipped as a public Guru Maker Skill. |
| `.codex-plugin/` | Codex discovery and presentation metadata only. |
| `.agents/plugins/` | The public Codex marketplace entry only. |
| `.claude-plugin/` | Claude Code discovery and marketplace metadata only. |
| `hooks/` | Optional cross-host edit guardrails; the Rust Runtime owns their decisions. |
| `.github/workflows/` | Cross-platform validation and tag-driven Runtime releases. |
| `tests/` | Rust integration, cross-project isolation, and product contract tests. |
| `scripts/` | Maintainer-only validation entrypoints and isolated Runtime test harnesses. |
| `docs/` | Durable product, architecture, record, pack, and execution behavior. |

The Rust core and Markdown contracts are canonical. Host adapters must not
fork product behavior. A future host adds only the metadata, installation
route, and behavioral smoke needed by that host unless a demonstrated
incompatibility requires more.

## Simplicity

- Prefer ordinary Markdown and Git over registries, projections, activation
  state, compatibility layers, or speculative extension points.
- Use the system `git` executable so existing SSH and credential configuration
  continues to work. Never accept, store, or print credentials.
- Git remote and commit are installed-pack provenance; do not duplicate them
  in a lockfile or database.
- Parse only authored relationships. Do not add GraphRAG, embeddings, inferred
  edges, a vector store, or a graph database.
- Scan project Markdown directly. Add an index or MCP surface only after a
  measured corpus-size, retrieval-quality, or tool-interface failure.
- Keep ordinary retrieval project-scoped: search, then read selected IDs. Do
  not require catalog, validation, raw Git, or parent-directory scans.
- Guru Mode defaults to `auto` for minimum, reviewable in-task memory
  improvements in an initialized project. `off` requires an explicit request
  for every memory change. Neither mode initializes projects, captures sessions
  in the background, or authorizes Pack, Git, or external execution actions.
- Keep Guru Packs data-only. Each Pack contains exactly one hierarchical Wiki,
  Lens, or Method collection, never Evidence, Decision, hooks, scripts, static
  payloads, MCP servers, or order code.
- Record only reusable knowledge and decision-grade work. Do not require a
  memory edit for every answer.

## Runtime delivery and updates

Read [Runtime distribution](docs/runtime-distribution.md) before changing
plugin installation, wrapper scripts, supported targets, versioning, or
release automation. Read [CI/CD](docs/ci-cd.md) before changing workflow
triggers, job boundaries, permissions, build matrices, or release publication.

- Host plugins expose the five public Skills. Their shared POSIX and PowerShell
  wrappers select the exact Rust Runtime pinned in
  `shared/gurumaker/runtime/version`.
- Keep the wrappers thin. They may locate, download, verify, and launch the
  canonical Rust binary; they must not reimplement CLI behavior.
- Keep Cargo, Codex plugin, Claude plugin, and pinned Runtime versions equal.
  Product contract tests and the release workflow fail on a mismatch.
- Release raw target binaries and `SHA256SUMS` under an immutable `v<version>`
  GitHub tag. Install into versioned user-data paths without changing `PATH`.
- A product update must not rewrite project memory or update Guru Packs.
  Guru Pack updates remain explicit project-local operations.
- Do not add npm, `npx`, a package-manager runtime, an install hook, a daemon,
  or an MCP server to solve Runtime delivery.

## Hook boundary

Read [Hook guardrails](docs/hooks.md) before changing `hooks/`, the internal
`gurumaker hook` command, workspace detection, or post-edit validation.

- Keep hooks optional. The public Skills and CLI must remain complete when a
  host disables or has not trusted plugin hooks.
- Limit plugin hooks to native file-edit interception: protect installed Guru
  Packs before an edit and validate local Markdown after an edit.
- Keep path inspection and validation in the Rust Runtime. POSIX, PowerShell,
  and host configuration files only locate the project and launch it.
- Do not add session-start context, prompt interception, automatic retrieval,
  automatic memory creation, graph generation, Git actions, or learning at
  turn end.
- Treat hooks as guardrails, not a permission or security boundary. Native
  host permissions remain authoritative, and not every external file mutation
  passes through a tool hook.

## Development

Target stable Rust and keep the runtime a single binary. Before materially
changing plugin or skill structure, follow the relevant platform creator
guidance. Validate incrementally:

```bash
cargo fmt --check
cargo clippy --all-targets --all-features -- -D warnings
cargo test --all-targets --locked
```

## Agent workflow

Start every change by reading this file, checking `git status --short`, and
identifying the canonical owner in the table above. Preserve unrelated
worktree changes; do not restage, reformat, or revert files outside the task.
Use `rg` for repository discovery and make the smallest change that keeps the
Rust behavior, public Skills, and durable documentation aligned.

Use the repository-local `guru-maker-development` Skill for maintenance work
in Codex. It is intentionally under `.agents/skills/`, outside the plugin's
`skills/` directory, so it helps contributors without becoming a sixth public
Guru Maker Skill.

Read the paired contract before changing these surfaces:

| Change | Read and update with it |
| --- | --- |
| Record parsing, search, packs, graph, or hooks | `docs/knowledge-model.md`, `docs/decision-memory.md`, or `docs/hooks.md`, plus focused Rust tests. |
| Public Skill trigger or shared reference | `README.md`, `installation.md`, `docs/architecture.md`, and `tests/product_contract.rs`. |
| Runtime wrapper, version, target, or release asset | `docs/runtime-distribution.md`, `docs/ci-cd.md`, shared wrapper tests, and all four version owners. |
| CI job or validation command | `docs/ci-cd.md`, `docs/development.md`, and the matching local script. |

Run `scripts/verify.sh` before handing off a material change. It runs the Rust
quality gate and a Runtime smoke in a freshly created temporary directory.
The harness sets its own `GURUMAKER_RUNTIME_HOME` and release fixture, never
uses a maintainer's installed Runtime or project memory. Use
`scripts/test-env.sh --keep` only when inspecting a failed fixture; it prints
the temporary directory for manual cleanup.

Validate both plugin manifests and run fresh Codex and Claude Code discovery
smokes after consequential skill changes. Preserve unrelated worktree changes.
Do not commit, push, publish, or create a release unless the user explicitly
asks.

---
> Source: [monarchjuno/guru-maker](https://github.com/monarchjuno/guru-maker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
