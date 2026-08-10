## mica-ppocr

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`mica-ppocr` is a **Java 17** port of PP-OCRv6 text detection + recognition, running pure
[ONNX Runtime](https://onnxruntime.ai/) inference with **zero PaddlePaddle dependency**. It is a
line-for-line port of the single-file Python reference `ppocrv6_onnx.py` from
[`AIwork4me/ppocrv6_onnx`](https://github.com/AIwork4me/ppocrv6_onnx), reproducing the exact
pre/post-processing (DB post-process, CTC greedy decode, pyclipper-equivalent polygon unclip).

**Bit-exactness with the Python reference is the primary correctness goal.** Defaults favor CPU
single-threaded execution (`intraOp=interOp=1`) to guarantee deterministic, cross-platform output.
When changing numeric logic, verify against the Python implementation rather than just "looks reasonable."

## Module structure

```
mica-ppocr/                     ← parent pom (packaging=pom)
├── mica-ppocr-core/            ← 核心引擎，零 Spring 依赖
│   └── src/main/java/net/dreamlu/mica/ai/ppocr/
│       ├── engine/PPOcrV6Engine.java
│       ├── config/PPOcrV6Config.java, PPOcrV6Result.java
│       ├── preprocessor/DetectionPreprocessor.java, RecognitionPreprocessor.java
│       ├── postprocessor/DbPostProcessor.java, CtcLabelDecoder.java
│       ├── utils/{BoxUtil, CropUtil, Offset, NdArrayUtils, OrtProviders}.java
│       └── cli/Main.java
└── mica-ppocr-spring-boot-starter/  ← Spring Boot 自动配置
    └── src/main/java/net/dreamlu/mica/ai/ppocr/autoconfigure/
        ├── PPOCRAutoConfiguration.java
        ├── PPOCRProperties.java
        └── OpenCVNativeLoader.java
```

## Commands

```bash
mvn -DskipTests package      # 全量构建
mvn package                  # 构建 + 运行测试
mvn test                     # 运行所有测试
mvn test -Dtest=NpUtilTest   # 运行单个测试
```

Native libs (OpenCV, ONNX Runtime) are pulled by Maven for Windows/Linux/macOS — no manual install.

## Model setup (required before running)

Models are **not** in the repo. Place under `models/ppocr-v6/{tier}/`，三档可选：

```
models/ppocr-v6/
├── tiny/        # 轻量，速度快，精度一般 (det 1.7MB + rec 4.3MB)
│   ├── det.onnx
│   ├── rec.onnx
│   └── dict.txt             # ~2855 字符
├── small/       # 平衡档 (det 9.4MB + rec 20.2MB)
│   ├── det.onnx
│   ├── rec.onnx
│   └── dict.txt             # ~2855 字符
└── medium/      # 高精度档 (det 59.2MB + rec 73.0MB)
    ├── det.onnx
    ├── rec.onnx
    └── dict.txt             # ~7180 字符
```

CLI 默认使用 `--tier tiny`；可用 `--tier small|medium` 切换，或用 `--det-model`/`--rec-model`/`--dict` 显式指定覆盖。

模型来源：`E:\codes\ai\mica-ai\model-tools\ppocr\model\out-by-spec`

## Architecture

The pipeline flows: **detect → sort boxes → crop → recognize**.

### mica-ppocr-core

- **`engine/PPOcrV6Engine`** — the orchestrator and only public entry point. Owns the two
  `OrtSession`s (det + rec), is `Closeable` (use try-with-resources), and exposes `detect()`,
  `recognize()`, and the full `run(Mat)`. Accepts a `PPOcrV6Config` (Lombok `@Builder`).
- **`config/PPOcrV6Config`** — `@Getter @Builder` config for all tunables.
- **`config/PPOcrV6Result`** — Java 17 `record` (text, score, box) returned to callers.
- **`preprocessor/DetectionPreprocessor`** — resize to limit-side constraints, normalize, HWC→NCHW.
- **`postprocessor/DbPostProcessor`** — DB binary-map → contours → boxes.
- **`preprocessor/RecognitionPreprocessor`** — batches crops, resizes, pads to common width.
- **`postprocessor/CtcLabelDecoder`** — loads char dict, CTC greedy decode → text + score.
- **`utils/`** — the numpy/cv2 equivalents:
  - `NdArrayUtils` — argmax/max/stack/pad/clip over float arrays.
  - `BoxUtil` — `sortQuadBoxes` (reading order), minAreaRect/boxPoints.
  - `Offset` — pyclipper `PyclipperOffset` equivalent via JTS `BufferOp` + `JOIN_ROUND`.
  - `CropUtil` — perspective-warp crop; returns `null` for invalid crops;
    `run()` filters these `null`s out — preserve this null-skip contract.
  - `OrtProviders` — picks the ORT execution provider; `forceCpu` (negation of
    `preferAccelerator`) is the default.
- **`cli/Main`** — arg parsing, validation, image auto-discovery, OpenCV visualization.

### mica-ppocr-spring-boot-starter

- **`PPOCRAutoConfiguration`** — auto-wires `PPOcrV6Engine` bean when `mica.ai.ppocr.enabled=true`.
- **`PPOCRProperties`** — `@ConfigurationProperties("mica.ai.ppocr")` binding.
- **`OpenCVNativeLoader`** — `@AutoConfigureBefore` the main config, eagerly loads OpenCV native libs.

### Porting conventions

Python↔Java mapping: `numpy`→`utils.NdArrayUtils`, `pyclipper`→`utils.Offset` (JTS),
`cv2.minAreaRect`→`Imgproc.minAreaRect`, `np.rot90`→`Core.ROTATE_90_COUNTERCLOCKWISE`,
`@dataclass(frozen=True)`→Java `record`.

### Known divergences from Python

- `pyclipper` uses scaled-integer math; JTS `BufferOp` uses doubles → unclip differs by <1px.
- No CoreML provider in ONNX Runtime Java API.
- For CUDA: swap `onnxruntime`→`onnxruntime_gpu` in pom.xml and set `preferAccelerator(true)`.

---
> Source: [lets-mica/mica-ppocr](https://github.com/lets-mica/mica-ppocr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
