## proctap

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ProcTap is a cross-platform Python library for capturing audio from specific processes. It provides platform-optimized backends for Windows, Linux, and macOS.

**Platform Support:**
- **Windows**: ✅ Fully implemented using WASAPI Process Loopback (C++ native extension)
- **Linux**: ✅ Fully implemented - PipeWire Native/PulseAudio (per-process isolation, v0.3.0+)
- **macOS**: ✅ **Officially supported** - ScreenCaptureKit (macOS 13+, bundleID-based)

**Key Characteristics:**
- Per-process audio isolation (not system-wide)
  - Windows/Linux: PID-based capture
  - macOS: bundleID-based capture (ScreenCaptureKit)
- Low-latency streaming (10-15ms on macOS, 10ms on Windows, 2-5ms on Linux with PipeWire Native)
- Platform-specific implementations:
  - Windows: WASAPI C++ extension (Windows 10 20H1+)
  - Linux: PipeWire Native API / PulseAudio (fully supported, v0.3.0+)
  - **macOS: ScreenCaptureKit Swift helper (macOS 13+) - RECOMMENDED**
- Dual API: callback-based and async iterator patterns

## Development Guidelines

### Test File Organization

**IMPORTANT:** Use `.claude_test/` directory for all temporary and experimental test files:

**Required Usage of `.claude_test/`:**
- All temporary test scripts (for quick testing/debugging)
- Experimental or throw-away code
- Test audio files and sample data
- Any files created for verification purposes
- Example/demo files used during development

**Do NOT use `.claude_test/` for:**
- Official test suite files (use `tests/` directory)
- Production examples (use `examples/` directory)
- Production code (use `src/` directory)

**Cleanup:**
- `.claude_test/` is gitignored
- Clean up files in `.claude_test/` after completing tests
- Only commit to `tests/`, `examples/`, or `src/` when ready for production

### Testing Standards

**IMPORTANT:** When creating official test code, ALWAYS follow pytest conventions:
- Use pytest framework for all tests
- Place tests in `tests/` directory or name files with `test_*.py` pattern
- Use pytest fixtures, parametrize, and markers
- Follow pytest discovery conventions
- Use `.claude_test/` for experimental scripts before moving to `tests/`

### Setup and Building

```bash
# Install in editable mode with dev dependencies
pip install -e ".[dev]"

# Build wheel (requires Visual Studio Build Tools + Windows SDK)
python -m build --wheel

# Build source distribution
python -m build
```

**Important:** After modifying C++ code in [_native.cpp](src/proctap/_native.cpp), you must rebuild:
```bash
pip install -e . --force-reinstall --no-deps
```

### Testing and Type Checking

```bash
# Run tests
pytest

# Type check
mypy src/
```

### Running Examples

```bash
# Windows example
python examples/windows_basic.py --pid 12345 --output audio.wav
python examples/windows_basic.py --name "VRChat.exe" --output audio.wav

# Linux example (requires pulseaudio-utils)
python examples/linux_basic.py --pid 12345 --duration 5 --output output.wav

# macOS example (requires macOS 14.4+, PyObjC)
python examples/macos_basic.py --pid 12345 --duration 5 --output output.wav
```

### CLI Usage (Pipe to FFmpeg)

The package installs a `proctap` command for direct use:

```bash
# Direct command (recommended)
proctap --pid 12345 --stdout | ffmpeg -f s16le -ar 48000 -ac 2 -i pipe:0 output.mp3

# Or using python -m (alternative)
python -m proctap --pid 12345 --stdout | ffmpeg -f s16le -ar 48000 -ac 2 -i pipe:0 output.mp3

# Using process name instead of PID
proctap --name "VRChat.exe" --stdout | ffmpeg -f s16le -ar 48000 -ac 2 -i pipe:0 output.mp3

# Low-latency mode with fast resampling (for real-time streaming)
proctap --pid 12345 --resample-quality fast --stdout | ffmpeg -f s16le -ar 48000 -ac 2 -i pipe:0 output.mp3

# Available quality modes:
# --resample-quality best   (highest quality, ~1.3-1.4ms latency, default)
# --resample-quality medium (medium quality, ~0.7-0.9ms latency)
# --resample-quality fast   (lowest quality, ~0.3-0.5ms latency)
```

