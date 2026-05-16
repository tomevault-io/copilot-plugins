## zero

> Zero is the programming language for agents. Keep public-facing changes honest

# Contributor Notes

Zero is the programming language for agents. Keep public-facing changes honest
about what works today without weakening that positioning.

## Development

- Build the local compiler with `make -C native/zero-c`.
- Use `bin/zero` for focused checks.
- Keep examples runnable and docs copyable.
- Prefer small, direct changes over broad refactors.
- Use direct emitters for compiler output.

## Useful Checks

```sh
npm run docs:test
npm run conformance
npm run native:test
npm run command-contracts
```

For focused compiler work:

```sh
bin/zero check --json <file-or-package>
bin/zero graph --json <file-or-package>
bin/zero size --json <file-or-package>
bin/zero explain <diagnostic-code>
bin/zero fix --plan --json <file-or-package>
```

## Project Layout

- `native/zero-c/`: native compiler implementation.
- `compiler-zero/`: Zero-authored compiler sources.
- `examples/`: small runnable programs and packages.
- `conformance/`: language and CLI fixtures.
- `docs-site/`: public documentation site.
- `scripts/`: validation and release support tooling.

## Public Docs Policy

Docs should describe current user-facing behavior, not internal development
history. Avoid release-planning language, validation-report narratives, and
implementation diary details in pages intended for external readers.

---
> Source: [vercel-labs/zero](https://github.com/vercel-labs/zero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
