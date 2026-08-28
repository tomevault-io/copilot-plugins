## remorafish

> This repository is an agent-first Web2API and MCP framework. Treat the Python

# RemoraFish agent guide

This repository is an agent-first Web2API and MCP framework. Treat the Python
framework and its tests as the product; `remorafish/webui/` is a bundled
reference client that demonstrates the public control and data planes.

## Read first

1. `README.md` for the three supported use cases.
2. `docs/architecture.md` for boundaries and runtime flow.
3. `docs/framework.md` before adding or changing a provider.
4. `docs/agent-control-plane.md` before operating local accounts.

Current source and tests are authoritative when historical `DESIGN.md` or
`RESEARCH.md` material differs from executable behavior.

## Architecture invariants

- Provider authors import the stable contract from `remorafish.framework`.
- A provider owns its manifest, credential mapping, health probe, endpoints,
  model catalog, and conversion of its upstream stream into neutral events.
- `ChatEngine` owns routing, account affinity, failover, conversations, and
  transport session lifecycle. Do not duplicate those policies in adapters.
- OpenAI, Anthropic, inference MCP, CLI, and the reference UI are surfaces over
  the same runtime; they must not implement provider protocols independently.
- External providers use the `remorafish.providers` Python entry-point group.
- The regular `remorafish mcp` server is inference-only. Account operations
  belong exclusively to the capability-gated `remorafish admin-mcp` server.

## Credential safety invariants

- Operate only accounts and browser profiles owned by the user or explicitly
  authorized for that user.
- Never print, log, serialize into an agent response, or place in a prompt a raw
  token, cookie, password, authenticated proxy URL, or vault key.
- Browser scans require explicit confirmation. They return opaque scan IDs and
  masked candidates; raw values remain in the local `CredentialBroker` for at
  most ten minutes.
- Import, login, enable, and removal operations require the corresponding Admin
  MCP server capability. Destructive removal also requires the exact account ID
  a second time.
- Keep account control local. Do not expose Admin MCP over the public HTTP data
  plane.
- Do not perform live-account or browser smoke tests unless the user explicitly
  authorizes them for the current task.

## Provider workflow

Generate a separate plugin project when possible:

```bash
remorafish providers init example
```

The plugin must declare `ProviderManifest`, implement the exact `chat()` and
`health_check()` signatures, provide a network-free `list_models_offline()`,
and register through:

```toml
[project.entry-points."remorafish.providers"]
example = "remorafish_provider_example:register"
```

Run the formal contract gate:

```bash
remorafish providers check
pytest -q
```

A provider is not complete merely because it imports. Its strict conformance
report must pass, parser/transport behavior must have deterministic fixtures,
and the full regression suite must remain green.

## Change discipline

- Preserve the public Provider API within a provider API version.
- Add contract tests with every new manifest field or lifecycle hook.
- Keep browser protocol drift isolated inside the corresponding provider or
  credential extractor.
- Preserve the reference UI unless the task explicitly changes its demo UX.
- Update English, Russian, and Simplified Chinese README positioning together.
- Run `git diff --check`, provider conformance, and the full test suite before a
  release or public commit.

For agents that need this guide injected persistently across sessions,
[choirboy-prompt](https://github.com/howdeploy/choirboy-prompt) can deliver this
file as custom instructions. Replace its bundled demonstration lore; never use
that demo biography unchanged as project context.

---
> Source: [howdeploy/remorafish](https://github.com/howdeploy/remorafish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