### macOS Setup

**Recommended: ScreenCaptureKit Backend (macOS 13+)**

```bash
# Build Swift helper binary
cd src/proctap/swift/screencapture-audio
swift build -c release

# Enable Screen Recording permission
# System Settings → Privacy & Security → Screen Recording → Enable for Terminal/IDE

# Test
python examples/macos_screencapture_test.py --bundle-id com.apple.Safari --duration 5
```

**Fallback: PyObjC Backend (Experimental, macOS 14.4+)**

```bash
# Install PyObjC dependencies
pip install pyobjc-core pyobjc-framework-CoreAudio

# Or install with optional dependencies
pip install -e ".[macos]"

# Note: PyObjC backend has IOProc callback issues and is not recommended
```

## Architecture

### Multi-Platform Backend Architecture

The library uses platform-specific backends selected at runtime:

```
ProcTap (core.py - Public API)
    ↓
backends/__init__.py (Platform Detection)
    ↓
┌─────────────────┬──────────────────┬──────────────────────────┐
│ Windows         │ Linux            │ macOS                    │
│ (Implemented)   │ (Implemented)    │ (Implemented)            │
├─────────────────┼──────────────────┼──────────────────────────┤
│ WindowsBackend  │ LinuxBackend     │ ScreenCaptureBackend     │
│ └─ _native.cpp  │ └─ PulseAudio/   │ └─ Swift CLI Helper      │
│    (WASAPI)     │    PipeWire      │    (ScreenCaptureKit)    │
└─────────────────┴──────────────────┴──────────────────────────┘
```

**Backend Selection** ([backends/__init__.py](src/proctap/backends/__init__.py)):
- Automatic platform detection using `platform.system()`
- Windows: Uses native C++ extension with WASAPI
- Linux: PulseAudio/PipeWire backend with multiple strategies
- **macOS: ScreenCaptureKit Swift helper (macOS 13+) - RECOMMENDED**

**Windows Backend** ([backends/windows.py](src/proctap/backends/windows.py)):
- Wraps `_native.cpp` C++ extension
- Per-process audio capture requires `ActivateAudioInterfaceAsync` (Windows 10 20H1+)
- Native WASAPI format: 48kHz, 2ch, float32 (IEEE 754) - optimal quality
- Fallback: 44.1kHz, 2ch, 16-bit PCM if float32 initialization fails
- **Audio Format Conversion** ([backends/converter.py](src/proctap/backends/converter.py)):
  - Python-based audio format conversion using scipy/numpy
  - Supports sample rate conversion (resampling)
  - Supports channel conversion (mono ↔ stereo)
  - Supports bit depth conversion (8/16/24/32-bit, int/float)
  - Automatically converts WASAPI output to match `StreamConfig`
  - No conversion overhead if formats already match

**Linux Backend** ([backends/linux.py](src/proctap/backends/linux.py)):
- ✅ Fully implemented with multiple strategies (v0.3.0+)
- **PipeWire Native API** ([backends/pipewire_native.py](src/proctap/backends/pipewire_native.py)):
  - 🚧 In development: Core functionality implemented, integration ongoing
  - Target latency: ~2-5ms (vs ~10-20ms subprocess-based)
  - Direct C API bindings via ctypes
  - Auto-selected when available
- **Strategy Pattern:** PipeWire Native → PipeWire subprocess (`pw-record`) → PulseAudio (`parec`)
- **Per-process Isolation:** True isolation using null-sink strategy
- Uses `pulsectl` library for stream management
- Requires: System-dependent (libpipewire-0.3-dev for native, pw-record or parec for subprocess)

**macOS Backend** ([backends/macos_screencapture.py](src/proctap/backends/macos_screencapture.py)):
- ✅ **RECOMMENDED** - ScreenCaptureKit API (macOS 13+, bundleID-based)
- Uses Swift CLI helper subprocess for audio capture
- **Advantages:**
  - Apple Silicon compatible (no AMFI/SIP hacks needed)
  - Simple TCC permissions (Screen Recording only)
  - Stable Apple official API
  - No Developer ID code signing required
  - Low latency (~10-15ms)
- **Requirements:**
  - macOS 13.0 (Ventura) or later
  - Swift helper binary: `cd src/proctap/swift/screencapture-audio && swift build`
  - Screen Recording permission (System Settings → Privacy & Security)
