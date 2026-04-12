## guru

> [datasets/llama_optimized.jsonl](mdc:datasets/llama_optimized.jsonl) contains 521 spiritual wisdom entries optimized for instruction tuning.

# Dataset Structure and Content Guide

## Primary Dataset
[datasets/llama_optimized.jsonl](mdc:datasets/llama_optimized.jsonl) contains 521 spiritual wisdom entries optimized for instruction tuning.

## Format Specification
- **File Type**: JSONL (JSON Lines)
- **Structure**: One JSON object per line
- **Schema**: `{"instruction": str, "input": str, "output": str}`
- **Input Field**: Always empty string `""` for this dataset
- **Content**: High-quality spiritual wisdom teachings

## Sample Entry
```json
{"instruction": "What happens in the world may not happen the way you want it, but at least what happens within you must happen your way, isn't it?", "input": "", "output": "So if what happens within you is not happening your way, fundamentally we have not figured what is the nature of our life."}
```

## Content Characteristics
- **Domain**: Spiritual wisdom and philosophical teachings
- **Style**: Question-answer format with deep insights
- **Quality**: Curated for coherence and depth
- **Size**: Perfect for fine-tuning (not too small, not overwhelming)

## Usage in Training
- Direct compatibility with Unsloth/Transformers SFTTrainer
- Requires prompt formatting for Llama 3.2 chat template
- No preprocessing needed beyond format conversion
- Ready for instruction-following fine-tuning

## Data Preparation Script
[generate_finetuning_dataset.py](mdc:generate_finetuning_dataset.py) was used to create this optimized dataset from source materials.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/r-huijts)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/r-huijts)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
