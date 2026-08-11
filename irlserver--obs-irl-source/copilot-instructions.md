## obs-irl-source

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

IRL Source is a third-party OBS Studio plugin (C11, AGPL-3.0) for receiving live IRL streams over SRT, RTMP, or any FFmpeg-supported protocol. It solves IRL-specific problems: audio jitter buffering, PTS discontinuity repair, adaptive playback speed, keyframe gating, hardware-accelerated decoding, and mid-stream resolution changes.

## Build commands

The plugin statically links its own FFmpeg, libsrt and mbedTLS (see `deps/README.md`), so the first step on every platform is building that stack. It is incremental, so this is a one time cost per version bump.

### Linux

```bash
sudo apt install build-essential cmake pkg-config nasm libobs-dev libva-dev
./deps/build-deps.sh
cmake -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo
cmake --build build --parallel
./scripts/verify-plugin.sh build/obs-irl-source.so
```

### Windows (MSVC)

`deps/build-deps.sh` runs inside MSYS2 with the MSVC environment active (FFmpeg's configure needs a POSIX shell even when driving `cl.exe`). See the `windows-x64` job in `.github/workflows/build.yml` for the exact setup.

```powershell
cmake -B build -G "Visual Studio 18 2026" -A x64 -DOBS_SOURCE_DIR=obs-src
cmake --build build --config RelWithDebInfo
```

### macOS (Apple Silicon)

```bash
brew install cmake pkg-config nasm simde uthash jansson
./deps/build-deps.sh
cmake -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo -DOBS_SOURCE_DIR=$PWD/obs-src \
    -DCMAKE_DISABLE_FIND_PACKAGE_PkgConfig=ON
cmake --build build --parallel
./scripts/verify-plugin.sh build/obs-irl-source.so
```

Output: `build/obs-irl-source.so` (Linux/macOS) or `build/RelWithDebInfo/obs-irl-source.dll` (Windows).

`-DIRL_BUNDLED_FFMPEG=OFF` falls back to linking a system or obs-deps FFmpeg. That path still works for a quick compile check, but it reintroduces the per OBS line binding the bundled stack exists to remove, so it is not what releases use.

`scripts/verify-plugin.sh` is not optional polish. It asserts the two properties that make the bundled stack correct and that a successful compile does not prove: that the binary carries no `libav*` dependency, and that it exports nothing but `obs_module_*`. CI runs it (and a `dumpbin` equivalent on Windows) on every build.

There are no tests.

## Architecture

Single OBS MODULE shared library. All source is C11.

### Data flow

```
[receiver thread]: FFmpeg URL, demux, decode, PTS repair
  audio: resample, write to jitter buffer
  video: keyframe gate, push decoded frame (PTS in ns) onto video queue

[video thread]: pop video queue, HW frame transfer, format conversion,
                OBS async video output

[audio thread]: drain jitter buffer, speed correction, concealment,
                OBS audio output
```

### Audio output contract (verified against libobs source)

The audio core is built around three facts about libobs:

1. OBS timestamps must be contiguous (`ts[n+1] = ts[n] + frames/rate`). Deviations under 70ms are smoothed, 70ms to 2s gaps are zero filled by OBS (audible), larger jumps flush all queued audio. The plugin therefore derives timestamps from a pure sample counter anchored once at prime time and never jumps the clock outside declared restarts.
2. Changing `samples_per_sec` between submissions makes OBS destroy and recreate its per source resampler with no crossfade (a click per change). Playback speed is instead applied inside the plugin with a persistent swresample compensation, and the rate submitted to OBS never changes.
3. The OBS mixer consumes 21.3ms ticks against wall clock. A source whose queued audio runs dry gets a tick of silence plus a time shifted splice (crackle), and a source that falls behind the mix window causes OBS to permanently add global audio buffering. After priming, the pump always emits (real audio or shaped concealment silence) and keeps a fixed lead ahead of wall clock.

Buffer regulation happens through playback speed only, asymmetric like IRLToolkit's player: builds at an inaudible -2%, drains post-stall backlog at up to +5% (mild chipmunk). Content is never skipped once playback has primed. Backlog beyond a fill ceiling is pushed back into the transport by pausing the read loop (TCP/RTMP backpressure; SRT bounds itself via its latency window), and startup backlog is trimmed only before priming.

### Source files

- **`src/plugin.c`**: OBS module entry point. Registers `irl_source_info` with callbacks.
- **`src/irl-source.c`**: Source lifecycle (create, destroy, update, tick, activate/deactivate/show/hide). Loads config, manages threads, registers `proc_handler` for stats. `update` diffs the new settings against the live config: URL, FFmpeg Options, Hardware Decode, and Low Latency Audio are latched at stream open and force a reconnect, everything else is swapped in place under `audio_state_lock` so a settings tweak neither drops the connection nor clears the stats counters. Retuning Target Buffer live goes through `audio_buffer_resize`, which grows the ring (never shrinks it) and moves the watermarks while keeping every queued sample. `tick` also runs the one-shot fit-to-canvas: a source created without a URL (so, freshly added rather than restored from a scene collection) applies the same `obs_transform_info` as the frontend's Fit to Screen action to every scene item referencing it, once, as soon as the source reports a non-zero size. When "Close Stream When Inactive" is enabled, the show/activate callbacks start the receiver and hide/deactivate stop it; otherwise those callbacks are no-ops and the stream runs from create to destroy. Every "the stream stopped" clear — hide/deactivate, a restart-forcing settings edit, and the disconnect in `receiver-stream.c` — is gated on "Show Nothing When the Stream Ends" (`clear_on_disconnect`, on by default), the port of the media source's `clear_on_media_end`. Turning it off restores the old behavior of leaving the last decoded frame frozen on screen until the stream returns.
- **`src/receiver.c`**: thread entry points. The receiver thread runs the `av_read_frame()` loop, the audio thread runs the output pump.
- **`src/receiver-internal.h`**: internal declarations shared across the `receiver-*.c` translation units (stream open/close, packet/frame handlers, the audio pump, the video thread, timing-state resets). Not part of the public `include/` API.
- **`src/receiver-stream.c`**: stream open/close, demuxer options, reconnection, disconnect fade out, periodic stats logging.
- **`src/receiver-decode.c`**: packet to decoder plumbing with corruption burst handling and throttled decoder flushes.
- **`src/receiver-audio.c`**: the audio core. Intake side (receiver thread): PTS repair, resample to interleaved float, write to the PTS aware jitter buffer. Pre-keyframe audio is discarded (not staged) to avoid decoder warm-up artifacts. Output side (audio thread): sample counter output clock, constant rate submission, swr based speed correction, dropout concealment, hidden backlog trims.
- **`src/receiver-video.c`**: decoded video frame handling, keyframe gate, resolution change detection. Also owns `irl_video_request_clear`: the receiver thread drops the queue and raises a flag, and the *video* thread is what actually calls `obs_source_output_video(source, NULL)`. Clearing from the receiver thread instead would race a frame already inside the format conversion, which would repaint the frozen frame right after the clear.
- **`src/audio-buffer.c`**: thread safe ring buffer sized in milliseconds with a parallel PTS chunk queue. Mutex protected. Supports fade-out reads.
- **`src/video-handler.c`**: converts AVFrames to OBS video. Maps pixel formats (I420, NV12, I010, P010, etc.), handles HW frame transfer, falls back to swscale for unsupported formats. Maps video PTS through the audio playout offset for lip sync.
- **`src/pts-repair.c`**: three tier PTS discontinuity repair. Small gaps interpolated, medium gaps get silence, large gaps trigger full reset.
- **`src/settings.c`**: OBS properties UI and default values.
- **`src/websocket-vendor.c`**: obs-websocket vendor extension (`obs-irl-source`), registered from `obs_module_post_load` because module load order between plugins is undefined and obs-websocket publishes its global proc from its own `obs_module_load`. Serves `GetStats`, `GetSourceList` and `GetVersion`. It does not read `struct irl_source`: it resolves a source by name (or the only IRL source present), calls that source's existing `get_stats` proc_handler and copies the calldata into the response, so the websocket and script transports cannot drift apart and the locked snapshot stays in `irl-source.c`. There is no teardown — the API has no vendor-unregister call, and the request-unregister proc would run at module unload when obs-websocket may already be gone. Vendored header in `third_party/obs-websocket-api.h`; nothing links against obs-websocket, and everything degrades to a log line when it is absent.

### Headers (`include/`)

- **`irl-source.h`** — Central header. Defines `struct irl_source` (main context), `struct irl_config`, all `#define` defaults, and function declarations for every module.
- **`audio-buffer.h`** — `struct audio_buffer` and ring buffer API.
- **`pts-repair.h`** — `struct pts_repair`, `enum pts_action`, and repair API.
- **`irl-threading.h`** — the plugin's mutex/condvar/thread primitives (`irl_mutex_*`, `irl_cond_*`, `irl_thread_*`). Win32 primitives on Windows, pthreads elsewhere. Plugin code must never call `pthread_*` directly: librist's bundled `contrib/pthread-shim.c` defines external `pthread_*` symbols for MSVC and wins the link ahead of w32-pthreads, so a `pthread_mutex_init` call wrote a 40-byte `CRITICAL_SECTION` into an 8-byte w32-pthreads field and corrupted the surrounding struct. `scripts/verify-plugin.sh` fails the build if a direct `pthread_*` call reappears.

### Threading model

- **Main/OBS thread**: calls create, destroy, update, tick, get_properties, and the activate/deactivate/show/hide callbacks (used only when "Close Stream When Inactive" is on)
- **Receiver thread**: owns demux/decode FFmpeg state. Writes to the audio buffer (mutex protected) and pushes decoded video frames (PTS pre-converted to nanoseconds) onto the video queue. Never blocks on GPU or OBS video delivery.
- **Video thread**: pops the video queue, does the HW frame transfer and format conversion (owns sws_ctx), and calls `obs_source_output_video`. Queue overflow drops the oldest frame (`video_queue_drops`).
- **Audio thread**: drains the jitter buffer and submits audio to OBS via `obs_source_output_audio`, paced against the sample counter output clock. Shared timing state is protected by `audio_state_lock` (lock order: `audio_state_lock` before the buffer mutex).

Config fields marked `/* hot */` in `struct irl_config` are written by `irl_source_update` while the worker threads run, so every cross-thread read goes through `os_atomic_load_long` / `os_atomic_load_bool` (not C11 `_Atomic`, which MSVC does not support without an experimental flag). The remaining fields are only written while the threads are stopped, where `irl_thread_create` and `irl_thread_join` supply the happens-before edge.

### OBS API conventions

- Memory: use `bfree()`/`bstrdup()`/`bzalloc()` (OBS allocators), not stdlib malloc/free
- Logging: `blog(LOG_INFO, "[irl-source] ...")` — always use the `[irl-source]` prefix
- Stats are exposed via `proc_handler` ("get_stats" call) for Lua/Python script consumption, and over obs-websocket through the vendor extension. A new stat field belongs in three places: the proc declaration in `irl_source_create`, the `calldata_set_*` block above it, and `irl_stat_fields[]` in `src/websocket-vendor.c` (plus the table in README.md)
- Source flags: `OBS_SOURCE_AUDIO | OBS_SOURCE_ASYNC_VIDEO | OBS_SOURCE_DO_NOT_DUPLICATE`

## CI

GitHub Actions (`.github/workflows/build.yml`) builds on three platforms: Linux x64 (Ubuntu 26.04), Windows x64 (VS 2026), macOS ARM64 (macos-15). Every job builds the bundled media stack first (cached on the hash of `deps/versions.env` plus `deps/build-deps.sh`), then the plugin, then runs the isolation checks. The Windows and macOS jobs also clone OBS source and patch it to build only libobs, which is the plugin's one remaining link against a specific OBS release. The workflow exposes `workflow_call` so the release workflow reuses it.

Releases are tag driven (`.github/workflows/release.yml`, see `RELEASING.md`). Pushing `vX.Y.Z` verifies the tag against the `project(VERSION)` in CMakeLists.txt, runs the build workflow, repackages the artifacts into install layout archives (one per platform: Linux tar.gz, Windows zip, macOS zip), generates sha256sums.txt, and creates a draft GitHub release whose body is `.github/release-notes-header.md` plus a changelog. Publishing the draft is manual, after testing the artifacts.

`scripts/changelog.sh` builds that changelog by grouping the commits since the previous `v*` tag on their conventional commit type (breaking, feat, fix, perf, refactor, docs, build/ci, everything else). It replaced GitHub's `generate-notes`, which only reports pull requests and therefore missed every commit pushed straight to `master`. Commit subjects are the release notes, so write them as such. Run `scripts/changelog.sh HEAD` to preview.

### One artifact per platform

There used to be a `matrix.include` over OBS lines, producing `-obs32.1` and `-obs32.2` binaries. That existed purely because the plugin dynamically linked obs-deps' FFmpeg, and OBS bumped FFmpeg 7 (`avcodec-61`) to 8.1 (`avcodec-62`) between those lines, so a binary linked against one would not load where the other was present. Bundling FFmpeg statically removed that constraint and the matrix with it.

libobs itself was never the problem. `obs_init_module` gates a plugin on `(mod.ver() & 0xFFFF0000) <= LIBOBS_API_VER`, so major and minor only, and it looks up nothing but `obs_module_*` symbols. Building against the oldest supported line therefore yields one binary that loads on that line and every newer one. `OBS_VERSION` and `OBS_DEPS_VERSION` at the top of `build.yml` pin that oldest line; raise them only to drop support for older OBS releases, never to chase a newer one.

`OBS_DEPS_VERSION` still exists because libobs needs obs-deps to build. The plugin no longer touches it.

## Contributing

If you wish to contribute PRs to this project, please understand what you are changing. You should be able to write any replies to reviews/PRs yourself — don't copy and paste replies directly from AI.

The initial version of this plugin was heavily built with LLM assistance. The author (datagutt) has experience with video and SRT(LA) protocols but is less familiar with C and the OBS Studio codebase. Tagged releases are fully tested; individual commits may not be.

## Other files

- **`irl-stats.lua`** - Example OBS Lua script that reads plugin stats via proc_handler and updates a text source overlay.
- **`third_party/`** - Verbatim copies of files from other projects, under their own licenses. Currently just `obs-websocket-api.h`. See `third_party/README.md` for provenance and how to update it.
- **`docs/audio-pipeline.md`** - Deep dive on the buffered vs low-latency audio paths, jitter buffer, adaptive latency control, PTS repair tiers, and timestamp handling.
- **`docs/viewer-quality-plan.md`** - The viewer-quality policy and the recovery/diagnostics behavior that implements it (what stats to watch and what healthy looks like).
- **`AGENTS.md`**, **`GEMINI.md`** - Symlinks to this file (`CLAUDE.md`).

---
> Source: [irlserver/obs-irl-source](https://github.com/irlserver/obs-irl-source) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
