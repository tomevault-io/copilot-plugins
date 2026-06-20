## whisperx

> Speech-to-text with word-level timestamps, speaker diarization, and forced alignment using WhisperX. Built on faster-whisper with batched inference for 70x realtime speed.


# WhisperX

Speech-to-text with **word-level timestamps**, **speaker diarization**, and **forced alignment** — built on faster-whisper with batched inference for up to **70x realtime** transcription speed.

WhisperX extends Whisper with three key capabilities that faster-whisper alone doesn't provide:
1. **Forced alignment** — precise word-level timestamps via phoneme ASR models (wav2vec2)
2. **Speaker diarization** — label who said what (via pyannote.audio)
3. **Batched inference** — process audio in parallel chunks for massive speedup

## When to Use

Use this skill when you need to:
- **Transcribe with word-level timing** — subtitles, captions, karaoke-style highlighting
- **Identify speakers** — meetings, interviews, podcasts, multi-speaker recordings
- **Generate subtitle files** — SRT, VTT with accurate word timestamps
- **Translate speech to English** — from any supported language
- **Batch transcribe** — efficient processing of multiple files
- **Transcribe a section** — extract just part of a long recording

**Trigger phrases:** "transcribe with speakers", "who said what", "diarize", "make subtitles", "word timestamps", "speaker identification", "meeting transcript", "karaoke subtitles"

**When NOT to use:**
- Simple transcription without speaker/timing needs → use **faster-whisper** (lighter, faster)
- Real-time/streaming transcription
- Files <10 seconds where setup overhead isn't worth it

**WhisperX vs faster-whisper:**
| Feature | faster-whisper | WhisperX |
|---------|---------------|----------|
| Basic transcription | ✅ | ✅ |
| Word timestamps | ✅ (approximate) | ✅ (precise, aligned) |
| Speaker diarization | ❌ | ✅ |
| Forced alignment | ❌ | ✅ |
| Batched inference | ❌ | ✅ |
| Word-level subtitles | ❌ | ✅ (karaoke-style) |
| Subtitle generation | Manual | Built-in (SRT/VTT/TSV) |
| Time range trimming | ❌ | ✅ (--start/--end) |
| Hotwords | ❌ | ✅ (boost specific terms) |
| Initial prompt | ❌ | ✅ (domain terms) |
| Speaker renaming | ❌ | ✅ (--speaker-names) |
| Stdin pipe | ❌ | ✅ (read from `-`) |
| Setup complexity | Simple | Requires HF token for diarization |

## Quick Reference

All commands use `./scripts/transcribe` (the skill wrapper), **not** the `whisperx` CLI directly. The wrapper applies a required PyTorch compatibility patch — see "PyTorch 2.6+ Compatibility" below.

| Task | Command | Notes |
|------|---------|-------|
| **Basic transcription** | `./scripts/transcribe audio.mp3` | Word-aligned by default |
| **With speakers** | `./scripts/transcribe audio.mp3 --diarize` | Auto-reads `~/.cache/huggingface/token` |
| **Clean speaker output** | `./scripts/transcribe audio.mp3 --diarize --merge-speakers` | Merges consecutive same-speaker segments |
| **Named speakers** | `./scripts/transcribe audio.mp3 --diarize --speaker-names "Alice,Bob"` | Replaces SPEAKER_00, SPEAKER_01 |
| **SRT subtitles** | `./scripts/transcribe audio.mp3 --srt -o subs.srt` | Ready for video players |
| **Word-level SRT** | `./scripts/transcribe audio.mp3 --srt --word-level` | Karaoke-style, one word per cue |
| **Wrapped subtitles** | `./scripts/transcribe audio.mp3 --srt --max-line-width 42` | Standard TV subtitle width |
| **VTT subtitles** | `./scripts/transcribe audio.mp3 --vtt -o subs.vtt` | Web-compatible with `<v>` speaker tags |
| **JSON output** | `./scripts/transcribe audio.mp3 --json` | Full data with word timestamps |
| **TSV output** | `./scripts/transcribe audio.mp3 --tsv` | Spreadsheet-friendly |
| **Translate to English** | `./scripts/transcribe audio.mp3 --translate` | Any language → English |
| **Fast, no alignment** | `./scripts/transcribe audio.mp3 --no-align` | Skip forced alignment |
| **Specific language** | `./scripts/transcribe audio.mp3 -l en` | Faster than auto-detect |
| **Partial transcription** | `./scripts/transcribe audio.mp3 --start 1:30 --end 5:00` | Only a section |
| **Boost terms** | `./scripts/transcribe audio.mp3 --hotwords "Kubernetes gRPC"` | Improve rare term recognition |
| **Domain accuracy** | `./scripts/transcribe audio.mp3 --initial-prompt "OpenAI, GPT-4"` | Condition the model |
| **From stdin** | `cat audio.mp3 \| ./scripts/transcribe -` | Pipe from other tools |
| **Auto-detect format** | `./scripts/transcribe audio.mp3 -o out.srt` | Format from extension |

