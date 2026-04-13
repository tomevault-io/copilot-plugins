## foundry-sandbox

> Docker-based sandbox environment for running Claude Code with isolated credentials.

# foundry-sandbox

Docker-based sandbox environment for running Claude Code with isolated credentials.

## Development

- `foundry_sandbox/` - Python CLI package (`cast` entry point via pyproject.toml)
- `unified-proxy/` - Credential isolation proxy (mitmproxy addons, git API)
- `stubs/` - Stub files injected into sandboxes (CLAUDE.md, etc.)
- `tests/` - Test scripts including security red-team tests

## Testing

```bash
./tests/redteam-sandbox.sh  # Security validation (run inside sandbox)
```

## Documentation

- `docs/README.md` - Documentation index
- `docs/architecture.md` - System architecture
- `docs/configuration.md` - Configuration options
- `docs/getting-started.md` - Setup guide
- `docs/usage/` - Commands and workflows
- `docs/security/` - Security model and threat analysis

**Important:** Read `docs/security/security-model.md` to understand the threat model and security boundaries before making changes to sandbox isolation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/foundry-works)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/foundry-works)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
