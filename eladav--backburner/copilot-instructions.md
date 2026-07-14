## backburner

> Backburner is a Node.js and TypeScript CLI for local GitHub-based agent orchestration. It uses GitHub labels and comments as the control plane, local worktrees as the implementation unit, and readable local JSON for configuration, state, and execution records.

# Contributor Guidance

Backburner is a Node.js and TypeScript CLI for local GitHub-based agent orchestration. It uses GitHub labels and comments as the control plane, local worktrees as the implementation unit, and readable local JSON for configuration, state, and execution records.

## Working Principles

- Read `README.md` before changing user-facing behavior.
- Read any intentionally authored public architecture or workflow docs before changing workflow behavior.
- Follow existing module boundaries and tests before adding new abstractions.
- Keep the design modular, but prefer explicit code over speculative frameworks.
- Use strict TypeScript and keep modules small enough to test directly.
- Minimize dependencies. Prefer Node built-ins unless a package clearly improves correctness or maintainability.
- Keep persisted config and state human-readable, pretty-printed JSON.
- Keep external integrations behind explicit gateway or runner boundaries. `gh`, `git`, and provider CLIs should be represented through testable interfaces or deliberate test doubles.
- Do not add new provider behavior, workflow transitions, or GitHub side effects without tests and matching documentation updates.
- Avoid private paths, personal repository names, generated run artifacts, and local incident residue in committed documentation.

## Product Boundaries

Preserve the product model unless a change explicitly expands it:

- GitHub labels and comments drive workflow decisions.
- Local worktrees are the unit of implementation.
- Context is rebuilt each cycle from current configuration, persisted state, and GitHub data.
- Management configuration stays separate from generated outputs and execution logs.
- Provider CLIs run locally with the user's installed tools and credentials.

Common boundaries to keep explicit include GitHub access, git/worktree operations, provider launch/runtime behavior, context building, task derivation, state loading, and state persistence.

## Workflow And Integration Changes

For changes that affect orchestration behavior:

- Update or add tests at the boundary where the behavior is observable.
- Include regression coverage for label gates, state transitions, task derivation, or GitHub publication paths touched by the change.
- Keep GitHub write behavior narrow and documented.
- Prefer deterministic task and state updates over implicit side effects.
- Do not rely on mocks alone for provider or GitHub permission semantics when the behavior depends on a real external API shape.

## Commands

Core checks:

```sh
npm run build
npm run typecheck
npm test
```

Targeted compiled tests can be run after `npm run build`:

```sh
node --import ./dist/test/setup.js --test dist/test/<name>.test.js
```

Useful CLI scripts:

```sh
npm run init
npm run run:backburner -- --help
npm run run:github-broker -- --help
npm run smoke:github-broker
```

For packaging or install-path changes, also check the package shape:

```sh
npm pack --dry-run
```

## Documentation-Only Changes

For docs-only edits, the full test suite is optional unless examples, package scripts, install instructions, or generated package contents changed. At minimum, run relevant grep, link, or package checks for the changed docs.

Recommended public-doc checks include scanning for old product names, private absolute paths, personal repository names, generated artifact paths, stale command aliases, and unresolved placeholder notes.

If docs mention a command, confirm it exists in `package.json` or in the compiled CLI help before publishing the change.

## Reporting

When finishing a change, report:

- changed files
- exact commands run
- checks that passed
- checks that failed
- live side effects, if any
- assumptions and remaining risks

Do not claim a real integration works unless that integration path was actually exercised.

---
> Source: [eladav/backburner](https://github.com/eladav/backburner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
