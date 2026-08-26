## gpu-ocr-processor

> This file provides guidance for Claude Code when working with this repository.

# CLAUDE.md

This file provides guidance for Claude Code when working with this repository.

## Project Overview

GPU-accelerated OCR pipeline for German documents with LLM-based error correction.

## Architecture

- `ocr_processor.py` - Main OCR script using PaddleOCR
- `ocr_postprocess.py` - LLM post-processor using Hugging Face Transformers

## Key Dependencies

- **PaddleOCR** - OCR engine with German language support
- **PaddlePaddle-GPU** - GPU backend (requires CUDA)
- **Transformers + Accelerate** - LLM inference for post-processing

## Development Commands

```bash
# Run OCR processing
python ocr_processor.py

# Run LLM post-processing
python ocr_postprocess.py

# Install dependencies (use PaddlePaddle mirror for GPU package)
pip install -r requirements.txt -i https://www.paddlepaddle.org.cn/packages/stable/cu129/
```

## Folder Structure

- `input/` - Place source images here
- `output/` - OCR text results
- `output_fixed/` - LLM-corrected text results
- `models/ocr/` - Cached PaddleOCR models
- `models/llm/` - Cached LLM models

## Configuration Notes

- OCR language is set to German (`lang="german"`)
- LLM model default: `Qwen/Qwen2.5-7B-Instruct` (requires ~14GB VRAM)
- All models auto-download on first run

## Code Style

- Use pathlib for file operations
- Keep scripts self-contained with configuration at the top
- Use type hints for function parameters

---
> Source: [masroorahmd/GPU-OCR-Processor](https://github.com/masroorahmd/GPU-OCR-Processor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
