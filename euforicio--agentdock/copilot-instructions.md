## agentdock

> These instructions apply to the entire repository.

# AgentDock Repository Guidance

These instructions apply to the entire repository.

## Start Here

- Read [README.md](README.md) for product behavior and supported boundaries.
- Use [docs/index.md](docs/index.md) to find architecture, data-flow,
  development, operations, and security documentation.
- Treat [Package.swift](Package.swift), source code, tests, packaging scripts,
  and the release workflow as the source of truth when documentation drifts.

## Change Boundaries

- Keep provider discovery, parsing, persistence, launch, process, and filesystem
  safety in `Sources/CodexerCore`.
- Keep transcript presentation provider-neutral in
  `Sources/TranscriptRenderer`.
- Do not weaken signature, ownership, path-containment, symlink, process
  identity, or bounded-I/O checks.
- Preserve stable transcript identity, exact source order, honest malformed
  records, and provider capability gating.
- Do not add mocks or stubs.

## Validation

Run the smallest relevant test target while iterating, then run:

```bash
swift test
```

For packaging changes, also run:

```bash
./script/build_app.sh
./script/package_app.sh
```

Installed-provider lifecycle tests are opt in and can open local applications.
Their commands and requirements are documented in
[docs/development.md](docs/development.md).

## Privacy and Repository Hygiene

- Never commit credentials, profiles, chats, logs, databases, local paths,
  account details, or screenshots containing real user or device data.
- Use synthetic fixtures and screenshots.
- Keep unrelated changes out of the same commit.
- Update focused documentation when behavior, contracts, security boundaries,
  commands, or release procedures change.

---
> Source: [euforicio/AgentDock](https://github.com/euforicio/AgentDock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
