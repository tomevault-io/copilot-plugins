## camera-hack

> Security research on Yoosee/Gwelltimes IP cameras. Current target: SC-B21 dual-lens PTZ (firmware 38.02.103).

# CLAUDE.md

## Project

Security research on Yoosee/Gwelltimes IP cameras. Current target: SC-B21 dual-lens PTZ (firmware 38.02.103).

## Attack Surface

| Port | Service | Auth | Notes |
|------|---------|------|-------|
| 554/tcp | RTSP | Partial | `/onvif1` needs auth, `/cam/realmonitor` open |
| 5000/tcp | ONVIF | **None** | Full enumeration, PTZ control, DoS via write ops |

Ports only respond to SYN scans (`sudo nmap -sS`), not TCP connect.

## RTSP Streams

| Path | Resolution | Auth |
|------|-----------|------|
| `/onvif1` | 1920x2160 H.265 15fps | Required |
| `/onvif2` | 640x720 H.265 15fps | Required |
| `/cam/realmonitor` | Varies | None |

## Structure

```
├── advisories/         # Formal vulnerability writeups
├── tools/rtsp/         # RTSP capture tools
├── sdcard_hack/        # SD card exploitation
├── arduino/            # UART serial bridge
├── esp32/              # ESP32 serial bridge
└── config.example      # Config template (no credentials)
```

## Hardware

- **OEM**: Shenzhen Gwelltimes Technology
- **Firmware**: 38.02.103, Hardware Ver 2.1
- **Video**: Dual lens stitched vertically (1920x2160 + 640x720)
- **WiFi**: RTL8188 variant
- **Audio**: G.711 PCM 16kHz mono
- **MAC OUI**: `4C:B0:08` (Gwelltimes), `A8:B5:8E` (Bilian/WiFi)

## Notes

- ONVIF write operations (CreateUsers, SetHostname) crash the IPC process permanently
- Camera requires power cycle to recover from DoS
- Scripts have placeholder IPs — copy `config.example` to `config.env`

---
> Source: [gabrielmaialva33/camera-hack](https://github.com/gabrielmaialva33/camera-hack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
