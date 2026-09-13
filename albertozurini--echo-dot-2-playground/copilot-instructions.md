## echo-dot-2-playground

> This file serves as a brain dump and knowledge base for the `echo-dot-assistant` app development.

# Echo Dot Assistant - Knowledge Base & Development Notes

This file serves as a brain dump and knowledge base for the `echo-dot-assistant` app development.

## Device Constraints
- **Hardware:** Amazon Echo Dot 2nd Gen (Biscuit).
- **OS:** Android 5.1 (API Level 22). 
- **Architecture:** `armeabi-v7a`.
- **Storage:** Very limited internal storage (`/data` gets full fast). Large assets (models, binaries) MUST be pushed to `/cache`.
- **Memory:** ~1GB RAM. Models must be kept extremely small (e.g., ~125M parameters, quantized to Q4).

## Architecture
The assistant is implemented as an Android Service (`AudioService.java`). 
It handles:
1. **Wake Word Detection:** Continuously parsing logcat for the native `CustomAssistant` pattern or Amazon's underlying wake-word logs.
2. **STT (Speech-to-Text):** Powered by `Sherpa-ONNX`. The model used is Zipformer (small).
3. **LLM (Language Model):** Powered by a natively compiled `llama-cli` binary. The model is `MobileLLM-125M-Q4_K_M`.

## Native Compilation Hurdles (Android NDK)
Due to Android 5.1 running an outdated linker, binaries and shared libraries compiled with newer NDKs (27+) will crash with `SIGSEGV` or `CANNOT LINK EXECUTABLE DEPENDENCIES` if not compiled carefully.

1. **DT_HASH vs DT_GNU_HASH:** Android 5.1 requires `DT_HASH`. Modern NDKs default to `DT_GNU_HASH`. You **must** pass `-Wl,--hash-style=both` to the linker (LDFLAGS) for both `Sherpa-ONNX` and `llama.cpp`.
2. **Unsupported POSIX Calls:** Functions like `posix_madvise` are not implemented in Android 5.1's Bionic libc. In `llama.cpp` (`llama-mmap.cpp`), these calls must be bypassed/commented out.
3. **FP16 Intrinsics:** The `armeabi-v7a` CPU on the Echo Dot lacks full FP16 intrinsic support. Compiling `llama.cpp` requires patching or emptying `sgemm.cpp` if compiler errors are thrown.
4. **OpenMP:** If `llama.cpp` is built with CMake, it relies on `libomp.so`. This library must be extracted from the NDK toolchain (`llvm/prebuilt/.../lib/linux/arm/libomp.so`) and pushed to the device (`/cache`), and `LD_LIBRARY_PATH=/cache` must be set when executing `llama-cli`.

## Pipeline Modes (Testing)
Because testing microphone features directly on the device is difficult when developing, `AudioService.java` supports `TEST_MODE` intents via ADB.

### 1. STT Test Mode
Triggers the Sherpa-ONNX initialization and `AudioRecord` listening phase.
```bash
adb shell am startservice -n com.custom.assistant/.AudioService --es TEST_MODE STT
```

### 2. LLM Test Mode
Bypasses the microphone and directly sends text to the LLM to benchmark inference speed and action parsing.
```bash
adb shell 'am startservice -n com.custom.assistant/.AudioService --es TEST_MODE LLM --es TEST_TEXT "turn on the light"'
```

### LLM Interactive Mode Issues
Modern `llama.cpp`'s `llama-cli` automatically defaults to interactive mode if a chat template is detected. This causes it to hang indefinitely waiting for input. To prevent this, the `-st` (single-turn) flag must be passed in the `ProcessBuilder` arguments.

## Caching & Performance
- **STT:** Inference is very fast and stays ahead of real-time speech.
- **LLM:** Execution takes ~15-20 seconds for a simple command because the model weights and prompt prefill are calculated from scratch on every run. 
  - *Optimization Strategy:* Run `llama-server` in the background indefinitely to keep the KV cache loaded in RAM. `llama-cli` caching (`--prompt-cache`) is not natively supported anymore for instruct models.

## Deployment
Use the `make deploy` command to build the `.apk` and push it to `/cache` (bypassing `/data` size limits) before installing it via `pm install`.

---
> Source: [albertoZurini/echo-dot-2-playground](https://github.com/albertoZurini/echo-dot-2-playground) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
