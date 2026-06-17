## sdwan-probe

> Guidance for AI coding assistants working in this repository.

# AGENTS.md

Guidance for AI coding assistants working in this repository.

## Project Summary

- Name: `sdwan-probe`
- Language: Python
- Purpose: Probe Cisco Catalyst SD-WAN control components (`vmanage`, `vbond`, `vsmart`) using TLS/DTLS without credentials.
- Primary audience: network and security engineers.

## Core Principles

1. Keep behavior read-only and non-intrusive.
2. Avoid introducing secrets, credentials, or private infrastructure identifiers.
3. Preserve backward-compatible CLI behavior unless explicitly asked to change it.
4. Prefer deterministic output and clear error messages.
5. Keep documentation aligned with user-visible behavior.

## Coding Standards

- Use Python type hints where practical.
- Prefer small, testable functions.
- Fail fast with explicit `ValueError`/`UsageError` messages for invalid user input.
- Keep external dependencies minimal.
- Avoid hidden network side effects beyond explicit probe/discovery actions.

## Security Requirements

- Never hardcode passwords, tokens, private keys, or API credentials.
- Treat hostnames, payload data, and certificate contents as untrusted input.
- Do not relax certificate verification in code paths that claim strict verification.
- Prefer modern crypto defaults; avoid adding SHA-1 or legacy trust anchors in new features.

## CLI and UX Requirements

- Any new CLI flag must be documented in `README.md`.
- Maintain both human-readable and `--json` compatibility where possible.
- Keep exit-code semantics stable:
  - `0` when at least one target is reachable
  - `1` otherwise

## Testing Requirements

- Add or update tests under `tests/` for every behavior change.
- Ensure full suite passes before finalizing:

```bash
.venv/bin/pytest -q
```

- For parser/discovery logic, prefer unit tests with mocked resolvers over live DNS.

## Files to Update Together

When changing any of the following, update all related docs/tests:

- CLI options in `sdwanprobe/cli.py`
- Parsing/discovery logic in `sdwanprobe/probe.py`
- Probe verification logic in `sdwanprobe/tls.py` or `sdwanprobe/dtls.py`
- End-user usage and examples in `README.md`

## Documentation Rules

- Use safe placeholder domains (`example.net`, `example.com`, RFC5737 test IPs).
- Do not include real customer hostnames, internal IPs, or production identifiers.
- Keep beginner-friendly installation and troubleshooting guidance.

## Release Hygiene

Before proposing a release/tag:

1. Run tests.
2. Verify README examples still match implemented flags.
3. Confirm license and notice files are present.
4. Ensure no real customer data appears in repository content.

---
> Source: [etychon/sdwan-probe](https://github.com/etychon/sdwan-probe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
