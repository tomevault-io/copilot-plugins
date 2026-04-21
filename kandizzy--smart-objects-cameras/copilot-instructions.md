## smart-objects-cameras

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Educational template repository for Discord bots that communicate with Luxonis OAK-D cameras (DepthAI 3.x) on Raspberry Pi 5. Students learn to build smart object systems with computer vision and interactive communication.

**Target Hardware:** Three Raspberry Pi 5s (16GB each) named `orbit`, `gravity`, `horizon` — each with an OAK-D camera. SSH via `ssh orbit`, `ssh gravity`, `ssh horizon`.

**Core concept:** Cameras as conversational agents — reconfigure and query them in real-time through Discord, not just passive sensors configured once via SSH.

## Workflow Orchestration

### 1. Plan Mode Default

- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately - don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy

- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution

### 3. Self-Improvement Loop

- After ANY correction from the user: update `tasks/lessons.md` with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for relevant project

### 4. Verification Before Done

- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### 5. Demand Elegance (Balanced)

- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes - don't over-engineer
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing

- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests - then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

### Task Management

1. **Plan First**: Write plan to `tasks/todo.md` with checkable items
2. **Verify Plan**: Check in before starting implementation
3. **Track Progress**: Mark items complete as you go
4. **Explain Changes**: High-level summary at each step
5. **Document Results**: Add review section to `tasks/todo.md`
6. **Capture Lessons**: Update `tasks/lessons.md` after corrections

### Core Principles

- **Simplicity First**: Make every change as simple as possible. Impact minimal code.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact**: Changes should only touch what's necessary. Avoid introducing bugs.

## Development Environment

```bash
# Activate shared venv (REQUIRED before running any script)
source /opt/oak-shared/venv/bin/activate
# Or use alias: activate-oak

# Key packages: depthai, depthai-nodes, opencv-python, numpy, discord.py, requests, aiohttp, python-dotenv
```

**Critical versions** (see `docs/WORKING_VERSIONS.md`): depthai 3.3.0, depthai-nodes 0.3.7, opencv-contrib-python 4.10.0.84, numpy 1.26.4 (numpy <2.0 required).

**Secrets:** Each user has `~/oak-projects/.env` with `DISCORD_WEBHOOK_URL` and `DISCORD_BOT_TOKEN`. Never commit `.env` files.

## Running Detectors

Each Pi has ONE camera — only ONE detector script should run at a time. Run with `--discord` so others see when the camera is free.

```bash
# Person detection (YOLO v6, COCO class 0)
python3 person_detector.py --discord --log --threshold 0.6
python3 person_detector.py --display              # requires X11/VNC

# Fatigue detection (YuNet + MediaPipe landmarks, EAR + head pose)
python3 fatigue_detector.py --dm                  # Discord DM notifications
python3 fatigue_detector.py --dm --dm-quiet       # only alert on fatigue

# Gaze direction (YuNet + head pose + gaze estimation ADAS)
python3 gaze_detector.py --display --log

# Whiteboard OCR - detection only (finds text regions)
python3 whiteboard_reader.py --discord --display

# Whiteboard OCR - full (detection + recognition, reads text content)
python3 whiteboard_reader_full.py --discord --log
```

Stop any detector with `Ctrl+C`. The systemd service (`sudo systemctl status person-detector`) can auto-start the person detector on boot.

## Code Architecture

### Repository Structure

```
├── person_detector.py           # YOLO person detection (template baseline)
├── person_detector_with_display.py  # Person detection with OpenCV visualization
├── fatigue_detector.py          # EAR + head tilt fatigue monitoring
├── gaze_detector.py             # 3-stage gaze direction estimation
├── whiteboard_reader.py         # OCR text detection only
├── whiteboard_reader_full.py    # OCR detection + recognition (reads text)
├── discord_bot.py               # Interactive bot (!status, !detect, !screenshot, !whiteboard, etc.)
├── discord_notifier.py          # Webhook notifications (sync & async)
├── discord_dm_notifier.py       # DM-based notifications for fatigue detector
├── utils/
│   ├── face_landmarks.py        # MediaPipe EAR calculation, head pose Euler angles
│   ├── process_keypoints.py     # Keypoint processing for gaze detection
│   ├── node_creators.py         # DepthAI node creation helpers
│   ├── host_concatenate_head_pose.py  # Head pose vector concatenation
│   ├── ocr_crop_creator.py      # OCR text region cropping
│   └── config_sender_script.py  # Configuration management
├── depthai_models/              # Model YAML configs (RVC2/RVC4 variants)
│   ├── yunet*.yaml              # Face detection
│   ├── head_pose_estimation*.yaml
│   ├── gaze_estimation_adas*.yaml
│   ├── mediapipe_face_landmarker*.yaml
│   └── paddle_text_*.yaml       # OCR detection + recognition
└── docs/                        # Extended documentation
```

