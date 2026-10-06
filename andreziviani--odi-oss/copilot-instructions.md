## odi-oss

> Guidance for any coding agent (or human) working in this repository. Read

# AGENTS.md

Guidance for any coding agent (or human) working in this repository. Read
the top-level `README.md` first for what this project is; this file is about
how to work in it safely and correctly. `docs/HACKING.md` has the detail
behind every rule here.

## Layout

    Makefile                the whole build; `make help` lists every target
    toolchain/               which toolchain images we build with: images.env (digest pins), image.sh; built in the odi-toolchain repo
    kernel/                  Linux 6.18 for the RTL9602C
      618/fetch.sh, mainline/, patches/, config
      618/debug/             debug-only patches, applied with CRUMBS_CORE=1
      build.sh
      extra/                 every file of our own, at its tree path (board, CPU cache, irqchip, timer, SPI NOR)
      extra/drivers/net/ethernet/odi/   our own switch/GPON/NIC/OMCI/watchdog/ramlog drivers
    packages/                upstream userland: busybox, dropbear, iproute2 (bridge-utils deliberately not built)
    rootfs/skeleton/         /etc and the init chain, ours; lib/firmware/odi/ the register replay tables
    image/                   assembles squashfs + uImage into a flashable tarball, and the on-device flasher (fwu.sh)
    src/                     our own tools: diag, omci (omcid/omcli/omciprobe/omcicap), igmp, nv — freestanding
    test/                    host-side and qemu-based test harnesses
    tools/                   developer tooling: remote-build.sh, kernel-footprint.sh, objdiff.sh, fetch-refs.sh, memprobe, regdump, regtrace, lint helpers
    docs/                    HACKING.md (contributor guide), BOOT.md, KERNEL.md, SWITCH.md, TOOLS.md, SETTINGS.md, BUILDING.md, FLASHING.md, ACCESS.md, CROSS-COMPILING.md, IMPROVEMENTS.md, LICENSING.md, REFERENCES.md; kb/ field notes

## Build and test commands

    make image-all           # everything, from a clean clone
    make image                # just the tarball, once the pieces exist
    make test                  # lint + test-host + test-diag + test-omci (~2 min)
    make test-host              # host-side unit tests, no Docker/stick needed
    make test-diag              # diag, including the exporter contract (byte-for-byte goldens)
    make test-rcs               # what rcS executes and writes to /proc, three flag sets, against goldens (needs out/busybox)
    make lint                   # shellcheck + repo-specific checks, see below

    kernel/build.sh              # the kernel alone (pulls the toolchain image itself)
    ODI_REMOTE=user@yourbuildhost tools/remote-build.sh 'make image-all'
                                  # build on a native x86_64 Linux host instead of
                                  # under emulation; see docs/BUILDING.md. ODI_REMOTE
                                  # is a placeholder you must set — this repo names
                                  # no host of its own.

`docs/BUILDING.md` has the full breakdown of every build target.

## House rules

- **No proprietary or vendor-sourced code or references, anywhere in this
  repo.** This project builds a complete replacement firmware from mainline
  Linux, public specifications (ITU-T G.984.3, G.988, SFF-8472) and our own
  independent implementation, written from hardware behaviour observed on
  the device; it ships and links no proprietary code. Do not
  add anything derived from, or copied from, a vendor's SDK or firmware
  source, and do not name proprietary vendor files, functions or symbols in
  code, comments, or docs — describing how the **stock (OEM) firmware
  behaves**, as an observed black box, is fine and often necessary; reading
  or citing its source is not.
- **No apostrophes in shell-script comments.** A single quote inside a
  single-quoted inline block (several scripts here run `docker run ... bash
  -c '...'`, where the whole block is one shell word) silently terminates
  the word and runs the rest of the line in the outer shell — this has cost
  real debugging time more than once. Rephrase; do not escape.
  `tools/check-inline-quotes.sh` (run by `make lint`) enforces this inside
  inline `bash -c` blocks specifically, but the safest habit is to avoid
  apostrophes in shell comments everywhere.
- **Every user-visible improvement goes into the README comparison table**
  (`README.md`, "area / stock / this firmware") in the same change, with
  a number where there is one, and only claims you can back: a
  measurement, a test, or the code.
