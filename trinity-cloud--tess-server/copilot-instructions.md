## tess-server

> Conventions for AI agents (and humans) working in this repository.

# AGENTS.md — Tess Server release repository

Conventions for AI agents (and humans) working in this repository.

## What this repository is

The **release repository** for `tess-server`: customer-facing README, docs, the open-source TUI, release tooling, and packaged model-profile integration. The proprietary engine source and detailed engineering evidence live in separate private repositories and must not be copied here.

## Hard rules

1. **No AI-attribution trailers.** Never add `Co-Authored-By: Claude`, `Generated with…`, or any AI sign-off to commits. Commits are authored by the repo owner; agents are tools.
2. **SSH-only git.** Do not authenticate `gh` or any token-based GitHub tooling. GitHub web-UI actions (repo settings, release-asset uploads, default-branch changes) are performed by the owner.
3. **No binaries in the git tree.** Executables, metallibs, archives, and models never enter version control here (`.gitignore` enforces this). The public npm package is assembled from this open-source TypeScript/Ink frontend plus an ignored, generated proprietary sidecar copied from an exact validated release archive. The sidecar always carries the binary, compiled metallib, profiles, launchers, checksums, notices, SBOM, and provenance manifest as one unit. The engine build comes from the isolated RC pipeline defined in the planning repo — never from an incremental dev tree.
4. **Every performance number must trace to evidence.** Claims in README or docs must be sourced from the campaign docs / A/B/A evidence records in the planning repo (adjacent same-thermal brackets, raw token-identity gates). No extrapolated, mixed-thermal, or "should be about" numbers. If a number is re-measured, update it with its evidence pointer.
5. **Losslessness language is precise.** "Bit-identical" claims are scoped to the validated probe lengths and configurations recorded in evidence. Known limitations during very long generations must not be papered over in customer docs, but their proprietary implementation details do not belong here.
6. **No hard-wrapped prose.** Markdown paragraphs and bullets are written one line each (soft wrap); never reflow prose to a fixed column width.
7. **Respect the split license.** The TypeScript/Ink TUI and npm tooling identified in `LICENSE` are MIT-licensed. The generated engine sidecar is governed by `LICENSE.md`; third-party rights listed in `THIRD_PARTY_NOTICES.md` remain untouched. Never copy proprietary engine code or payload files into the open-source TUI boundary.
8. **Do not disclose engine trade secrets.** Public Markdown may describe installation, supported product behavior, customer controls, measured release results, methodology, limitations, privacy, API compatibility, and licensing. It must not describe proprietary kernels, graph transformations, operation counts, dispatch geometry, internal fallbacks, optimization progressions, dead ends, private source layout, or the mechanisms behind performance gains.

## Repository layout

```
README.md          — marketable front page (numbers must stay evidence-true)
AGENTS.md          — this file
docs/
  index.md         — docs entry point
  performance.md   — common-protocol release results and public methodology
  models.md        — the verified model profiles
  running.md       — install, launch, API, Tess integration, context rules
profiles/          — (release) versioned profile descriptors, hash-bound
scripts/           — reproducible build, profile verification, safe launchers, staging, and tests
SHA256SUMS         — (release) written by the packaging pipeline only
```

## Release flow (summary)

Defined in full in the private release-planning package. Short form: frozen private engine commit → isolated arm64 release build → verification gates → staging allowlist → inject the validated archive into the ignored npm sidecar → verify npm contents → publish `@trinity-cloud/tess-server`. Apple Developer ID signing and notarization remain optional future hardening, not an npm-release prerequisite.

## Style

Customer-facing voice: confident, specific, honest. Negative results and constraints are stated plainly (it is a differentiator, not a weakness). Numbers get their conditions (model, quant, depth, thermal state, probe).

---
> Source: [trinity-cloud/tess-server](https://github.com/trinity-cloud/tess-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
