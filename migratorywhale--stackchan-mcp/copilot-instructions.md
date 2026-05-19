## stackchan-mcp

> This repository contains firmware and host-side tooling for a push-based

# Stack-chan Project Guide

This repository contains firmware and host-side tooling for a push-based
Stack-chan voice avatar running on M5Stack CoreS3.

## Scope

- `firmware/` contains the Arduino/PlatformIO firmware for M5Stack CoreS3.
- `mcp-server/` contains a Python MCP server that lets clients control
  Stack-chan over HTTP.
- `faces/` and `firmware/data/` contain the PNG face assets used by the device.
- `start-http.sh` starts the MCP HTTP server and the public tunnel used by
  remote clients.

## Important Rules

- Do not read or use any `CLAUDE.md` files. They are for a different assistant.
- Do not call memory, identity, or introspection MCP tools such as
  `imprint-memory`, `memory_search`, `breath`, or similar tools.
- Be careful with live-device endpoints. `GET /audio` consumes and clears the
  current recording buffer on the device.
- Do not overwrite local secrets or Wi-Fi settings in `firmware/src/config.h`.
  Use `firmware/config.h.example` for documented defaults.
- Avoid destructive git operations unless explicitly requested.

## Firmware Notes

The firmware exposes a local HTTP API on port 80:

- `POST /play` queues a WAV URL for playback.
- `POST /mode` switches between `api` and `mcp` recording behavior.
- `GET /audio/status` reports whether a recording is ready.
- `GET /audio` returns the latest WAV recording and then clears it.
- `POST /move`, `POST /home`, `POST /nod`, and `POST /shake` control servos.
- `POST /face` and `GET /face` control or inspect the current expression.
- `GET /snapshot` returns a 320x240 JPEG from the camera.

Key services live in:

- `firmware/src/http_server.cpp`
- `firmware/src/mic_service.cpp`
- `firmware/src/playback_service.cpp`
- `firmware/src/face_service.cpp`
- `firmware/src/servo_service.cpp`
- `firmware/src/camera_service.cpp`
- `firmware/src/wifi_manager.cpp`

Use PlatformIO from `firmware/` for builds and uploads:

```sh
pio run
pio run -t upload
pio device monitor
pio run -t uploadfs
```

The expected serial device on this Mac is commonly `/dev/cu.usbmodem101`, but
verify it before upload because the port can change.

## Quality Checks

Use the project-root `Makefile` as the default quality gate:

```sh
make lint
make test
```

- `make lint` runs Python `ruff` plus firmware `pio check`.
- `make test` runs Python `pytest` plus a firmware `pio run` build.
- `make test-mcp` runs only the MCP server unit tests.

Run the smallest useful check while iterating, then run the broader gate before
handing off changes:

- Python or MCP server changes: `uv run ruff check .` and `uv run pytest`.
- MCP server behavior changes: also run `make test-mcp`.
- Firmware changes: run `cd firmware && pio run`.
- Firmware safety/lint checks: run
  `cd firmware && pio check --severity=high --fail-on-defect=high`.
- Cross-boundary HTTP contract changes: update firmware, MCP server callers,
  tests, and docs together.

MCP tests are written to avoid live-device side effects. They mock the MCP
package and must not call Stack-chan HTTP endpoints or consume `GET /audio`.

## MCP Server Notes

`mcp-server/server.py` provides tools for speaking, listening, moving the head,
changing faces, checking status, and taking snapshots.

Important environment variables:

- `STACKCHAN_IP`: device IP address. Current known default in code is
  `10.83.20.187`.
- `STACKCHAN_PORT`: device HTTP port, usually `80`.
- `MAC_IP`: host IP used in generated audio URLs.
- `AUDIO_SERVE_PORT`: local HTTP port for generated WAV files.
- `TTS_ENGINE`: `fish-audio` or `edge-tts`.

For local HTTP MCP mode:

```sh
./start-http.sh
./start-http.sh stop
```

## Live Device Checks

Useful non-destructive checks:

```sh
curl -sS --max-time 5 http://$STACKCHAN_IP/audio/status
curl -sS --max-time 5 http://$STACKCHAN_IP/face
curl -sS --max-time 10 -o /tmp/stackchan_snapshot.jpg http://$STACKCHAN_IP/snapshot
```

Avoid `GET /audio` unless the task explicitly needs to consume the pending
recording.

## Development Style

- Prefer small, focused changes that match the existing Arduino C++ and Python
  style.
- Keep hardware behavior responsive: avoid blocking work in the main firmware
  loop when a task can run asynchronously.
- Preserve PSRAM-aware allocation patterns for audio, face assets, and camera
  buffers.
- When changing HTTP contracts, update both firmware and MCP server callers.
- For face assets, keep filenames and SPIFFS paths aligned between
  `firmware/data/` and `firmware/src/face_service.cpp`.

## Troubleshooting Documentation

- Record non-trivial debugging sessions under `docs/`.
- Use this filename pattern: `docs/<topic>-troubleshooting-YYYY-MM-DD.md`.
- Include the symptoms, investigation steps, false leads, root cause or current
  best hypothesis, final fix, verification commands/results, and references.
- Prefer concrete artifacts over vague notes: HTTP responses, serial log
  excerpts, build errors, image-diff numbers, firmware versions, and source
  links.
- If the issue changes established project behavior, update
  `docs/development-guide.md` as well so the general guide stays current.


<claude-mem-context>
# Memory Context

# [stackchan] recent context, 2026-05-16 8:56pm GMT+9

No previous sessions found.
</claude-mem-context>

---
> Source: [migratorywhale/stackchan-mcp](https://github.com/migratorywhale/stackchan-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