**⚠️ Do NOT run `whisperx` CLI directly** — it will crash on PyTorch 2.6+ with pyannote models. Always use this skill's `./scripts/transcribe` wrapper.

## Model Selection

| Model | Size | Speed | Accuracy | Use Case |
|-------|------|-------|----------|----------|
| `tiny` | 39M | Fastest | Basic | Quick drafts, testing |
| `base` | 74M | Very fast | Good | General use |
| `small` | 244M | Fast | Better | Default for whisperx CLI |
| `medium` | 769M | Moderate | High | Quality transcription |
| `large-v2` | 1.5GB | Slower | Excellent | Best diarization compat |
| `large-v3` | 1.5GB | Slower | Best | Maximum accuracy |
| **`large-v3-turbo`** | 809M | Fast | Excellent | **Recommended (default)** |

**Note:** WhisperX defaults to `small` but this skill defaults to `large-v3-turbo` for the best speed/accuracy balance with GPU.

## Setup

### First-Time Setup

**Prerequisites:** Python 3.10+, ffmpeg, NVIDIA GPU with CUDA (strongly recommended)

**Step 1: Install whisperx**

```bash
# Option A: Run the setup script (auto-detects GPU, creates venv if needed)
./setup.sh

# Option B: Install globally (if you prefer)
pip install whisperx
```

**Step 2 (optional, for diarization): Set up Hugging Face token**

Speaker diarization requires a free Hugging Face account and access to gated models. Skip this if you only need transcription/alignment.

