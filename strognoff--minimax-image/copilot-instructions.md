## minimax-image

> Generate images from text prompts using MiniMax's Image-01 model.

# MiniMax Text-to-Image Skill

Generate images from text prompts using MiniMax's Image-01 model.

## Usage

```bash
python3 /home/cechinel/.openclaw/workspace/minimax-image/minimax_image.py "your prompt" --output /path/to/output.png --aspect_ratio 16:9
```

## Arguments

- `prompt` (required): Text description of the image to generate
- `--output`, `-o`: Output filename (default: generated_image.png)
- `--aspect_ratio`, `-a`: Aspect ratio - `16:9`, `9:16`, `1:1`, `4:3`, `3:4` (default: 16:9)
- `--model`, `-m`: Model name (default: image-01)
- `--response_format`, `-f`: `base64` or `url` (default: base64)
- `--api_key`, `-k`: MiniMax API key (optional, uses `MINIMAX_API_KEY` env if not provided)

## Environment

Requires `MINIMAX_API_KEY` set in credentials. Already configured at:
`/home/cechinel/.openclaw/credentials/minimax_credentials`

## Examples

```bash
# Generate a logo
python3 minimax_image.py "minimalist coffee shop logo" --output logo.png --aspect_ratio 1:1

# Generate a banner
python3 minimax_image.py "tech startup banner, modern design" --output banner.png --aspect_ratio 16:9

# Generate a story illustration
python3 minimax_image.py "epic mountain landscape at sunset" --output landscape.png --aspect_ratio 16:9
```

## Notes

- Output is a JPEG image
- API can be slow (up to 60 seconds)
- Aspect ratio 16:9 works well for banners
- Aspect ratio 1:1 works well for logos and social posts

---
> Source: [strognoff/minimax-image](https://github.com/strognoff/minimax-image) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-23 -->
