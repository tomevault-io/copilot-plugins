## open-tts-eval

> This is the operating manual for AI coding agents (Codex, Claude Code, Cursor, …) working in this

# Agent guide — sample, evaluate, and report TTS with `tts-assess`

This is the operating manual for AI coding agents (Codex, Claude Code, Cursor, …) working in this
repo. It covers the three workflows the toolkit exists for: **sample** audio from TTS providers,
**evaluate** it against the reference text, and build **reports** (single-run and cross-run).

Golden rules:
- **Never commit API keys or audio.** Keys come from env vars / key files; audio (`*.wav`,
  `*.flac`, `*.mp3`, …) and `tmp/`, virtualenvs, and `.measure_cache/` are git-ignored — keep it
  that way.
- The **evaluation core is offline**; only the *sampling* subsystem calls external provider APIs.
- After code changes, run `ruff check .` and `pytest -q` (both must pass).
- `results.jsonl` is the canonical per-sample record; the HTML reports are derived from it.

---

## Setup

```bash
python3 -m venv .venv && . .venv/bin/activate
pip install -e ".[asr,quality]"     # asr = faster-whisper; quality = torch + NISQA (torchmetrics)
```

Install extras by need: core (`pip install -e .`) is enough for **sampling** and **mock-ASR**
tests; `[asr]` adds real Whisper; `[quality]` adds NISQAv2; `[similarity]` adds ECAPA speaker
similarity; `[all]` = everything.

The CLI entry point is `tts-assess` (module `tts_assess.cli`). Commands: `sample`, `run`,
`compare`, `voices`, `init-config`, `preview`.

---

## Providers & API keys

| provider | key env (default) | default model | notes |
|---|---|---|---|
| `inworld` | `INWORLD_API_KEY` | `inworld-tts-1.5-max` | also `inworld-tts-2`, `inworld-tts-1-max`, `inworld-tts-1` |
| `elevenlabs` | `ELEVENLABS_API_KEY` | `eleven_multilingual_v2` | `eleven_v3`, `eleven_turbo_v2_5` |
| `hume` | `HUME_API_KEY` | `octave-2` | `octave-1`; sits behind Cloudflare (a UA is set for you) |

Provide the key with `--api-key-env NAME` (reads that env var) or `--api-key-file PATH`. List a
provider's prebuilt voices:

```bash
tts-assess voices --provider inworld --api-key-env INWORLD_API_KEY
```

---

## Workflow 1 — Sample

Synthesize a text dataset with chosen voices/models. One run directory per model, named
`<provider>-<model>`, each holding `audio/`, `manifest.jsonl`, `sampling_meta.json`.

```bash
tts-assess sample data/inworld.tts.open_benchmak.en.json \
  --provider inworld --api-key-file ~/inworld.key \
  --model inworld-tts-2 --model inworld-tts-1.5-max \
  --voice Ashley --voice Sarah --voice Oliver \
  --language en-US --format WAV --sample-rate 24000 \
  --limit 20 -o out/samples
```

Key flags: `--provider/-p`, `--model` (repeatable, latest first), `--voice/-v` (repeatable) **or**
`--num-voices N` (+ `--shuffle-voices --seed S` for a seeded diverse pick), `--format`
(WAV/LINEAR16/MP3/FLAC/OGG_OPUS/…; **WAV is the safe default** — `soundfile` must decode it),
`--sample-rate`, `--limit N` (first N texts, for smoke runs), `--concurrency`, `--overwrite`,
`--language`, `--speaking-rate`, `--temperature`.

- Datasets: `.txt` (one utterance/line), `.json`/`.jsonl` (objects or bare strings with a `text`
  field; optional `id`, `language`), or `.csv` with a `text` column. Bundled benchmark:
  `data/inworld.tts.open_benchmak.en.json` (100 messy dialogue utterances).
- Audio is **cached by existence** — re-running resumes; dead keys / quota caps are logged per
  sample and skipped (the run continues). Here "cached by existence" also requires a non-empty
  file and a matching full-request fingerprint; older unfingerprinted rows are refreshed once.
- Voices become `speaker_id` in the manifest.

