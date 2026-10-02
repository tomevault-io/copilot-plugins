## no-commit-images

> Never commit or push scan images, captures, or personal photo data


# No images in git

Never commit, stage, force-add, or push scan captures, decoded PNGs/TIFFs, or personal photograph data.

- `captures/` is gitignored on purpose — do not `git add -f` anything under it.
- Do not commit large image binaries elsewhere either (decoded strips, frame exports, sample scans).
- Tools and docs that *produce* or document decoding are fine; the image files themselves are not.
- If asked to "commit the strip / scan / PNG", commit only the code path — refuse the image and say why.

---
> Source: [gazzdingo/pakon-mac](https://github.com/gazzdingo/pakon-mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
