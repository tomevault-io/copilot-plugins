## vision-serve

> > Read this BEFORE touching any code (AI coding agents AND humans).

# CLAUDE.md — VisionServe development rules

> Read this BEFORE touching any code (AI coding agents AND humans).
> Goal: avoid "vibe-coding the wrong thing" — wrong license, wrong architecture, wrong scope.

## What VisionServe is

VisionServe is a **fully free, open-source community project** released under
**Apache-2.0 (permissive)**. There is **no paid tier and no closed-source component** —
every feature lives in this repo and is free for anyone to use, including in commercial,
edge, and closed-source products. Think "Ollama for Computer Vision": one lean Go binary
that serves CV models locally, with no account, no API key, no cloud, no telemetry.

In scope, all free:

- **Detection** — RF-DETR.
- **Segmentation** — MobileSAM (box/point prompted).
- **Open-vocabulary detection** — GroundingDINO (text prompted) — a free feature, Apache-2.0.
- **Grounded-SAM** — GroundingDINO boxes → MobileSAM masks (text-prompted segmentation).

## Top-priority rules

1. **License discipline — accept ONLY permissively-licensed models (Apache-2.0 / MIT / BSD).
   STRICTLY forbid AGPL models (Ultralytics YOLO, FastSAM, YOLO-World).**
   This is *because* the project is free and community-owned, not in spite of it. AGPL is a
   strong copyleft: pulling in a single AGPL model would virally relicense the entire project
   — and every downstream deployer — under AGPL, destroying the permissive freedom that lets
   anyone (including commercial and closed products) build on VisionServe. "Free / community"
   does NOT relax this rule; it is precisely *why* the rule exists.
   - "It's on HuggingFace" says **nothing** about the license. Check each model's actual
     license; it must be Apache-2.0 / MIT / BSD.
   - Every model **MUST** declare a `license` field in its manifest, and the registry parser
     **MUST reject** any license not in the allowlist.
2. **Do NOT write your own inference engine.** All inference goes through **ONNX Runtime**.
3. **Do NOT pull Python into the runtime.** This is a lean **Go binary** for edge devices.
4. **Avoid cgo/OpenCV unless strictly necessary.** If you must, document why and mark it
   `// CGO`. Prefer pure-Go imaging (`disintegration/imaging`, the standard `image` package).
5. **Extension points are for community contributions / optional integrations** — not for a
   paid tier. Keep them as clean interfaces with sane no-op defaults (e.g. the data-collection
   hook in `internal/extension/`), so people can plug in their own integrations without
   touching core. Local-first means the default behavior collects and sends nothing.

## Architecture rules

- **Plain model = new package in `internal/models/<name>/` implementing the `Model`
  interface + calling `models.Register()` in `init()`.** NEVER touch core
  (`server` / `engine` / `lifecycle`) to add a model. This is the contribution path.
- **Prompted / multi-session models implement the `PipelineModel` interface instead**
  (`Roles()` + `Infer(img, prompt, Runner)`). Use it when a model needs a `Prompt`
  (box / point / text) and/or chains multiple ONNX sessions:
  - MobileSAM — encoder + decoder, box/point prompt.
  - GroundingDINO — single session, text prompt.
  - Grounded-SAM — GroundingDINO + MobileSAM chained.
  Such a model loads several ONNX sessions via the manifest `files:` map (role → path) and
  drives the chaining itself through the `Runner`. **`lifecycle.Manager` still owns and keeps
  alive every session (VRAM-safe)** — the model only orchestrates calls by role, it never
  creates or frees sessions.
- **`Detection.BBox` is ALWAYS in ORIGINAL image coordinates** (mapped back via
  `PreprocessMeta`), as `[x, y, w, h]`. This is the single most common bug — check it
  carefully.
- **Unified `Result` schema across ALL tasks** (`Detections` + `Masks`; masks encoded as
  **column-major RLE**). Do NOT invent a per-model schema.
- ONNX sessions are heavy resources: they **must** go through `lifecycle.Manager`, never
  created directly in a handler, and must be released correctly (avoid VRAM leaks).
- `Infer` (calling an ONNX session) is **NOT** part of the plain `Model` interface — `engine`
  + `lifecycle` own it. A plain `Model` focuses only on pre/postprocess. (A `PipelineModel`
  drives inference itself, but still only through the `Runner` gateway.)

## Go code rules

- Follow the standard Go project layout (`cmd/`, `internal/`, `pkg/`).
- Every function returns a clear `error`; **no panics** in normal code paths.
- Concurrency: the server handles many requests in parallel; access to a model session must
  be **thread-safe** (mutex or pool). This is an easy place to get wrong.
- Write **tests for the pre/postprocess of EVERY model** (the most error-prone part).

## When unsure

- **Do NOT guess a model's output format.** Verify the real tensor shapes before writing
  postprocess. If you are not sure, write a stub + `TODO` + record the open question —
  **do not fabricate.**
- **RF-DETR postprocess is DIFFERENT from YOLO.** RF-DETR is **NMS-free** (output is a set of
  object queries). Never apply YOLO-style NMS to RF-DETR.

## Scope (don't over-engineer)

All four capabilities are free and in scope: **detection (RF-DETR)**, **segmentation
(MobileSAM)**, **open-vocab detection (GroundingDINO)**, and **Grounded-SAM**. Keep the
working spirit: get a clean end-to-end path working per capability, verify real tensor
shapes before writing decode logic, and don't build speculative machinery (remote registry,
premature optimization) until it's actually needed.

## Build environment

- Requires **Go >= 1.22**.
- Needs an **ONNX Runtime** shared library at runtime (`libonnxruntime.so`) for the
  `yalue/onnxruntime_go` binding — point to it via the **`ORT_DYLIB_PATH`** environment
  variable. On edge (Jetson) use an ORT build with the TensorRT/CUDA execution providers so
  the manifest's `tensorrt → cuda → cpu` fallback chain works.
- **Supported execution providers** (EP allowlist in `internal/engine/provider.go`):
  `tensorrt`, `cuda` (NVIDIA), `coreml` (Apple Silicon), `directml` (Windows GPU),
  `openvino` (Intel), and `cpu` (always appended last). Each EP only works if the ORT build
  on the host actually ships that provider; a missing EP silently falls back to the next.
  Only EPs the `yalue/onnxruntime_go` binding exposes can be wired (no ROCm binding yet).
- Default server port is **11435** (avoids clashing with Ollama's 11434).

---
> Source: [mtbui2010/vision_serve](https://github.com/mtbui2010/vision_serve) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
