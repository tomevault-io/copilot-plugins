## shapeshift-labs-frontier-lang-cli

> - Keep runtime imports dependency-light and package-boundary explicit.

# Frontier Lang CLI Agent Notes

- Keep runtime imports dependency-light and package-boundary explicit.
- Do not commit release credentials, npm tokens, .env files, temporary npm configs, generated tarballs, node_modules, dist, or agent-runs.
- Preserve the package role documented in README; target-specific adapters should not leak into the kernel.
- Use npm test and npm pack --dry-run before publishing.

---
> Source: [siliconjungle/-shapeshift-labs-frontier-lang-cli](https://github.com/siliconjungle/-shapeshift-labs-frontier-lang-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
