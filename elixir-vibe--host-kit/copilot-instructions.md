## host-kit

> - Use the project Mix aliases; prefer `mix ci` for the full validation suite.

# Agent Guidelines

## Development

```sh
mix deps.get
mix ci
```

## Conventions

- Use the project Mix aliases; prefer `mix ci` for the full validation suite.
- Keep changes small, tested, and formatted.
- Core owns systemd/systemdkit and unitctl primitives.
- Integrations like Caddy, Forgejo, object storage, and monitoring are providers.
- Recipes compose higher-level deployments from plain resources; they must not hide uninspectable runtime behavior.
- DSLs must compile to plain, inspectable structs; runtime API must be primary and Mix tasks should wrap it.
- For HostKit config authoring conventions, use `SKILL.md`; keep it current whenever DSL/resource/path/file/template conventions change.
- Rollback is modeled as a down plan. Keep changes centered on `Project -> Plan -> Apply` and avoid introducing new entities unless existing plan/change/resource abstractions cannot express the behavior.
- Apply progress is mailbox-first: use `reporter: pid` and `%HostKit.Apply.Event{}` for user-facing progress. Telemetry may mirror events but must not become the primary apply API.

## Documentation upkeep

When changing public behavior, DSL, resource structs, recipes, providers, Mix tasks, plan/apply semantics, events, or integration workflows, update docs in the same change:

- README stays concise and product-oriented; link to guides instead of expanding it.
- `guides/reference/full-reference.md` documents user-facing DSL/API details and examples.
- `guides/reference/cli.md` documents Mix task flags, command shape, and examples.
- `guides/reference/internal-architecture.md` documents internal entities, plan/apply/down-plan flow, event flow, and Mermaid diagrams. Update diagrams when relationships change.
- Deployment guides under `guides/deployment/` document provider/recipe-specific operational flows.
- Livebook demos under `notebooks/learn/` should remain teaching material with the flow: settings -> declaration -> plan -> apply -> verify.
- Examples under `examples/` should stay executable and reflect current preferred APIs.

Doc quality rules:

- Prefer one canonical explanation and link to it from README/other guides.
- Keep Mermaid diagrams valid and focused; update them when adding/removing core concepts.
- Document new event types and event payload fields where they are introduced.
- Document new convention roots, default paths, and artifact/run tracking behavior.
- If a public API is intentionally not documented yet, call that out in the PR/commit rationale.

## Tests and integration coverage

- Test structure should mirror code structure where practical, e.g. `lib/host_kit/plan.ex` -> `test/host_kit/plan/` and `lib/mix/tasks/*.ex` -> `test/mix/tasks/`.
- Prefer exercising public APIs in integration tests. For rollback/down behavior, use down plans and `HostKit.IntegrationCase.on_exit_rollback/3` rather than only hand-rolled cleanup.
- In tests and helpers, do not build HostKit DSL snippets with interpolated strings. Use quoted AST (`quote`/`unquote` with `Code.eval_quoted/1`) or plain runtime structs/APIs so dynamic values remain syntax-safe and refactorable.
- Keep fallback cleanup for resources that are intentionally not reversible yet.
- Livebook integration tests belong under `test/integration/livebook/`.

---
> Source: [elixir-vibe/host_kit](https://github.com/elixir-vibe/host_kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
