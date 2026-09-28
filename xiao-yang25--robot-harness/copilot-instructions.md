## robot-harness

> This is the product repository. Supported, isolated simulation support belongs in

# Robot Harness engineering entry

This is the product repository. Supported, isolated simulation support belongs in
`integrations/ros2/simulation`. Keep exploratory research fixtures, sealed evidence,
personal environment details, and unrelated reference sources outside it.

## Rules and knowledge entry points

Apply the host-provided global instructions. When the host selects the shared
`agent-engineering-workflow` guide, resolve its location from that host entry or
the task handoff, read its `AGENTS.md`, and load the relevant guides there. Its
relative references resolve within the shared repository. Do not assume another
agent or CLI has read it; pass the applicable entry points and verify access when
delegating. If a necessary entry is unavailable, report the gap and continue work
that does not depend on it. Do not add personal paths or copies of shared policy
to this repository.

| Concern | Project source |
|---|---|
| Product purpose, implemented status, next increment | [README](README.md) |
| Contributor onboarding and runnable examples | [Contributing](CONTRIBUTING.md), [Examples](examples/README.md) |
| Accepted execution boundary and M1–M4 scope | [Design](docs/DESIGN.md) |
| General software design principles | [Design principles](docs/DESIGN.md#design-principles) |
| Commands, environments, acceptance mapping, result locations | [Testing](docs/TESTING.md) |
| PR review workflow and revision-linked evidence | [PR review and evidence](docs/TESTING.md#pr-review-and-evidence) |
| C++ naming, headers, formatting and generated-file boundaries | [Coding style](docs/CODING_STYLE.md) |
| Project-specific integrity constraints | [D-068 product application](docs/standards/MINIMAL_INTEGRITY_STANDARD.md) |

## Product constraints

- Preserve the D-073 ownership and recovery boundaries in `docs/DESIGN.md`.
  Core remains C++17, ROS-independent, payload-independent, and host-driven.
- Enforce authority at the adapter's declared submission boundary. Keep cancel
  acknowledgement, native termination, required settlement, and task verdict
  distinct; evidence gaps must not become success or fresh authority.
- Task strategy and model dependencies belong to the separate Agent project.
  A model cannot allocate execution authority or override native safety.
- Follow D-068 for new documents, tools, and experiments: no MD5, no routine
  source snapshots/digests, and no repeated full semantic adjudication. Preserve
  sealed historical evidence under its original contract.

## Change and verification entry points

- Follow `docs/CODING_STYLE.md` for new and modified product code; use
  `.clang-format` for layout and review names separately. Keep build trees ignored.
- Core or adapter changes use the corresponding rows in `docs/TESTING.md` and
  the shared guide's applicable review requirements. Planned M1–M4 checks are
  not implemented tests or evidence of PASS.
- Changes to an accepted execution boundary update `docs/DESIGN.md`; changes to
  build/test commands, environment requirements, or CI update `docs/TESTING.md`
  and the affected README examples. Update README status only with supporting
  implementation and validation evidence.
- Keep continuing-task state in the existing host/task record supplied at
  handoff, with the Git revision, uncommitted scope, evidence, gaps, and next step.
  Do not import private history into the product repository. Ordinary public
  product status belongs in README; do not add a parallel progress registry.

---
> Source: [xiao-yang25/robot-harness](https://github.com/xiao-yang25/robot-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
