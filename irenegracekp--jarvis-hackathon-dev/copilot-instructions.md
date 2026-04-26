## jarvis-hackathon-dev

> An AI-powered Reachy Mini robot demo running on Jetson Orin. The robot ("The Witness") observes people, holds conversations, controls desktop apps via gestures, and reacts with emotions, lighting, and head/body movement.

# Jarvis - Reachy Mini Hackathon Robot

An AI-powered Reachy Mini robot demo running on Jetson Orin. The robot ("The Witness") observes people, holds conversations, controls desktop apps via gestures, and reacts with emotions, lighting, and head/body movement.

## Platform

- **Hardware:** Jetson Orin Nano, Reachy Mini robot, ZED 2 stereo camera, Hollyland wireless mic, Bluetooth speaker
- **OS:** JetPack 6 (L4T R36.4.3), CUDA 12.6, TensorRT 10.3
- **Python env:** Conda `trt_pose` env has torch 2.5 (CUDA), torchvision 0.19, OpenCV 4.8, ultralytics
- **Run with:** `/home/orin/miniconda3/envs/trt_pose/bin/python` (needs `LD_LIBRARY_PATH` for cusparselt, see below)
- **Docker alternative:** `run_docker.sh` runs `witness:latest` container with NVIDIA runtime, host network, device access

```bash
# Required env for running outside Docker:
export LD_LIBRARY_PATH="/home/orin/miniconda3/envs/trt_pose/lib/python3.10/site-packages/nvidia/cusparselt/lib:${LD_LIBRARY_PATH}"
```

## Architecture Overview

```
                    main.py (orchestrator)
                    State machine: IDLE → AMBIENT → ENGAGED
                    4 threads: listen, brain, output, gesture
                         │
    ┌────────────────────┼────────────────────────┐
    │                    │                         │
 INPUT               COGNITION                 OUTPUT
 ─────               ─────────                 ──────
 listen.py            brain.py                  speak.py
  (mic→ASR)            (LLM orchestration)       (Piper TTS)
 vision.py            memory.py                 reachy_bridge.py
  (camera+VLM)         (person facts store)       (head/body/antenna)
 faces.py             openclaw_bridge.py        hue.py
  (YuNet+SFace)        (desktop commands)         (Philips Hue lights)
 gestures.py          llm_proxy.py              actions.py
  (hand pose)          (Agora LLM proxy)          (gesture→keyboard)
 person_tracker.py                              dashboard.py
  (YOLOv8-pose+ReID)                              (MJPEG web UI)
```

## main.py - The Orchestrator

Entry point. Runs 4 daemon threads coordinated through a shared `State` object and queues.

### State Machine
- **IDLE** - No faces detected. Waiting.
- **AMBIENT** - Face(s) present. Robot observes, greets arrivals (after 3s confirmation), makes periodic ambient comments.
- **ENGAGED** - Active conversation locked to one person (closest to center). Speech routed to LLM, responses spoken via TTS.

### Threads
1. **listen_loop** - Mic → faster-whisper (local) or OpenAI Whisper API (cloud) → `audio_queue`. Ignores speech while TTS is playing (prevents feedback loop).
2. **brain_loop** - Polls `audio_queue` for speech. Grabs camera frame, runs face events (arrivals/departures), gets scene description from VLM, builds memory context, calls LLM, puts response on `response_queue`. Also handles: goodbye detection, OpenClaw command routing, silence timeout (→ ambient), face departure (→ ambient).
3. **output_loop** - Reads `response_queue`. Executes robot actions (emotion, head, antenna). Sets Hue light emotion. Speaks response via TTS (blocking).
4. **gesture_loop** - Independent. Camera → hand pose → gesture classification → xdotool desktop actions.

### Key CLI Flags
```
--no-vlm          Skip vision-language model
--no-tts          Print instead of speak
--no-robot        Skip robot commands (default: True)
--no-listen       Skip mic input
--no-gestures     Skip hand gesture recognition
--no-openclaw     Skip desktop command routing
--no-hue          Skip Philips Hue lights
--agora           Use Agora cloud ASR+LLM+TTS (replaces local listen/output)
--llm-backend     local | openai | openrouter
--asr-backend     local | cloud
--vlm-backend     transformers | llama_cpp
--ambient-interval   Seconds between ambient lines (default: 30)
--silence-timeout    Seconds before engaged → ambient (default: 30)
--departure-buffer   Seconds face must be gone before counting as departed (default: 10)
```

