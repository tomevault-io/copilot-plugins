## codex-cli-bridge-for-lark-feishu

> summary: root routing contract for current Codex CLI Bridge for Lark (Feishu) work

<!-- docmeta
role: agent
layer: 1
parent: null
children:
  - docs/AGENTS.md
  - src/AGENTS.md
  - scripts/AGENTS.md
  - skills/AGENTS.md
summary: root routing contract for current Codex CLI Bridge for Lark (Feishu) work
read_when:
  - first agent entry into the repository
skip_when:
  - already inside the correct domain router
source_of_truth:
  - AGENTS.md
  - docs/AGENTS.md
  - src/AGENTS.md
-->

# Agent Router

The product is Codex CLI Bridge for Lark (Feishu). The repository/package compatibility name remains `telegram-codex-bridge`; do not treat that retained name or shared compatibility types as product positioning. The active Feishu direction is not yet fully shipped, so keep three categories separate:

1. current truth in product, architecture, operations, generated docs, and code
2. active Feishu plan in `docs/plans/2026-07-27-feishu-native-bridge-refactor.md`
3. older Web, future, plan, and archive context

## Default Retrieval

Read this file, then one domain router, then one leaf or narrow source owner. Expand only for a demonstrated cross-cutting dependency or conflict.

| Task | Read next |
|---|---|
| current behavior, architecture, operations, or planning | `docs/AGENTS.md` |
| implementation, refactor, tests, or bug fix | `src/AGENTS.md` |
| GitHub bootstrap scripts | `scripts/AGENTS.md` |
| bundled setup skills | `skills/AGENTS.md` |

## Source Priority

1. active user instruction
2. current-truth docs
3. current implementation and tests
4. Codex protocol evidence
5. active plan
6. future and historical material

If code and current docs disagree, report and resolve the mismatch according to the task. Do not blend them.

## Feishu Rule

For Feishu development, deployment, debugging, or fixes, use the workspace's local Feishu official-document mirror before changing platform fields, permissions, event names, callback shapes, limits, or release steps. Open the matched official raw Markdown. The local full mirror excludes some server API, client API, and Feishu CLI columns; explicitly supplement those gaps from official sources when required.

Shared Telegram-compatible types are implementation reuse, not evidence of Feishu parity. A desired single-session Feishu UI is not evidence that multi-session state or concurrency can be deleted.

## Fast Leaves

| Need | Leaf |
|---|---|
| scope and trust | `docs/product/v1-scope.md` |
| project/session flow | `docs/product/auth-and-project-flow.md` |
| settings and Codex commands | `docs/product/codex-command-reference.md` |
| runtime and delivery | `docs/product/runtime-and-delivery.md` |
| callbacks | `docs/product/callback-contract.md` |
| current decoupling gaps | `docs/architecture/platform-decoupling-status.md` |
| state and recovery | `docs/architecture/runtime-and-state.md` |
| pack boundary/capabilities | `docs/architecture/platform-pack-boundary.md` or `platform-capability-matrix.md` |
| code ownership | `docs/architecture/current-code-organization.md` |
| install and administration | `docs/operations/install-and-admin.md` |
| active Feishu refactor | `docs/plans/2026-07-27-feishu-native-bridge-refactor.md` |
| volatile versions/counts | `docs/generated/current-snapshot.md` |

## Change Rules

- Preserve compatibility names unless migration is explicitly in scope.
- Update the matching current-truth doc with user-visible or operational code changes.
- Plans must state what is not implemented.
- Historical archive content is preserved, not rewritten as present fact.
- Do not start destructive platform-code deletion until imports, tests, and Feishu replacements prove the boundary.
- Never include secrets, owner IDs, raw tokens, or private project paths in docs, tests, screenshots, or logs.

## Verification

Use focused tests during development, then `npm run check`, relevant tests, and `npm run build`. CI targets Ubuntu and macOS. Compare Windows full-suite failures with the documented baseline before classifying a regression.

---
> Source: [Sunne927/Codex-CLI-Bridge-for-Lark-Feishu](https://github.com/Sunne927/Codex-CLI-Bridge-for-Lark-Feishu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
