## bluewake

> No public releases until this repo is marked Clear in the maintainer's private release audit. Do not publish, re-publish, or restore any release, IPA, or app build, and do not add download links, until then.

# Agent instructions

## Releases paused

No public releases until this repo is marked Clear in the maintainer's private release audit. Do not publish, re-publish, or restore any release, IPA, or app build, and do not add download links, until then.

Before any future public release, every artifact must pass `scripts/release/check_public_assets.sh <artifact>...`, which runs `python3 ~/.codex/release-gate/release_gate.py` on the maintainer's machine. A failure is a stop, not a note.

## Personal builds stay personal

The game module (`gGZLE01_recomp.dylib`) is translated from the player's own disc. An IPA or app that contains it is a personal build: never upload, attach, commit or link it anywhere. Public releases are source (plus notices) only; players make their own IPA with `scripts/ios/build_device.sh DISC.iso --ipa OUT.ipa`.

---
> Source: [chrissotraidis/bluewake](https://github.com/chrissotraidis/bluewake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-28 -->
