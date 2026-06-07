## localscribe

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`localscribe` is a **fully local, offline** CLI that takes an audio (or video) file and produces a
Markdown transcript with diarization ("who spoke when") and recognition of enrolled voices
("that turn was Me"). Targets macOS Apple Silicon; ASR runs on the Apple GPU via MLX,
diarization/embeddings run on CPU via sherpa-onnx. No cloud, no NVIDIA GPU, no Hugging Face token.

## Commands

Everything is driven through the Makefile (`make help` lists targets). The CLI itself is
`uv run --no-sync python -m localscribe <command>`.

```bash
make setup                                              # uv sync + download diarization/voiceprint models
make enroll NAME="Me" FILES="enroll/a.wav enroll/b.wav" # store a voiceprint
make list                                               # list enrolled voiceprints
make transcribe FILE=audio/x.m4a [SPEAKERS=N] [LANG=fr] # main entry point
make record OUT=audio/x.wav [DEVICE=:3]                 # capture mic to 16 kHz mono WAV (press q, not Ctrl+C)
make devices                                            # list avfoundation audio inputs
```

There is **no test suite, linter, or formatter** configured. The only way to exercise changes is
to run `make transcribe` on a sample file.

`LANG` is only honored when passed explicitly on the make command line (`make transcribe ... LANG=fr`);
a `LANG` inherited from the shell environment (the locale) is deliberately ignored so it never
accidentally switches the ASR backend (see the `ifeq ($(origin LANG),command line)` guard in the Makefile).

## Architecture

The pipeline (`pipeline.py::process_file`) is a linear 5-stage flow; each stage lives in its own module:

1. **`audio.py`** — decode any ffmpeg-readable input to mono 16 kHz float32 via a subprocess (no decode libs).
2. **`speakers.py`** — diarize into `DiarSegment(start, end, speaker_id)` using sherpa-onnx (pyannote
   segmentation + TitaNet embeddings + fast clustering), then identify: concatenate each anonymous
   speaker's segments, embed, and cosine-match against enrolled voiceprints. Returns `{speaker_id: name}`.
3. **`enroll.py`** — voiceprint storage. Enrolled embeddings live in `personas/voiceprints.npz`
   (local, gitignored); enrollment averages embeddings across the provided files.
4. **ASR** — two interchangeable backends producing the same `AsrSentence(text, start, end)`:
   - **`asr.py`** (Parakeet, default): multilingual auto-detection. Always transcribes in overlapping
     30s/15s chunks — this is **load-bearing**, not just for memory: single-pass TDT greedy decoding can
     collapse a whole transcript to a few words on some inputs; chunking forces multiple passes that merge correctly.
   - **`whisper_asr.py`** (Whisper large-v3): used only when `--language` is set, because Whisper takes a
     forced language token and avoids mid-sentence code-switching.
   The backend is chosen in `pipeline.py` purely by whether `language` is truthy.
5. **`report.py`** — merge ASR sentences with diar segments by **max time overlap**, group consecutive
   same-speaker turns, render Markdown to `output/<stem>.md`.

`__main__.py` is the argparse CLI (`enroll` / `list` / `transcribe`). The shared data contract between
stages is the three dataclasses: `DiarSegment`, `AsrSentence`, `Turn`.

## Models

Two kinds, fetched differently:
- **Diarization + voiceprint** ONNX models (~45 MB) → `models/` via `models/download_models.sh` (run by `make models`).
  Paths are hardcoded in `speakers.py` (`SEGMENTATION_MODEL`, `EMBEDDING_MODEL`); both are checked for
  existence with a "run download_models.sh" hint before use.
- **ASR** models (Parakeet ~2.3 GB, Whisper ~3 GB) → auto-downloaded by MLX on first transcription into the
  shared Hugging Face cache (`~/.cache/huggingface/hub/`, overridable via `HF_HOME`). Not in `models/`.

## Packaging gotcha

PyPI sherpa-onnx wheels are broken on macOS arm64 (missing onnxruntime dylib). `pyproject.toml` pins
`sherpa-onnx` and `sherpa-onnx-core` to the official k2-fsa wheel index. Keep that index config when
touching dependencies, or imports break at runtime.

---
> Source: [omansour/localscribe](https://github.com/omansour/localscribe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-07 -->