### DepthAI 3.x Pipeline Pattern

All detectors follow this pattern:

```python
with dai.Device() as device:
    platform = device.getPlatformAsString()
    with dai.Pipeline(device) as pipeline:
        # 1. Camera node
        cam = pipeline.create(dai.node.ColorCamera)
        cam.setPreviewSize(W, H)  # match model input
        cam.setInterleaved(False)
        cam.setFps(15)

        # 2. Neural network with automatic parsing (from depthai-nodes)
        model_desc = dai.NNModelDescription(model_ref, platform=platform)
        nn_archive = dai.NNArchive(dai.getModelFromZoo(model_desc))
        nn = pipeline.create(ParsingNeuralNetwork).build(cam.preview, nn_archive)

        # 3. Output queues — create BEFORE pipeline.start()
        q_det = nn.out.createOutputQueue(maxSize=4, blocking=False)
        q_preview = cam.preview.createOutputQueue(maxSize=4, blocking=False)

        pipeline.start()
        # 4. Main loop: poll queues, apply temporal debouncing, update status files
```

**Critical depthai 3.x API rules:**
- **No XLinkOut/XOut nodes** — create queues directly: `node.output.createOutputQueue()`
- **Use `ColorCamera`** not `Camera` — the new Camera node API is different and poorly documented
- **Use `getDeviceId()`** not deprecated `getMxId()`
- **Create all queues BEFORE `pipeline.start()`**

### Inter-Process Communication

Detectors and the Discord bot communicate via JSON status files in `~/oak-projects/`:

| File | Writer | Reader | Contents |
|------|--------|--------|----------|
| `camera_status.json` | person_detector.py | discord_bot.py | detection state, person count, username, hostname |
| `fatigue_status.json` | fatigue_detector.py | discord_bot.py | fatigue state, EAR values, head pose |
| `whiteboard_status.json` | whiteboard_reader*.py | discord_bot.py | detected text, confidence, region count |
| `whiteboard_history.jsonl` | whiteboard_reader*.py | discord_bot.py | time-series OCR readings (append-only) |
| `latest_frame.jpg` | person_detector.py | discord_bot.py | screenshot for !screenshot command |
| `latest_fatigue_frame.jpg` | fatigue_detector.py | discord_bot.py | screenshot with landmarks overlay |
| `latest_whiteboard_frame.jpg` | whiteboard_reader*.py | discord_bot.py | screenshot with text region boxes |

### Detection Logic

All detectors use temporal debouncing (1.5-2 seconds) — state must persist before triggering a notification to prevent flickering. State changes are logged to console, optionally to file (`--log`), and optionally sent to Discord (`--discord` or `--dm`).

### Discord Bot Commands

The bot (`discord_bot.py`) reads status files to respond to queries:

```
!ping                    !status              !detect              !screenshot
!whiteboard              !whiteboard-status   !whiteboard-history  !whiteboard-screenshot
!whiteboard-consensus    !set-confidence <v>  !set-fps <v>         !toggle-notifications
!help
```

### Multi-Stage Pipelines

- **Person detector:** Single-stage (YOLO v6, 80 COCO classes, person = class 0)
- **Fatigue detector:** Two-stage (YuNet face detection → MediaPipe landmarks → EAR + head pose via solvePnP)
- **Gaze detector:** Three-stage (YuNet → head pose estimation → gaze estimation ADAS)
- **Whiteboard reader:** Two-stage (PaddlePaddle text detection → PaddlePaddle text recognition)

## Modifying Code

### Changing Detection Models
```bash
python3 person_detector.py --model luxonis/yolov8-nano:r2-coco-640x640
```
Browse models at https://models.luxonis.com. Model YAML configs live in `depthai_models/`.

