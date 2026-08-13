## agent-general-staff

> This checkout is the self-contained public distributable edition of Agent

# AGENTS.md

## Repository contract

This checkout is the self-contained public distributable edition of Agent
Governance Suite 0.4.13. It must not depend on private repositories,
machine-local state, credentials, or unpublished Skill bodies.

Before AGS-governed work, call MCP `ags_preflight` or run:

```bash
ags session preflight --for <agent> --target .
```

The Host reads `ags://capabilities/current-host`, interprets the complete user
context, and submits one typed `HostRouteProposal` to read-only
`ags_route_request`. `ags_apply_action` consumes only a returned
connection-bound action. Existing `## 任务卡` input is validated before request
classification.

## Hard boundaries

- Preserve user-owned content and unrelated working-tree changes.
- Do not publish local memory, receipts, credentials, build output, Host config,
  machine paths, or runtime state.
- Recommendations are discovery metadata, never installation authority.
- InstalledSkillRecord and verified Host activation are machine-local truths.
- AGS MCP is the suite adapter, not a governed third-party MCP.
- External MCP registration remains advice-only.
- Task-card authority comes from `Execution mode`, `Execution topology`, and
  `Delegation planning`; Heavy adds review but cannot expand authority.
- Destructive, external-write, credential, protected-path, and release actions
  require matching authorization.

## Read when relevant

- Repository/publication boundary: `WORKSPACE.md`
- Governance overview: `AGENT_SUITE_PROTOCOL.md`
- Task lifecycle: `protocol/agent-task-protocol.md`
- Runtime adapters: `protocol/runtime-adapters.md`
- MCP contract: `protocol/mcp-server.md`
- Skill governance: `protocol/skill-governance.md`
- Entrypoint ownership: `protocol/entrypoint-guidelines.md`

## Verification

Use the narrowest relevant check during development. Before delivery:

```bash
cargo clippy --all-targets --all-features -- -D warnings
cargo run -q -p ags-cli -- verify --scope local --target .
./target/release/ags verify --scope release --target .
git diff --check
```

After context compaction, re-check the request, repository root, and
`git status --short` before editing.

---
> Source: [FernandeZ-hjm/Agent-General-Staff](https://github.com/FernandeZ-hjm/Agent-General-Staff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