- **Implementation:**
  - Swift CLI helper (`screencapture-audio`) captures via ScreenCaptureKit
  - Python backend manages subprocess and PCM streaming
  - PID → bundleID translation using `lsappinfo`
  - See [backends/macos_screencapture.py](src/proctap/backends/macos_screencapture.py)
  - See [swift/screencapture-audio/](src/proctap/swift/screencapture-audio/) for Swift implementation

**Experimental/Archived Backends**:
- PyObjC backend: [backends/macos_pyobjc.py](src/proctap/backends/macos_pyobjc.py) - IOProc callback issues (Fallback only)
- Archived experimental backends: [archive/experimental-backends/](archive/experimental-backends/) - Process Tap implementations (AMFI limitations)
- Process Tap investigation: [archive/apple-silicon-investigation-20251120/](archive/apple-silicon-investigation-20251120/) - AMFI limitations on Apple Silicon

### Threading Model

Audio capture runs on a background thread to prevent blocking:

1. **Worker Thread**: Reads from WASAPI capture buffer continuously
2. **Main Thread**: Receives data via callbacks or async queue
3. **Synchronization**: Thread-safe queue for async iteration, direct callbacks for callback mode

**Data Flow:**
```
Audio Source (Process-specific)
  → Platform Backend (WASAPI/PulseAudio/CoreAudio)
  → Worker Thread
  → Queue/Callback
  → User Code
```

### Key Components

**[core.py](src/proctap/core.py)** - Main API surface:
- `ProcessAudioCapture`: User-facing class with two operation modes:
  - Callback mode: `start(on_data=callback)`
  - Async mode: `async for chunk in tap.iter_chunks()`
  - Uses platform-specific backend via `get_backend()`
  - Accepts `resample_quality` parameter for controlling resampling performance:
    - `'best'`: Highest quality, ~1.3-1.4ms latency (default)
    - `'medium'`: Medium quality, ~0.7-0.9ms latency
    - `'fast'`: Lowest quality, ~0.3-0.5ms latency

**[backends/](src/proctap/backends/)** - Platform-specific implementations:
- `base.py`: `AudioBackend` abstract base class
- `windows.py`: Windows implementation (wraps `_native.cpp` + format conversion)
- `linux.py`: Linux PipeWire/PulseAudio implementation (fully supported, v0.3.0+)
- `pipewire_native.py`: Native PipeWire API bindings (in development)
- `macos_screencapture.py`: macOS ScreenCaptureKit backend (recommended, macOS 13+)
- `macos_pyobjc.py`: macOS PyObjC fallback backend (experimental, has IOProc issues)
- `converter.py`: Audio format converter (sample rate, channels, bit depth)

**[_native.cpp](src/proctap/_native.cpp)** - Windows C++ Extension:
- `ProcessLoopback` class: WASAPI capture implementation
- Uses `ActivateAudioInterfaceAsync` for process-specific capture
- COM/WRL integration with proper apartment threading
- Exposes methods: `start()`, `stop()`, `read()`, `get_format()`

## Build System Details

**Platform-Specific Builds:**

The build system ([setup.py](setup.py)) automatically detects the platform and builds appropriate extensions:

**Windows Build Requirements:**
- Visual Studio Build Tools (MSVC compiler)
- Windows SDK
- Python 3.10+ (supports 3.10, 3.11, 3.12, 3.13)
- C++20 compiler (`/std:c++20` flag)

**Windows Linked Libraries:**
- `ole32`: COM infrastructure
- `uuid`: GUID support
- `propsys`: Property system

**Extension Module:** Builds as `_native.cp3XX-win_amd64.pyd` (Windows only)

**Linux Builds:**
- No C++ extension required (pure Python)
- Multiple backend strategies:
  - PipeWire Native: `libpipewire-0.3-dev` (optional, for ultra-low latency)
  - PipeWire subprocess: `pw-record` from `pipewire-utils`
  - PulseAudio: `parec` from `pulseaudio-utils`
- Python dependencies: `pulsectl` library (automatically installed)

**macOS Builds:**
- Primary: ScreenCaptureKit backend via Swift CLI helper (automatic build)
  - Swift helper built during `pip install` if Swift toolchain available
  - Bundled binary included in package distribution