- **Keep the mainline footprint minimal.** `kernel/618/patches/` holds only
  edits to files that exist in mainline; every new file lives in the
  `kernel/extra` overlay. Before adding a hunk to a mainline file, look for
  a hook that already exists (mach headers, board callbacks, our own Kconfig
  `select`s). Fold a fix into the patch it belongs to rather than stacking
  a new one, so each patch stays one coherent change, and check
  `tools/kernel-footprint.sh` before and after: the count only goes down
  without a stated reason. This is what makes the next LTS port cheap.
- **Name the test sticks by their line, ISP1 and ISP2**, in code, comments,
  docs and commit messages, and never write a credential, a serial number
  or another identity value from a real stick into the tree. `make lint`
  does not check this; the review does.
- **Every change that affects users, the build, or the docs adds an entry
  under `## Unreleased` in `CHANGELOG.md`, in the same commit.** A release
  moves `Unreleased` into a version section named after the tag
  (`tools/release-notes.sh` reads that section by heading).
- **Every wait on another process is bounded.** A stuck omcid has, on
  hardware, left both a shell script and the exporter waiting on it
  forever (`CHANGELOG.md`, "Unreleased"). See `docs/SETTINGS.md`, "Bounded
  waits", for the pattern per case — a forked command goes through busybox
  `timeout` (already built; `packages/busybox/config.fragment`), a bare
  `echo verb > /proc/...` write goes through `write_proc_bounded`/
  `read_proc_bounded` (`rootfs/skeleton/etc/scripts/rcs-lib.sh`), and a
  freestanding C daemon polls its child with a timeout and `SIGKILL`s it on
  expiry rather than blocking in `read()`/`waitpid()`. Pick a real bound for
  the specific call, not a copy-pasted one, and document it next to the
  call when it is not obvious. Two documented exceptions exist: `rcS.dev`'s
  dev-hook diag probes (a process parked in an uninterruptible-sleep kernel
  wait ignores `timeout`'s signal too, so a hang of that specific kind is
  not fixable this way and is not pretended to be), and `rcS`'s 23-step
  switch SDK-init loop, every boot with no exception, left unbounded on
  cost rather than risk -- `write_proc_bounded` turns a zero-fork builtin
  into a fork pair, and no step there has ever been observed to hang. Bound
  a hot, unproven path only once it actually hangs.
- **Check for existing lint/contribution rules before adding a new pattern.**
  `make lint` (see below) is the authority. `docs/HACKING.md` is the full
  contributor guide (the recovery mechanism, the gates, the known traps,
  every feature toggle) and `CONTRIBUTING.md` the short version; this file
  stays the short list of rules. Search `tools/*lint*` and the `lint:`
  target in the Makefile before assuming a check does not exist.

### What `make lint` actually checks

- `shellcheck` over every build/host script (bash), and separately, in POSIX
  `sh` mode, over the scripts that run **on the device** (`rootfs/skeleton/`,
  `tools/regdump/dump.sh`) — those are interpreted by busybox `ash`, not
  bash, and a failure there costs a stick, not just a build.
- `tools/check-inline-quotes.sh` — the apostrophe/inline-quote rule above.
- A `git grep` guard against references to the private investigation
  workspace this project was developed alongside (paths, document names,
  internal task-tracking IDs). If you did your work in, or copied notes
  from, an external workspace, restate the point in this repo's own words
  rather than naming or linking that workspace.
- A `git grep` guard against machine-local and private paths: a home
  directory in any spelling, the root home other than the documented
  remote build directory, the backups directory of a stick. A tool that needs such a path takes it from a
  variable (`STOCK_ROOTFS` for the extracted stock rootfs).
- A `git grep` for known vendor source citations: file and function names,
  and the headers of the vendor SDK (`rtk/*.h`, `rtk_*.h`).
- A `git grep` for trial history in `kernel/extra`, `rootfs/skeleton` and
  `src`: trial, image and boot ids, dates, old patch numbers, "this pass".
  A comment says what is true of the code now; the story goes in the
  commit message (`docs/HACKING.md`, "Repo rules").

## Stick safety rules

See `docs/FLASHING.md` for the full procedure; the essentials, because
getting these wrong can brick a stick or take down the fiber service behind
it:

- **Flash only the inactive slot**, from the stock (OEM) firmware or from
  this image — never the slot you are currently running from. `fwu.sh`
  refuses to write the running slot itself, but do not rely on that as your
  only check.
