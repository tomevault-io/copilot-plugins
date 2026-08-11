## nullspace

> validates the live ownership, security, signer, and workflow configuration.

# Repository Guidelines

This repository owns Nullspace's public API, protocol contracts, SDKs, console,
documentation, and self-hosted runtime.

Use `README.md` for the repository map and existing component-level instructions
for local commands. Keep HTTP and WebSocket changes synchronized with
`specs/openapi.yaml`, public SDK callers, console callers, and public docs.

Do not add platform deployment, billing, fleet/provider, private operations, or
customer-control-plane source. Do not add submodules, subtrees, source mirrors,
or reverse synchronization with the private platform repository. Cross-repository
consumption must use immutable, digest-pinned release artifacts.

Before tagging anything, run `python3 tools/ci/check_release_trust.py`, which
validates the live ownership, security, signer, and workflow configuration.

---
> Source: [ns-rocks/nullspace](https://github.com/ns-rocks/nullspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
