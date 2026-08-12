## agentsbox

> Infrastructure-only repo (no application source, no tests, no package manager for

# Agent Instructions

Infrastructure-only repo (no application source, no tests, no package manager for
this repo itself): Bash/nushell/Node scripts, a `Containerfile`, pinned-tool Nix
files, and a `flake.nix` that builds the user-facing `agentsbox` command. For
build/run usage and repo layout see `README.md`.

## Commands

| Task | Command |
|------|---------|
| Dev shell (sets `AGENTS_TOOLS_DIR`, prepends `bin/`) | `nix develop` |
| Build image | `nu make.nu build` |
| Force rebuild (no cache) + drop nix store volume | `nu make.nu update` |
| Run container in a project dir | `nu make.nu run --workdir <dir>` |
| Verify host env (nix, podman, image) | `nu make.nu doctor` |

No test suite exists — do not create placeholder test files. The dev shell
(`nix develop`, which the container entrypoint re-execs into) puts `nu`, `bash`,
`podman`, and the repo's own `agentsbox`/`make.nu` on PATH (`AGENTS_TOOLS_DIR`
set, `bin/` prepended by the flake `shellHook`): exercise the real `config_*`
readers and source `make.nu` to run its real helpers (e.g. `port-flags`) directly
rather than reasoning about them in the abstract. `nu make.nu build`/`run`/
`doctor` still need a host — the `podman` binary is present in-box but nested
podman does not run inside an agentsbox container, so defer those to a host shell.

## External References

| Need | File |
|------|------|
| Build/run usage, repo layout, features | `README.md` |

## Conventions & Invariants

- **Rootless only:** never use `sudo` or run Podman as root.
- **Every `podman run`:** include `--rm` and `--security-opt no-new-privileges:true`.
- **Project hash parity:** `make.nu` `workdir-hash`, `bin/agentsbox` `sha1_8`, and
  `bin/list-secrets` all use SHA-1 (first 8 hex) of the canonical absolute workdir
  path. Do not change the algorithm without re-verifying all three.
- **Flake package:** `dontPatchShebangs = true` — scripts are `COPY`'d into a
  container with different nix store paths; patching shebangs breaks them.
- **Bash strict mode:** `set -euo pipefail` in every Bash script. Exception:
  `shell-entrypoint` uses only `set -e` (it re-execs through `nix develop` and
  runs fallback logic after failed commands).
- **Config is parsed, never sourced:** `.agentsbox/config.toml` and
  `~/.config/agentsbox.toml` are read with `nu`, never executed — a cloned repo
  must not run code at `enter` time.
- **`nu -c` TOML reads (Bash→nushell boundary only):** wrap in
  `2>/dev/null || true` for missing-key handling, but that also hides real parse
  errors — run the expression bare against a real TOML file first and confirm
  output before wrapping. A `config_*` reader that silently returns empty (its
  feature no-ops) is the signature of a swallowed parse error in its `nu -c` body —
  run the body bare to surface it. Inside `.nu` files, do not spawn `nu -c`; use
  `try { open $file | get k } catch { fallback }` inline.
- **`installed_agents` is global-only** (`~/.config/agentsbox.toml`): the image
  is a host-level artifact shared by all projects. Fixed set
  `{claude, codex, pi, opencode}`; absent key → all four. Switching requires a
  rebuild.
- **`--a2a` is incompatible with a custom `network`** — `enter` aborts rather
  than run a broken A2A mesh. Under `host`/`container:` network modes, `--auth`,
  `--web`, and `[[ports]]` port publishing are skipped (Podman rejects them).
- **`[[volumes]]` validation:** `name` matches `^[A-Za-z0-9_.-]+$`; `target` is
  absolute and must not collide with a built-in mount (`/workspace`, `/nix`,
  `/root/.claude`, …). Malformed entries are hard errors. Read from both
  `.agentsbox/config.toml` (project, prefixed `agent-<hash>-<name>`) and
  `~/.config/agentsbox.toml` (global, prefixed `agent-global-<name>`); on a shared
  target the project mount wins (mirrors secrets).
- **`[[ports]]` is project-only** (`.agentsbox/config.toml`): there is no global
  ports equivalent — host ports are scarce, not shareable like volumes. Fields
  `host`/`container` are integers; `bind` defaults to `127.0.0.1` (`0.0.0.0` →
  LAN). Transported to `make.nu` as `AGENTSBOX_PORTS` (one `host:container:bind`
  per line); `port-flags` emits `-p <bind>:<host>:<container>`, gated on
  `not no_publish` (same flag as `--auth`/`--web`).
- **Nushell list-typed flags** (`--foo: list<string>`) take a single `[a,b]`
  value, not repeated `--foo a --foo b`. Pass variable-length lists from Bash via
  env vars (one entry per line) — see `AGENTSBOX_VOLUMES` /
  `AGENTSBOX_GLOBAL_VOLUMES` / `AGENTSBOX_PORTS`.
- **`default (expr)` is eager** — use an `if` when the fallback has side effects.
- **Non-zero externals abort nushell** — wrap in `try { … } catch { … }` to
  convert to a custom message/exit.
- **Secrets are podman secrets**, never env vars or baked into
  `Containerfile`/`make.nu`. Project secrets are prefixed `agent-<hash>-<name>`,
  globals `agent-global-<name>`; on a shared mount target the project secret wins.
- **`:Z`** on all Podman bind mounts (SELinux-aware systems). Create host dirs
  with `mkdir -p` before mounting (`make.nu run` already does this).
- **Comments:** one-line header after the shebang stating *what* the file does;
  inline `#` only for non-obvious *why* (ordering deps, cross-file invariants,
  footgun guards). Never restate the code or leave commented-out code.

## Naming

- `bin/` scripts + config files: `kebab-case`, no extension.
- `make.nu`: `const` in `SCREAMING_SNAKE_CASE`; subcommands/helpers in
  `kebab-case`. Tasks are `export def "main <name>"`; helpers are plain `def`.
- Nix: `camelCase` (`buildInputs`, `shellHook`, `runtimeDeps`).

---
> Source: [mrdaak/agentsbox](https://github.com/mrdaak/agentsbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
