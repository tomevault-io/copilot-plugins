## docker-airgap-upgrade

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Six standalone Bash scripts for upgrading Docker Engine 29.1.5 → 29.6.2 (and containerd.io 2.2.1 → 2.2.6) on **air-gapped RHEL 8/9 servers**. There is no application code, no build system, no package manager, and no test framework — the deliverable is the scripts themselves, bundled into a tarball and hand-carried to disconnected servers.

The scripts run as root on RHEL. They cannot be executed on the macOS dev machine.

## Working on this repo

There is no lint/test tooling configured. The practical checks:

```bash
bash -n upgrade-docker.sh                 # syntax check (works on macOS)
shellcheck upgrade-docker.sh              # available via homebrew
```

All six scripts are currently `bash -n` clean and `shellcheck` clean. Keep them that way — where a suppression is genuinely warranted, use an inline `# shellcheck disable=SCxxxx` with a reason, not a blanket ignore.

**Real execution happens through `tests/vm/`** — an OrbStack-based harness that runs the actual scripts against Rocky Linux 9 (x86_64, systemd PID 1) on the dev Mac:

```bash
tests/vm/bootstrap-vm.sh      # build the S1 baseline
tests/vm/build-bundle.sh      # real download-docker-packages.sh run
tests/vm/tier2-run.sh         # Tier 2 cases: reject / upgrade / rollback
tests/vm/negative-control.sh  # prove test 2.4 catches the regression
tests/vm/reset-baseline.sh    # back to S1 between destructive runs
```

The S1 baseline deliberately puts containerd's root on a **separate XFS filesystem at `/data/containerd`** with real images, containers and volume data on it. That is the configuration the pre-v2.0.0 phase 6 destroyed, and it is the only way to test it. See `tests/vm/README.md` for what this proves and — importantly — what it does not (no Swarm, not real RHEL, no GPU, not bare metal).

`simulate-upgrade.sh` remains a separate dnf-path smoke test. It is **not** the same code path as `upgrade-docker.sh` (see below), so passing it does not prove the air-gapped path works.

## Two different install strategies (important)

The repo deliberately contains two incompatible ways of installing the same packages:

| Script | Strategy | Why |
|--------|----------|-----|
| `simulate-upgrade.sh` | `createrepo` + `dnf install` then `dnf distro-sync --allowerasing` | Two-phase dnf; plain `dnf upgrade` is a no-op here |
| `upgrade-docker.sh` | `rpm -Uvh --force` on a validated file list | Corporate satellite servers break dnf with `SSL certificate problem: EE certificate key too weak` |

`recover-dnf.sh` prints recovery commands that reference an `--enablerepo=docker-local` repo. That repo only exists on machines that ran the simulation path; on production air-gapped hosts it must be created first or the commands will fail.

## Current upgrade scope (read this before changing phase logic)

The cluster is on 29.1.5 / containerd.io 2.2.1. This upgrade stays **inside the containerd 2.2.x line**. That is why several things the previous round required are gone:

- **No containerd config migration or regeneration.** 2.2.1 and 2.2.6 share config v3. Regenerating would discard a relocated `root` path, registry mirrors and runtime config — silently repointing a node at an empty `/var/lib/containerd`. Phase 6 verifies; it does not rewrite.
- **No XFS `ftype=1` check.** Any node running containerd 2.x has already satisfied it.
- **No automatic orphaned-network cleanup.** Extracted to `clean-swarm-networks.sh`, run on demand.

All three are recoverable from git history at `upgrade-docker.sh` v1.2.3 (commit `974683a`) if a future containerd **major** upgrade needs them back. Do not resurrect them for a point release.

**The whole-cluster-together rule does not apply here.** 29.1.5 and 29.6.2 both speak containerd 2.2.x gRPC, so a mixed 29.1.5/29.6.2 Swarm is fine and nodes roll one at a time. The rule still holds across the 1.7 ↔ 2.x boundary.

## Invariants the scripts depend on

