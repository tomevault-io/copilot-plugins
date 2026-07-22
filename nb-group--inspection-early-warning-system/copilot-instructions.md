## inspection-early-warning-system

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

人脸识别预警系统 (Face Recognition Early Warning System) - A real-time face recognition alert system that detects specific personnel and displays full-screen warnings.

**Tech Stack**: Python, OpenCV, DeepFace (Facenet512), PyWebView, TensorFlow

## Development Commands

### Environment Setup
```bash
pip install -r requirements.txt
```
Note: First run will auto-download DeepFace models (~300MB)

### Running the Application
```bash
python main.py
# Or use the Windows launcher
启动.bat
```

### Testing Environment
```bash
python test_environment.py
```

### IP Camera Simulator (for testing)
```bash
python tools/ipcam_sim.py
```

## Architecture Overview

### Core Application Flow
1. **Single Instance**: Uses Win32 mutex (`Global\\NullifyBlightFaceRecognition`) to ensure only one instance runs
2. **IPC Communication**: Socket-based IPC server allows subsequent launches to wake the settings window
3. **Multi-threaded Design**:
   - Main thread: PyWebView UI
   - Camera thread: Continuous frame capture
   - Monitor thread: Face recognition processing
   - IPC thread: Listen for wake requests
   - Audio thread: Optional audio level monitoring

### Key Modules

**main.py**: Entry point with single-instance enforcement via Win32 mutex

**app/application.py**:
- `Application` class orchestrates all components
- `SettingsAPI` provides WebView JavaScript API bridge (uses `__slots__` to prevent attribute enumeration errors)
- Manages lifecycle: camera, face recognizer, alert windows, IPC server
- Handles state persistence (monitoring_enabled saved to config)

**app/camera.py**:
- `CameraMonitor` supports both local cameras and IP cameras (RTSP/HTTP streams)
- Dual camera support (front/back) for IP cameras
- Video recording functionality (MP4 output)
- Auto-reconnect on failure (after 30 consecutive failures)
- Thread-safe frame access with locks

**app/face_recognition.py**:
- Two-stage detection: Fast OpenCV Haar Cascade → DeepFace feature extraction
- Only processes detected face regions for performance
- Uses cosine similarity for matching (threshold typically 0.5-0.7)
- Returns detailed results: matched persons with name, similarity, and live face image
- Handles Unicode paths via `np.fromfile` + `cv2.imdecode`

**app/alert_window.py**:
- Runs in separate process to avoid PyWebView main-thread limitations
- Full-screen, frameless, topmost alert window
- Displays: warning icon, person photos, names, similarity scores
- Auto-hides after N seconds without detected faces (configurable)
- Manual dismiss button available

**app/audio_monitor.py**:
- Monitors audio stream from IP camera
- Triggers separate alert (`DbAlertWindow`) when decibel threshold exceeded
- Configurable threshold and hold duration

**app/config.py**:
- JSON-based configuration in `data/config.json`
- Auto-migrates from old format (string paths) to new format (dict with name/path/angle)

**app/ipc.py**:
- Socket server listens on localhost:9527
- Allows secondary launches to send wake command

### Face Data Format

**New format** (current):
```json
{
  "faces": [
    {
      "name": "张三",
      "path": "data/faces/zhangsan.jpg",
      "angle": "正面"
    }
  ]
}
```

**Old format** (still supported):
```json
{
  "faces": ["data/faces/person1.jpg"]
}
```

The system auto-converts old to new format on load.

## Important Implementation Details

### PyWebView API Boundary
- `SettingsAPI` uses `__slots__ = ('_app',)` to hide complex objects from WebView
- This prevents JavaScript from triggering Python attribute enumeration errors
- All API methods must be simple return types (dict, str, bool, int)

### Unicode/Chinese Path Handling
- Use `np.fromfile()` + `cv2.imdecode()` instead of `cv2.imread()` for images
- Use `cv2.imencode()` + `tofile()` instead of `cv2.imwrite()` for writing
- Applies to both face images and configuration paths

### Image Transfer to WebView
- Use Base64 DataURL format: `data:image/jpeg;base64,<base64_string>`
- Avoids file path encoding issues
- Used for: preview frames, face thumbnails, live detection results

### Window Management
- Settings window never truly closes (only hides) to allow re-opening
- `on_closing()` handler returns `False` to prevent destruction
- Alert window runs in separate process with Pipe communication

### Face Recognition Performance
1. OpenCV Haar Cascade detects face bounding boxes (~fast)
2. DeepFace processes only detected regions with `detector_backend="skip"`
3. Pre-loads target face embeddings at startup (avoid repeated model loading)
4. Configurable detection interval (default 0.5s) controls CPU usage

### Camera Backend Selection
- Windows: Try CAP_DSHOW → CAP_MSMF → CAP_ANY in order
- IP cameras: Use OpenCV VideoCapture with URL directly
- Supports: RTSP, HTTP (MJPEG), local video files

## Configuration Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| camera_type | "local" | "local" or "ip" |
| camera_index | 0 | Local camera device index |
| ip_camera_url | "" | RTSP/HTTP stream URL |
| enable_dual | false | Enable dual camera mode (front/back) |
| recognition_threshold | 0.6 | Match threshold (0.5-0.7 recommended) |
| alert_duration | 3 | Seconds to keep alert after face disappears |
| detection_interval | 0.5 | Seconds between recognition attempts |
| face_model | "Facenet512" | DeepFace model name |
| detector_backend | "opencv" | Face detection backend |
| monitoring_enabled | false | Auto-restore monitoring on startup |
| db_threshold | 75 | Audio decibel alert threshold |

## Common Development Patterns

### Adding New Configuration Options
1. Add to `data/config.json` default in `config.py`
2. Add UI controls in `app/settings_ui.html`
3. Handle updates in `SettingsAPI.save_config()`
4. Trigger component restart if needed (e.g., camera, face recognizer)

### Extending Camera Support
- Modify `CameraMonitor.__init__()` to accept new parameters
- Update `Application._init_components()` to pass parameters
- Add UI controls and API handlers in `SettingsAPI`

### Adding New Alert Types
- Create new window class (see `AlertWindow`, `DbAlertWindow`)
- Use multiprocessing if needs to coexist with PyWebView
- Communicate via `multiprocessing.Pipe` or Queue

## Data Directories

```
data/
├── config.json          # Configuration (auto-generated)
├── faces/              # Target face images with Chinese filenames supported
├── screenshots/        # Saved snapshots (snapshot_YYYYMMDD_HHMMSS.jpg)
└── records/            # Video recordings (record_YYYYMMDD_HHMMSS.mp4)
```

## Debugging Tips

- Set `webview.start(debug=True)` in `application.py` for WebView console
- Check console output for face loading confirmation
- Use `test_environment.py` to verify dependencies
- Monitor `data/config.json` for configuration changes
- Use IP camera simulator `tools/ipcam_sim.py` for testing without hardware

---
> Source: [NB-Group/Inspection_Early_Warning_System](https://github.com/NB-Group/Inspection_Early_Warning_System) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
