## bluesky-gradient

> Guidance for agents working on the unmaintained Rust gradient avatar/banner generator.

# AGENTS.md

Guidance for agents working on the unmaintained Rust gradient avatar/banner generator.

## Current implementation

- `cli.rs` selects profile (400x400), banner (1500x500), or custom output. Multiple mode flags use priority order rather than enforcing the comment's “exactly one” rule; custom height is currently `--custom-height`, while the README shows `--height`.
- `config.rs` deserializes hour-string RGB arrays and selects a new suffixed directory on every run. `gradient.rs` interpolates in linear RGB and fades toward greyscale; `render.rs` sizes centered text and adds random per-pixel grain.
- `main.rs` loads fixed CWD-relative `config/generation.json` and `config/fonts/madecarvingsoft.ttf`, then writes `00.png` through `23.png` under a new `src/profile_pics*`, `src/banners*`, or `src/custom_<w>x<h>*` directory.

## Invariants

- Validate dimensions before allocation; zero height breaks gradient math and huge custom values can exhaust memory.
- Validate palettes before `interpolate_colour`: keys are unwrapped as integers, arrays indexed as three bytes, fewer than two keys underflow the loop bound, and out-of-range hours may require exact keys.
- Keep text legible and within bounds for long/Unicode names. Make randomness injectable before asserting exact pixels.
- Preserve the non-overwriting output-directory suffix behavior and do not commit generated batches.

## Validation

Run `cargo fmt --check`, `cargo clippy --all-targets`, `cargo test`, and `cargo build --release`. There are currently no tests, so add focused coverage when changing CLI conflicts/names, suffix selection, palette parsing/interpolation, sizing, or noise. Generate all three modes from the repository root and inspect 24 PNGs, dimensions, contrast, missing font/config, malformed palettes, zero/huge sizes, and long names.

---
> Source: [ewanc26/bluesky-gradient](https://github.com/ewanc26/bluesky-gradient) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
