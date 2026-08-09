## jaitts-easy

> The upstream repo is **benchmark/evaluation code only** — it measures TTS output, it

# JaiTTS — working notes

## What this repo actually is

The upstream repo is **benchmark/evaluation code only** — it measures TTS output, it
does not generate speech. README says so in one line and never mentions where the
speech model lives.

The speech model is a **separate HuggingFace repo**: `JTS-AI/JaiTTS-F5TTS`
(`model.pt` + `vocab.txt`). It is a Thai-finetuned **F5-TTS** — checkpoint inspection
gives `depth=22`, `text_dim=512`, i.e. the `F5TTS_Base` architecture, not `v1`.

Run it through the vendored `flowtts/` pipeline (copied from `biodatlab/thonburian-tts`,
MIT, `runtime/` stripped). Calling `f5_tts.api.F5TTS` directly with the same checkpoint
and architecture *works* but produces markedly quieter, mushier audio — measured RMS
0.040 vs flowtts' 0.116 on the same prompt, and ~0.009 when driven from Gradio. The
difference is in flowtts' pre/post-processing, not the weights. Do not "simplify" by
dropping flowtts without re-measuring output RMS and CER.

The HF model card's usage snippet is wrong regardless: it calls `pipeline.generate()`,
which does not exist. The pipeline object is called directly — `pipe(text=..., ref_voice=...,
ref_text=..., output_file=...)`.

## Layout added on top of upstream

| Path | What |
|---|---|
| `jaitts_synth.py` | CLI synthesis |
| `jaitts_webapp.py` | Local Gradio UI on 127.0.0.1:7860 |
| `flowtts/` | vendored inference pipeline (third-party, MIT) |
| `.venv` | eval env — Python 3.11 |
| `.venv-tts` | synthesis env — Python 3.11 |
| `data/jaitts_testset/th/` | benchmark set (1000 wavs, 802 prompt-wavs, `meta.lst`) |
| `checkpoints/wavlm_large_finetune.pth` | WavLM for SIM scoring |
| `outputs/` | generated audio (gitignored) |

**Two venvs are mandatory.** `f5-tts` pulls `transformers 5.x`; the eval scripts pin
`4.57.3`. One env cannot serve both.

Only `.gitignore` differs from upstream in tracked files — everything else added here
is ignored. Remote is the official `JTS-AI-Team/JaiTTS`, so think before pushing.

## Synthesis

```bash
.venv-tts/bin/python jaitts_synth.py "<text>" <ref.wav> "<ref transcript>" outputs/out.wav
.venv-tts/bin/python jaitts_webapp.py          # web UI
```

Voice cloning needs a reference clip **plus its exact transcript** — a wrong transcript
degrades the output badly. The benchmark's 802 prompt-wavs come with transcripts in
`meta.lst` (column 2 = text, column 3 = wav path).

~20s per sentence on MPS.

## Evaluation

```bash
source .venv/bin/activate
bash cal_wer.sh data/jaitts_testset/th/meta.lst <synth_audio_dir> th
bash cal_sim.sh data/jaitts_testset/th/meta.lst <synth_audio_dir> checkpoints/wavlm_large_finetune.pth th
```

Scripts call bare `python3`, so the venv must be activated, not just referenced.
Audio must be at `{synth_audio_dir}/{filename}.wav` matching `meta.lst` column 1.
Relative `prompt_wav` paths resolve against the directory holding `meta.lst`.

Requirements are incomplete: `accelerate` (needed by `run_wer.py`'s
`low_cpu_mem_usage=True`) and `s3prl` (WavLM upstream via `torch.hub`) are missing from
`requirements.txt`.

Both scripts pick MPS automatically when CUDA is absent. Whisper runs ~10s per clip on
MPS, so a full 1000-file run takes ~3 hours.

## Verified baselines

Feeding the benchmark's own ground-truth audio back through `cal_wer.sh` gives ~1.96%
CER, matching the paper's 1.98% human reference. Use that as the sanity check that the
eval stack is wired correctly.

---
> Source: [aiunlocked1412/JaiTTS-Easy](https://github.com/aiunlocked1412/JaiTTS-Easy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
