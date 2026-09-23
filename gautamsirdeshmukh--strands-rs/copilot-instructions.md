## strands-rs

> This document provides guidance for AI agents working in the Strands Agents

# Agent Development Guide - Strands Agents for Rust

This document provides guidance for AI agents working in the Strands Agents
Rust repository. For human contributor guidelines, see
[CONTRIBUTING.md](CONTRIBUTING.md).

This file is shared by agents with different goals - writing code, opening PRs,
and helping contributors - and is organized by task.

## Context

### Repository Layout

```text
strands-rs/
├── crates/strands-agents/ # Rust SDK (Cargo)
├── tools/strandly/        # CLI tooling
├── website/               # Documentation site (Astro) - see website/AGENTS.md
├── team/                  # Governance and cross-SDK process
├── test-infra/            # Integration-test infrastructure
├── .agents/               # Agent skills and references
├── Cargo.toml             # Cargo workspace root
└── .github/workflows/     # CI (ci.yml is the merge gate)
```

Determine which part of the repository you are changing and follow its
conventions. Documentation work must also follow `website/AGENTS.md`.

### Where the "why" lives: `team/`

Before designing a feature or changing an API, read the relevant context in
`team/`. It captures the reasoning the code itself does not:

- **`team/designs/`** - RFC-style proposals for significant features (numbered
  `NNNN-*.md`). This is the richest source of architectural context: problem
  framing, the chosen approach, alternatives considered, and consequences.
- **`team/DECISIONS.md`** - lightweight architecture decision records for
  smaller calls.
- **`team/TENETS.md`** - the principles a contribution should align with.
- **`team/API_BAR_RAISING.md`** and **`team/FEATURE_LIFECYCLE.md`** - the bar
  and process for API changes and feature deprecation.

## Writing Code

- **Code conventions**: Follow the Rust patterns in
  `crates/strands-agents/`. Use `snake_case` for modules, files, functions, and
  variables and `UpperCamelCase` for types and traits.
