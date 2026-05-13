## good-gym

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Good-GYM is a dual-platform AI fitness tracking application that uses RTMPose for real-time pose detection to count exercise repetitions. It has a **desktop app** (Python/PyQt5) and a **web app** (React/TypeScript).

## Commands

### Desktop App (Python)

```bash
# Install dependencies
pip install -r requirements.txt

# Run the application
python run.py

# Build Windows executable (uses PyInstaller, outputs to dist/)
python build_exe.py
```

### Web App (Good-GYM-Web/)

```bash
cd Good-GYM-Web
npm install
npm run dev        # Vite dev server
npm run build      # TypeScript check + Vite production build
npm run lint       # ESLint
```

### Testing

No test framework is configured. Testing is done manually via video/camera input.

## Architecture

### Data Flow

```
Camera/Video → VideoThread (QThread) → RTMPoseProcessor (rtmlib/ONNX) → ExerciseCounter (angle calc) → WorkoutTracker (JSON persistence) → UI
```

### Desktop App Layers

- **Entry point**: `run.py` → creates `WorkoutTrackerApp` (PyQt5 QMainWindow)
- **`app/`** — Application-level orchestration (managers):
  - `main_window.py` — Main window, wires all components together
  - `video_processor.py` — Coordinates pose detection and frame rendering
  - `mode_manager.py` — Switches between Workout mode (camera) and Stats mode
  - `counter_manager.py` — Rep counting adjustments and recording
  - `stats_manager.py` — Statistics display updates
  - `menu_manager.py` — Menu bar construction
- **`core/`** — Business logic and processing:
  - `rtmpose_processor.py` — Wraps rtmlib.Wholebody for pose keypoint extraction
  - `video_thread.py` — QThread that captures frames from camera or video file
  - `workout_tracker.py` — Saves/loads workout history to `data/workout_history.json`
  - `sound_manager.py` — Audio cues for completed reps
  - `translations.py` — i18n (English/Chinese)
- **`ui/`** — PyQt5 widget components:
  - `video_display.py`, `control_panel.py`, `workout_stats_panel.py`, `styles.py`
- **`exercise_counters.py`** — Angle-based rep counting logic (calculates joint angles from keypoints)

### Web App (`Good-GYM-Web/src/`)

- `App.tsx` — Main component
- `hooks/usePoseDetection.ts` — RTMPose via ONNX Runtime Web
- `utils/exercise-logic.ts` — Angle calculations (mirrors desktop logic)
- `utils/pose-utils.ts` — Keypoint processing

### Exercise Configuration

Exercises are defined in `data/exercises.json` using COCO 17-point pose format. Each exercise specifies:
- `down_angle` / `up_angle` — Thresholds for rep counting
- `keypoints.left` / `keypoints.right` — Indices into the 17-point skeleton (e.g., `[11, 13, 15]` = left shoulder→elbow→wrist)
- `is_leg_exercise` — Affects which keypoints are used for angle display

New exercises can be added via JSON config without code changes.

### Key Technical Details

- All ML inference runs on CPU only (ONNX Runtime, no GPU required)
- Video capture runs in a separate QThread to prevent UI freezing
- ONNX model files live in `models/` (RTMPose wholebody detection + pose estimation)
- Workout data persists in `data/workout_history.json` and `data/workout_goals.json`

---
> Source: [yo-WASSUP/Good-GYM](https://github.com/yo-WASSUP/Good-GYM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
