## ui

> PartnerUp UI is a pnpm workspace for shared PartnerUp UI and design-system packages.

# AGENTS.md of PartnerUp UI

PartnerUp UI is a pnpm workspace for shared PartnerUp UI and design-system packages.
This file is the root SVC front door for agents. Package-local rules are
additive and closest-to-target rules win for tactical execution.

## Repository Layout

```text
/
|-- .changeset/           # Changesets release metadata
|-- .github/workflows/    # CI and publish automation
|-- packages/
|   |-- web/              # Vue web design-system package
|   `-- uniapp/           # UniApp design-system package
|-- tasks/                # Agent-owned task workspaces and implementation records
|-- package.json          # Root scripts and release entry
`-- pnpm-workspace.yaml
```

## Package Boundaries

- `packages/web/`: standard Vue web DOM package. Read `packages/web/AGENTS.md` before editing this subtree.
- `packages/uniapp/`: UniApp-compatible source package. Read `packages/uniapp/AGENTS.md` before editing this subtree.
- Shared release and workspace behavior is owned by the repository root.
- Avoid moving behavior between packages unless the change explicitly targets cross-package API alignment.

## SVC Operating Model

Classify each request before editing:

- Intent: package or product behavior promise changes. This repo has no PRD
  layer today; create one only when concrete product-intent pressure appears.
- Constraint: technical boundaries, package topology, compatibility, release
  surface, environment, or verification rules. Route to Product TDD or Unit TDD.
- Reality: observed behavior disagrees with expected behavior. Gather evidence
  first, then fix and add local tripwires when warranted.
- Artifact: bounded deliverables such as task packets, generated references,
  migration notes, or one-off analysis.

Choose mode as the current posture, not the durable owner:

- Explore: map unknowns and impact.
- Solidify: turn findings into explicit claims or decisions.
- Execute: implement a clear change with verification.
- Diagnose: investigate mismatches evidence-first.

For non-trivial work, open or update a task packet under `tasks/`. Keep task
state volatile until it passes the promotion test.

## Development Workflow

- Install from the root with `pnpm install`.
- Run workspace builds from the root with `pnpm run build`.
- Use package filters for package-local work, for example `pnpm --filter @partner-up-dev/ui-web run verify`.
- Keep changesets focused on release-worthy package changes. Do not create a changeset to republish an already-versioned package.
- For breaking package API, export, runtime, or package-content changes, update
  the affected package's `MIGRATION.md` with consumer migration steps. If the
  change is technically breaking but requires no consumer action, record that
  explicitly in the migration entry.
- Keep `tasks/` as task-local workspace state for investigation notes, migration plans, verification matrices, and implementation records.
- Generated output such as `dist/`, `.histoire/dist/`, and `node_modules/` is not durable source.

## Documentation Index

- `README.md`: consumer-facing package list and registry setup.
- `CONTRIBUTING.md`: concise contribution entry; durable release truth lives in `docs/40-deployment/`.
- `docs/00-meta/`: SVC routing, mode dispatch, destination map, and implementation taste.
- `docs/20-product-tdd/`: workspace topology, cross-package contracts, generated docs/skills, and release surface.
- `docs/30-unit-tdd/`: package-level durable technical contracts.
- `docs/40-deployment/`: publish workflow, release runbook, republish, and rollback guidance.
- `packages/web/README.md`: consumer usage for `@partner-up-dev/ui-web`.
- `packages/web/MIGRATION.md`: consumer migration guide for `@partner-up-dev/ui-web`.
- `packages/web/skills/ui-web/`: generated agent skill and component references for web package usage.
- `packages/uniapp/README.md`: consumer usage for `@partner-up-dev/ui-uniapp`.
- `packages/uniapp/MIGRATION.md`: consumer migration guide for `@partner-up-dev/ui-uniapp`.
- `packages/uniapp/CONTRIBUTING.md`: legacy UniApp package contribution notes.
- `tasks/*/README.md`: task packet entry points for non-trivial agent work.

## Agent Operating Notes

- Prefer reading the nearest `AGENTS.md` before changing a package subtree.
- Keep root SVC docs about workflow, durable ownership, cross-package contracts,
  unit structure, and deployment truth.
- Keep package README files consumer-facing.
- Keep package-local AGENTS files about closest edit rules, local hazards, and
  verification commands.
- Keep volatile reasoning in `tasks/` until it is stable enough to promote into root or package documentation.
- Verify user claims against repository state before changing release metadata.
- Do not add a changeset for SVC documentation topology changes unless package
  public API, package contents, runtime behavior, or consumer-facing package
  documentation changed.

---
> Source: [partner-up-dev/ui](https://github.com/partner-up-dev/ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
