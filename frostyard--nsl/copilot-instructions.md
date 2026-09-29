## nsl

> nsl is a Go CLI for WSL-style Linux machines on atomic Linux hosts: systemd-nspawn containers in one systemd-vmspawn/QEMU VM, from signed Frostyard machine images ([ADR-0016](docs/adr/0016-wsl-style-machines.md), [ADR-0017](docs/adr/0017-shared-vm-and-machine-images.md)). The [implementation plan](docs/plans/shared-vm-implementation.md) records how it was built and what each phase proved. Start at [docs/README.md](docs/README.md); user documentation is the [site](site/content/index.md), and [README.md](README.md) is its short entry.

# frostyard/nsl

nsl is a Go CLI for WSL-style Linux machines on atomic Linux hosts: systemd-nspawn containers in one systemd-vmspawn/QEMU VM, from signed Frostyard machine images ([ADR-0016](docs/adr/0016-wsl-style-machines.md), [ADR-0017](docs/adr/0017-shared-vm-and-machine-images.md)). The [implementation plan](docs/plans/shared-vm-implementation.md) records how it was built and what each phase proved. Start at [docs/README.md](docs/README.md); user documentation is the [site](site/content/index.md), and [README.md](README.md) is its short entry.

This is the canonical agent instruction file. `CLAUDE.md`, `GEMINI.md`, and `.github/copilot-instructions.md` link here; `.claude/skills` links to `.agents/skills` ([ADR-0002](docs/adr/0002-agent-portable-instruction-surface.md)). Edit the canonical targets only.

## Skills

Procedures belong in [.agents/skills/](.agents/skills/). Add a skill based on its template when a multi-step procedure repeats.

- [validate-image](.agents/skills/validate-image/SKILL.md): build the VM and machine images, accept them in disposable VMs, record the evidence and diagnose integration failures.

## Live code conventions

- The root `main.go` owns command parsing. `state.go` keeps the VM records (the shared VM and each isolated machine's), `vm.go` launches the VM and checks readiness through the agent, `storage.go` handles the data disk, `update` and `list`, `machines.go` the machine records, commands and directory translation, `archive.go` export and import, `ports.go` the port forwarder, `desktop.go` desktop sessions and the `nsl-open` broker, and `remote.go` `ssh-config` and `logs`. `image_*.go` verify and cache signed catalogue images. `config.go` reads `nsl.conf`. The `runner` interface abstracts local tools.
- `cmd/nsl-agent` is the VM agent: the forced SSH command, the machine operations, the VM's boot services and its idle monitor. `internal/protocol` holds the request types and validation both ends share. `image/vm/` is the VM image layer and `image/machines/` the machine images.
- Tests use fake runners, fake systemd and local processes; do not require root or a VM for unit tests. Language and dependency changes are allowed when justified.
- The specs describe the target system: the [CLI](docs/specs/cli.md), [agent protocol](docs/specs/agent.md), [VM image](docs/specs/vm-image.md), [machine images](docs/specs/machine-images.md) and [image delivery](docs/specs/image-delivery.md). Follow the [implementation plan](docs/plans/shared-vm-implementation.md) phase by phase, with its working rules and carried requirements, and record each phase's evidence there. Publication and release are Phase 10.
- Signed image delivery uses `sigstore-go` and bounded zstd under [ADR-0015](docs/adr/0015-image-verification-and-catalogue-policy.md). Keep publisher identity, trust root, rollback/freshness and cache checks intact. Public image artifacts must exclude private VM evidence and archives; follow [the publication design](docs/design/image-publication.md).
- Pass commands as argument arrays (`exec.Command`, and `ExecStartEx` in the agent), never as a shell command line on the host or guest. Validate ownership and permissions of state files (`privateFile`, `checkPrivateDir`) and unit descriptions before changing state. Never overwrite preexisting images, machines, archives or files. Reserve `--root` for explicit administrative commands.
- Storage and archives follow [ADR-0008](docs/adr/0008-offline-storage-management.md) and [ADR-0006](docs/adr/0006-stopped-vm-backups.md): stopped units only, resumable removal and growth, no shrinking, and validation before publishing. Lifecycle calls must reject a replacement ID after waiting for a lock. Never take a trust tier or host share from an archive.
- Keep the host CLI and the agent independent of guest distribution. Distro differences belong in image adapters under the [machine-image contract](docs/specs/machine-images.md) ([ADR-0009](docs/adr/0009-distribution-neutral-guest-contract.md)).
- Regenerate `THIRD_PARTY_NOTICES.txt` with `python3 scripts/license-notices.py` when Go dependencies change; CI checks it against shipped packages.
- Run `make ci` before claiming a change is complete. CI runs the same recipe. It is the last of the gate triad (frostyard/core ADR-0043, ADR-0044): `make verify` is credential-free and leaves a clean checkout clean (module tidiness, license notices, vet, formatting, the pinned golangci-lint, script and unit tests), `make check` formats and then verifies, and `make ci` adds race-enabled tests with coverage and the cross builds. `mise install` provides the pinned linter; fix a finding rather than loosening `.golangci.yml`. GoReleaser Pro (not OSS) validates `.goreleaser.yaml` in CI using the org secret; run Pro's `goreleaser check` when changing release configuration.

## Repository boundary

Never commit binaries, `build/`, `dist/`, `site/dist/`, coverage artifacts, local agent state, credentials or a host-specific VM image. The CLI does not install host packages, change device permissions or sudoers, or share D-Bus, GPU or SSH sockets. Host storage is shared only through the `/mnt/host` allowlist in [ADR-0016](docs/adr/0016-wsl-style-machines.md). Releases are built from `v*` tags by `.github/workflows/release.yml` using GoReleaser Pro and GitHub provenance attestations. Tag one with `make bump`: on a clean `main` that matches `origin/main`, it runs `make ci`, tags the version `svu next` derives from conventional commits (`.svu.yml`), and pushes the tag. Conventional commit subjects therefore set the version and make the release changelog readable. Tools beyond Go are pinned in `mise.toml` with a committed `mise.lock` (frostyard/core ADR-0043); `mise install` provides them, and Go comes from `go.mod`.

## Documentation

Docs follow [the index](docs/README.md): `adr/` for rationale, `design/` for living mechanisms, `specs/` for testable contracts, `plans/` for phased work. Start new documents from the category's `TEMPLATE.md`; update the index and link related docs in both directions. User documentation is the site in `site/content/`, published to GitHub Pages ([ADR-0018](docs/adr/0018-user-documentation-site.md)); `README.md` is its short entry. Change site pages in the same change as the behavior they describe, and run `make site` when they change; it builds in strict mode. Record significant decisions in an ADR, new or edited, before changing the related design or spec. nsl is pre-release and has no users. Rewrite ADRs, specs, plans and code in place when decisions change. Keep no compatibility bridges, migrations or version shims for state, schemas, catalogues, archives, CLI grammar or images, and delete what a change replaces in the same change. Security checks are not bridges: keep them.

---
> Source: [frostyard/nsl](https://github.com/frostyard/nsl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-29 -->
