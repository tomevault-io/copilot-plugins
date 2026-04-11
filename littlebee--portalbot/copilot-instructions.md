## portalbot

> Portalbot is a telepresence robot platform built on `basic_bot`.

# Portalbot Agent Notes

## Purpose
Portalbot is a telepresence robot platform built on `basic_bot`.

It supports two WebRTC media paths:
- View path: robot camera/mic -> browser (any participant in a space can view)
- Control path: controlling browser camera/mic -> robot onboard UI (single active controller)

## Runtime Components

### Public signaling server (`src/public_server.py`)
- FastAPI + WebSocket endpoint at `/ws`
- REST endpoints:
  - `GET /health`
  - `GET /spaces`
- Routes signaling/control message types to modular handlers:
  - `src/server/space_manager.py`
  - `src/server/webrtc_signaling.py`
  - `src/server/robot_control_handler.py`

### Robot relay service (`src/portalbot_service.py`)
Responsibilities:
- Connect/authenticate to public server via websocket
- Relay view-stream offers to basic_bot vision service
- Relay control-stream offers to onboard UI service
- Forward ICE candidates to both local services
- Relay robot control commands to `central_hub`

This service no longer renders pygame UI.

### Onboard UI service (`src/onboard_ui_service.py`)
Responsibilities:
- Own pygame display loop (`RobotDisplay`)
- Show robot eyes while idle
- Show controlling user's incoming video when available
- Handle control WebRTC peer termination through `src/ui/webrtc_peer.py`
- Expose local HTTP endpoints used by `portalbot_service`:
  - `POST /offer`
  - `POST /ice_candidate`
  - `GET /health`

### Frontend (`webapp/`)
- React + TypeScript app
- Main hook: `webapp/src/hooks/useWebRTC.ts`
- Uses two peer connections via `webapp/src/services/webrtcPeer.ts`:
  - `viewPeer`: receive robot A/V
  - `controlPeer`: send operator A/V to onboard UI

## Message and Media Flow

### Space and identity
1. Browser connects to `/ws` over `WebSocket`, receives `connected`.
2. Browser sends `join_space` over `WebSocket`.
3. Robot service connects and sends `robot_identify` over `WebSocket`.

### View media flow (robot -> browser)
1. Browser sends `offer` over `WebSocket` to public server.
2. Public server forwards `offer` over `WebSocket` to robot service.
3. Robot service POSTs offer over local `REST` to vision service `/offer`.
4. Robot service sends `answer` over `WebSocket` back through public server.
5. Browser and vision exchange media over `WebRTC peer connection` (robot video + audio to browser).

### Control media flow (browser -> onboard UI)
1. Browser creates control peer and sends `control_offer` over `WebSocket`.
2. Public server forwards `control_offer` over `WebSocket` to robot service.
3. Robot service POSTs offer over local `REST` to onboard UI `/offer`.
4. Onboard UI returns answer over local `REST`; robot service forwards `control_answer` over `WebSocket`.
5. Browser and onboard UI exchange media over `WebRTC peer connection` (operator video + audio to robot).

### ICE flow
- Browser sends `ice_candidate` over `WebSocket` via public server.
- Robot service forwards ICE to both:
  - vision `/ice_candidate` over local `REST`
  - onboard UI `/ice_candidate` over local `REST`

### Control workflow messages
- `control_request`, `control_granted`, `control_release`, `set_angles` all use `WebSocket` signaling via public server.
- Control-state validation is currently stubbed to auto-grant in `ControlStateManager`.

## Config and Services

### Service list (`basic_bot.yml`)
Important services:
- `central_hub`
- `vision`
- `onboard_ui` (new)
- `portalbot`

### Robot config (`portalbot_robot.yml`)
Loaded by `src/commons/robot_config.py`.
Key fields include:
- `public_server_url`
- `vision_service_url`
- `onboard_ui_service_url`
- `ice_servers`

### Secrets
- Robot auth keys are loaded from `robot_secrets/*.key`
- Never commit real key files

## Development Environment
- Use conda environment: `basic_bot`
- Example interpreter: `/opt/anaconda3/envs/basic_bot/bin/python`

## Running and Testing

### Full test command
Use the project script for full test runs:
```bash
./test.sh
```
`test.sh` runs:
1. `python -m pytest -vv tests/`
2. `cd webapp && npm install && npm run test`

## Validating

### run the full suite of code validation scripts

To identify python types, flake8, typescript, eslint and prettier errors.  Fix what things (like spacing and punctuation) that can be fixed by the various tools.

This script should be run by the agent on all proposed changes.

```bash
./validate.sh
```


### Pytest rule
Only run `pytest` directly when targeting an individual module/test during focused debugging.

### Helpful targeted checks
```bash
conda run -n basic_bot python -m pytest -vv tests/test_space_config.py
cd webapp && npm run test -- App.test.tsx
```

## Operational Notes
- `restart_robot.sh` now calls `bb_killall` before restarting services.
- Logging helper lives in `src/commons/logger_utils.py`.
- Onboard UI face detection uses OpenCV Haar cascade via `src/commons/face_detector.py`.
- Onboard audio playback from incoming WebRTC track uses `src/commons/audio_stream_player.py`.

## Known Limitations / TODOs
- `webrtc_signaling` currently broadcasts `answer` and `control_answer` to the space; target-specific routing is still TODO.
- UI/control validation (audio + face checks before granting control) is not fully implemented yet.
- TURN/STUN configuration is static in config and should be reviewed for production credential handling.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/littlebee)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/littlebee)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
