## zimage-skill

> Generate images using ModelScope Z-Image-Turbo API. Use when user asks to generate, create, or make images, pictures, or illustrations.


# Z-Image Generation

Generate images using ModelScope's Tongyi-MAI/Z-Image-Turbo model.

## Requirements

```bash
pip install requests Pillow
```

## Usage

```bash
python3 ~/.claude/skills/zimage-skill/generate.py "prompt" [output_path]
```

## Environment

Optional: Set `MODELSCOPE_API_KEY` to use your own API key.

## Examples

```bash
python3 ~/.claude/skills/zimage-skill/generate.py "A golden cat" cat.jpg
python3 ~/.claude/skills/zimage-skill/generate.py "Sunset over mountains"
```

## Notes

- Default output: `result_image.jpg`
- Timeout: 120 seconds max
- Supports Chinese and English prompts

---
> Source: [rustectersehj226/zimage-skill](https://github.com/rustectersehj226/zimage-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