### Adjusting Performance
- **Threshold:** `--threshold 0.7` (higher = fewer false positives)
- **Frame rate:** `cam.setFps(15)` in script, or `--fps-limit` on fatigue/gaze detectors
- **Resolution:** `cam.setPreviewSize(W, H)` must match model input size
- **Debounce:** `DEBOUNCE_SECONDS` constant in each detector script

### Adding New Detectors
Follow the pattern in existing scripts:
1. Copy structure from the closest existing detector
2. Use `ParsingNeuralNetwork` from depthai-nodes for automatic output parsing
3. Write status to a JSON file for Discord bot integration
4. Add periodic screenshot capture for bot queries
5. Include `--discord`/`--log`/`--display` CLI arguments
6. Add startup/shutdown Discord announcements with username and hostname

### Config File Polling (Dynamic Reconfiguration)
The bot can write config changes that detectors pick up without restart:
```python
CONFIG_FILE = Path.home() / "oak-projects" / "camera_config.json"
# In detection loop: periodically read config, update threshold/mode/etc.
```

## Common Issues

### Camera Not Found
```bash
lsusb | grep Myriad  # Should show Intel Movidius MyriadX
# Try: different USB 3.0 port, powered hub, reload udev rules:
sudo udevadm control --reload-rules && sudo udevadm trigger
```

### Import Errors
Activate the shared venv first: `activate-oak` (alias for `source /opt/oak-shared/venv/bin/activate`)

### depthai 3.x API Errors
```python
# ❌ XLinkOut doesn't exist in 3.x:
xout = pipeline.create(dai.node.XLinkOut)
# ✅ Create queues directly:
q = cam.preview.createOutputQueue(maxSize=4, blocking=False)

# ❌ Camera node has different API:
cam = pipeline.create(dai.node.Camera)
cam.setPreviewSize(...)  # AttributeError
# ✅ Use ColorCamera (deprecated but works):
cam = pipeline.create(dai.node.ColorCamera)
```

### Display Issues
`--display` requires X11. Works via VNC; for SSH use `ssh -X orbit`. Press 'q' to exit the OpenCV window.

### Discord Issues
Test webhook: `python3 discord_notifier.py "test message"`. Check `~/oak-projects/.env` exists with valid URLs/tokens.

## Extending This Template — Next Frontiers

The template already implements config polling (`!set-confidence`, `!set-fps`), whiteboard OCR, per-camera routing, and screenshot capture. These are the next directions for students to explore:

### Cross-Detector Intelligence

Combine multiple detectors for richer insights instead of running them in isolation.

**Fusion via status files** (simplest — no code changes to existing detectors):
```python
# A new "attention_monitor.py" that reads multiple status files
fatigue = json.loads(FATIGUE_STATUS.read_text())
gaze = json.loads(GAZE_STATUS.read_text())

if fatigue['fatigue_detected'] and gaze['gaze_direction'] != 'center':
    send_notification("⚠️ Student appears fatigued AND not looking at board")
```

**Fusion within a single pipeline** (advanced — multi-model on one camera):
```python
# Chain models: face detection → landmarks + gaze in parallel
# Use GatherData node to synchronize outputs from multiple NNs
# See fatigue_detector.py and gaze_detector.py for multi-stage patterns
```

**Example bot commands:**
- `!attention` — Combined fatigue + gaze + person count summary
- `!alert-rules add "fatigue AND gaze!=center"` — Custom compound triggers
- `!compare detectors` — Side-by-side status from all running detectors

### Depth Features (Untapped OAK-D Hardware)

The OAK-D's stereo cameras are currently unused. Depth opens up physical-space awareness.

**Adding stereo depth to the pipeline:**
```python
# Create mono cameras and stereo depth node
left = pipeline.create(dai.node.MonoCamera)
right = pipeline.create(dai.node.MonoCamera)
stereo = pipeline.create(dai.node.StereoDepth)

left.setResolution(dai.MonoCameraProperties.SensorResolution.THE_400_P)
right.setResolution(dai.MonoCameraProperties.SensorResolution.THE_400_P)

left.out.link(stereo.left)
right.out.link(stereo.right)

# Get depth map
q_depth = stereo.depth.createOutputQueue(maxSize=4, blocking=False)
```

**Student project ideas:**
- **Proximity zones:** "Alert if someone is within 1 meter of the whiteboard"
- **Occupancy heatmaps:** Track where people spend time (depth + position over time)
- **Distance measurement:** Report how far detected people are from the camera
- **Spatial detection:** Combine YOLO detections with depth for 3D bounding boxes
- `!zone add "whiteboard" 0.5 2.0` — Define distance-based alert zones via Discord