- Fallback: PyObjC backend (no compilation needed)
  - PyObjC dependencies installed automatically on macOS via environment markers
- **Archived experimental backends** in `archive/experimental-backends/` (not used in production)

## Python Dependencies

**Runtime:**
- **Core**:
  - `numpy>=1.20.0` - Array operations for audio processing
  - `scipy>=1.7.0` - Signal processing (fallback resampling)
- **Optional**:
  - `samplerate>=0.1.0` - Professional-grade audio resampling (libsamplerate)
    - Install with: `pip install proc-tap[hq-resample]`
    - **Note**: May fail to build on some platforms (Windows with Python 3.13+)
    - Falls back to scipy if not available
    - Supports three quality modes via `resample_quality` parameter:
      - `'best'`: Uses `sinc_best` converter (default)
      - `'medium'`: Uses `sinc_medium` converter
      - `'fast'`: Uses `sinc_fastest` converter
- **Windows**: Uses native C++ extension + Python format conversion
- **Linux**: `pulsectl>=23.5.0` (automatically installed via environment markers in pyproject.toml)
- **macOS**: `pyobjc-core>=9.0`, `pyobjc-framework-CoreAudio>=9.0` (automatically installed via environment markers in pyproject.toml)

**System Dependencies (Linux only):**
- One of the following (auto-detected with graceful fallback):
  - **PipeWire Native** (recommended): `libpipewire-0.3-dev`
  - **PipeWire subprocess**: `pw-record` from `pipewire-utils`
  - **PulseAudio**: `parec` from `pulseaudio-utils`
- PulseAudio or PipeWire with pulseaudio-compat

**Examples:**
- `psutil`: Used in examples for process name → PID resolution

**Development:**
- `pytest`: Test framework
- `mypy`: Type checking
- `types-setuptools`, `types-psutil`, `scipy-stubs`: Type stubs for type checking

**Contrib (optional):**
- `faster-whisper>=1.0.0`: For real-time transcription features
  - Install with: `pip install proc-tap[contrib]`

## Audio Format

**Windows Backend:**

