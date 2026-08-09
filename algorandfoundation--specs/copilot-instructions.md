## specs

> Before changing specification content, read

# Guidance for AI coding agents

Before changing specification content, read
[`src/CONTRIBUTIONS.md`](./src/CONTRIBUTIONS.md) for the authoring rules covering
Markdown, MathJax, admonitions, links, and diagrams.

Use [`README.md`](./README.md) as the source of truth for repository setup,
toolchain management, CI/CD, previews, and releases.

Choose the validation path based on the available environment:

- With Docker and Docker Compose, use `make docker-check` while iterating, then
  run `make ci` once before handoff. `make ci` is the authoritative
  GitHub-equivalent Linux AMD64 validation and HTML build.
- Without Docker, use `make check` with the pinned native toolchain. Report any
  version-drift warnings and state that Docker-only validation was not run.
  Targeted commands such as `make lint` or `make test` are useful while
  iterating, but do not replace `make check`.

Run additional checks only when relevant:

- When external links or Lychee configuration change, run
  `make docker-links-check` with Docker or `make links-check` without it.
- `make docker-release` is a heavy HTML and PDF release build. Run it only when
  changes affect the release workflow or image, Pandoc/PDF configuration,
  release scripts or dependencies, or when the user explicitly requests PDF
  validation. Do not run it for routine Markdown or content changes.
- If release validation is required but Docker is unavailable, report that it
  remains unverified rather than substituting a native build.
- Inspect the relevant HTML, PDF, or deployment preview when rendering changes.

---
> Source: [algorandfoundation/specs](https://github.com/algorandfoundation/specs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
