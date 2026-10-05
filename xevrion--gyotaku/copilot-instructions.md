## gyotaku

> gyotaku indexes every screenshot by its OCR'd text so it can be searched. `crates/core` (index, config) and `crates/ocr` (PP-OCRv6 on ONNX Runtime) know nothing about the window; `crates/cli` is the `gyotaku` command and background reader; `crates/app` is the gpui search window.

# Project coding standards

gyotaku indexes every screenshot by its OCR'd text so it can be searched. `crates/core` (index, config) and `crates/ocr` (PP-OCRv6 on ONNX Runtime) know nothing about the window; `crates/cli` is the `gyotaku` command and background reader; `crates/app` is the gpui search window.

## Communication

- Be succinct. Prefer code over prose.
- Do not summarise your changes afterwards unless asked.
- Do not apologise when corrected; give the correct answer.
- State what you did not verify. An honest gap costs less than a confident wrong claim.

## Correctness

- Every number in the README and docs is measured on real hardware, never estimated or extrapolated. Paste the measurement when you change one.
- A behaviour is not finished until it has run for real: the app opened, a real screenshot read, a real search returned.
- Fail soft. No gpu, no systemd, no network, old glibc, a broken config or a huge image must never crash anything or harm the machine.
- Never write to, move or delete a user's screenshots. The one exception is an explicit, confirmed move to the system trash (`crates/core/src/trash.rs`), which is always undoable and never deletes.
- Keep it generic. No code path or text assumes one desktop, compositor, distro or gpu vendor.

## Rust

- Rust 1.95, pinned in `rust-toolchain.toml` because gpui needs `std::hint::cold_path`.
- `cargo clippy --workspace -- -D warnings` clean, `cargo fmt --all` formatted.
- `anyhow` for errors in binaries, with context on anything touching the filesystem or network.
- Avoid allocation on hot paths: frame rendering and per keystroke search.
- Comments explain why, not what, and read like a person wrote them.

## Traps already fallen into

- gpui's `img(path)` loader leaks about 9 MB per page of scrolling. Thumbnails are decoded in `crates/app/src/images.rs` and passed as `ImageSource::Render`. Never switch back.
- Animation steps by real elapsed time (capped at 0.25 s). A fixed step per frame made motion crawl on slow machines and swallowed escape.
- The app stays resident and later launches wake it over a unix socket. Run it from source with `--once`, or you see the installed build.
- The wayland clipboard dies with the app that set it; copying goes through `wl-copy` when present.
- `pkill -f` with a pattern that appears in your own command line kills your own shell. Use `pgrep -x` or a PID.
- `wtype` decodes the first key of each call with the previous call's keymap. Start every call with `-k Shift_L`.
- Dependencies build without debug info in dev. Full debug info once filled 18 GB of `target/debug`.

## UI

- The screenshots are the colour; chrome is off white or near black, IBM Plex Sans.
- Shu (vermilion) only ever means "the search found this". Selection is ink.
- Nothing animates on its own. Motion answers input and is interruptible.
- Without a gpu, motion becomes short fades.

## Copy

- README and docs are written in clear, neutral, professional English, with sentence case headings. No first person.
- No em dashes and no unicode arrows anywhere: code, comments, commits, docs.

## Commands

```sh
cargo run -p gyotaku-app -- --once     # the window, from source
cargo run -p gyotaku -- ocr some.png    # read one image
cargo test --workspace
cargo clippy --workspace -- -D warnings
cargo fmt --all
```

Distro builds run in GitHub Actions (`.github/workflows/ci.yml`); don't build distro containers locally.

---
> Source: [xevrion/gyotaku](https://github.com/xevrion/gyotaku) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
