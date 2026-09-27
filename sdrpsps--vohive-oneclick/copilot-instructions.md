## vohive-oneclick

> This repository publishes VoHive v1.5.5 one-click installers for Linux. It does not contain the VoHive application source. The application binaries are opaque upstream artifacts packaged for `amd64` and `arm64`.

# Repository Instructions

## Purpose and scope

This repository publishes VoHive v1.5.5 one-click installers for Linux. It does not contain the VoHive application source. The application binaries are opaque upstream artifacts packaged for `amd64` and `arm64`.

These instructions apply to the entire repository.

## Supported behavior

- Target only Linux systems using systemd.
- Support both `amd64`/`x86_64` and `arm64`/`aarch64`.
- Keep the online path as one command through `install.sh`.
- Keep each offline `.run` file self-contained and network-free when executed directly.
- Install under `/opt/vohive` and register `/etc/systemd/system/vohive.service`.
- Preserve an existing `/opt/vohive/config/config.yaml` during reinstall or upgrade.
- Back up an existing binary to `/opt/vohive/bin/vohive.bak` before replacement.
- Install `/opt/vohive/uninstall.sh`. Interactive execution must ask whether to remove config, data, and logs, defaulting to preservation. Non-interactive execution and `--keep-data` must preserve them; `--purge` may remove them without prompting.
- Never execute the README's modem VID/PID-changing AT commands from an installer. They modify hardware-persistent state and must remain an explicit manual operation.

## Repository map

- `README.md`: user-facing VID/PID, online installation, offline installation, and maintenance instructions.
- `install.sh`: POSIX-shell online bootstrap. It detects the CPU architecture, downloads the matching `.run` file, verifies the top-level SHA-256, installs dependencies, and executes the installer. GitHub Raw failures fall back to `ghfast.top`.
- `uninstall.sh`: POSIX-shell uninstaller. It interactively asks whether to clear state, supports explicit `--keep-data` and `--purge`, and defaults to preserving state when non-interactive.
- `packaging/install-offline.sh`: Bash systemd installer embedded in both offline packages.
- `packaging/self-extract.sh`: POSIX-shell header prepended to each compressed payload.
- `packaging/config.default.yaml`: default configuration installed only when no configuration exists.
- `packaging/vohive.service`: systemd unit installed on the target.
- `packaging/mcc-mnc-table.json`: MCC/MNC-to-country-and-operator lookup data installed at `/opt/vohive/data/mcc-mnc-table.json`.
- `scripts/build-offline-installers.sh`: builds both self-extracting `.run` files from explicitly supplied binaries and rewrites `checksums.txt`.
- `tests/test-installers.sh`: validates shell syntax, top-level checksums, payload extraction, internal checksums, and ELF architectures.
- `vohive-offline-v1.5.5-linux-{amd64,arm64}.run`: generated release artifacts; do not edit by hand.
- `checksums.txt`: generated SHA-256 values for the two `.run` artifacts.

## Shell and portability rules

- `install.sh`, `uninstall.sh`, and `packaging/self-extract.sh` must remain POSIX `sh` compatible.
- `packaging/install-offline.sh`, the builder, and tests may use Bash.
- Do not add dependencies when standard shell tools are sufficient.
- Do not hardcode developer-specific absolute paths. The build script must continue to require both binary paths as arguments.
- Quote paths and variables. Keep `set -eu` or `set -euo pipefail` enabled as appropriate.
- The online bootstrap's temporary-directory signal traps must exit after `INT` or `TERM`; never clean the directory and then continue execution.
- Online mode sets `VOHIVE_INSTALL_DEPS=1` and installs `socat`, `usbutils`, and `pciutils` with `apt-get`, `dnf`, or `yum`.
- Direct execution of an offline `.run` file must not contact package repositories or any other network endpoint. It requires Bash, `awk`, `tail`, `tar`, `sha256sum`, and systemd; `socat`, `lsusb`, and `lspci` are optional tools that it only reports as missing.

