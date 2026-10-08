## whisper-cz-en

> Local audio/video transcription using whisper.cpp on macOS (Apple Silicon, Metal GPU). Two scripts: Czech (`prepsat.sh`) and English (`transcribe.sh`).

# Whisper CZ/EN — Project Context

Local audio/video transcription using whisper.cpp on macOS (Apple Silicon, Metal GPU). Two scripts: Czech (`prepsat.sh`) and English (`transcribe.sh`).

## Project Structure

```
whisper-cz-en/
├── transcribe.sh          # English transcription script
├── prepsat.sh             # Czech transcription script
├── README.md              # Public documentation
├── CLAUDE.md              # This file — project context for AI agents
├── LICENSE                # MIT license
├── .gitignore             # Ignores media, venv, outputs, .claude/
└── whisper.cpp/           # Git submodule — whisper.cpp v1.8.3
    ├── build/bin/whisper-cli
    └── models/
        ├── ggml-large-v3.bin          # 3.1 GB — primary transcription model
        └── ggml-silero-v6.2.0.bin     # 865 KB — VAD model
```

## Scripts

Both scripts share identical architecture — only language and output extension differ.

### `transcribe.sh` (English)

- Usage: `./transcribe.sh [--txt|--srt] <file>`
- Language flag: `-l en`
- Output: `.txt` + `.srt` (default both, `--txt` or `--srt` for single)

### `prepsat.sh` (Czech)

- Usage: `./prepsat.sh [--md|--srt] <file>`
- Language flag: `-l cs`
- Output: `.md` + `.srt` (default both, `--md` or `--srt` for single)

## Processing Pipeline

Both scripts follow the same 3-phase pipeline:

1. **Convert** — ffmpeg converts input to WAV (16kHz, mono, PCM s16le)
2. **Split** — Audio is split into 5-minute chunks (configurable via `CHUNK_MINUTES`)
3. **Transcribe** — Each chunk is transcribed independently by whisper-cli

SRT output is post-processed: timestamps are offset by chunk position and sequence numbers are renumbered globally. The transcription loop's stdout is redirected to a temp file, then moved to the final `.srt`. Whisper-cli stdout/stderr must be `>/dev/null 2>&1` to prevent raw output from leaking into the SRT file.

## Anti-Hallucination Strategy

Whisper hallucinates repetitive text during silence or unclear audio. Three countermeasures:

1. **Chunking** — 5-minute segments; model starts fresh each chunk and cannot propagate loops
2. **Silero VAD** — Voice Activity Detection skips silent segments, removing hallucination triggers
3. **Decoder parameters:**
   - `-mc 0` — zero max context (no previous text carried over between segments)
   - `-et 2.0` — lower entropy threshold (faster fallback on confusion)
   - `-ml 80` — max segment length 80 chars
   - `--suppress-nst` — suppress non-speech tokens

## Key Implementation Details

- Whisper-cli is called with `-otxt -osrt` (file output flags) regardless of user's output choice — the unwanted output is simply not collected
- Progress messages use `>&2` to avoid contaminating the stdout-to-SRT redirect
- The `for` loop redirects stdout to a temp file (`/tmp/${BASENAME}_srt_tmp.txt`), which is then moved or deleted based on output flags
- `cat "${CHUNK_OUT}.txt" >> "${BASENAME}.txt"` works inside the redirected loop because `>>` is an explicit redirect that overrides the loop-level redirect
- ffmpeg uses `-y` flag to overwrite without prompting
- All `/tmp/` paths are properly quoted to handle filenames with spaces

## Dependencies

- **whisper.cpp** — compiled with Metal support (`whisper-cli` binary in `whisper.cpp/build/bin/`)
- **ffmpeg / ffprobe** — audio conversion and duration detection
- **bash** — scripts use bash-specific features (`BASH_REMATCH`, `[[ ]]`)

## Updating whisper.cpp

```bash
cd whisper.cpp
git fetch origin
git checkout v1.X.X          # desired tag
cd ..

# Rebuild
cmake -B whisper.cpp/build -S whisper.cpp -DGGML_METAL=ON
cmake --build whisper.cpp/build --config Release -j$(sysctl -n hw.ncpu)

# Commit submodule update
git add whisper.cpp
git commit -m "Update whisper.cpp to vX.X.X"
```

After updating, run `whisper-cli --help` to check for flag changes and update scripts if needed.

## Development Notes

- Model files are large (3.1 GB) — not tracked in git; download via `whisper.cpp/models/download-ggml-model.sh`
- VAD model: download via `whisper.cpp/models/download-vad-model.sh silero-v6.2.0`
- Media files in the project root are user data, not project assets (ignored by `.gitignore`)
- `.md`, `.txt`, and `.srt` files in the root (other than README/CLAUDE) are transcription outputs
- When modifying whisper-cli flags, verify against `whisper-cli --help` — the flag set varies by whisper.cpp version

---
> Source: [faborsky/whisper-cz-en](https://github.com/faborsky/whisper-cz-en) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