- **Service order is not optional.** Stop `docker` → `docker.socket` → `containerd`. Start `containerd` → wait for readiness → `docker`. Readiness means polling `ctr version` (up to 30×2s) **and** `ctr snapshots --snapshotter overlayfs ls`, because systemd reports containerd active before its snapshotter is usable. A bare `sleep` is not readiness.
- **Verify the stop, don't assume it — and fail closed.** After stopping, confirm `docker`, `docker.socket` and `containerd` are all conclusively stopped before touching packages or network state. Use the `verify_unit_stopped()` helper, not `systemctl is-active`: `is-active` returns nonzero for `activating`, `deactivating`, *and* for failing to reach systemd, so treating nonzero as "stopped" fails open. The helper requires a successful `systemctl show`, an ActiveState of `inactive` or `failed`, and `MainPID=0`. `docker.socket` specifically: if it survives while dockerd is down, anything touching the socket socket-activates dockerd again, mid-transaction.
- **The package is `containerd.io`, never `containerd`.** The standalone `containerd` package is a different, wrong thing.
- **Validate before destroying.** Everything checkable runs before the node is touched: `upgrade-docker.sh` phase 0 and `rollback-docker.sh` phase 0 verify digests, RPM metadata, expected versions, `el${RHEL_VER}` release, arch, and duplicates, then `rpm -Uvh --test` dry-runs the exact transaction — all while services are still up and the node is still in the Swarm.
- **Assert on RPM metadata, never on filenames.** The previous bundle has an identical directory layout, so a filename check would let an operator "upgrade" 29.1.5 → 29.1.5 and be told it succeeded.
- **One rpm transaction, not several.** Splitting containerd.io and docker-ce into separate invocations can leave a downgraded runtime under a newer engine. Resolving the set together closes that gap, and rpm — not argument order — decides install ordering. This is **not** atomicity: rpm has no general rollback once execution starts, so a failing scriptlet can still leave partial state. That is what `PKG_STATE="attempted"` exists to report.
- **`rollback-docker.sh` needs `--replacepkgs`.** `--oldpackage` permits a lower EVR but not reinstalling an identical one, so a rerun after a partially applied downgrade would be refused with "already installed" — on precisely the node that needs the rerun. (`upgrade-docker.sh` uses `--force`, which already implies it.)
- **Whole clusters upgrade together across a containerd major.** containerd 2.x's gRPC API is incompatible with 1.7.x; mixed-version Swarm clusters produce ALPN handshake errors. Does not apply within 2.2.x.
- **`docker-buildx-plugin` and `docker-compose-plugin` version independently** of docker-ce. Don't pin them to the docker-ce version.

## Version constants are duplicated — change all of them together

Package versions (`29.6.2`, `2.2.6`, `0.35.0`, `5.3.1`, rollback `29.1.5`/`2.2.1`) appear in:

- `download-docker-packages.sh` — four download loops (rhel8, rhel9, rollback-rhel8, rollback-rhel9)
- `upgrade-docker.sh` — `EXPECTED_DOCKER_VERSION` / `EXPECTED_CONTAINERD_VERSION` constants, plus the header and banner
- `rollback-docker.sh` — `ROLLBACK_DOCKER_VERSION` / `ROLLBACK_CONTAINERD_VERSION` constants, plus the header and banner
- `simulate-upgrade.sh` — its own download loop, the `dnf install` pins, and the `assert_pkg` calls
- `README.md` — the version table and verification section

The two upgrade/rollback scripts now hold their versions in named constants at the top rather than scattered through the body — change the constant, not the occurrences.

Directory layout is assumed by path in several scripts: `/opt/docker-offline/rhel${RHEL_VER}`, `rollback-rhel${RHEL_VER}`, `nvidia`. RHEL version comes from `rpm -E %rhel`.

The bundle is `/opt/docker-upgrade-bundle.tar.gz` everywhere (this was previously inconsistent with README and the `upgrade-docker.sh` header; both are now fixed).

## Releasing — every shipment gets a GitHub release

Not optional, and not a per-upgrade decision. The deliverable is a ~330 MB bundle of
RPMs that gets hand-carried onto disconnected servers; a git tag records which scripts
shipped but says nothing about which packages an operator installed, and an air-gapped
operator cannot rebuild the bundle. See `docs/RELEASING.md`.

```bash
tools/make-release.sh v29.6.2-1        # --draft to review first
```

Tag scheme is `v<TARGET_DOCKER_VERSION>-<BUNDLE_REVISION>`. The script refuses to run
on a dirty tree, with unpushed commits, on an existing tag, or with failing static
checks — and it **always rebuilds the bundle from the current checkout** rather than
reusing whatever is in the VM. Release notes enumerate every package from RPM
metadata, never filenames.

## Script versioning convention

Every script except `simulate-upgrade.sh` declares `VERSION="x.y.z"` on ~line 4 and echoes it in its startup banner. Only the scripts actually changed get bumped — versions across scripts drift on purpose. Currently:

| Script | Version |
|---|---|
| `upgrade-docker.sh` | 2.0.0 |
| `rollback-docker.sh` | 2.0.0 |
| `download-docker-packages.sh` | 2.1.0 |
| `clean-swarm-networks.sh` | 1.0.0 |
| `recover-dnf.sh` | 1.2.2 |

Commit subjects carry the new version in parens, e.g. `Fix NVIDIA toolkit upgrade failures (v1.2.2)`, with a bullet list body.

## upgrade-docker.sh structure

Numbered phase blocks, each fenced by a `####` comment banner: **0 validate payload** → 1 Swarm detect/drain → 2 pre-upgrade checks → 3 backup to `/root/docker-backup-<timestamp>/` → 4 stop services (+ verify stopped) → 5 rpm upgrade → 6 verify containerd config → 7 NVIDIA toolkit → 8 start services → 9 verification + version assertion → 10 Swarm reactivation. Add new work as a new numbered phase rather than inlining it elsewhere.

