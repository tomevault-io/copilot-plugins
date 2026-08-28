## keys-auto-receipts-studio

> Local receipt/document scanner for **Autonomous OS + Autonomous Lamp**.

# keys-automatic-receipt-doc-scanner

Local receipt/document scanner for **Autonomous OS + Autonomous Lamp**.
Lamp camera captures; a GPU box runs the model. Nothing cloud.

## Hardware split (do not blur this)

| Where | Fits | Does not fit |
|---|---|---|
| **Lamp** (ARM64, **6 GB RAM**) | This skill (`SKILL.md` + `scripts/`), HAL snapshot, SQLite | Gemma 4 12B Unified, Qwen3.8-27B, Nemotron 3.5 Lightning |
| **GPU box** (DGX Spark / Omen) | Gemma 4 12B Unified (omni: vision + embed) or Qwen3.8-27B VLM + Embed-1B | — |

Gemma 4 12B Unified is ~12B dense (`hidden_size` **3840**, `Gemma4UnifiedForConditionalGeneration`). Weights alone exceed the Lamp's 6 GB even in NVFP4. Treat the Lamp as eyes/hands; treat the GPU box as the brain.

## Default brain: Gemma 4 12B Unified (omni)

Encoder-free VLM. One OpenAI-compatible server does:

1. **Vision extract** — `POST /v1/chat/completions` with `image_url` data URI (receipt/doc JPEG)
2. **Embed** — `POST /v1/embeddings` against the **same** server (mean-pool / convert-embed). Dim **3840**.

Never mix 3840 (Gemma) and 2048 (Nemotron-3-Embed-1B) in one sqlite-vec index.

## Fallback brains (same skill, env only)

- **Qwen3.8-27B ADay777** VLM at `:8078` (`qwen38-nvfp4`) for extract; Nemotron-3-Embed-1B 2048-d for embed
- **Lightning** is text-only. Never send images to it.

Fleet GB10: `--gpu-memory-utilization` **0.85** hard cap.

## Pipeline

```
voice / phone / drop → Lamp camera or inbox/
        → optional OCR assist
        → Gemma4 (or Qwen) vision JSON extract + category
        → omni embed (or Nemotron-3-Embed-1B)
        → sqlite-vec vendor/SKU/category match
        → review / speak summary
```

## Rules

- No cloud APIs. Backends behind `OCRBackend` / `LLMBackend` / `EmbedBackend`.
- Lightning: `accepts_images=False`. Never attach image parts.
- Python 3.12, typed, pytest. No notebooks. Don't vendor weights.
- Idle-batch inbox 30s. Dedup sha256.

## Extract JSON

```json
{
  "doc_kind": "receipt|invoice|document",
  "category": "groceries|dining|transport|household|health|entertainment|utilities|office|travel|other",
  "vendor": "string|null",
  "date": "YYYY-MM-DD|null",
  "tax": "number|null",
  "total": "number|null",
  "currency": "string|null",
  "line_items": [
    {"description": "string", "qty": "number|null", "unit_price": "number|null",
     "amount": "number|null", "sku": "string|null"}
  ]
}
```

Money stored as integer cents.

## Match (cosine similarity = 1 - sqlite-vec distance)

| | Auto | Review | Unmatched |
|---|---|---|---|
| SKU / line | ≥ 0.88 | 0.72–0.88 | < 0.72 |
| Vendor | ≥ 0.82 | 0.65–0.82 | < 0.65 |

Exact catalog SKU wins first.

## Layout

```
app/          config, schemas, media, camera, extract, embed, db, match,
              pipeline, watcher, cli, ui
backends/     base, openai_compat, gemma, nvidia, ollama, apple, cpu
skills/keys-receipt-scanner/   Autonomous OS built-in skill (Lamp)
inbox/ processing/ processed/ failed/ exports/
```

## Autonomous OS skill

`skills/keys-receipt-scanner/` is a **built-in skill** in Autonomous OS format:

- `SKILL.md` + `skill.json` (`capabilities: ["vision"]`)
- Installs on any body that declares vision (Lamp, Reachy Mini — not Intern)
- Acts via HAL `GET :5001/camera/snapshot` then `python -m app.cli scan --image PATH`
- Does not load 12B weights on the robot

---
> Source: [drowzeys/keys-Auto-Receipts-Studio](https://github.com/drowzeys/keys-Auto-Receipts-Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
