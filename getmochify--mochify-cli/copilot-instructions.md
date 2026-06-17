## mochify-cli

> Use this skill whenever the user wants to compress, convert, resize, crop, or rotate images. Triggers on requests like "compress this image", "convert to WebP/AVIF/JXL", "resize to X pixels wide", "make this image smaller for the web", or any image optimisation task. Calls the mochify `squish` tool with the appropriate parameters.


# Mochify — Image Compression & Conversion

Use the `squish` tool to process images via the mochify.app API.

## Parameters

- **file_path** (required) — absolute path to the image file
- **type** — output format: `jpg`, `png`, `webp`, `avif`, `jxl`
- **width** — target width in pixels (height scales proportionally unless `height` is also set)
- **height** — target height in pixels
- **crop** — set `true` to crop to exact `width`×`height` rather than letterboxing
- **rotation** — degrees: `0`, `90`, `180`, `270`
- **output_dir** — directory to write the result (defaults to same directory as input)

## Format guidance

| Goal | Recommended format |
|---|---|
| Web photos | `avif` or `webp` |
| Lossless / transparency | `png` |
| Maximum compression | `jxl` |
| Broad compatibility | `jpg` |

## Tips

- If the user doesn't specify a format, default to `avif` for photos and `png` for images with transparency.
- If the user says "resize" without a format, keep the original format.
- For "web-optimised" or "compress for web" requests, use `avif` at the user's desired width (or 1200px if unspecified).
- Multiple files can be processed in sequence with separate `squish` calls.
- Always confirm the output path back to the user after processing.

---
> Source: [getmochify/mochify-cli](https://github.com/getmochify/mochify-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