**Phase 4.5 is intentionally vacant.** It held the orphaned-network cleanup. The number is left unused so phases 5–10 keep the identities they have in the runbook and in logs from prior upgrades.

Things the phase structure encodes:

- **Phase 0 runs before anything mutates the node** — including before the Swarm drain. A payload problem must not leave a node drained.
- **The containerd 1.x hard stop is evaluated unconditionally**, before the already-at-target / partial-upgrade / unexpected-version branches. It was previously nested inside the unexpected-version branch, which made it bypassable: a node with docker-ce already at 29.6.2 but containerd still 1.x took the partial-upgrade branch and never reached it. Any guard that must always fire belongs outside the `if/elif/else`, not in one arm of it.
- **Already-at-target means all five packages**, not the core three. A partially applied transaction can leave correct core packages beside stale plugins, and that node still needs the run.
- **`rollback-docker.sh` has a phase 0b** that selects and confirms the containerd config backup *before* services stop. Choosing it in phase 3 (after the downgrade) left an operator who saw the wrong backup named with no option but to interrupt a half-finished rollback.
- **Swarm workers cannot drain or inspect themselves** — only managers can run `docker node` commands. Workers get printed instructions and a "has this been drained?" prompt; managers drain themselves interactively. `IS_MANAGER` comes from `.Swarm.ControlAvailable`.
- **Failure is reported, not guessed at.** An EXIT trap tracks `CURRENT_PHASE`, `SERVICES_STOPPED`, and a tri-state `PKG_STATE` (`untouched`/`attempted`/`installed`). It deliberately does **not** auto-restart services: after an rpm transaction, retry-vs-rollback is an operator judgement call. `PKG_STATE` is set to `attempted` *before* the rpm call, because a transaction can change host state and still fail.
- **NVIDIA is best-effort.** `libnvidia-container-devel` and `libnvidia-container1-debuginfo` are force-removed first because they pin old versions; failures warn instead of aborting. A corrupt NVIDIA RPM skips the toolkit upgrade rather than aborting the run. `nvidia-ctk runtime configure --runtime=containerd` is deliberately skipped — nvidia-ctk doesn't understand containerd config v3 yet.

## clean-swarm-networks.sh

Standalone remedy for a node that returns from an upgrade unable to attach to overlay networks (`failed adding service binding`). Deletes VXLAN interfaces, `/var/run/docker/netns/*`, `<data-root>/network/files/local-kv.db`, and `docker_gwbridge`. The data root is scraped out of `/etc/docker/daemon.json`.

Two things are load-bearing and must not be "simplified":

- **The inventory is enumerated after services stop, and deletion iterates exactly that captured list.** Enumerating before the stop, or re-globbing at deletion time, means the confirmation prompt can describe something different from what actually gets deleted.
- **Enumeration failure is not treated as an empty inventory.** `mapfile < <(cmd || true)` cannot see the command's exit status, so a failing `ip` would silently read as "nothing to clean". Output and status are captured separately, and a failed enumeration aborts.

Exits 2 when cleanup was incomplete, so a wrapper checking `$?` doesn't record a partial remedy as success.

## Bash conventions used here

- `set -e` everywhere, so any command whose failure is acceptable needs an explicit `|| true`.
- `exec > >(tee -a /var/log/docker-<action>.log) 2>&1` at the top of the long-running scripts.
- `RED`/`GREEN`/`YELLOW`/`NC` color vars with `echo -e`.
- Interactive prompts go through `prompt_yes_no "text [Y/n]" "y"`; the default is passed separately from the prompt string. Destructive prompts default to `n`. It **refuses EOF** rather than applying the default — otherwise a non-interactive run (ssh without `-t`, any wrapper) silently auto-answers every prompt, including the drain prompt, which defaults to yes.
- **Every script that calls a helper must also define it.** These are standalone files; there is no shared library. `rollback-docker.sh` once called `prompt_yes_no` without defining it, and because the call sat in an `if !` condition the missing command's exit 127 inverted to "abort", exiting **0** — a silent no-op emergency rollback that reported success. `bash -n` cannot see this and shellcheck does not flag unresolved commands, so `tests/static-checks.sh` check 1.12 enforces it.
- **Compare Swarm state exactly, never `grep -q "active"`** — a non-Swarm host reports `inactive`, which contains `active`.
- **Any new harness check must be mutation-tested.** A check that greps for a function *name* passes even when the definition is deleted, because the call site supplies the string. Break the thing deliberately and confirm the harness fails and exits non-zero before trusting it.
- EXIT traps do `local rc=$?` first, `[ "$rc" -eq 0 ] && exit 0`, and end with `exit "$rc"`. Paired with `trap 'exit 130' INT` / `trap 'exit 143' TERM` so signals route through the same reporting.
- Scripts assume GNU userland (`grep -oP`, `findmnt`, `xfs_info`) with fallbacks where it mattered — don't test behavior against macOS tooling.

---
> Source: [Mazyod/docker-airgap-upgrade](https://github.com/Mazyod/docker-airgap-upgrade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
