## reachyminichat

> Manages conversation context for Ollama API.

# AGENTS.md — Reachy Mini Chat

This file contains essential information for AI coding agents working on the Reachy Mini Chat project.

---

## Project Overview

**Reachy Mini Chat** is a voice-enabled conversational AI demo for the Reachy Mini desktop robot. It integrates:

- **Ollama LLM** for conversational AI (local inference)
- **Piper-TTS** for offline text-to-speech synthesis
- **faster-whisper** for automatic speech recognition (ASR)
- **WebRTC VAD** for voice activity detection
- **Reachy Mini SDK** for robot motion control and animation

The project enables a fully offline conversational robot experience with emotion-driven gestures, synchronized movements, and lip-sync animations.

---

## Technology Stack

| Component | Technology |
|-----------|------------|
| Language | Python 3 |
| Robot SDK | `reachy-mini` (MuJoCo simulation support) |
| LLM Backend | Ollama (local API at `http://localhost:11434`) |
| TTS Engine | Piper-TTS (offline ONNX models) |
| ASR Engine | faster-whisper (CPU inference) |
| VAD | webrtcvad-wheels |
| Audio I/O | sounddevice + soundfile |
| HTTP Client | aiohttp (async) |

---

## Project Structure

```
ReachyMiniChat/
├── emo_v1.py → emo_v9.py       # Versioned emotion controllers (see below)
├── utils/                       # Utility modules
│   ├── asr.py                   # FasterWhisper ASR engine with VAD
│   ├── test_actions.py          # Test robot recorded moves
│   ├── test_edge_tts_voices.py  # Edge-TTS voice discovery
│   ├── test_emotion_analysis.py # Emotion analysis testing
│   ├── test_audio.py            # Audio playback testing
│   ├── test_ollama_connection.py # Ollama connectivity check
│   ├── latency_harness.py       # Performance testing
│   └── simple_interact.py       # Manual robot testing
├── models/                      # Piper-TTS voice models (.onnx + .json)
├── docs/                        # Archived version documentation
├── requirements.txt             # Python dependencies
├── README.md                    # Main project documentation
├── EMO_README.md                # Version comparison table
├── EMO_V6_README.md             # v6 detailed documentation
├── EMO_V7_README.md             # v7 ASR pipeline documentation
├── emo_v9_todo.md               # Development todo list (Chinese)
├── install-rocm.md              # AMD ROCm installation guide
└── FAQ.md                       # Frequently asked questions
```

---

## Version History

| Version | Key Feature | Status |
|---------|-------------|--------|
| emo_v1.py | Baseline high-intensity emotion controller | Archived |
| emo_v2.py | RecordedMoves integration | Archived |
| emo_v3.py | Streaming-triggered actions | Archived |
| emo_v4.py | Offline TTS (espeak) + lip-sync | Archived |
| emo_v5.py | Edge-TTS integration | Archived |
| emo_v6.py | Continuous synchronized actions + cartoon voices | Stable |
| emo_v7.py | ASR → LLM → TTS pipeline (Edge-TTS) | Stable |
| emo_v7_vad.py | VAD-enhanced ASR variant | Archived |
| emo_v8.py | Piper-TTS offline version | Stable |
| emo_v9.py | Conversation history + performance timing | Development |

**Current development focus**: `emo_v9.py` with conversation history, VAD optimization, and performance statistics.

---

## Setup and Installation

### System Requirements

- **OS**: Ubuntu 24.04 (developed on AMD Ryzen™ AI Max+ 395)
- **Python**: 3.8+
- **Hardware**: CPU sufficient; AMD ROCm optional for GPU acceleration

### System Dependencies

```bash
sudo apt update
sudo apt install -y python3 python3-venv python3-pip espeak ffmpeg libsndfile1 portaudio19-dev
```

### Python Environment

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
pip install "reachy-mini[mujoco]"
```

### Ollama Setup

```bash
# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Pull recommended model
ollama pull qwen3:0.6b
```

### Piper Voice Models

Download `.onnx` and `.onnx.json` files from:
- https://github.com/rhasspy/piper/releases/tag/v0.0.2
- https://huggingface.co/rhasspy/piper-voices

Place in `models/` directory.

---

## Running the Application

### Start Robot Simulation

```bash
# Terminal 1: Start Reachy Mini simulator
reachy-mini-daemon --sim

# Use this if GUI fails on Wayland (Ubuntu 24.04 default)
export PYGLFW_LIBRARY_VARIANT=x11
```

### Run Chat Application

```bash
# Text chat mode (emo_v9 - latest)
python emo_v9.py --model qwen3:0.6b --piper-model models/en-us-blizzard_lessac-medium.onnx

# ASR mode with VAD
python emo_v9.py --asr --model qwen3:0.6b --piper-model models/zh_CN-huayan-medium.onnx --gentle

# With conversation history disabled
python emo_v9.py --no-history

