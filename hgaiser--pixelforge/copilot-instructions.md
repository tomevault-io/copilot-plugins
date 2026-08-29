## pixelforge

> Pixelforge is a Rust library for video encoding using Vulkan Video.

# Agent Instructions

## Project Overview
Pixelforge is a Rust library for video encoding using Vulkan Video.

## Build & Test
```bash
cargo build
cargo test
cargo run --example encode_h264
```

## README Generation
The `README.md` is generated from the doc comments in `src/lib.rs` using the `README.tpl` template.
To regenerate:
```bash
cargo readme --no-title --no-indent-headings > README.md
```
Do not edit `README.md` directly; update the doc comments in `src/lib.rs` instead.

To verify the quality of the encoded videos, run:

```bash
cargo run --example encode_h265 \
    && rm -f decoded.yuv \
    && ffmpeg -hide_banner -loglevel error -y -i output.h265 -pix_fmt yuv420p -f rawvideo decoded.yuv \
    && ffmpeg -hide_banner -loglevel info -s 320x240 -pix_fmt yuv420p -f rawvideo -i testdata/test_frames.yuv -s 320x240 -pix_fmt yuv420p -f rawvideo -i decoded.yuv -lavfi psnr -f null -
```

Make sure there are no Vulkan validation layer errors during execution.

## Code Style
- Follow `rustfmt.toml` formatting rules
- Run `cargo fmt` before committing
- Use clippy for linting
- If a comment is a sentence, it should end with a period
- Avoid long files; split into modules if necessary

## Project Structure
- `src/` - Library source code
- `examples/` - Usage examples for encoding
- `testdata/` - Test input files

## Key Dependencies
- Vulkan Video API
- ash (Vulkan bindings for Rust)

---
> Source: [hgaiser/pixelforge](https://github.com/hgaiser/pixelforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