### Two Modes of Operation
1. **Local mode** (default): listen_loop + brain_loop + output_loop + gesture_loop + dashboard on :8080
2. **Agora mode** (`--agora`): Agora web server on :8080, browser handles RTC voice. Agent manages cloud ASR+LLM+TTS. Brain loop still runs for face detection. Optional MCP server for OpenClaw tool calling.

## Pipeline Modules

### listen.py - Audio Input / ASR
- Captures from Hollyland mic via `sounddevice` (48kHz → resample to 16kHz)
- VAD for speech/silence detection
- Backends: `faster_whisper` (local, GPU) or OpenAI Whisper API
- Output: `text_queue` of (text, timestamp) tuples

### speak.py - TTS Output
- `SpeakPipeline`: Piper TTS CLI → WAV → `paplay` (PulseAudio, routes to Bluetooth speaker)
- `DummySpeakPipeline`: prints text (for `--no-tts`)
- `say_blocking(text)` blocks until audio finishes playing

### brain.py - LLM Orchestration
- Maintains per-face conversation history
- Loads system prompt from `config/system_prompt.txt`
- `engage(text, scene_desc, face_id, memory_context)` → JSON response
- `greet(face_id, person_info, scene_desc)` → greeting response
- `ambient_react(context)` → random ambient line from `config/ambient_lines.json`
- LLM response format: `{speech, emotion, head_direction, antenna_state, save_memory}`
- Backends: local GGUF (llama-cpp-python), OpenAI (GPT-4o-mini), OpenRouter (Claude Sonnet)

### vision.py - Camera + VLM
- `VisionPipeline`: opens ZED camera (left half of stereo frame when width > 2000)
- `grab_frame()` → BGR numpy array
- `detect_faces(frame)` → Haar cascade fallback
- `identify_faces(frame)` → YuNet + SFace (delegates to `faces.py`)
- `get_face_events(frame)` → `{arrivals, departures, present}` with stable ID tracking via bbox IoU
- `get_scene_description(frame)` → VLM text (SmolVLM2-256M via transformers, or simple fallback)

### faces.py - Face Detection + Recognition (OpenCV)
- `FaceRecognizer`: YuNet detection + SFace recognition
- `detect_and_identify(frame)` → list of `{face_id, bbox, confidence, is_known, embedding}`
- `enroll(face_id, embedding)` → persist to `data/known_faces/embeddings.json`
- Cosine threshold: 0.30
- `get_closest_to_center(faces, frame_width)` → face nearest to horizontal center

### faces_trt.py - TensorRT Face Pipeline
- `FaceRecognizerTRT`: same API as `FaceRecognizer` but with TensorRT engines
- Falls back to OpenCV if TRT engines not found
- Engines: `models/trt/yunet_640x480_fp16.engine`, `models/trt/sface_fp16.engine`

### person_tracker.py - Person Re-ID + Hand Raise Detection
- `PersonTracker`: YOLOv8n-pose (detection + keypoints) + ResNet18 (re-ID embeddings)
- State machine: SCANNING → TRACKING → LOST
- Hand raise detection: wrist keypoint above shoulder keypoint (COCO indices 9/10 vs 5/6)
- Re-ID: 512-d ResNet18 crop embeddings, cosine similarity matching, EMA-updated target embedding
- `process_frame(frame)` → `{state, target_bbox, target_center, all_persons, hand_raised_idx}`

### gestures.py - Hand Gesture Recognition
- `GestureRecognizer`: detects hand keypoints (21 joints) + classifies gestures
- Backends: trt_pose_hand (TensorRT, primary) or MediaPipe (fallback)
- Gestures: fist, pan, stop (open palm), fine (OK sign), peace, no_hand
- Classification: SVM on 441 pairwise joint distances (TRT) or rule-based (MediaPipe)
- `process_frame(frame)` → `{gesture, hand_position, motion: {direction, speed, dx, dy}}`

### actions.py - Gesture → Desktop Actions
- `ActionMapper`: maps gesture + motion → xdotool commands
- pan + left/right → Alt+Left/Right (browser nav)
- pan + up/down → scroll
- fist → click, fine → Enter, peace + motion → fast scroll
- 0.5s debounce per action

### memory.py - Persistent Person Memory
- `MemoryStore`: JSON-backed at `data/memories.json`
- Schema: `{face_id: {name, facts[], last_seen, times_seen, pre_loaded}}`
- `get_context_string(face_id)` → formatted string for LLM prompt injection
- Facts are deduplicated on add