## Generated artifacts and release invariants

The version appears in filenames, `install.sh`, the builder, and documentation. A version change must update all of them together.

Never modify a `.run` file or `checksums.txt` manually. Rebuild both architectures whenever any of these inputs changes:

- either VoHive binary;
- any file under `packaging/`;
- `uninstall.sh`;
- the version or generated filename format.

Do not rebuild or modify the `.run` files and `checksums.txt` for changes limited to `README.md`, `AGENTS.md`, the online `install.sh`, or tests. Those files are not part of the offline payload. Binary artifacts should change only when an embedded payload input or its packaging format actually changes.

Rebuild with explicit inputs:

```bash
./scripts/build-offline-installers.sh \
  /path/to/vohive_v1.5.5_linux_amd64 \
  /path/to/vohive_v1.5.5_linux_arm64
```

The builder must reject binaries whose ELF architecture does not match the supplied slot. It must regenerate both root checksums and the checksums embedded in each payload.

Raw binaries are intentionally not tracked as separate files. For a packaging-only change, an agent may extract the unchanged `vohive` binaries from the two current `.run` payloads into a temporary directory and pass those temporary paths to the builder; follow the extraction logic in `tests/test-installers.sh` and never commit the extracted binaries. For a VoHive binary or version upgrade, require user-supplied trusted binaries instead of downloading or substituting unknown artifacts. Add or update a `Binary provenance` subsection in `README.md` with the source, original filenames, and SHA-256 of both input binaries; keep `checksums.txt` limited to generated `.run` hashes.

`install.sh` is pinned to `https://raw.githubusercontent.com/sdrpsps/vohive-oneclick/main`. If the repository identity changes, update the default base URL and every user-facing direct and `ghfast.top` command in `README.md` together.

## Installation safety invariants

- Validate the operating system, architecture, required payload files, and SHA-256 values before installing.
- Do not weaken checksum failures into warnings.
- Keep the default configuration mode at `0600`, the binary at `0755`, and data/service files at `0644`.
- Do not overwrite an existing user configuration.
- Do not delete configuration, data, or logs without an affirmative interactive answer or the explicit `--purge` argument. Empty input, a negative answer, non-interactive execution, and `--keep-data` must preserve them.
- Preserve rollback behavior when service restart or health verification fails.
- Keep the default Web warning visible: the initial credentials are `admin / admin` and must be changed after login.
- Treat `mcc-mnc-table.json` as a required runtime asset. It is operator metadata, not APN or modem firmware configuration.
- Do not add Docker support unless explicitly requested; this repository currently targets native systemd installation only.

## Validation

Run the repository check after every change:

```bash
./tests/test-installers.sh
git diff --check
```

If a packaging input changes, rebuild both `.run` files first and then run the checks. A valid result must confirm:

- both top-level checksums;
- POSIX/Bash syntax as applicable;
- successful extraction of both payloads;
- all internal payload checksums;
- x86-64 ELF in the amd64 package;
- AArch64 ELF in the arm64 package.

The local test does not prove a real systemd deployment, USB modem operation, config preservation, installed permissions, rollback behavior, package-manager behavior, or absence of network calls in offline mode. Those require code review or integration testing. Do not claim systemd or hardware behavior was tested unless it was exercised on a supported Linux host with the relevant hardware.

## Change checklist

Before finishing a change:

1. Keep `README.md` commands consistent with actual script behavior.
2. Confirm online and direct-offline modes still have different dependency behavior: online installs dependencies; offline only reports missing optional tools.
3. Rebuild generated artifacts if any payload input changed.
4. Run `./tests/test-installers.sh` and `git diff --check`.
5. Review `git status` so generated files and `checksums.txt` are not accidentally omitted.

---
> Source: [sdrpsps/vohive-oneclick](https://github.com/sdrpsps/vohive-oneclick) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