# Debug mode with timing statistics
python emo_v9.py --debug --asr
```

### Common CLI Flags

| Flag | Description |
|------|-------------|
| `--asr` | Enable microphone ASR input |
| `--model` | Ollama model name (default: qwen3:0.6b) |
| `--piper-model` | Path to Piper .onnx voice model |
| `--piper-config` | Path to Piper .json config |
| `--gentle` | Enable gentle mode (subtle motions) |
| `--debug` | Enable debug output with timing stats |
| `--history-size N` | Conversation history rounds (default: 5) |
| `--no-history` | Disable conversation history |
| `--asr-model` | ASR model size: tiny/base/small/medium/large |
| `--vad-silence` | VAD silence threshold in seconds (default: 0.8) |
| `--vad-aggressive` | VAD aggressiveness 0-3 (default: 1) |
| `--no-vad` | Use fixed 4s recording instead of VAD |

---

## Code Organization

### Core Classes

#### `PiperTTSEngine` (emo_v8.py, emo_v9.py)
Offline TTS wrapper around piper-tts library.
- Loads ONNX voice models
- Synthesizes speech to temporary WAV files
- Plays audio via sounddevice

#### `FasterWhisperASREngine` (utils/asr.py)
ASR engine with VAD support.
- `transcribe_from_mic(duration)` — Fixed-length recording
- `transcribe_from_mic_vad(max_duration, silence_threshold)` — VAD-based recording

#### `EmotionControllerV6` (emo_v6.py)
Base emotion controller with:
- Emotion analysis from text
- Recorded moves library integration
- Lip-sync controller
- Combined action sequences (eye blink + body yaw + head + antennas)

#### `EmotionControllerV71` (emo_v8.py, emo_v9.py)
Extended controller replacing Edge-TTS with Piper-TTS.

#### `ConversationHistory` (emo_v9.py)
Manages conversation context for Ollama API.
- Configurable history size
- Automatic message formatting

#### `ChatAppWithPiper` (emo_v8.py, emo_v9.py)
Main application orchestrating:
- Ollama API communication (async)
- ASR/TTS coordination
- Robot animation control

---

## Development Conventions

### Code Style

- Use **type hints** for function signatures
- Use **async/await** for I/O-bound operations (HTTP, TTS)
- Use **threading** for CPU-bound operations (ASR inference)
- Prefix private methods with underscore: `_helper_method()`
- Use emoji indicators in user-facing output: ✅ ❌ ⚠️ 🤖

### Error Handling

```python
try:
    result = await async_operation()
except asyncio.TimeoutError:
    print("⚠️ Operation timed out")
    return None
except Exception as e:
    if self.debug:
        import traceback
        traceback.print_exc()
    return None
```

### Adding New Features

1. **Incremental changes**: Make small, testable modifications
2. **Version files**: Create new `emo_v{N}.py` for major changes
3. **CLI flags**: Add argparse arguments for configuration
4. **Documentation**: Update relevant README files

---

## Testing

### Test Robot Actions

```bash
python utils/test_actions.py
```

### Test ASR

```bash
python utils/asr.py  # Interactive test
```

### Test Ollama Connection

```bash
python utils/test_ollama_connection.py
```

### Component Tests (emo_v6+)

```bash
# Test TTS only
python emo_v6.py --test-tts

# Test robot actions
python emo_v6.py --test-actions

# Test Edge-TTS voices
python utils/test_edge_tts_voices.py
```

---

## Key Dependencies

See `requirements.txt` for full list:

```
requests>=2.31.0
numpy>=1.24.0
scipy>=1.10.0
edge-tts>=1.2.0
soundfile>=0.12.1
sounddevice>=0.4.8
aiohttp>=3.9.0
faster-whisper>=1.2.1
webrtcvad-wheels>=2.0.14
piper-tts>=1.4.0
```

---

## Troubleshooting

### Audio Issues
- Ensure `libsndfile1` and `portaudio19-dev` are installed
- Check audio devices: `aplay -l` (Linux)
- Verify `sounddevice` is installed in venv

### Robot Connection
- Ensure `reachy-mini-daemon --sim` is running
- Check `PYGLFW_LIBRARY_VARIANT=x11` for Wayland issues

### Ollama Issues
- Verify Ollama is running: `curl http://localhost:11434/api/tags`
- Check model availability: `ollama list`

### VAD Issues
- Increase silence threshold: `--vad-silence 1.5`
- Reduce aggressiveness: `--vad-aggressive 1`
- Disable VAD: `--no-vad`

---

## Security Considerations

- Ollama API runs locally on `localhost:11434` — no external exposure
- Piper-TTS is fully offline — no cloud dependencies for speech
- ASR runs locally on CPU — no audio data leaves the machine
- Edge-TTS (emo_v5-emo_v7) requires internet — cloud-based Microsoft voices

---

## Language Notes

- Project documentation uses **English** primarily
- `emo_v9_todo.md` contains **Chinese** text (development notes)
- Code comments and docstrings use **English**
- The robot supports **multilingual** TTS (English, Chinese, etc.)

---

## References

- Reachy Mini SDK: https://docs.pollen-robotics.com/
- Ollama: https://ollama.com/
- Piper TTS: https://github.com/rhasspy/piper
- faster-whisper: https://github.com/SYSTRAN/faster-whisper

---
> Source: [alexhegit/ReachyMiniChat](https://github.com/alexhegit/ReachyMiniChat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
