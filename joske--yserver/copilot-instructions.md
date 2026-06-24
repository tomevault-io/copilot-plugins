## yserver

> A modern X11 server written from scratch in Rust.

# yserver

A modern X11 server written from scratch in Rust.

Info about the project is in README.md
Current status is in docs/status.md and should be kept up to date
focus is on yserver (KMS) now

## Instructions

- it's fine not to use clippy pedantic in this repo but DO use regular clippy
- use `cargo +nightly fmt` for formatting
- design docs (specs) go in docs/superpowers/specs
- impl plans go in docs/superpowers/plans
- work on feature branch for phases
- squash merge when ready (ask confirmation)
- Spec compliance is the goal, but if Xorg deviates from spec (unlikely), we need to follow Xorg, clients are tested for 40+ years on Xorg.

## environment

- you are most likely running in a bwrap sandbox, if you see /home/jos/realhome, you are.
- the project dir is rw mounted
- in /home/jos/Projects/xserver/hw/kdrive/ephyr/ you can find the source to Xephyr for reference
- in /home/jos/Projects/xserver/hw/xnest/ you can find the sources to Xnest for reference
- to test you can run ynest with RUST_LOG=debug, capture its output
- x11trace is available if you want to trace how Xephyr/Xnest does things, x11trace always needs -n flag

---
> Source: [joske/yserver](https://github.com/joske/yserver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
