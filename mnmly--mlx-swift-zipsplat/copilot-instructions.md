## mlx-swift-zipsplat

> Swift/MLX port of [ZipSplat](https://github.com/cvg/ZipSplat). Read `README.md` first for

# mlx-swift-ZipSplat — agent notes

Swift/MLX port of [ZipSplat](https://github.com/cvg/ZipSplat). Read `README.md` first for
setup and the verification story.

## Layout

```
Sources/MLXZipSplat/            library: model, ZipSplatSession, .ply export
  Documentation.docc/           DocC catalog (landing page + Topics)
Tools/zipsplat-tool/            CLI: reconstruct + bench
Examples/ZipSplatDemo/          SwiftUI viewer (own .xcodeproj, local package refs)
Tests/MLXZipSplatTests/         parity + session contract tests
  Fixtures/                     small committed safetensors (stats + slices)
Scripts/                        weight conversion, fixture dump, PLY/render comparison
```

## Build and test

Use `xcodebuild`, never `swift build` / `swift test` — the latter cannot load mlx-swift's
`default.metallib` at runtime, so any test touching an MLX op crashes.

```bash
xcodebuild -scheme MLXZipSplat-Package -destination 'platform=macOS' test
xcodebuild -scheme zipsplat-tool -configuration release -derivedDataPath .xcdd build
xcodebuild -project Examples/ZipSplatDemo/ZipSplatDemo.xcodeproj -scheme ZipSplatDemo \
  -destination 'platform=macOS' -derivedDataPath Examples/ZipSplatDemo/.xcdd build
```

Benchmarks must be Release — Debug is roughly 5x slower and the numbers are meaningless.

## Invariants

**Never change the model without re-running parity.** `ParityTests` and `SessionTests` are
the contract. If a change moves the numbers, that is a regression until proven otherwise —
the port is verified to 46 dB render PSNR against the reference and should stay there.

**The `.ply` stays in the model's native OpenCV frame** (+Y down, +Z forward), matching the
reference writer. Only the renderer bridge in `Examples/ZipSplatDemo/SplatBridge.swift`
applies the Y-up flip. Do not "fix" the exporter to be Y-up.

**LayerNorm eps differs by call site.** Backbone blocks' `norm1`/`norm2` use 1e-6; everything
else — qk-norms inside attention, and all of ZipSplat's own blocks — uses PyTorch's default
1e-5 (`zipSplatLayerNormEps`). Python's `SelfAttentionBlock` does not forward `norm_layer` to
`SelfAttention`, which is why the two differ. Getting this wrong is a silent accuracy loss.

**`ZipSplatSession` is single-writer.** It holds MLXArray state and is not thread-safe.
Callers serialise access; the demo funnels everything through one queue.

**The Session takes materialized pixels, never file paths.** `CGImageSourceCreateImageAtIndex`
decodes lazily, so a path opened later can fail once a sandbox or drag grant has lapsed. The
frontend bakes pixels first (`ImageIntake.bake`).

**Preprocessing mirrors PIL exactly, including its quirks** — decode in the image's own colour
space (the reference ignores ICC profiles), and round-and-clamp to uint8 between resize passes
(PIL's `clip8`). Both were bugs found during the port; `PreprocessingTests` guards them at
max 1.00 / mean 0.05 uint8 levels.

## Regenerating fixtures

Fixtures come from the PyTorch reference and are committed small (statistics + slices, with
the final Gaussians in full). Generate them **without xformers installed** — the Swift port
mirrors the pure-PyTorch SwiGLU fallback, so the fallback is the correct oracle.

```bash
.venv/bin/python Scripts/dump_fixtures.py --images <dir> --views 2 \
  --out Tests/MLXZipSplatTests/Fixtures/office2.safetensors
```

## Documentation

This package ships DocC (`Sources/MLXZipSplat/Documentation.docc/MLXZipSplat.md`), and
`.github/workflows/docs.yml` publishes it to GitHub Pages on every push to `main`. **`///`
comments on public symbols become published reference docs**, so they are load-bearing, not
optional hints:

- Every `public` symbol gets a `///` comment.
- Parameter docs use the **internal** name: `func resize(_ image:, to size:)` documents
  `- size:`, not `- to:`.
- Cross-references use signature-sensitive double backticks: `` ``kMeans(_:k:iterations:chunkSize:)`` ``.
- A new top-level public symbol must be filed under a `## Topics` group in the landing page,
  or DocC drops it from the site.

Verify with `BUILD_DOC=1 Scripts/build_docs.sh` — it should emit no `warning: Parameter …`
or `doesn't exist at` lines.

## Known open questions

- **Clustering layer.** The reference clusters on the deepest prepared layer (commit
  `e1b592d`), but the checkpoint's training config says `clustering_layer: 1` (middle).
  `ZipSplatConfiguration.clusteringLayer` exposes both. Only matters below compression 1.0.
- **Packed-encoding ranges in the viewer** (resolved). Satin-Spark's `sh1Max` defaults to
  1.0 while ZipSplat's degree-1 coefficients reach ±6, so `encodeSH1` clamped the strongest
  terms per channel and skewed hue. `SplatBridge.fittedEncoding` now fits `sh1Max` per scene.
  Uses `.quantile(1.0)` rather than `.standard`; the reasoning and the measurements are in
  the doc comment. Requires Satin-Spark 0.1.7+, which fixed an SH decode bug that had been
  the dominant cause of false iridescence. Verify with `ZIPSPLAT_VERIFY_BRIDGE=1`.
- **Camera priors** (`usePriors`) are implemented but not parity-tested; the default path is
  pose-free.

---
> Source: [mnmly/mlx-swift-ZipSplat](https://github.com/mnmly/mlx-swift-ZipSplat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
