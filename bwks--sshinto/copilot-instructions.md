## sshinto

> This project is used to connect to network devices,

## About
This project is used to connect to network devices,
initally over SSH and apply config or return data.

The goal is to abstract the nuances of handling this process across
multiple network vendors and linux/bsd hosts.

## Documentation
- Always update the README.md file with relevant information based on the changes made in code.

## Project Structure
- `lib_sshinto` - Library functionality for project.
- `sshinto` - CLI utility crate.
- `models` - Shared data models.
- `test-specs/` - Test specifications for every supported device type (see below).

## Guidelines
- Never use `.unwrap()` except in tests.
- Prefer the `gh` CLI for all git and GitHub actions (commits, pushes, pull requests, etc.) over raw `git` commands.

## Test Specifications (`test-specs/`)

The `test-specs/` directory contains a mandatory test specification for every
device type defined in `models::DeviceKind`. These specs exist to ensure
consistent, verifiable behaviour across all supported platforms and to give
clear acceptance criteria when a new device type is added or an existing one
is modified.

### Structure

```
test-specs/
  lab-devices.md     # Current lab host for each device type — edit this, not the spec files
  common.md          # Cases that every device type must satisfy
  cisco_ios.md       # Device-specific cases for cisco_ios
  cisco_iosxr.md
  cisco_nxos.md
  juniper_junos.md
  arista_eos.md
  nokia_srlinux.md
  mikrotik_ros.md
  aruba_aos.md
  cumulus_linux.md
  sonic_linux.md
  linux.md
```

`common.md` defines six baseline test cases (connectivity, single command,
multiple commands, SCP upload, paging, invalid command handling) that apply
to every device type without exception. Each device-specific file extends
these with cases that target platform-specific behaviour such as privileged
mode entry, prompt format variants, and pager commands.

### Rules

1. **Every `DeviceKind` variant must have a spec file.** When adding a new
   device type to `models::DeviceKind`, create a corresponding
   `test-specs/<device_type>.md` file before merging.

2. **All common cases must pass.** The six cases in `common.md` are the
   minimum bar. A device type is not considered complete until all six pass
   against a real device.

3. **Run specs against lab devices before merging.** Consult
   `test-specs/lab-devices.md` for the current host assigned to each type.
   If none is listed, document this and test as soon as a device becomes
   available.

   Before running tests, copy `sherpa_ssh_config` to `~/.ssh/config` and
   remove any `ProxyJump` lines from it (they are only needed when accessing
   the lab from outside the local network). The `sshinto` binary does not
   read SSH config directly, but the file must be clean for any underlying
   SSH operations to resolve correctly.

4. **Update the spec when behaviour changes.** If a bug fix or new feature
   changes how a device is handled, update the affected spec file in the
   same PR.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bwks)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bwks)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