- **A freshly flashed slot is inert until `nv setenv sw_tryactive <slot>`**,
  which boots it exactly once with the watchdog armed.
- **Never `sw_commit` a trial before it has proven itself.** Commit only
  from the booted trial image, with `nv commit <slot>`, once you have
  checked it yourself. Committing before that discards the only free safety
  net this mechanism gives you, and a committed image that later crashes
  reboots into itself with no way back to the other slot.
- **rcS confirms userland to the watchdog within 120 s on every boot**, or
  the board resets (`docs/FLASHING.md`). The confirmation needs no network;
  only with `/etc/config/confirm-arp` (development) does it wait for an ARP
  reply from the `.2` address of the management subnet. Do not change the
  confirmation or the deadline without reading `rcS`, `rcS.dev` and
  `odi_wdt.c` together.
- **A reboot with no commit returns to the stock (or previously committed)
  firmware.** That is the whole point of the mechanism; do not "fix" a
  stuck trial by writing `sw_commit` to make it stick — power-cycle it
  instead, or fix the image and try again.
- **Nothing commits a trial automatically, and nothing should.** A trial is
  committed only by a person, with `nv commit <slot>`. Until then the image
  says so (`slot-state.sh`: the ssh login banner, `/var/run/odi-slot`,
  `gpon_uncommitted`); do not add code that acts on that state.

## Dangerous commands

- **`omcicli get tables` wedges the stock OMCI daemon.** omcid answers it
  safely, but the same command on the stock slot takes the line down. Do not
  run it.
- **`diag` reading from a stdin that never closes waits forever** for the
  next command. Always give it a pipe or a file that ends, and wrap it in
  `timeout` in anything that must not stall (rcS, network.sh).
- **Reading an undecoded SoC register address with `devmem`** can stall the
  bus until the hardware watchdog resets the stick. Do not probe addresses
  you cannot already account for from `docs/KERNEL.md` or the driver source.
- **`omciprobe` (writes) and `omcicap` (capture)** both interfere with
  `omcid` — see `docs/TOOLS.md` before running either against a device that
  needs to stay provisioned.

## Where the logs are

- **The DRAM ramlog** (`docs/KERNEL.md`, `tools/memprobe/README.md`)
  survives a watchdog reset (not a power cycle) and is the only way to see
  console output from a boot that never came up on the network. After a
  reset, read it from the *other* (working) slot: `cat
  /proc/odi_ramlog_prev` when that slot runs this image (check its slot and
  build id are the failed boot's), `tools/memprobe` when it runs the stock
  one. With `print-fatal-signals=1` it also holds the registers of any
  process killed by a signal.
- **`/etc/config/breadcrumbs`**, enabled with `: > /etc/config/breadcrumbs.on`
  on the currently running image before a trial: one timestamped line per
  boot stage, on the config partition, which survives a revert the same way
  the ramlog does. `docs/FLASHING.md` has the procedure.
- **`/var/log/`** on a booted image — `omcid.log` in particular records
  every OMCI frame and driver call for that boot.

## Release checklist

A release is cut only when the maintainer asks for one. Before tagging:

1. **Every change since the last tag is in the changelog.** Walk
   `git log --oneline <last-tag>..origin/main` and check that each merged
   change has its entry under `## Unreleased`. Add any that are missing in
   the release commit.
2. **No merge debris.** `grep -nE '^(<<<<<<<|=======|>>>>>>>)' CHANGELOG.md`
   finds nothing.
3. **Move `## Unreleased` into the version section** named after the tag,
   with the date, and leave an empty `## Unreleased` above it. Commit it as
   `CHANGELOG: <tag>`, then tag that commit (`git tag -s`).
4. **Check the published release** (`gh release view <tag>`): the assets are
   there and the notes are the new section, not an empty one.
5. **odi-oss only: the pins match.** `src/fetch-releases.sh` pins the latest
   odi-ui (`CONFD_TAG`) and odi-sfp-exporter (`METRICSD_TAG`) releases that
   should ship. If a sibling repo has unreleased changes the image needs,
   release it first and bump the pin, with its own changelog entry, before
   tagging odi-oss.

---
> Source: [AndreZiviani/odi-oss](https://github.com/AndreZiviani/odi-oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
