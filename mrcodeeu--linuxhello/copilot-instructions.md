## linuxhello

> **LinuxHello** is an open-source face authentication system for Linux, designed to mimic Windows Hello functionality. It provides secure, hands-free authentication for `sudo`, login screens (GDM/SDDM), and other PAM-aware services.

# LinuxHello Project Context

## Overview
**LinuxHello** is an open-source face authentication system for Linux, designed to mimic Windows Hello functionality. It provides secure, hands-free authentication for `sudo`, login screens (GDM/SDDM), and other PAM-aware services.

### Key Features
*   **Face Authentication:** Uses IR cameras and deep learning (ArcFace) for secure recognition.
*   **PAM Integration:** Seamlessly integrates with system authentication via a custom PAM module.
*   **Liveness Detection:** Prevents spoofing using depth/IR analysis.
*   **Modern Web UI:** React 19 + TypeScript interface with real-time enrollment progress, live camera preview, authentication testing, and system logs.
*   **Service Management:** Control systemd services, configure settings, and manage PAM integration through the GUI.
*   **Privacy-First:** All processing happens locally on the device.

## Architecture
The system relies on a hybrid architecture:
1.  **LinuxHello GUI (`cmd/linuxhello-gui/main.go`):** HTTP server on port 8080 serving React frontend and REST API. Handles enrollment, camera streaming, auth testing, configuration, and service management.
2.  **Python Inference Service (`python-service`):** gRPC server on port 50051 running ONNX models (SCRFD for detection, ArcFace for recognition) for face detection and embedding generation.
3.  **PAM Module (`pkg/pam`):** C-shared Go library (`pam_linuxhello.so`) that hooks into the Linux authentication stack.
4.  **Frontend (`frontend`):** React 19 + TypeScript + Vite SPA with tabs for enrollment, auth testing, user management, PAM config, settings, and logs.

### Component Communication
```
Browser (localhost:8080) ↔ HTTP API (Go) ↔ gRPC (Python ONNX)
                                    │
                            Camera (go4vl V4L2)
```

### GUI Features (v1.7.0+)
- **Real-time Enrollment:** Progress bar showing "Sample X/Y captured" with status messages
- **Live Camera Preview:** MJPEG stream with bounding box overlays for detection
- **Authentication Testing:** Test face recognition and see confidence scores + bounding boxes
- **System Logs:** View and download journalctl logs with level filtering (error/warn/info/debug)
- **Configuration UI:** Adjust camera settings, thresholds, and enrollment parameters
- **Service Control:** Start/stop/restart inference and GUI services via systemctl integration

## Tech Stack
*   **Backend:** Go (1.24+), gRPC, SQLite (auth data), CGO (PAM).
*   **AI/ML:** Python 3, ONNX Runtime, NumPy, OpenCV.
*   **Frontend:** React 19, TypeScript, Vite, Tailwind CSS.
*   **Build System:** Make, Shell scripts.

## Development Guide

### Prerequisites
*   **System:** Linux (Fedora/RHEL/CentOS recommended, others supported).
*   **Tools:** Go 1.24+, Python 3.9+, Node.js 18+, Make, GCC.
*   **Hardware:** IR-capable webcam (recommended) or standard webcam.

### Quick Start
The `Makefile` is the primary entry point for all development tasks.

```bash
# 1. Full Setup (Dependencies, Models, Python Venv)
make setup

# 2. Build All Components (Daemon, GUI, PAM, CLIs)
make build

# 3. Start Inference Service (Required for any operation)
make start-service-bg
```

### Key Commands
| Command | Description |
| :--- | :--- |
| `make setup` | Full setup: Python venv, Go deps, download models, build frontend |
| `make build` | Compiles linuxhello-gui HTTP server and PAM module |
| `make build-frontend` | Build React frontend (npm run build in frontend/) |
| `sudo linuxhello` | Launch GUI at localhost:8080 (starts inference service automatically) |
| `make start-service-bg` | Start Python inference service in background |
| `make status` | Check systemd service status |
| `make clean` | Removes build artifacts |
| `make lint` | Runs Go linters (fmt, vet, golangci-lint) |

### REST API Endpoints (localhost:8080)
| Endpoint | Method | Description |
| :--- | :--- | :--- |
| `/api/stream` | GET | MJPEG camera stream |
| `/api/users` | GET | List enrolled users |
| `/api/enroll` | POST | Start enrollment (username in JSON body) |
| `/api/enroll/status` | GET | Poll enrollment progress in real-time |
| `/api/config` | GET/POST | Get or update configuration |
| `/api/pam` | GET/POST | PAM status and management actions |
| `/api/logs` | GET | View systemd logs with journalctl integration |
| `/api/logs/download` | GET | Download full service logs |
| `/api/authtest` | POST | Test authentication with camera frame |
| `/api/service` | POST | Control services (start/stop/restart/enable/disable) |

### Configuration
Configuration is managed via `configs/linuxhello.conf` (local dev) or `/etc/linuxhello/linuxhello.conf` (production).
*   **Key Settings:** Camera device path (`/dev/video0`), authentication thresholds, and liveness checks.
*   **Struct:** Defined in `internal/config/config.go`.

### Project Structure
*   `cmd/linuxhello-gui/main.go` - HTTP server with REST API endpoints for enrollment, auth testing, config, logs, etc.
*   `internal/` - Core Go packages:
    *   `auth/` - Authentication engine, liveness detection, lockout management
    *   `camera/` - V4L2 camera interface via go4vl
    *   `config/` - Configuration management with Viper
    *   `embedding/` - SQLite-based face embedding storage
*   `pkg/pam/` - CGO PAM module (builds pam_linuxhello.so)
*   `python-service/` - ML inference engine:
    *   `inference_service.py` - gRPC server with SCRFD and ArcFace models
    *   `requirements.txt` - Python dependencies (onnxruntime, opencv, numpy, grpc)
*   `frontend/` - React 19 frontend:
    *   `src/components/` - EnrollmentTab, AuthTestTab, LogsTab, SettingsTab, etc.
    *   `src/hooks/` - React hooks for data fetching (useEnrollment, useAppData, useConfiguration)
*   `models/` - ONNX model files (downloaded by `make setup`):
    *   `det_10g.onnx` - SCRFD face detection from buffalo_l (640x640 input, 10G FLOPS)
    *   `arcface_r50.onnx` - Face recognition (112x112 input, 512-dim embeddings)
*   `scripts/linuxhello-pam` - PAM management script for enable/disable/status
*   `systemd/` - Service files for linuxhello-inference and linuxhello-gui

## Development Conventions
*   **Go:** Follows standard Go idioms. Use `make fmt` and `make vet` before committing.
*   **Python:** Dependencies in `python-service/requirements.txt`. Use the venv created by `make setup`.
*   **Frontend:** Modern React hooks pattern. Build with `npm run build` inside `frontend`.
*   **Safety:** The project interacts with system security (PAM). Always test changes with `make test-auth` before enabling PAM integration system-wide to avoid lockouts.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MrCodeEU) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
