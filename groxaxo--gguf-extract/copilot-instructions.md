## gguf-extract

> >


# GGUF Extract

## When to use

- A HuggingFace repo contains `.gguf` files
- You need the full tensor list (name, shape, precision) per file
- You need to generate tensor-type override files for llama.cpp
- You need metadata/chat template from a GGUF file

## Extraction modes

### 1. API path (preferred)

```bash
python3 extract_gguf.py <repo-id-or-url> [options]
```

Supports both CDN-backed and Xet-backed repos. Uses HTTP Range requests first,
falls back to HfFileSystem for Xet/gated repos.

### 2. Browser path

For dynamic pages where the API path is insufficient. Requires Chrome MCP
tools and the local sink server.

1. Start sink: `python3 sink_server.py`
2. Navigate to HF blob page in Chrome
3. Run `extract.js` via `chrome_javascript`

## Prerequisites

| Requirement | Notes |
|---|---|
| Python 3.8+ | Runtime |
| requests | HTTP Range requests |
| huggingface_hub | Xet/LFS fallback (optional) |
| Chrome MCP | Browser path only |
| HF token | Required for gated/Xet repos |

## CLI options

```
repo              HuggingFace repo ID or URL
-a, --all         Fetch all GGUF files
-f, --file FILES  Target specific .gguf file (repeatable)
-o, --output DIR  Save JSON results to directory
--token TOKEN     HF token for gated/Xet repos
--max-bytes N     Header bytes to fetch per file (default 16MB)
```

## Output

Per-file JSON with `metadata` dict and `tensors` array. Use
`build_tensor_type_files.py` to convert to llama.cpp `.tensor-types.txt` files.

## Troubleshooting

| Symptom | Fix |
|---|---|
| API fails on Xet repo | Provide `--token` or set `HF_TOKEN` env var |
| `Tensors` text never appears (browser) | Page 5xx — retry after a few minutes |
| Sink CORS error | Verify `sink_server.py` is running on port 8123 |
| 429 / CAPTCHA from HF | Increase delay between requests |

---
> Source: [groxaxo/gguf-extract](https://github.com/groxaxo/gguf-extract) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
