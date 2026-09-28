## nemo3-battleships

> An all-NVIDIA speech stack on Modal (Nemotron 3 diarization + Nemotron 3.5 ASR, coupled; MagpieTTS)

# nemo3-battleships

An all-NVIDIA speech stack on Modal (Nemotron 3 diarization + Nemotron 3.5 ASR, coupled; MagpieTTS)
and on it a voice game of Battleships for two players on one microphone: diarization tells their
voices apart so one player can't take the other's turn, a Jev referee (TypeSafe) reads squares
and game instructions, and one LLM (`LLM=luna|phonellm`: GPT-5.6 Luna or PhoneLLM) handles names
and confirmations and narrates. `README.md` is the quick start; this file is the map.

| Package | What | Start here |
|---|---|---|
| `nemo3/` | The Modal app `nemo3`: class `Speech` (diarizer + ASR via NeMo `SpeakerTaggedASR`, WebSocket) and class `TTS` (Magpie, HTTP streaming); a `nemo3-tools` app (`verify_image`, `prepare_weights`, `inspect`, `speech_sweep`); torch-free fakes; scripts | `nemo3/AGENTS.md`, `nemo3/README.md` |
| `server/` | The Pipecat 1.11 bot: `game` (Battleships: floor, speaker gate, Jev referee, LLM host and non-interruptible narrator), plus `stt` and `tts` modes to test the services | `server/AGENTS.md` |
| `deploy.sh` | Idempotent Modal setup + deploy (speech/TTS, and the PhoneLLM endpoint with `--phonellm`); writes the endpoint URLs into `.env` | `./deploy.sh --help` |
| `client/` | Vite + React 19 + Tailwind 4 + shadcn (`base-nova`) + Pipecat UI + Zustand: the game screen with a lazy 3D pirate board, mirrored from the bot over the RTVI UI channel | `client/AGENTS.md` |
| `eval-dashboard/` | Vite + React, no bot: a full-screen replay of one diarization eval run (an 8-voice group call with overlaps, one live session) for a demo video; runs come from `nemo3/scripts/group_eval.py` | `eval-dashboard/AGENTS.md` |

Known limit, measured: the streaming diarizer loses a second voice across long pauses and
short turns, which a turn-based game is full of, so turn enforcement is only as good as the
labels (`server/AGENTS.md`, "Known limit").

Secrets live only in the repo-root `.env` (gitignored; `.env.example` lists every key). Only
`HF_TOKEN` is ever copied into Modal (secret `huggingface-secret`). `MODAL_API_KEY` is a proxy
token that the bot sends as `Authorization: Bearer`.

Conventions: uv per package with `pyproject.toml`; ruff line-length
100 with `I` + `UP`; pyright; pytest; Modal-specific code only in `nemo3/modal_app.py`,
everything else pure and tested offline; each service has a torch-free fake for the local loop
(`.claude/launch.json` → `nemo3-fake`, `server`, `client`, `eval-dashboard`).

Licence: `Nemotron-3-Diarization-preview` is under NVIDIA's evaluation licence. Private endpoint
(proxy auth), weights only on the Modal Volume, never run on the Mac, no public results. The
model card and integration guide are in the gitignored `.model-docs/`.

Cost: with `NEMO3_WARM_CONTAINERS=1` two L4s stay warm at ≈ $1.85/h. `./deploy.sh --warm 0` is the
off switch; `--warm 1` restores it.

---
> Source: [pipecat-ai/nemo3-battleships](https://github.com/pipecat-ai/nemo3-battleships) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