The Windows native extension attempts 48kHz float32 first, with fallback to 44.1kHz int16 ([_native.cpp:329-365](src/proctap/_native.cpp#L329-L365)):

**Primary Format (Preferred):**
- **Sample Rate:** 48,000 Hz
- **Channels:** 2 (stereo)
- **Bits per Sample:** 32-bit
- **Format:** IEEE float (WAVE_FORMAT_IEEE_FLOAT)
- **Value Range:** -1.0 to +1.0 (normalized)
- **Block Align:** 8 bytes (2 channels × 32 bits / 8)
- **Byte Rate:** 384,000 bytes/sec

**Fallback Format (If float32 fails):**
- **Sample Rate:** 44,100 Hz (CD quality)
- **Channels:** 2 (stereo)
- **Bits per Sample:** 16-bit
- **Format:** PCM (WAVE_FORMAT_PCM)
- **Block Align:** 4 bytes (2 channels × 16 bits / 8)
- **Byte Rate:** 176,400 bytes/sec

**Format Behavior:**

The Windows backend **always returns 48kHz float32** to user code:

- **If native is 48kHz float32**: No conversion (zero overhead)
- **If fallback to 44.1kHz int16**: Automatically converts to 48kHz float32 using AudioConverter

This ensures consistent output format regardless of which WASAPI format succeeds.

**Important Notes:**
- `StreamConfig` has been deprecated and removed
- All backends now return their native high-quality format
- Windows: 48kHz float32
- Linux: 44.1kHz int16 (configurable)
- macOS: 48kHz int16 (configurable)

**For WAV file output:**

Users must convert float32 to int16 for standard WAV files:

```python
import numpy as np

def on_data(pcm: bytes, frames: int):
    # Convert float32 to int16
    float_samples = np.frombuffer(pcm, dtype=np.float32)
    int16_samples = (np.clip(float_samples, -1.0, 1.0) * 32767).astype(np.int16)
    wav.writeframes(int16_samples.tobytes())
```

**Linux Backend:**

The PulseAudio backend default format:
- Default: 44,100 Hz, 2 channels, 16-bit PCM
- Returns raw int16 PCM data

**macOS Backend:**

The Core Audio Process Tap backend default format:
- Default: 48,000 Hz, 2 channels, 16-bit PCM
- Returns raw int16 PCM data

Raw PCM data is returned as `bytes` to user callbacks/iterators.

## Known Issues and TODOs

**Windows Backend:**
1. ✅ **LOOPBACK Format Detection** - COMPLETED
   - LOOPBACKモードでは実際のミックスフォーマットが指定と異なる可能性がある
   - `Initialize()`成功後、`GetMixFormat()`で実際のフォーマットを取得
   - 実際のフォーマットと指定が異なる場合は`m_waveFormat`を更新
   - Python側の`AudioConverter`が自動的に48kHz float32に変換
   - 詳細ログで実際のフォーマットを確認可能（OutputDebugString）

2. **Frame Count Calculation** ([core.py:207](src/proctap/core.py#L207)):
   - Currently returns `-1` for frame count in callbacks
   - TODO: Calculate from backend format info (needs to account for conversion)

3. **Buffer Size Control** ([core.py:29](src/proctap/core.py#L29)):
   - `buffer_ms` parameter exists but note indicates limited control

**Linux Backend:**
1. **Native PipeWire API Implementation** ([backends/pipewire_native.py](src/proctap/backends/pipewire_native.py)):
   - 🚧 IN DEVELOPMENT (v0.3.0+):
     * ✅ Core API bindings (pw_init, pw_main_loop, pw_context, pw_stream)
     * ✅ Stream capture framework (pw_stream_new_simple, dequeue/queue buffers)
     * ✅ Registry API for node discovery
     * ✅ Basic thread management
     * ⚠️  Incomplete: SPA POD format parameters optimization
     * ⚠️  Integration with LinuxBackend: Experimental, may fall back to subprocess
   - ✅ Testing: Basic unit tests and examples added

2. **Cross-distribution Testing** (Ongoing):
   - Verify on Ubuntu, Fedora, Arch Linux, Debian
   - Test with various PipeWire and PulseAudio versions
   - Validate fallback behavior

**macOS Backend:**
1. **ScreenCaptureKit Backend** ([backends/macos_screencapture.py](src/proctap/backends/macos_screencapture.py)):
   - ✅ Production-ready (macOS 13+)
   - ✅ BundleID-based capture
   - ✅ Swift CLI helper with automatic build
   - ✅ Screen Recording permission handling
   - TODO: Improve error messages for common TCC permission failures
   - TODO: Add universal binary support for Swift helper

2. **PyObjC Fallback Backend** ([backends/macos_pyobjc.py](src/proctap/backends/macos_pyobjc.py)):
   - ⚠️  Experimental fallback only
   - ⚠️  IOProc callback reliability issues
   - TODO: Investigate callback timing issues on different macOS versions

**General:**
1. **Test Coverage:**
   - ✅ Audio format converter tests added ([test_converter.py](test_converter.py))
   - TODO: Add platform-specific backend tests
   - TODO: Add integration tests for ProcessAudioCapture with real processes

## CI/CD Workflows

GitHub Actions workflows in [.github/workflows/](.github/workflows/):

- **[build-wheels.yml](.github/workflows/build-wheels.yml)**: Multi-platform wheel builds
  - Builds for Windows, Linux, macOS
  - Python versions: 3.10, 3.11, 3.12, 3.13
  - Platform-specific setup: PulseAudio (Linux), Swift verification (macOS)

- **[publish-pypi.yml](.github/workflows/publish-pypi.yml)**: PyPI release workflow
  - Builds wheels for all platforms
  - Merges artifacts from multiple runners
  - Manual trigger with git tag input

- **[release-testpypi.yml](.github/workflows/release-testpypi.yml)**: TestPyPI releases
  - Triggered on version tags (v*.*.*)
  - Multi-platform wheel generation
  - Automatic upload to TestPyPI

**Platform-Specific Build Steps:**
- **Windows**: C++ extension compilation (Visual Studio Build Tools)
- **Linux**: PulseAudio system package installation
- **macOS**: Swift CLI helper compilation (SwiftPM)

---
> Source: [m96-chan/ProcTap](https://github.com/m96-chan/ProcTap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