### reachy_bridge.py - Robot Hardware Interface
- `ReachyBridge`: direct reachy_mini SDK control
- `move_head(direction)` → goto_target with preset angles (left/right/up/down/front/nod)
- `play_emotion(emotion_id)` → plays recorded JSON motion from `models/emotions/`
- `feed_audio_chunk(level)` → head wobble during TTS (pitch oscillation at 6Hz, amplitude proportional to audio level)
- `wiggle_antennas()` → quick antenna snap
- Movement API: `set_target(head=create_head_pose(yaw, pitch, roll))` for real-time, `goto_target(..., duration=X)` for timed moves
- Head ranges: yaw ±20°, pitch ±15°, roll ±15°. Body yaw ±45°.
- Antennas: `set_target(antennas=np.deg2rad([angle, angle]))`. Body: `set_target(body_yaw=np.deg2rad(angle))`

### robot.py - High-Level Robot Abstraction (Stub)
- `RobotController`: maps abstract commands to reachy_mini calls
- `set_head_pose(direction)`, `set_emotion(emotion)`, `set_antenna_state(state)`
- `execute_response(response_dict)` → apply full brain response
- Mostly stubs; `reachy_bridge.py` is the real hardware interface

### hue.py - Philips Hue Lights
- `HueBridge`: emotion → light color mapping via phue library
- States: idle (dim warm), ambient (breathing pulse), engaged (emotion-driven)
- `set_emotion(emotion)` → color change (excited=yellow, curious=blue, calm=green, surprised=orange, amused=pink, skeptical=purple)
- `flash(emotion)` → quick blink for greetings
- Config via env: `HUE_BRIDGE_IP`, `HUE_LIGHT_IDS`
- `DummyHueBridge`: no-op fallback

### openclaw_bridge.py - Desktop Command Router
- `OpenClawBridge`: classifies text as conversation vs computer command
- `is_agent_command(text)` → keyword/regex classification
- `send_command(text)` → `openclaw agent --agent main --message <text> --json`
- Agent keywords: open, close, search, click, navigate, play, type, etc.
- Conversation keywords override: what, how, who, tell me, etc.

### dashboard.py - Web Dashboard
- `DashboardServer`: HTTP server on port 8080
- `GET /` → HTML page with video + status
- `GET /video` → MJPEG stream (~10 FPS) with face bbox overlay
- `GET /api/status` → JSON: mode, current_face, last_speech, faces, events, uptime

### agora_web_server.py - Agora Voice Web Server
- FastAPI server for Agora Conversational AI mode
- Serves `static/agora/index.html` (browser RTC client)
- `POST /api/agora/agent/start|stop` → manage agent lifecycle via `AgentManager`
- `POST /api/datastream/message` → dispatch robot actions from LLM (display_emotion, move_head, dance, wiggle)
- `POST /api/motion/audio-chunk` → feed TTS audio level for head wobble
- Uses `ReachyBridge` for all robot control

### agent_manager.py - Agora Agent Lifecycle
- `AgentManager`: REST API calls to Agora ConvAI to start/stop agents
- Loads config from `agent_config.json` with `{{placeholder}}` template support
- Handles 409 conflicts (agent already running)
- Configures MCP server endpoint if `MCP_PUBLIC_URL` env var is set

### agora_rtc.py - Agora RTC Audio Bridge
- `AgoraRTCClient`: bridges local mic → Agora RTC channel → speaker
- Captures mic at 48kHz, resamples to 16kHz, pushes PCM frames
- Receives TTS audio and pipes to `paplay`
- Monitors TTS speaking state (silence timeout 0.5s)

### llm_proxy.py - Agora LLM Proxy
- FastAPI OpenAI-compatible endpoint for Agora agent
- Intercepts `/v1/chat/completions`, injects system prompt + memory context
- Streams from OpenRouter, strips `[emotion]` and `[save:...]` tags
- Routes OpenClaw commands when detected

### mcp_server.py - MCP Tool Server
- FastMCP HTTP server exposing `execute_desktop_command` tool
- Agora agent calls this to run OpenClaw commands
- Runs `openclaw agent --agent main --message <cmd>` with 30s timeout

## Scripts