For a reproducible example of a multi-provider run, use `scripts/sample_multi.py` (pinned voices/models; keys
from env or file; presents `inworld-tts-2` as `inworld-tts-2-preview` via a model alias).
Build your own sampling script to support the new providers, or just provide already sampled audios.


## Workflow 2 — Evaluate

Assess a manifest of `{id, text, audio_path}` (the sampler writes these). Runs ASR → normalize →
WER/CER + audio-health + optional NISQA/speaker/prosody → thresholds.

```bash
tts-assess run out/samples/inworld-inworld-tts-2/manifest.jsonl \
  --config eval.yml -o out/samples/inworld-inworld-tts-2
```

- **Evaluate a run into its own dir** (as above): audio, manifest, results, and report end up
  together, and `audio_path` is stored **relative** → the report's `<audio>` players work when the
  folder is opened or moved. Evaluating into a *different* dir keeps absolute paths.
- **Measurement cache** (`.measure_cache/`, keyed by decoded audio + sample rate/channels + text +
  reference-audio content when used + ASR/normalization/metric config): re-runs are instant
  (`--no-cache` / `--cache-dir` to control). Only thresholds and the report are re-applied each run,
  so tuning bands or the report needs **no** recompute.
- Use `asr.backend: mock` (config) for plumbing tests without downloading Whisper.
- Outputs per run: `results.jsonl`, `summary.json`, `results.csv`, `report_data.json`,
  `report.html`.

## Workflow 3 — Reports & Compare

Every `run` already writes a single-run `report.html`. To compare runs:

```bash
tts-assess compare out/samples/inworld-inworld-tts-2 out/samples/inworld-inworld-tts-1.5-max \
  --label "TTS 2" --label "TTS 1.5 Max" --config eval.yml -o out/comparison
```

Positional args are run dirs (or `results.jsonl` paths); `--label` (repeatable, one per run) sets
column names. Runs must have matching text/language cohorts and equivalent per-speaker sampling
profiles, although provider-specific sample and voice IDs may differ. Writes `comparison.html` +
`comparison.json`.

Both reports share one **minimal black-and-white** layout (no CDN/scripts, fully offline):
- **Metric Comparison** — metrics grouped **Accuracy** (WER, insertion/deletion/substitution, CER),
  **NISQAv2** (MOS + noisiness/discontinuity/coloration/loudness), **Subjective** (chars/sec,
  arousal, expressiveness), **Silence** (silence ratio, lead/tail silence). Each cell = mean with a
  95% bootstrap CI beneath; best run per metric green, worst red, `*` when a CI doesn't overlap the
  best; Silence is uncolored.
- **Model Health** — per threshold-backed metric, the share of clips passing its per-sample
  threshold (a `pass if …` rule), graded good ≥99% / warn ≥95% / fail <95% (configurable).
  WER/CER/insertions are omitted here; vowel prolongation follows its configured threshold.
- **Threshold Violations** (single-run report only) — warn/fail counts + worst examples showing
  normalized `expected` vs `heard`, an `<audio>` player if the clip exists, each sample once.

---

## Config

`tts-assess init-config eval.yml` writes an editable default. Shape:

```yaml
asr:
  backend: faster-whisper   # or "mock"
  model: base.en            # small, medium, large-v3, …
  language: en
normalization:
  backend: english-basic    # or "nemo", or "plugin" (module:function)
  expand_numbers: true
optional_metrics:
  nisqa_v2: false           # needs [quality]
  speaker_similarity: false # needs [similarity] + reference_audio_path
  vowel_prolongation: true
  voice_lens: true          # expressiveness/arousal proxies
thresholds:                 # per-sample pass/warn/fail bands (see below)
  wer: {warn: 0.05, fail: 0.10}
  vowel_prolongation_score: {fail: 0.8}
  nisqa_mos: {warn_below: 3.3, fail_below: 2.8}
reporting:
  confidence_level: 0.95
  bootstrap_resamples: 2000
  health_good_rate: 0.99    # Model Health bands
  health_warn_rate: 0.95
  embed_audio: true
  max_violation_examples: 5
  max_worst_samples: 25
  question: "Are these TTS audios acceptable?"
```

Unknown config fields are rejected instead of being silently ignored.