1. Create account at [huggingface.co](https://huggingface.co) (if you don't have one)
2. Go to [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens) and create a **read** access token
3. Accept the model agreement(s) (click "Agree and access repository"):
   - **whisperx ≥3.8.0** (recommended): Accept [pyannote/speaker-diarization-community-1](https://huggingface.co/pyannote/speaker-diarization-community-1) — uses pyannote v4 with better accuracy
   - **whisperx <3.8.0**: Accept **both** [pyannote/speaker-diarization-3.1](https://huggingface.co/pyannote/speaker-diarization-3.1) and [pyannote/segmentation-3.0](https://huggingface.co/pyannote/segmentation-3.0)
4. Save the token so the script auto-detects it:
   ```bash
   mkdir -p ~/.cache/huggingface && echo -n "hf_YOUR_TOKEN" > ~/.cache/huggingface/token && chmod 600 ~/.cache/huggingface/token
   ```
   Alternatively: set `HF_TOKEN` env var, or pass `--hf-token` per-command.

**Note:** The token and model access are completely free. The models are just gated behind a click-to-agree license. Without step 3, you'll get a 403 error even with a valid token.

### Subsequent Runs

No setup needed — just run `./scripts/transcribe`. The wrapper script:
- Auto-detects GPU/CPU and picks optimal compute type
- Auto-reads the HF token from `~/.cache/huggingface/token` for diarization
- Applies the PyTorch 2.6+ compatibility patch automatically (see below)
- First run for a new model downloads it to `~/.cache/huggingface/` (one-time per model)

### Checking If It's Working

```bash
# Quick test — should print transcript to stdout
./scripts/transcribe some_audio.mp3

# Test diarization — should show [SPEAKER_00], [SPEAKER_01], etc.
./scripts/transcribe some_audio.mp3 --diarize

# Check version
./scripts/transcribe --version

# If diarization fails with 403: model agreements not accepted (see step 3 above)
# If it crashes with pickle/weights_only error: you're running `whisperx` CLI directly instead of the wrapper
```

### Platform Support

| Platform | Acceleration | Speed |
|----------|-------------|-------|
| **Linux + NVIDIA GPU** | CUDA (batched) | ~70x realtime 🚀 |
| **WSL2 + NVIDIA GPU** | CUDA (batched) | ~70x realtime 🚀 |
| macOS Apple Silicon | CPU | ~3-5x realtime |
| macOS Intel | CPU | ~1-2x realtime |
| Linux (no GPU) | CPU | ~1x realtime |

## Usage

All commands use `./scripts/transcribe` — resolve the path relative to this skill's directory.

```bash
# Basic transcription (word-aligned)
./scripts/transcribe audio.mp3

# With speaker diarization (auto-reads ~/.cache/huggingface/token)
./scripts/transcribe audio.mp3 --diarize

# Diarize with merged same-speaker segments (cleaner output)
./scripts/transcribe audio.mp3 --diarize --merge-speakers

# Rename speakers to real names
./scripts/transcribe audio.mp3 --diarize --speaker-names "Alice,Bob,Charlie"

# Generate SRT subtitles
./scripts/transcribe audio.mp3 --srt -o subtitles.srt

# SRT with line wrapping (standard TV width)
./scripts/transcribe audio.mp3 --srt --max-line-width 42 -o subtitles.srt

# Word-level karaoke subtitles (one word per cue with precise timing)
./scripts/transcribe audio.mp3 --srt --word-level -o karaoke.srt

# VTT with speaker voice tags (<v Speaker>text</v>)
./scripts/transcribe audio.mp3 --vtt --diarize -o subtitles.vtt

# Auto-detect format from output filename
./scripts/transcribe audio.mp3 -o transcript.json
./scripts/transcribe audio.mp3 -o subtitles.vtt

# TSV for spreadsheets/data analysis
./scripts/transcribe audio.mp3 --tsv -o transcript.tsv

# Transcribe only a section (useful for long recordings)
./scripts/transcribe podcast.mp3 --start 10:30 --end 15:00

# Boost recognition of specific terms (hotwords)
./scripts/transcribe meeting.mp3 --hotwords "Kubernetes gRPC OAuth2"

# Improve accuracy for domain context (initial prompt)
./scripts/transcribe meeting.mp3 --initial-prompt "Attendees: Alice, Bob. Topics: Kubernetes, gRPC, OAuth2"

# Maximum accuracy
./scripts/transcribe audio.mp3 --model large-v3

# Translate non-English audio to English
./scripts/transcribe audio.mp3 --translate -l ja

# Fast mode (skip alignment)
./scripts/transcribe audio.mp3 --no-align

# JSON with full metadata and word timestamps
./scripts/transcribe audio.mp3 --json -o transcript.json

# Specify known speaker count for better diarization
./scripts/transcribe audio.mp3 --diarize --min-speakers 2 --max-speakers 4

# Read audio from stdin (pipe from other tools)
cat audio.mp3 | ./scripts/transcribe -
ffmpeg -i video.mp4 -f wav - | ./scripts/transcribe - --diarize
```

## Options

```
AUDIO_FILE               Path to audio/video file, or '-' to read from stdin

Model options:
  -m, --model NAME       Whisper model (default: large-v3-turbo)
  --batch-size N         Batch size for inference (default: 8, lower if OOM)
  --beam-size N          Beam search size (higher = slower but more accurate)
  --initial-prompt TEXT   Condition the model with domain terms, names, acronyms
  --hotwords TEXT         Space-separated hotwords to boost recognition of rare terms

Device options:
  --device               cpu, cuda, or auto (default: auto)
  --compute-type         int8, float16, float32, or auto (default: auto)
  --threads N            CPU threads for CTranslate2 inference (default: 4)

Language options:
  -l, --language CODE    Language code (auto-detects if omitted)
  --translate            Translate to English

Time range:
  --start TIME           Start time — seconds (90), MM:SS (1:30), HH:MM:SS
  --end TIME             End time — same formats as --start

Alignment options:
  --no-align             Skip forced alignment (no word timestamps)
  --align-model MODEL    Custom phoneme ASR model for alignment

Speaker diarization:
  --diarize              Enable speaker labels
  --hf-token TOKEN       Hugging Face access token (also reads ~/.cache/huggingface/token or HF_TOKEN env)
  --min-speakers N       Minimum speaker count hint
  --max-speakers N       Maximum speaker count hint
  --merge-speakers       Merge consecutive segments from same speaker (cleaner output)
  --speaker-names NAMES  Comma-separated names to replace SPEAKER_00, SPEAKER_01, etc.

Output options:
  -j, --json             JSON output with segments and word timestamps
  --srt                  SRT subtitle format
  --vtt                  WebVTT subtitle format (uses <v> voice tags for speakers)
  --tsv                  TSV (tab-separated values) for data analysis
  --word-level           Word-level subtitles (SRT/VTT only) — karaoke-style
  --max-line-width N     Maximum characters per subtitle line (wraps at word boundaries)
  --output-format FMT    Explicit format (srt, vtt, txt, json, tsv)
  -o, --output FILE      Save to file (format auto-detected from extension)

Miscellaneous:
  -V, --version          Show version
  -q, --quiet            Suppress progress messages
```

## Output Formats

### Plain Text (default)
```
Hello and welcome to the show.
Today we're talking about AI transcription.
```

### Plain Text with Diarization (`--diarize`)
```
[SPEAKER_00] Hello and welcome to the show.
[SPEAKER_01] Thanks for having me.
```

### Plain Text with Named Speakers (`--diarize --speaker-names "Alice,Bob"`)
```
[Alice] Hello and welcome to the show.
[Bob] Thanks for having me.
```

### SRT (`--srt`)
Standard subtitle format, compatible with VLC, YouTube, etc.
```
1
00:00:00,000 --> 00:00:03,500
Hello and welcome to the show.

2
00:00:03,500 --> 00:00:06,200
Today we're talking about AI transcription.
```

### Word-Level SRT (`--srt --word-level`)
One word per cue — for karaoke-style highlighting or precise editing.
```
1
00:00:00,000 --> 00:00:00,320
Hello

2
00:00:00,320 --> 00:00:00,560
and

3
00:00:00,560 --> 00:00:01,100
welcome
```

### WebVTT (`--vtt`)
Web-native subtitle format for HTML5 `<video>` and `<track>`. When used with `--diarize`, uses proper VTT `<v>` voice tags for speaker identification.
```
WEBVTT

00:00:00.000 --> 00:00:03.500
<v Alice>Hello and welcome to the show.</v>

00:00:03.500 --> 00:00:05.000
<v Bob>Thanks for having me.</v>
```

### JSON (`--json`)
Structured output with word-level timestamps and confidence scores.
```json
{
  "segments": [
    {
      "start": 0.0,
      "end": 3.5,
      "text": "Hello and welcome to the show.",
      "speaker": "SPEAKER_00",
      "words": [
        {"word": "Hello", "start": 0.0, "end": 0.32, "confidence": 0.98},
        {"word": "and", "start": 0.32, "end": 0.56, "confidence": 0.95}
      ]
    }
  ]
}
```

### TSV (`--tsv`)
Tab-separated values for spreadsheets and data pipelines.
```
start	end	text
0.000	3.500	Hello and welcome to the show.
3.500	6.200	Today we're talking about AI transcription.
```

## Examples

```bash
# Transcribe a meeting recording with speakers (clean output)
./scripts/transcribe meeting.mp3 --diarize --merge-speakers \
  --speaker-names "Alice,Bob,Charlie" \
  --min-speakers 3 --max-speakers 5 --json -o meeting.json

# Generate subtitles for a video (wrapped to standard width)
./scripts/transcribe video.mp4 --srt --max-line-width 42 -o video.srt

# Karaoke-style word-level subtitles
./scripts/transcribe song.mp3 --vtt --word-level -o karaoke.vtt

# Transcribe just the interesting part of a podcast
./scripts/transcribe podcast.mp3 --start 45:00 --end 1:02:30 --diarize --merge-speakers

# Improve accuracy for technical content (hotwords + initial prompt)
./scripts/transcribe lecture.mp3 \
  --hotwords "PyTorch CTranslate2 whisperx" \
  --initial-prompt "A lecture on ML model optimization"

# Batch transcribe a folder
for file in recordings/*.mp3; do
  ./scripts/transcribe "$file" --json -o "${file%.mp3}.json"
done

# Transcribe YouTube audio (with yt-dlp)
yt-dlp -x --audio-format mp3 <URL> -o audio.mp3
./scripts/transcribe audio.mp3 --diarize --merge-speakers

# Pipe directly from ffmpeg (extract audio on the fly)
ffmpeg -i video.mp4 -f wav -ac 1 -ar 16000 - 2>/dev/null | ./scripts/transcribe -

# Quick draft (fast, no alignment)
./scripts/transcribe audio.mp3 --model base --no-align

# German audio with TSV output for analysis
./scripts/transcribe audio.mp3 -l de --tsv -o transcript.tsv

# Auto-detect format from filename
./scripts/transcribe audio.mp3 -o transcript.srt   # → SRT
./scripts/transcribe audio.mp3 -o data.json         # → JSON
./scripts/transcribe audio.mp3 -o export.tsv        # → TSV
```

## Common Mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| **Using CPU when GPU available** | 10-70x slower | Check `nvidia-smi`; verify CUDA |
| **Missing HF token for diarize** | Diarization fails | Get token from huggingface.co/settings/tokens |
| **Not accepting model agreements** | 403 error on diarization model | whisperx ≥3.8.0: accept community-1. Earlier: accept **both** pyannote/speaker-diarization-3.1 AND segmentation-3.0 (see Setup) |
| **Running `whisperx` CLI directly** | Crashes on PyTorch 2.6+ | Always use `./scripts/transcribe` wrapper (applies torch.load patch) |
| **batch_size too high** | CUDA OOM | Lower `--batch-size` (try 4 or 2) |
| **Using large-v3 when turbo works** | Unnecessary slowdown | `large-v3-turbo` is faster with near-identical accuracy |
| **Forgetting --language** | Wastes time auto-detecting | Specify `-l en` when you know the language |
| **Using WhisperX for simple transcription** | Heavier setup for no benefit | Use faster-whisper for basic transcription |
| **--word-level without --srt/--vtt** | Flag is ignored | Word-level only applies to subtitle formats |
| **--merge-speakers without --diarize** | Flag is ignored | Merge only works when speakers are identified |

## Performance Notes

- **First run**: Downloads model + alignment model (one-time)
- **GPU batched inference**: Up to 70x realtime with large-v2
- **Diarization**: Adds ~30-60s overhead for model loading
- **Memory** (GPU VRAM):
  - `large-v3-turbo`: ~2-3GB
  - `large-v3` + diarization: ~4-5GB
  - Reduce `--batch-size` if OOM
- **Completion stats**: The tool prints segment/word counts, speed ratio, and speaker breakdown (when diarizing) at the end

## Supported Languages

WhisperX supports all languages that Whisper supports (99 languages). **Forced alignment** (word timestamps) is available for a subset — if alignment fails for a language, the tool falls back gracefully to segment-level timestamps.

**Languages with alignment support** (common subset):
`en` English, `zh` Chinese, `de` German, `es` Spanish, `fr` French, `it` Italian, `ja` Japanese, `ko` Korean, `pt` Portuguese, `ru` Russian, `nl` Dutch, `pl` Polish, `tr` Turkish, `ar` Arabic, `sv` Swedish, `da` Danish, `fi` Finnish, `hu` Hungarian, `uk` Ukrainian, `el` Greek, `cs` Czech, `ro` Romanian, `vi` Vietnamese, `th` Thai, `hi` Hindi, `he` Hebrew, `id` Indonesian, `ms` Malay, `no` Norwegian, `fa` Persian, `bg` Bulgarian, `ca` Catalan, `hr` Croatian, `sk` Slovak, `sl` Slovenian, `ta` Tamil, `te` Telugu, `ur` Urdu

For the full list, see [whisperx/alignment.py](https://github.com/m-bain/whisperX/blob/main/whisperx/alignment.py).

## Hotwords vs Initial Prompt

Both improve accuracy for specific terms, but they work differently:

| | `--hotwords` | `--initial-prompt` |
|---|---|---|
| **How it works** | Boosts probability of specific tokens during decoding | Conditions the model as if these words appeared earlier |
| **Best for** | Rare terms, proper nouns, technical jargon | Setting domain context, style, formatting |
| **Example** | `--hotwords "Kubernetes gRPC OAuth2"` | `--initial-prompt "A technical meeting about cloud infrastructure"` |
| **Can combine** | ✅ Yes, use both together for best results | ✅ |
| **Requires** | whisperx ≥3.7.5 | Any version |

**Tip:** Use hotwords for the specific words you need recognized correctly, and initial prompt for broader context about the audio content.

## PyTorch 2.6+ Compatibility (CRITICAL)

**⚠️ PyTorch 2.6 changed `torch.load()` to default to `weights_only=True`.** This breaks pyannote.audio's model loading (used by both VAD and diarization), because the model checkpoints contain globals like `omegaconf.listconfig.ListConfig` and `torch.torch_version.TorchVersion` that aren't allowlisted.

**Symptoms:**
- `_pickle.UnpicklingError: Weights only load failed` when loading VAD or diarization models
- `'NoneType' object has no attribute 'to'` (pipeline silently returns None)

**How this skill handles it:**

The `scripts/transcribe.py` uses the **whisperx Python API directly** (not as a subprocess) so it can monkey-patch `torch.load` before any model loading happens:

```python
import torch
_original_torch_load = torch.load
def _patched_torch_load(*args, **kwargs):
    kwargs['weights_only'] = False  # Must FORCE, not setdefault — lightning_fabric passes True explicitly
    return _original_torch_load(*args, **kwargs)
torch.load = _patched_torch_load
```

**Key details:**
- The patch **must** use `kwargs['weights_only'] = False` (forced override), NOT `kwargs.setdefault('weights_only', False)` — because `lightning_fabric` explicitly passes `weights_only=True`, which `setdefault` won't override
- The patch **must** be applied before importing `whisperx`, `pyannote`, or any model loading code
- This is why the skill uses the Python API instead of shelling out to `whisperx` CLI — a subprocess can't inherit the monkey-patch
- VAD defaults to **silero** (not pyannote) to avoid a second torch.load issue in the VAD pipeline. Silero loads fine without the patch, but diarization still needs it

**Note:** whisperx ≥3.8.0 migrated to pyannote-audio v4 with `speaker-diarization-community-1`, which may resolve some of these compatibility issues. The patch is kept for broad version support.

**If whisperx CLI is updated to fix this upstream**, the monkey-patch can be removed and the script could switch back to subprocess mode. Track: [whisperX#972](https://github.com/m-bain/whisperX/issues/972)

## Troubleshooting

**`_pickle.UnpicklingError: Weights only load failed`**: PyTorch 2.6+ compat issue. If running via CLI (`whisperx` command directly), this can't be fixed without patching the installed library. Use this skill's `scripts/transcribe` wrapper instead, which applies the patch automatically. See "PyTorch 2.6+ Compatibility" section above.

**"CUDA not available"**: Install PyTorch with CUDA (`pip install torch --index-url https://download.pytorch.org/whl/cu121`)

**"No module named whisperx"**: Run `./setup.sh` or `pip install whisperx`

**Diarization 403 error**: You must accept the model agreement(s). For whisperx ≥3.8.0: accept [community-1](https://huggingface.co/pyannote/speaker-diarization-community-1). For earlier versions: accept **both** [speaker-diarization-3.1](https://huggingface.co/pyannote/speaker-diarization-3.1) and [segmentation-3.0](https://huggingface.co/pyannote/segmentation-3.0). See Setup above.

**Diarization fails but transcription continues**: v1.1.0+ gracefully handles diarization failures — it prints a diagnostic error and continues without speaker labels instead of crashing.

**`'NoneType' object has no attribute 'to'`**: Either the HF token is invalid, the model agreements haven't been accepted, or the torch.load patch isn't applied. Check all three.

**OOM on GPU**: Lower `--batch-size` to 4 or 2

**Alignment fails for language X**: The language may not have a wav2vec2 alignment model. The tool will fall back to segment-level timestamps and print a warning. Check supported languages in [whisperx alignment.py](https://github.com/m-bain/whisperX/blob/main/whisperx/alignment.py).

**Slow on CPU**: Expected — use GPU for practical transcription. Even `tiny` model on CPU is ~5-10x slower than `large-v3-turbo` on a mid-range GPU.

**Empty output / no segments**: Audio may be silence or too short. Check with `ffprobe audio.mp3` to verify the file has actual audio content. v1.1.0+ prints a warning and produces valid empty output instead of crashing.

**Timestamps wrong after trimming**: If using `--start`, timestamps in the output reflect the original file's timeline (not relative to the trim point). This is by design — subtitle timecodes stay correct for the source video.

**"No speech detected" warning**: The audio file may contain only music, silence, or non-speech sounds. This is expected behavior, not an error.

## Upstream Changes

**whisperx 3.8.0** (Feb 2026): Migrated to pyannote-audio v4 with `speaker-diarization-community-1`. This model has lower diarization error rates across all benchmarks compared to the older `speaker-diarization-3.1`. Upgrade recommended: `pip install --upgrade whisperx`

**whisperx 3.7.5**: Added `--hotwords` support for boosting recognition of specific terms. This skill exposes it via the `--hotwords` flag.

## References

- [WhisperX GitHub](https://github.com/m-bain/whisperX)
- [WhisperX Paper (INTERSPEECH 2023)](https://arxiv.org/abs/2303.00747)
- [pyannote.audio](https://github.com/pyannote/pyannote-audio) — speaker diarization
- [faster-whisper](https://github.com/SYSTRAN/faster-whisper) — CTranslate2 backend

---
> Source: [ThePlasmak/whisperx](https://github.com/ThePlasmak/whisperx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