| Script | Purpose |
|--------|---------|
| `scripts/person_follow.py` | Hand raise → enroll → robot head+body tracks person. Two threads: tracking (YOLO+ReID) + control (30Hz). `--no-robot --debug` for testing. |
| `scripts/dance_to_music.py` | Live beat detection → 4-beat dance sequence (sway + bop). `--no-robot` for testing. |
| `scripts/live_faces.py` | Camera viewer with face detection/recognition overlay |
| `scripts/enroll_face.py` | Register face from photo: `--photo img.jpg --name Alice --face-id alice` |
| `scripts/enroll_live.py` | Interactive live face enrollment from camera |
| `scripts/test_gestures.py` | Real-time hand pose + gesture classification viewer |
| `scripts/test_actions.py` | Live gesture → desktop action demo |
| `scripts/test_reachy.py` | Basic robot connectivity test (antenna wiggle, head nod) |
| `scripts/test_reachy_emotions.py` | Play recorded emotions: `python test_reachy_emotions.py dance1` |
| `scripts/live_viewer.py` | Tkinter GUI for dashboard MJPEG + status |
| `scripts/setup_hand_pose.py` | Download trt_pose_hand models + train SVM classifier |
| `scripts/setup_host.sh` | Full Jetson setup (apt, venv, models, TRT engines) |
| `scripts/convert_face_trt.sh` | Convert YuNet/SFace ONNX → TensorRT FP16 engines |

## Config Files

- `config/system_prompt.txt` — Robot persona ("The Witness": curious, sarcastic, opinionated about tech, keeps responses to 1-3 sentences)
- `config/ambient_lines.json` — Random ambient commentary lines
- `config/system_prompt_agora.txt` — Agora-specific system prompt (if exists)
- `agent_config.json` — Agora ConvAI agent preset (OpenAI GPT-4 + Minimax TTS)

## Data Files

- `data/known_faces/embeddings.json` — Enrolled face embeddings (face_id → float array)
- `data/memories.json` — Person memory store (face_id → {name, facts, times_seen, last_seen})

## Models

- `models/opencv/face_detection_yunet_2023mar.onnx` — Face detection
- `models/opencv/face_recognition_sface_2021dec.onnx` — Face recognition
- `models/trt/yunet_640x480_fp16.engine` — TensorRT face detection
- `models/trt/sface_fp16.engine` — TensorRT face recognition
- `models/trt_pose/` — Hand pose ResNet18 weights + SVM classifier
- `models/emotions/*.json` — 30+ recorded emotion motion sequences for Reachy Mini
- `yolov8n-pose.pt` — YOLOv8 nano pose estimation (person detection + 17 keypoints)

## Environment Variables

```
# LLM backends
OPENAI_API_KEY          — For GPT-4o-mini
OPENROUTER_API_KEY      — For Claude Sonnet via OpenRouter

# Agora (cloud voice)
AGORA_APP_ID
AGORA_CUSTOMER_ID
AGORA_CUSTOMER_SECRET
AGORA_CHANNEL           — RTC channel name
AGORA_RTC_TOKEN         — Optional RTC token

# MCP / OpenClaw
MCP_PUBLIC_URL          — Public tunnel URL for MCP server (e.g., cloudflared/ngrok)

# Philips Hue
HUE_BRIDGE_IP           — Hue bridge IP address
HUE_LIGHT_IDS           — Comma-separated light IDs

# TTS (Agora mode)
MINIMAX_GROUP_ID
MINIMAX_API_KEY
```

## Common Commands

```bash
# Run full system (local mode)
python main.py --llm-backend openrouter --no-robot

# Run with Agora cloud voice
python main.py --agora --no-robot

# Person follow demo
python scripts/person_follow.py --no-robot --debug

# Dance demo
python scripts/dance_to_music.py --no-robot

# Enroll a face
python scripts/enroll_face.py --photo face.jpg --name "Alice" --face-id alice --facts "Engineer"

# Test gestures
python scripts/test_gestures.py
```

## Key Integration Points for New Features

- **Add a new robot behavior:** Add method to `reachy_bridge.py`, call from `agora_web_server.py` `_dispatch_action()` or `output_loop` in `main.py`
- **Add a new LLM tool:** Add to `mcp_server.py` for Agora mode, or handle in `brain_loop` `_handle_speech()` for local mode
- **Add a new gesture:** Add class to `GESTURE_CLASSES` in `gestures.py`, train new SVM or add rule in `_classify_rule_based()`
- **Add a new emotion:** Add to `models/emotions/` as JSON, add color mapping in `hue.py`, add to brain's valid emotions list
- **Integrate person_tracker with main.py:** Add `--person-follow` flag, create `person_follow_loop` thread, share camera or use separate VideoCapture

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/irenegracekp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
