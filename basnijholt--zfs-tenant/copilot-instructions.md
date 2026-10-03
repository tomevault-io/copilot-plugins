## zfs-tenant

> - **KISS**: The kernel enforces the boundary (ZFS delegation, `zfs zone`, quota, limits). The Python code only removes the shell, scopes names, and keeps the zone's namespace alive; keep it small enough to audit in one sitting.

# zfs-tenant Development Guidelines

## Core Principles

- **KISS**: The kernel enforces the boundary (ZFS delegation, `zfs zone`, quota, limits). The Python code only removes the shell, scopes names, and keeps the zone's namespace alive; keep it small enough to audit in one sitting.
- **YAGNI**: Only the owners' requirements: the tenant sees nothing of the host, changes nothing outside its root, manages datasets inside it, stays under a quota, and uses sanoid/syncoid. No pull mode, no zrepl, no config files, no retention features. New gate commands only when a real client (syncoid, a restore) needs them.
- **DRY**: `names.py` is the single source of truth for what a dataset or snapshot name may look like and whether it is inside the tenant root.

## Architecture

```
src/zfs_tenant/
├── __init__.py   # Version only
├── __main__.py   # python -m zfs_tenant (and the .pyz entry point)
├── cli.py        # argparse: gate, setup, zone, authorized-key
├── names.py      # dataset/snapshot validation and scoping
├── grammar.py    # SSH_ORIGINAL_COMMAND -> Reply | Run | Receive | ResumeSend | Rejected (pure)
├── gate.py       # executes a request, encryption policy, syslog
├── setup.py      # idempotent tenant root: properties, zoned=on, delegation
├── zone.py       # holder process + zfs zone (root side); setns into it (gate side)
└── zfs.py        # captured subprocess runner, CommandError
nix/
├── host-module.nix       # services.zfs-tenant
├── integration-test.nix  # two-node VM test, real OpenZFS, sender on nixpkgs services.syncoid
└── integration/          # throwaway SSH keys for the VM test only
docs/                     # Zensical site, deployed to https://zfs-tenant.nijho.lt
```

## Key Design Decisions

1. **Kernel first**: `zfs allow -l -u U create,mount,receive R` plus `zfs allow -d -u U create,destroy,mount,receive,send R`. The root itself can never be destroyed or reconfigured by the tenant, even with a shell. Never add `-l` destroy rights or any property permission.
2. **The grammar is pure and never shells out**: `grammar.parse` tokenizes with `shlex` (punctuation split out), validates every token with `names.py`, and returns an argv the gate executes with `subprocess.run` and no shell. Reasons in `Rejected` depend only on the input, so they are safe to show the tenant.
3. **Scope before zfs runs**: names outside the root are rejected by the grammar, so zfs error messages about other datasets never reach the tenant.
4. **Standard library only**: the gate must run from a single `zfs-tenant.pyz` on TrueNAS SCALE. Do not add dependencies.
5. **Encryption policy never destroys pre-existing data**: the gate refuses to receive into an existing unencrypted dataset and destroys only datasets that the current receive left unencrypted, whether it created them or a forced receive replaced them.
6. **Zone with the tenant's own uid**: `zfs-tenant zone` (root) forks a holder that drops to the tenant user and creates a user namespace mapping that uid to itself, then `zfs zone`s the root to it and writes the holder's pid. The gate `setns()`es into it before parsing and fails closed when it cannot. Never map the tenant to root inside the namespace: ZFS treats namespace root as the zone administrator, which bypasses `zfs allow` (verified: it can destroy the tenant root).
7. **Probes answer with nothing**: `command -v` exits 1 (so syncoid uses neither mbuffer nor compression on the host), `ps -Ao args=` prints nothing (a real `ps` would leak the host's process list).
8. **Sender runs as non-root with `send,hold`**: `zfs send -I` takes temporary holds; syncoid 2.3.0 pastes the receiver's resume token into a local shell, so a root sender would trust the host with root. The README's sender example is nixpkgs `services.syncoid` with `localSourceAllow = [ "send" "hold" ]` (its default also grants `snapshot` and `destroy`); the VM test's sender node runs that same configuration, so change both together. This project ships no sender module.

## Development Commands

Use `just` for common tasks. Run `just` to list available commands:

| Command | Description |
|---------|-------------|
| `just install` | Install dev dependencies |
| `just test` | Run the unit tests (parallel) |
| `just lint` | Lint, format, and type check |
| `just pyz` | Build `dist/zfs-tenant.pyz` |
| `just vm-test` | Two-node NixOS VM test with real OpenZFS and syncoid |
| `just docs` | Regenerate `docs/` from README sections and build the site |
| `just docs-serve` | Serve the docs site locally |
| `just clean` | Clean build artifacts |

## Testing

- Unit tests never touch ZFS. `tests/test_gate.py` injects fake `_Effects` (runner, spawner, log, error writer); `tests/test_cli.py` runs `python -m zfs_tenant gate` against a fake `zfs` shell script to check the environment, stdin, and exit-status wiring.
- `nix/integration-test.nix` is the ground truth for ZFS and syncoid behavior. When the grammar changes, run `just vm-test`. When a syncoid push fails there, the test prints the push journal and the gate's auth log.
- Any new accepted command needs rejection tests for its near misses (outside the root, the root itself, extra flags).

## Documentation

- `README.md` is the single source of truth. Its `<!-- SECTION:name:START/END -->` markers feed the `docs/*.md` pages through `markdown-code-runner`'s `include_section`; only `docs/design.md` is written by hand.
- After editing the README, run `just docs` and commit the regenerated `docs/*.md`. CI regenerates them before every build anyway, so the site never goes stale.

## Communication Notes

- Clarify ambiguous wording (e.g., homophones like "right"/"write", "their"/"there").

## Git Safety

- Never amend commits.
- **NEVER merge anything into main.** Always commit directly or use fast-forward/rebase.
- Never force push.
- Stage files by name; `git add -A` is blocked by a hook.

## Pull Requests

- Never include unchecked checklists (e.g., `- [ ] ...`) in PR descriptions. Either omit the checklist or use checked items.
- **NEVER run `gh pr merge`**. PRs are merged via the GitHub UI, not the CLI.

## Releases

Use `gh release create` to create releases. The tag is created automatically; `release.yml` publishes to PyPI through trusted publishing and attaches `zfs-tenant.pyz` to the release.

```bash
# IMPORTANT: Ensure you're on latest origin/main before releasing!
git fetch origin
git checkout origin/main

# Check current version
git tag --sort=-v:refname | head -1

# Create release (minor version bump: v0.1.0 -> v0.2.0)
gh release create v0.2.0 --title "v0.2.0" --notes "release notes here"
```

Versioning:
- **Patch** (v0.1.0 → v0.1.1): Bug fixes
- **Minor** (v0.1.1 → v0.2.0): New features, non-breaking changes

Write release notes manually describing what changed. Group by features and bug fixes.

---
> Source: [basnijholt/zfs-tenant](https://github.com/basnijholt/zfs-tenant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
