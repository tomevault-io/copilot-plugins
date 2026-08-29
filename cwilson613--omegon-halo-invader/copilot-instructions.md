## omegon-halo-invader

> Expose selected [Invader](https://github.com/SnowyMouse/invader) 0.55.0 workflows to Omegon through typed, bounded, auditable contracts. Invader constructs and transforms legacy Halo content; Ringhopper remains the preferred structured read-only inspector; native Bevy content remains owned by its independent project.

# omegon-halo-invader agent directives

## Mission

Expose selected [Invader](https://github.com/SnowyMouse/invader) 0.55.0 workflows to Omegon through typed, bounded, auditable contracts. Invader constructs and transforms legacy Halo content; Ringhopper remains the preferred structured read-only inspector; native Bevy content remains owned by its independent project.

## Repository and licensing boundary

- This is an independent Git repository, not a submodule of the adjacent Halo reconstruction repository.
- Extension code is GPL-3.0-only. Invader 0.55.0 is GPL-3.0-only and is executed as a separate process.
- Do not copy Invader source or generated definitions into this repository unless deliberately vendoring with full provenance and license compliance.
- Never commit Halo executables, maps, tags, textures, sounds, HEK packages, or unclear-rights derivatives.
- Tests use synthetic executables and project-authored fixtures. Restricted qualification artifacts stay under `HALO_RESOURCE_ROOT`.

## Version policy

The only qualified Invader release baseline is:

```text
version: 0.55.0
commit:  33518109f2d7d1694a41b4ea7e957d244eb1866f
```

Do not accept floating `master`, a newer release, or an arbitrary compatible-looking binary. Version upgrades require source/release review, dependency and license review, fixture qualification, protocol review, and an explicit release decision.

## Tool-contract policy

- Never add generic argv, shell, or executable-name passthrough tools.
- Each tool owns its complete argument schema and executable choice.
- Unknown JSON fields are rejected.
- Mutation requires separate plan/apply contracts; plans are read-only and apply must reproduce or reference the approved normalized request.
- Return extension-owned DTOs. Do not expose raw process internals as the only result contract.
- Keep stdout reserved for JSON-RPC. Child stdout/stderr are piped and bounded.

## Filesystem and process safety

- Canonicalize configured executable, read, and write roots.
- Read inputs must exist beneath a read root. Outputs must resolve beneath a distinct write root.
- Reject overlapping read/write roots, traversal, escaping symlinks, non-regular executables, and unexpected executable names.
- Spawn with argument arrays, `stdin` null, and piped stdout/stderr. Never invoke a shell or use inherited stdio.
- Set operation-specific timeouts, kill owned children on timeout, and bound captured output.
- Build/apply operations write to clean staging directories and atomically promote validated outputs where possible.
- Never overwrite source content by default.

## Initial supported surface

- `halo_invader_probe`: verify configured executable identities and exact release version.
- `halo_invader_map_inspect`: run bounded, stable `invader-info` queries and return typed metadata.
- `halo_invader_build_plan`: validate and normalize an Xbox/PC/MCC cache-build request without executing it.

Cache-build apply and extraction remain explicitly unavailable until qualification fixtures and plan-digest semantics exist.

## Required validation

For every non-trivial change:

```bash
cargo fmt --check
cargo test --locked
cargo clippy --locked --all-targets -- -D warnings
cargo build --locked --release
python3 -m unittest discover -s tests -v
python3 scripts/protocol-smoke.py target/release/omegon-halo-invader
```

Tests must cover malformed parameters, path containment, symlink escapes, executable identity, timeout/output bounds when process execution changes, and protocol stdout hygiene.

## Agent-session startup

Before implementation work:

1. Read this file, `README.md`, `docs/invader-0.55.0-qualification.md`, and `docs/release-policy.md`.
2. Inspect `git status` and the active Workbench plan.
3. Distinguish implemented, planned, and qualified capabilities.
4. Do not use operator-supplied Halo content in ordinary tests.
5. Keep decisions and fixture evidence in repository documentation rather than relying on chat history.

---
> Source: [cwilson613/omegon-halo-invader](https://github.com/cwilson613/omegon-halo-invader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
