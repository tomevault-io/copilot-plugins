## voice-clone-lab

> Guidance for agents working in this repo.

# AGENTS.md

Guidance for agents working in this repo.

## What this is

Voice Clone Lab: an installable pipeline (`src/voice_clone_lab/`, console script
`vcl`) for fine-tuning `Qwen3-TTS-12Hz-1.7B-Base` on a user's own voice and
generating speech. CLI (`cli.py`) + Gradio UI (`ui.py`) over shared modules.

## Layout

- `src/voice_clone_lab/` — the package. `config.py` (dataclass config + per-voice
  path derivation), `utils.py` (IO + overwrite safety), `audio.py`,
  `transcribe.py`, `dataset.py`, `train.py`, `generate.py`, `sources.py`
  (yt-dlp URL downloads), `system.py`, `cli.py`, `ui.py`.
- `config/default.yaml` — the ONLY config source. Every key is used; if you add a
  setting, add it here AND to the dataclasses in `config.py`.
- `patches/sft_12hz.patch` — local patch to the vendored training script. See
  "Vendor rule" below.
- `third_party/Qwen3-TTS/` — vendored upstream clone (gitignored, recreated by
  `vcl setup`).
- `data/`, `outputs/`, `models/`, `attic/` — gitignored runtime dirs. `data/` and
  `outputs/` contain biometric voice data: never commit, never upload.
- `tests/` — pytest, CPU-only (synthetic audio, tmp_path, monkeypatching).

## Conventions (keep them when editing)

- **Overwrite safety**: every step that writes files must refuse to overwrite
  unless `force=True`, via `utils.refuse_overwrite` / `refuse_any_existing`.
  No unconditional `ffmpeg -y`, no unguarded `rm -rf`/`shutil.rmtree`. The one
  destructive path (train `--force`) must stay guarded to
  `outputs/checkpoints/` only.
- **Relative paths in artifacts**: files under `data/` that reference audio
  (chunk metadata, transcripts, dataset rows) store project-relative paths via
  `config.rel_to_root`; readers resolve with `utils.resolve_artifact_path`
  (accepts legacy absolute paths too).
- **Config idiom**: values come from the `Config` dataclasses; function
  parameters override (`None` = use config). No `config.get(...).get(...)`
  chains, no second YAML reader.
- **Speaker-derived paths**: never hardcode a voice name or a data path; use
  `cfg.paths_for(speaker)`.
- **Lazy heavy imports**: `torch`, `qwen_tts`, `soundfile`, ASR backends,
  `webrtcvad`, `noisereduce` import inside functions so `vcl` stays fast and
  CPU-only tests pass.
- **User-facing errors**: `raise SystemExit(clear message)`, not tracebacks.
- Ported behavior is faithful to the original scripts — don't change filter
  chains, thresholds, or scoring formulas without a reason and a README note.

## Vendor rule

`third_party/Qwen3-TTS` is pinned to commit `022e286b…` (`qwen.repo_commit` in
`config/default.yaml`) and modified ONLY by `patches/sft_12hz.patch`. If you
change the vendored tree, regenerate the patch (`git -C third_party/Qwen3-TTS
diff > patches/sft_12hz.patch`) and keep `train.setup_vendor` applying it
idempotently. Never commit the vendored tree (it is gitignored).

## Verify before done

```bash
python -m pytest tests/ -q          # must stay green, CPU-only
vcl check                           # sanity on a real machine
```

If you touched generation code: `vcl generate --speaker <existing voice> --text
"smoke" --out /tmp/smoke.wav` against a real checkpoint (GPU machine only).
Do not run full training without being asked — it is GPU-heavy and writes
gigabytes of checkpoints.

---
> Source: [tetsuo-ai/voice_clone_lab](https://github.com/tetsuo-ai/voice_clone_lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
