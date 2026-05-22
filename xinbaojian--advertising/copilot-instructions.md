## advertising

> This file provides guidance to Qoder (qoder.com) when working with code in this repository.

# AGENTS.md

This file provides guidance to Qoder (qoder.com) when working with code in this repository.

## Commands

### Running the application
```bash
python main.py
```

### Installing dependencies
```bash
pip install opencv-python>=4.5.0 Pillow>=8.0.0 numpy>=1.19.0
```
Or:
```bash
pip install -r requirements.txt
```

## Architecture

This is a macOS automation script for the "汽水音乐" (Qishui Music) iOS app using iPhone Mirroring. The project consists of a single main script that performs image recognition and automated clicking.

### Core Components

**Image Recognition System** (`find_image_on_screen()`):
- Uses OpenCV template matching to locate UI elements on screen
- Handles Retina display scaling by converting between logical coordinates (pyautogui) and pixel coordinates (screenshots)
- Supports optional window bounds to limit search area
- Returns logical coordinates suitable for `pyautogui.click()`

**Application Launcher** (`call_iphone()`):
- Uses AppleScript to launch and position the iPhone Mirroring application
- References app by Bundle ID (`com.apple.ScreenContinuity`) rather than localized name to work across different language environments
- Dynamically retrieves the app's process name for UI scripting operations

**Main Automation Loop** (`loop_process()`):
- Cycles through task completion states by detecting specific UI elements in the `images/` directory
- Handles special cases like live streaming interfaces (detects `guan-zhu.png` and closes with `gz-close.png`)
- Implements random delays (1-10 seconds) between actions to mimic human behavior
- Tracks progress and stops after 90 iterations or when `finished.png` is detected

### Image Assets

All template images are stored in the `images/` directory:
- `finished.png` - Task completion indicator
- `success.png` - Success button to click after ad completion
- `reward.png` - Reward claim button
- `speaker.png` - Speaker icon to continue tasks
- `guan-zhu.png` - Follow button (indicates live stream interface)
- `gz-close.png` - Close button for live stream overlay

### System Requirements

- macOS with Python 3.6+
- Accessibility permissions enabled for Python to control mouse/keyboard
- iPhone Mirroring feature must be available and functional
- Dependencies: opencv-python, Pillow, pyautogui, numpy

### Code Conventions

- Chinese comments and print statements throughout (this is intentional for the project's target audience)
- Uses logical coordinates from pyautogui (screen points) not pixel coordinates
- Template matching threshold default is 0.8, adjustable per call

---
> Source: [xinbaojian/advertising](https://github.com/xinbaojian/advertising) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
