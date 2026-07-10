## aiblindcaptcha

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI Blind Captcha is a time-encoded video CAPTCHA system that generates verification codes where content is only visible during video playback. The system implements Algorithm 2-style temporal encoding - single frames appear as structured noise, but moving foreground elements become visible when played as video.

## Architecture

### Core Components

- **`captcha_generator.py`**: Core CAPTCHA generation engine
  - `_render_text_mask()`: Text-based CAPTCHA rendering with large fonts (40% of video height)
  - `_render_shape_mask()`: Shape-based CAPTCHA (circle, rectangle, triangle, heart, arrow)
  - `_render_depth_mask()`: Depth image CAPTCHA with configurable thresholds
  - `generate_time_captcha()`: Main function that creates time-encoded MP4 videos

- **`server.py`**: FastAPI service layer
  - In-memory CAPTCHA store with TTL (180 seconds)
  - POST `/captcha/new`: Generate new CAPTCHA with mode/depth configuration
  - POST `/captcha/verify`: Validate user answers
  - GET `/captcha/hint/{captcha_id}`: Retrieve hints with attempt penalties

- **`static_demo.html`**: Frontend demo interface
  - Video loop playback with autoplay handling
  - Mode switching (text/shape/depth/random)
  - Depth image upload with threshold controls
  - Real-time validation and hint system

### CAPTCHA Modes

1. **Text Mode**: 3-5 character random words with 40% font size ratio
2. **Shape Mode**: Geometric shapes (circle, rectangle, triangle, heart, arrow)
3. **Depth Mode**: Custom depth images with configurable thresholds (tl, tu)
4. **Random Mode**: Randomly selects from above modes

## Development Commands

### Running the Server
```bash
# Activate virtual environment
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Start development server
python -m uvicorn server:app --reload

# Access demo at: http://127.0.0.1:8000
```

### Dependencies
Core dependencies (installed in .venv):
- fastapi==0.104.1
- uvicorn==0.37.0
- opencv-python==4.12.0.88
- pillow==11.3.0
- numpy==1.26.4

## Key Technical Details

### Video Generation Process
1. Create binary mask (text/shape/depth) representing foreground elements
2. Generate tiled noise pattern as background
3. Apply vertical motion to foreground pixels across frames (y + v*t)
4. Encode frames as MP4 using H.264 codec for browser compatibility
5. Return Base64-encoded video for frontend delivery

### Security Considerations
- In-memory storage with 180-second TTL
- 5 maximum attempts per CAPTCHA
- Answer validation with fuzzy matching for shapes
- Depth image processing with threshold masking
- Fallback to text mode if generation fails

### Frontend Video Handling
- Base64 video data with `data:video/mp4;base64,` prefix
- Loop playback with muted autoplay for browser compatibility
- Event listeners for debugging and user interaction
- Smart unmuting after successful autoplay start

## File Structure Notes
- `static/` directory contains demo HTML (copy from root)
- Virtual environment `.venv` contains all Python dependencies
- No external configuration files - settings are embedded in code
- Temporary MP4 files created in `/tmp/` during generation and cleaned up automatically

---
> Source: [fengcunhan/AIBlindCaptcha](https://github.com/fengcunhan/AIBlindCaptcha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
