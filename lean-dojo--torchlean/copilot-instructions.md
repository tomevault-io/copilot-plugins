## torchlean

> Before making any claim about TorchLean's architecture, APIs, semantics, supported devices,

# TorchLean Agent Guidance

## Source Of Truth

Before making any claim about TorchLean's architecture, APIs, semantics, supported devices,
backend providers, verification guarantees, or current implementation status, inspect the current
checkout.

- Do not infer current behavior from earlier conversations, documentation, filenames, or planned
  architecture alone.
- Distinguish clearly between functionality that is:
  - implemented and wired into runtime execution;
  - represented only in metadata, contracts, or planning;
  - planned but not yet supported.
- Cite the relevant source files when explaining the implementation.
- Run the relevant build or test before claiming executable behavior works.
- Treat the current code as authoritative when it disagrees with stale documentation, and identify
  the documentation that needs correction.

## Working Tree

- Preserve existing local changes and work with them.
- Do not commit, push, merge, delete branches, or modify remote state unless the user explicitly
  requests that action.

---
> Source: [lean-dojo/TorchLean](https://github.com/lean-dojo/TorchLean) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