- **Branching**: `git checkout -b agent-tasks/{ISSUE_NUMBER}`
- **Commits**: Use [conventional commits](https://www.conventionalcommits.org/)
  - `feat:`, `fix:`, `refactor:`, `docs:`, and so on.
- **CI**: `.github/workflows/ci.yml` is the merge gate and delegates to the
  Rust, package, security, and documentation workflows.
- **Skills**: Reusable repository workflows live under `.agents/skills/` - for
  PRs (`pr-create`, `pr-writer`, `pr-feedback`), docs (`docs-writer`,
  `docs-reviewer`, `docs-audit`, `docs-planner`), and code review
  (`strands-review`). See [`.agents/skills/README.md`](./.agents/skills/README.md)
  for what each does and when to use it.
- **Doc `sourceLinks` track source files**: Documentation pages under
  `website/` point at their backing implementation through `sourceLinks`
  frontmatter. When you rename or move a Rust source file, update every
  `sourceLinks` reference to the old path in the same change. The site build
  cannot detect a stale path that still parses, so search with
  `rg -n "<old/path>" website/src/content/docs`.

### Cross-SDK Conventions

The SDKs aim for parity in concepts and names, not identical code.

- **Plugin / construct naming**: Name a construct for what it does, not for the
  interface it implements. Use `AgentSkills`, `ContextOffloader`, and
  `GoalLoop`, never an `...Plugin` suffix.
- **Cross-SDK parity**: When a name, constant, or hook event exists in another
  SDK, keep it in sync.
  - Identifiers match, re-cased to Rust idiom (`camelCase` to `snake_case`;
    type names remain `UpperCamelCase`).
  - Single-word string-literal values are byte-identical (`"user"`,
    `"success"`).
  - Multi-word string-literal values use the casing required by the shared
    protocol. Convert with an explicit map; do not emit ad hoc variants.
  - Wire field names keep their wire format even when it breaks Rust casing
    conventions (`inputSchema`, `tool_use_id`). Use explicit Serde renames.
  - Hook event names are shared across SDKs, modulo language suffix
    conventions. Add the matching name when adding an event.
- **Public vs internal API**: Keep internal modules and symbols private to the
  crate. Do not make an item public merely to share it internally; use
  `pub(crate)` where appropriate.
- **Structured logging format**:
  `field=<value>, field=<value> | lowercase human-readable message`, with no
  punctuation and a pipe between multiple statements. Use structured `tracing`
  fields or Rust formatting, never printf placeholders.
- **Evergreen comments**: Comments explain what the code does and why, never
  how it changed or what it used to do. Regression tests link the issue they
  guard against and state the guaranteed behavior; tests written during
  feature development carry no issue reference.
- **Directory and file naming parity**: Use Rust `snake_case`, but preserve the
  conceptual stem word-for-word so names remain mechanically translatable
  (`conversation-manager` to `conversation_manager`,
  `vended-plugins` to `vended_plugins`).

### Rust Quality Rules

- Preserve provider wire names explicitly with typed adapters and Serde
  attributes.
- Translate provider failures into the SDK's typed errors and preserve the
  original source where the error contract permits it.
- Keep async streams incremental; do not buffer complete model responses.
- Make cancellation, timeout, cleanup, and ownership behavior explicit.
- Keep implementation and tests in the owning module unless a test exercises
  public integration behavior.
- Avoid `unsafe`; workspace lints forbid it.
- Avoid unrelated refactors and formatting churn.
- Add behavior-focused tests for every changed contract.

### Testing

Use the target that matches the source test's responsibility:

- Unit tests live with their owning modules under `src/`.
- Public integration and provider tests live under
  `crates/strands-agents/tests/`.
- Credentialed end-to-end tests live under `crates/strands-agents/tests/integ/`
  and run through the dedicated manifest.
- Examples under `crates/strands-agents/examples/` are executable verification
  artifacts, not illustrative snippets only.

Run focused checks while developing and the full local gate before submission:

```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-targets --all-features --locked -- --test-threads=1
cargo check -p strands-agents --all-targets --no-default-features --locked
cargo doc --workspace --all-features --no-deps --locked
cargo package -p strands-agents --allow-dirty --locked
```

**`test-infra/` guardrails.** The test-infrastructure crate describes real AWS
resources used by a small subset of integration tests. Most tests do not need
it and run without provisioned infrastructure.

- Do not deploy this stack unless you are explicitly changing the
  infrastructure or iterating on tests that resolve its SSM parameters.
- Never set `STRANDS_TEST_INFRA_INTERNAL=true` unless deploying to the Strands
  team's own test account. It attaches broad internal permissions and GitHub
  OIDC trust that are inappropriate elsewhere.
- To run infrastructure-dependent tests without deploying anything, open a PR;
  CI runs them against pre-provisioned resources.

## Creating PRs

See [PR guidelines](./team/PR.md). Use the `pr-create` and `pr-writer` skills
under `.agents/skills/` to draft and open PRs.

If you open a PR on behalf of a contributor, the human is the author and is
accountable for everything submitted. A small, focused change the author fully
understands is the strongest predictor of a fast review and an accepted PR.

- **Understand before you submit.** The contributor must be able to explain why
  every line works and defend the design. Simplify or explain code they cannot
  describe plainly.
- **Keep it small and focused.** Use one logical change per PR. Changes spanning
  the SDK, website, and infrastructure are usually several PRs.
- **Open an issue first for anything significant**, so maintainers can align on
  the approach before implementation.
- **Do not pad the change.** Avoid drive-by formatting, unrelated refactors, and
  speculative abstractions.
- **Verify before opening.** Run the relevant checks and the local equivalent of
  the `ci.yml` merge gate. Do not open a PR with known format, lint, build,
  package, or test failures.
- **Actually exercise the change.** Automated checks establish validity, not
  that the feature works. Run an example, integration program, or focused
  end-to-end probe, including edge cases. State explicitly when credentials or
  provisioned infrastructure prevent execution.
- **Self-review the complete diff.** Confirm every PR-template assertion is
  true, including that the author reviewed and understands AI-generated code.
  Then use `pr-writer` so the description explains why the change is needed.

## Reviewing

### Documentation Changes

When a change touches `website/`, apply the documentation skills in
`.agents/skills/` in addition to normal code review:

- **`.agents/skills/docs-reviewer/SKILL.md`** - voice consistency, structure,
  terminology, and code-example quality.
- **`.agents/skills/docs-audit/SKILL.md`** - technical accuracy against current
  Rust sources, including import paths, method signatures, and API behavior.

Verify terminology against `.agents/references/terminology.md` and MDX patterns
against `.agents/references/mdx-authoring.md`. Read those source files before
reviewing.

## Working with the Community

When helping someone contribute, be a guide, not a gatekeeper or substitute
author. The contribution is theirs.

- **Point people to the community.** Design discussion belongs on
  [Discord](https://discord.gg/strands) and in
  [GitHub Discussions](https://github.com/gautamsirdeshmukh/strands-rs/discussions).
- **Assume good faith.** Meet contributors where they are; ready-for-contribution
  issues should bring newcomers in.
- **Talk with contributors, not at them.** Be warm, plain, and concise. Ask one
  question at a time and explain why requested changes matter.

---
> Source: [gautamsirdeshmukh/strands-rs](https://github.com/gautamsirdeshmukh/strands-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