`BoundThreshold` fields: `warn`/`fail` (upper-bound: value ≥ band → warn/fail), `warn_below`/
`fail_below` (lower-bound, e.g. similarity/MOS), `fail_if_true` (booleans like `tail_click_detected`).

## Metric glossary (direction = better)

- **Accuracy (lower better):** `wer`, `cer`, `insertion_rate`, `deletion_rate`, `substitution_rate`.
- **NISQAv2 (higher better, 1–5):** `nisqa_mos` + `nisqa_noisiness/discontinuity/coloration/loudness`.
- **Audio health (lower better):** `clipping_ratio`, `silence_ratio`, `leading/trailing_silence_sec`,
  `tail_click_score` (+ boolean `tail_click_detected` at score ≥ 4.0).
- **Pacing/subjective (neutral):** `duration_sec`, `chars_per_second`, `arousal_proxy`;
  `expressiveness_proxy`; `vowel_prolongation_score` (seconds of longest vowel run; the default
  threshold fails values at or above 0.8s).
- **Speaker (higher better):** `speaker_similarity` (needs `reference_audio_path`).

## Repo map

```
src/tts_assess/
  cli.py                       # Typer CLI: run / sample / compare / voices / init-config / preview
  config.py                    # pydantic config + default thresholds
  pipeline.py                  # run_assessment: ASR→normalize→metrics→thresholds→report; measure cache
  asr/backends.py              # faster-whisper + mock (model cached via lru_cache)
  audio/features.py            # decode once; rms/peak/clipping/silence/tail-click
  normalization/               # english-basic (+ nemo, plugin)
  metrics/                     # text (jiwer WER/CER + hallucination heuristics), audio, optional (NISQA/ECAPA/prosody)
  reporting/
    aggregate.py               # summarize: means, bootstrap CI, violations, by-voice
    stats.py                   # bootstrap CI + CI-overlap significance
    metrics_meta.py            # metric direction + display names
    thresholds.py              # classify_row / evaluate_thresholds
    compare.py                 # build_comparison + build_run_report (+ Model Health, violations)
    comparison_html.py         # the black-and-white renderer (both reports)
  sampling/
    datasets.py                # .txt/.json/.jsonl/.csv loaders
    sampler.py                 # run_sampling: texts × voices × models → manifests (+ model aliases)
    providers/                 # base.py + inworld.py / elevenlabs.py / hume.py (+ _http.py)
scripts/sample_multi.py        # reproducible multi-provider sampling
scripts/eval_multi.py          # evaluate all runs in place + build compare_all
data/…open_benchmak.en.json    # bundled benchmark dataset
out/eval_multi/                # example evaluated runs + compare_all report (audio git-ignored)
```

## Gotchas

- **WER/CER also capture ASR errors, not just TTS errors.** Whisper mishears accents, proper names,
  and expressive delivery, inflating WER even when synthesis is fine. Cross-check NISQA (needs no
  transcript), read the `expected` vs `heard` columns, and confirm by listening. Bigger/multilingual
  Whisper reduces it.
- Committed reports reference audio by **relative** path; audio isn't shipped, so per-run `<audio>`
  players are inert in a fresh checkout. The **comparison report has no audio** and always works.
- `tail_click_score` is an unbounded ratio (heavy-tailed): its mean is outlier-dominated — prefer
  the boolean `tail_click_detected` or a clipped/percentile view.
- Competitor keys may be dead or quota-limited; the sampler records per-sample errors and continues.
  Use `--limit` for cheap smoke runs.

## Extending — add a provider

1. Subclass `TTSProvider` in `src/tts_assess/sampling/providers/<name>.py`, implementing
   `synthesize(SynthesisRequest) -> SynthesisResult` and `list_voices() -> list[Voice]` (use the
   injectable `transport` and the `providers/_http` helpers; set `forced_encoding` if the API only
   emits one decodable format).
2. Register it in `providers/__init__.py` (`_PROVIDERS`).
3. Add tests with a fake transport (see `tests/test_sampling_competitors.py`).

## Dev conventions

- `ruff check .` and `pytest -q` must pass; line length 100.
- Prefer the measurement cache over re-synthesizing/re-transcribing.
- Match existing style; keep audio and secrets out of git.

---
> Source: [inworld-ai/open-tts-eval](https://github.com/inworld-ai/open-tts-eval) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
