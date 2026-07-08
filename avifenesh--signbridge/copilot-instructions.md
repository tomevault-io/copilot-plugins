## signbridge

> 1. **NEVER suggest AudioPlaybackCapture (Path A) as a solution.**

# SignBridge — AI Agent Context

## Critical Rules (Read First)

1. **NEVER suggest AudioPlaybackCapture (Path A) as a solution.**
   *WHY: Android OS hardcodes `USAGE_VOICE_COMMUNICATION` as non-capturable. Telegram uses this for all calls. Confirmed from AOSP source. No workaround exists.*

2. **ALWAYS use quaternion bone format for signs — never position-based.**
   *WHY: The app parser expects `"bones": { "name": { "rotation": [x,y,z,w] } }`. The old format `"joints": [x,y,z]` is pipeline-internal only (before IK conversion).*

3. **Use relay (Path B) as the audio source.** Default `useRelay=true` in `Preferences.kt`.
   *WHY: Path A is dead. Bot relay via pytgcalls is the only way to capture Telegram call audio.*

4. **Run `./gradlew testDebugUnitTest` after any Kotlin change.** 58 tests must stay green.

5. **Store all user preferences in `Preferences.kt` via SharedPreferences.** No server, no account.

---

## What This Project Is

An open source Android app giving deaf people real-time ASL signing during Telegram calls.
The hearing person speaks normally. A 3D avatar overlay signs everything said, with English subtitles.
Free, Apache 2.0, no account required.

## Stack

- **Android app**: Kotlin, OpenGL ES 2.0, ONNX Runtime, Vosk STT
- **Relay server**: Python, FastAPI, pytgcalls (joins Telegram calls as a user account)
- **ASL translation**: 3-tier engine — pattern hash → MiniLM vector similarity → grammar rules
- **Sign pipeline**: Python, MediaPipe, IK solver → quaternion keyframes

---

## Build & Test

```bash
# Android (requires ANDROID_HOME set)
./gradlew assembleDebug          # build APK
./gradlew testDebugUnitTest      # 58 unit tests — must pass

# Dictionary rebuild (after adding signs)
python3 -c "
from dictionary.build.common_signs import get_all_common_sign_entries
from dictionary.build.common_signs_extended import get_all_extended_entries
from dictionary.build.builder import build_dictionary
signs = list(get_all_common_sign_entries().values()) + list(get_all_extended_entries().values())
build_dictionary(signs)
"
python3 dictionary/build/convert_for_app.py \
  dictionary/output/dictionary.json \
  app/src/main/assets/dictionary/signs.json

# Relay server
cd relay && uvicorn main:app --host 0.0.0.0 --port 8080

# Python venv (for dictionary/translation pipelines)
source .venv/bin/activate
```

---

## Architecture (Key Facts)

```
Telegram call → relay bot (pytgcalls) → WebSocket PCM → Android phone
                                                              │
                                          ┌───────────────────┼──────────────────────┐
                                    Silero VAD          Vosk STT              ASL Translation
                                    (speech detect)     (on-device)           (3-tier engine)
                                                              │                      │
                                                        English text            ASL gloss
                                                              │                      │
                                                         subtitle bar        Sign Dictionary
                                                                                     │
                                                                            Avatar Renderer
                                                                           (OpenGL ES overlay)
```

**Relay joins as a Telegram user account.** Hearing person sees it as a call participant.

---

## Key Files

| File | Purpose |
|------|---------|
| `app/.../overlay/OverlayService.kt` | Main service — full pipeline orchestration |
| `app/.../stt/SttPipeline.kt` | audio → VAD → STT → translation → signs |
| `app/.../translation/AslTranslationEngine.kt` | 3-tier cascade + grammar rules |
| `app/.../renderer/AvatarRenderer.kt` | OpenGL ES LBS skinned mesh |
| `app/.../translation/SignDictionary.kt` | Loads quaternion keyframes from JSON |
| `app/.../ui/DemoActivity.kt` | **Demo mode** — type text → avatar signs (no relay needed) |
| `app/.../util/Preferences.kt` | All app settings (SharedPreferences, no server) |
| `relay/main.py` | Relay server API |
| `relay/bridge.py` | pytgcalls Telegram integration |
| `dictionary/build/common_signs_extended.py` | Add new ASL signs here |
| `dictionary/build/convert_for_app.py` | Position→quaternion IK converter |
| `agent4/asl_engine/engine.py` | Python ASL translation engine |

## Assets (bundled in APK)

| Asset | Content |
|-------|---------|
| `app/src/main/assets/dictionary/signs.json` | 115 signs in quaternion format |
| `app/src/main/assets/translation/patterns.json` | 224 ASL translation patterns |
| `app/src/main/assets/translation/vector_index.json` | 686 MiniLM-embedded vectors |
| `app/src/main/assets/models/avatar.glb` | 17KB procedural placeholder |
| `app/src/main/assets/shaders/avatar_vert.glsl` | LBS vertex shader (48 bones) |
| `app/src/main/assets/shaders/avatar_frag.glsl` | Diffuse + confidence glow |

---

## How To Add Signs

1. Add function in `dictionary/build/common_signs_extended.py`
2. Register in `EXTENDED_SIGNS` dict with gloss, category, duration
3. Rebuild dictionary + convert (see Build section)

Use helpers: `_make_frame(time_ms, hand_pose, rh_offset)` / `_make_two_hand_frame(...)`.
Hand poses are dicts of joint name → `[x, y, z]` — the IK converter handles position→quaternion.

## How To Add Translation Patterns

Edit `app/src/main/assets/translation/patterns.json`:
```json
{"english": "i want {THING}", "asl": "{THING} I WANT", "category": "request"}
```

Then rebuild the vector index:
```bash
source .venv/bin/activate
PYTHONPATH=. python3 agent4/scripts/build_index.py
cp agent4/data/pattern_embeddings.json app/src/main/assets/translation/vector_index.json
```

---

## Conventions

| Convention | Rule |
|------------|------|
| UI | Programmatic (no XML layouts, view binding off) |
| Preferences | `SharedPreferences` via `Preferences.kt` — no server |
| STT | Vosk on-device default; cloud optional via settings |
| Audio path | Path B relay (`useRelay=true`); Path A code kept but inactive |
| Tests | JUnit unit tests only; `isReturnDefaultValues=true` for Android mocks |
| Sign format | Quaternion keyframes in `signs.json`; never position-based |

---

## Current State

- Build: passing, 58 unit tests green, APK ~111MB debug
- Dictionary: 115 signs (79 content + 26 A-Z + 10 numbers)
- Translation: 224 patterns + 686 MiniLM vectors
- **Demo mode works without relay** — `DemoActivity` for standalone testing
- **Not yet tested on a real device**
- Relay: complete, not deployed (needs Telegram credentials from my.telegram.org)
- Avatar: procedural placeholder — needs 3D artist (48 joint names in `tools/generate_avatar_glb.py`)

---

## Docs

| Document | Content |
|----------|---------|
| `README.md` | User-facing overview + quick start |
| `HANDOVER.md` | Complete technical state — every file, every decision |
| `NEXT_TASKS.md` | Prioritized tasks with effort estimates |
| `SPEC.md` | Full product specification |
| `docs/ARCHITECTURE.md` | System design, data flow, rendering details |
| `docs/PLANS.md` | Roadmap, phase status, open questions |
| `docs/RELAY_SETUP.md` | Relay deployment guide |

---
> Source: [avifenesh/signbridge](https://github.com/avifenesh/signbridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
