## ocrspine

> Spine 家族的 AI / 人类协作契约。先读家族 `../README.md`,本文件是 ocrspine 的操作指南。

# CLAUDE.md — ocrspine

Spine 家族的 AI / 人类协作契约。先读家族 `../README.md`,本文件是 ocrspine 的操作指南。

## 这是什么

**ocrspine —— 家族里 domain-neutral 的纯 Rust OCR 引擎。** 输入图像(RGB / 灰度像素,
或 PNG/JPEG/TIFF/BMP 编码字节),输出 `Vec<OcrWord>`(每个词带 `text` / `BBox` /
`confidence` / `quad`)。底层用 PP-OCRv5 ONNX 模型(DBNet 检测 + 180° 方向分类 +
CRNN/CTC 识别),经 `tract-onnx` 在 CPU 上跑,无 Python、无 C/C++、无云、无网络,
离线确定性。

## 宪章(不可违背)

- **零领域泄漏。** 这里**只有像素 → 词**。任何 PDF / PPT / 文档 / 页面 概念**一律不准进**。
  判据:这段代码搬到一个完全不同的图像来源里还成立吗?不成立就不属于 ocrspine。
- **独立可测。** `cargo test` 必须独立通过(加载 `models/` 下真实 ONNX 模型识别 fixture),
  不依赖任何兄弟包。
- **离线、纯 Rust。** 默认路径只用 `tract` + `image` + `rayon` + `thiserror`,不碰网络。
  模型文件运行时从磁盘加载(`OCRSPINE_MODELS` 环境变量,否则 `CARGO_MANIFEST_DIR/models`)。
- **机制,不是保证。** 只提供 OCR 机制;具体阈值 / 后处理由消费者在自己的缝里绑。

## 模块地图(按文件夹定位)

```
src/
  lib.rs            #![forbid(unsafe_code)];公共 API 再导出
  error.rs          OcrError(thiserror)+ kind() + Result
  geom.rs           BBox { x0,y0,x1,y1: f64 } + width/height/center
  input.rs          OcrImage:from_rgb / from_gray / from_encoded(内部持 image::RgbImage)
  engine.rs         OcrWord(text,bbox,confidence,quad)+ OcrEngine trait
  paddle/
    mod.rs          PaddleOcr + impl OcrEngine(detect → classify → recognize)
    model.rs        懒加载 shape-bucket tract runnable + 嵌入式字典
    preprocess.rs   to_tensor / resize_exact / crop / crop_rotated / sample_bilinear
    detect.rs       DBNet 检测 → 最小外接旋转矩形
    classify.rs     180° 方向分类
    recognize.rs    CRNN+CTC 贪心解码
models/             4 个数据文件 + PROVENANCE.md(Apache-2.0,PaddlePaddle Authors)
tests/ocr.rs        验收测试:加载 fixture PNG,跑真实模型,断言参考行被识别
```

## 跑(始终从包根)

```bash
OCRSPINE_MODELS=$(pwd)/models cargo build --release
cargo test --release          # 期望 GREEN(加载真实 ONNX 模型)
cargo clippy --release -- -D warnings
```

## 约定

- Rust 2021,rust-version 1.96;`#![forbid(unsafe_code)]`;简体中文 docstring/注释。
- **最小改动**——只改需求要求的部分。代码核心从 pdfspine `pdf-ocr` 移植,
  仅断开 PDF 耦合(Pixmap→OcrImage、Rect→BBox),推理逻辑保持一致。

---
> Source: [VoldemortGin/ocrspine](https://github.com/VoldemortGin/ocrspine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