**Resources:** Check `/opt/oak-shared/oak-examples/` for stereo depth examples. See https://docs.luxonis.com/software-v3/depthai/ for spatial detection documentation.

### Multi-Camera Coordination

Three Pis, three cameras — make them work together as a network.

**Shared state via network** (simplest approach):
```python
# Each detector already writes status files. Share them across Pis:
# Option 1: Periodic SCP/rsync of status files between Pis
# Option 2: Simple HTTP server on each Pi serving status JSON
# Option 3: Shared MQTT broker for real-time pub/sub

# Example: lightweight Flask endpoint on each Pi
from flask import Flask, jsonify
app = Flask(__name__)

@app.route('/status')
def status():
    return jsonify(json.loads(STATUS_FILE.read_text()))
```

**Camera-triggered actions:**
```python
# orbit detects person → gravity starts recording → horizon watches the door
# Implement via Discord bot commands or direct HTTP calls between Pis

@bot.command(name='coordinate')
async def coordinate(ctx, action: str):
    """Send a command to all cameras via their status endpoints."""
    for pi in ['orbit', 'gravity', 'horizon']:
        requests.post(f'http://{pi}:5000/command', json={'action': action})
    await ctx.send(f"📡 Sent '{action}' to all cameras")
```

**Student project ideas:**
- **Distributed occupancy:** Combine person counts from all cameras for room-wide tracking
- **Handoff tracking:** Person leaves camera 1's view, appears in camera 2's
- **Synchronized snapshots:** `!snapshot-all` — get images from all three cameras at once
- **Cross-camera alerts:** "Orbit sees a person but Gravity doesn't — they went left"
- **Camera mesh status:** `!mesh-status` — single embed showing all three cameras' state
- **Consensus detection:** Only alert if 2+ cameras agree something is happening

### Combining All Three

The most ambitious projects combine these directions:

- **3D room mapping:** Depth from all cameras → reconstructed room model
- **Attention analytics dashboard:** Cross-detector fusion across cameras → "Classroom engagement: 72%"
- **Smart room automation:** Multi-camera coordination triggers real-world actions (lights, displays)
- **Adaptive detection:** Cameras dynamically adjust thresholds based on what other cameras see

### Implementation Pattern: Shared Config + Status

All extensions should follow the established pattern:
1. **Detector writes** a JSON status file (for bot queries)
2. **Bot reads** status files and responds to commands
3. **Config file polling** lets the bot reconfigure detectors without restart
4. **Discord announces** state changes with username and hostname
5. **Screenshots** are captured periodically for visual queries

## Key Documentation

| Document | Purpose |
|----------|---------|
| `docs/STUDENT_QUICKSTART.md` | Getting started fast (includes Claude Code setup) |
| `docs/WORKFLOW.md` | File copying workflow (laptop → Pi) |
| `docs/CHEATSHEET.md` | Command quick reference |
| `docs/WORKING_VERSIONS.md` | Package version compatibility (critical!) |
| `docs/DISCORD_BOT_PLAN.md` | Full bot setup guide |
| `docs/discord-integration.md` | Webhook setup (simpler, one-way) |
| `docs/GIT_COLLABORATION.md` | Git workflow for teams |
| `docs/NEXT_IDEAS.md` | 5 project extension ideas with code |
| `docs/EQUIPMENT_LIST.md` | Hardware reference and pricing |
| `docs/INITIAL_SETUP.md` | Instructor: Pi setup from scratch |
| `docs/multi-user-access.md` | Multi-user coordination |
| `docs/wifi-management.md` | WiFi network switching |
| `WHITEBOARD_READER.md` | OCR feature documentation |

**Slides:** Most docs have [slide versions](https://kandizzy.github.io/smart-objects-cameras/) for quick reference.

## External Resources

- **DepthAI 3.x docs:** https://docs.luxonis.com/software-v3/depthai/
- **depthai-nodes:** https://github.com/luxonis/depthai-nodes
- **Luxonis Hub models:** https://models.luxonis.com
- **discord.py:** https://discordpy.readthedocs.io/
- **Oak examples on Pi:** `/opt/oak-shared/oak-examples/` (symlinked to `~/oak-examples/`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kandizzy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
