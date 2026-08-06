## portzero

> Repo-specific guidance only. Shared instruction modules are under

# Port Zero Local — agent notes

Repo-specific guidance only. Shared instruction modules are under
`.instructions/` and included at the bottom of this file.

## Configuration & service endpoints

The client has no staging-vs-production environment of its own — only which
backend URLs it targets. Those URLs (cloud API, edge server, dashboard, marketing site)
and their `PZ_TUNNEL_*` environment overrides live in **one** place:
`portzero_domain::endpoints`. When you need a service URL, call its accessor
(`endpoints::api_url()`, `endpoints::edge_url()`, `endpoints::dashboard_url()`,
`endpoints::web_url()`); when you add a new endpoint, add its `DEFAULT_*`
constant and accessor there. Never redefine a `DEFAULT_*` URL constant or
re-read a `PZ_TUNNEL_*` variable with its own fallback inside a `cli`/`daemon`
crate — that reintroduces the duplication this module exists to prevent.

## Terminology

- In code, `service` is an acceptable umbrella term for the discovered thing when
  the implementation is modeling both processes and Docker containers together.
- In user-facing text, prefer `process`, `Docker container`, `Local tunnel`, or
  `Cloud tunnel` when that wording is clearer.
- Avoid `service` in user-facing copy unless it is the most precise term in the
  local context.
- Release channels: **stable** and **unstable** (not “edge” or “prerelease” in
  prose). Product **edge servers** (cloud data plane) are unrelated to the
  unstable release channel.

## Documentation layout

- `docs/developers/` — contributors, has `target-audience.md`.
- User-facing product documentation is **not** in this repo — it lives at
  [portzero.net/docs](https://portzero.net/docs), sourced from portzero-cloud's
  `cloud/landing/blog/src/data/docs/`. When a user-facing behavior changes here,
  update the corresponding page there.

## CI: self-hosted macOS runner and Parallels VMs

The org has exactly one self-hosted macOS runner, and it is a developer's
personal MacBook Pro — not a throwaway/ephemeral box. It also hosts the
Parallels VMs used for VM-based E2E testing (see `vmtest/README.md` and
`.github/workflows/vm-e2e.yml`), driven through `vmkit`
(private, internal tool — installed from a checkout via `just install`, NOT
from the public portzero Homebrew tap).

**Never add a workflow step that runs `sudo` (or anything else host-mutating
— installing system packages, modifying trust stores, etc.) directly on that
self-hosted runner.** Unprivileged steps (`cargo build`, `cargo test`
without elevated privileges, `cargo clippy`, etc.) are fine there.

Anything on macOS that needs `sudo` or otherwise mutates host state must run
in one of these two places instead:

1. **GitHub-hosted `macos-15` runners** — privileged steps that don't need
   real hardware/VM integration.
2. **Inside a Parallels guest, via `vmkit`** — hardware-in-the-loop coverage.

## Delivery model (pointers)

Downloadable product. Shared CD modules apply; in-repo detail:

- **Unstable Release** — automatic on every push to `staging` (`release.yml`)
- **Trigger Stable Release** — gated (`trigger-stable-release.yml`) stamps `vX.Y.Z`
- **Stable Release** — signed build off that tag (`release.yml` run-name)
- `docs/developers/sdlc.md`, `docs/developers/unstable-channel.md`,
  `docs/developers/release-conventions.md` (keep in sync with portzero-cloud)

@.instructions/semble.md
@.instructions/ticketry.md
@.instructions/start-with-production.md
@.instructions/continuous-integration.md
@.instructions/continuous-delivery-shared.md
@.instructions/continuous-delivery-downloadable.md
@.instructions/documentation-layout.md
@.instructions/just.md
@.instructions/user-facing-errors.md

---
> Source: [PortZeroNetwork/portzero](https://github.com/PortZeroNetwork/portzero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
