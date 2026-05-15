## costuma

> Costuma is a browser-based AR costume experience built entirely with AI collaboration. It uses TensorFlow.js PoseNet for real-time pose detection to overlay costume images on people's faces/bodies through a webcam feed.

# Costuma - AR Costume Overlay Project

## Project Overview

Costuma is a browser-based AR costume experience built entirely with AI collaboration. It uses TensorFlow.js PoseNet for real-time pose detection to overlay costume images on people's faces/bodies through a webcam feed.

**Live Demo**: https://chaboud.github.io/costuma/site/basic-costume.html

## Quick Start

### Development Server
```bash
cd site
node https-server.js
# Opens https://localhost:8443
# Accept self-signed certificate warning
```

### Testing
- Open https://localhost:8443/basic-costume.html
- Grant camera permissions
- Click costume buttons to switch overlays

## Architecture

### Core Technology Stack
- **Pose Detection**: TensorFlow.js PoseNet (pose-detection model)
- **Rendering**: Canvas 2D (switched from Three.js for simplicity/performance)
- **Hosting**: GitHub Pages (static, client-side only)
- **Camera API**: getUserMedia with iOS/Android compatibility

### Main Application File
**`site/basic-costume.html`** (~700 lines) - Self-contained single-file app
- All HTML, CSS, and JavaScript in one file
- No build process required
- Works offline once loaded

## Key Technical Concepts

### 1. Pose Detection Pipeline
```javascript
// Flow:
video → inputCanvas → PoseNet → poses → matching → smoothing → scaling → rendering
```

**Input Resolution**: 513x513 (configurable in PoseNet config)
**Detection Threshold**: 0.3 score minimum
**Max People**: 5 simultaneous detections

### 2. Person Tracking System

**Nearest Neighbor Matching** (lines 173-205):
- Tracks people across frames by nose position proximity
- 300px matching threshold
- Handles dropout/re-acquisition
- Uses `previousPoses` array for frame-to-frame continuity

**Key Variables**:
- `previousPoses`: Last frame's pose data for matching
- `MATCH_THRESHOLD`: 300px max distance for same person
- `EMA_ALPHA_POSITION`: 0.25 (position smoothing)
- `EMA_ALPHA_SCALE`: 0.1 (scale smoothing)

### 3. Scaling System

**Triangle Distance Method** (lines 295-315):
- Calculates max edge length of eye-nose-eye triangle
- More robust than IOD (intra-ocular distance) for face turns
- Divisor: `/50` to convert triangle distance to scale factor

**Scale Smoothing** (lines 317-341):
- EMA smoothing per person (keyed by position bucket)
- Rolling average of last 30 observations for default
- Prevents "popping" on first detection

**Current Issue**: "Scale pumping" - rhythmic growing/shrinking
- Partially mitigated by `seenScale` global EMA
- Not fully resolved

### 4. Costume Types

**Face Costumes** (overlay on face):
- `scream` (Candy): 280x280
- `grumpy` (Grumpy Cat): 320x320
- `disaster` (Disaster Girl): 384x384
- `derpy` (Derpy): 640x640

**Body Costume**:
- `rick` (Rick Roll): 720x450 - offsets down from nose by triangle distance

**Debug Mode**:
- `debug`: Shows skeleton with keypoint labels

### 5. Camera Handling

**Platform-Specific Logic** (lines 343-420):
- **iOS**: Uses `facingMode` ('user' vs 'environment') instead of deviceId
  - iOS Safari changes deviceIds on enumeration (unstable)
- **Desktop/Android**: Uses deviceId for camera selection
- **Auto-mirroring**: Front camera horizontally flips for natural selfie view

**Default Camera**:
- Mobile: Rear camera (environment)
- Desktop: Front camera (user)

## File Structure

```
costuma/
├── CLAUDE.md                 # This file - project context for Claude Code
├── README.md                 # User-facing documentation
├── vibe-coding.md            # Development log with technical details
├── site/
│   ├── basic-costume.html    # Main optimized AR application
│   ├── https-server.js       # Local dev server (port 8443)
│   ├── assets/
│   │   ├── RickAstley.gif
│   │   ├── GrumpyCat.png
│   │   ├── DisasterGirl.png
│   │   ├── Scream.png
│   │   └── Derpy.png
│   └── qr_costuma_512.png
├── docs/                     # Screenshots
└── reference/                # Original AI execution plans
```

## Code Patterns

### Adding a New Costume

1. **Add asset** (line ~437):
```javascript
const assets = {
    newcostume: './assets/NewCostume.png'
};
```

2. **Add button** (line ~102):
```html
<button class="costume-btn" onclick="setCostume('newcostume')">New Costume</button>
```

3. **Add to appropriate category** (line ~624):
```javascript
// For face costume:
if (costumeType === 'scream' || ... || costumeType === 'newcostume') {

// For body costume: add separate else-if block like 'rick'
```

4. **Set base size** (line ~662):
```javascript
} else if (costumeType === 'newcostume') {
    baseWidth = 400;
    baseHeight = 400;
}
```

### Adjusting Costume Size

Find the costume in the size section (lines 660-674) and modify `baseWidth`/`baseHeight`:
```javascript
} else if (costumeType === 'grumpy') {
    baseWidth = 320;  // Change this
    baseHeight = 320; // And this
}
```

## Known Issues

### 1. Scale Pumping
**Symptom**: Costumes rhythmically grow/shrink
**Cause**: Multiple people at different distances updating shared scale averages
**Partial Fix**: Global `seenScale` EMA (lines 124-125, 529-535)
**Status**: Reduced but not eliminated

### 2. iOS Camera Quirks
**Issue**: iOS Safari changes camera deviceIds on enumeration
**Workaround**: Use `facingMode` constraints instead of deviceId on iOS
**Status**: Working reliably

### 3. Fade-In/Fade-Out (Attempted)
**Attempted Feature**: Smooth opacity transitions for dropout handling
**Issues**:
- Position-based keys caused flickering (fixed with stable IDs)
- Score filtering removed fading people (fixed)
- Overall complexity too high
**Status**: Abandoned via git reset, code reverted to simpler version

## Development History

### Original Build (Oct 30-31, 2024)
- Built in ~4 hours with Claude Code
- Switchover from MediaPipe to PoseNet (WASM loading issues)
- Canvas 2D approach (simpler than Three.js)
- Multi-person depth sorting
- iOS camera switching solved

### Post-Halloween Session (Nov 26, 2024)
- Attempted fade-in/fade-out (abandoned)
- Costume size refinements (all scaled down 0.7x-0.9x)
- Added Derpy costume (640x640)
- Investigated scale pumping (partially improved)
- Git reset when complexity spiraled

See `vibe-coding.md` for detailed development log.

## Testing Checklist

When making changes:
- [ ] Test on iOS Safari (camera switching, performance)
- [ ] Test on Android Chrome
- [ ] Test with multiple people in frame
- [ ] Test camera switch (front/back)
- [ ] Check console for errors
- [ ] Verify all costumes load
- [ ] Check costume sizing at different distances

## Performance Notes

- **Target FPS**: 30+ on modern smartphones
- **Detection Resolution**: 513x513 (balance of accuracy/speed)
- **Canvas Size**: Matches video dimensions exactly (1:1 coordinate mapping)
- **Bottleneck**: PoseNet inference time (~30-50ms on mobile)

## Git Workflow

- **Main branch**: Stable, deployable to GitHub Pages
- **Git resets**: Used when experiments fail (see vibe-coding.md Phase 8)
- **No feature branches**: Fast iteration, reset on failure

## Future Ideas (Not Implemented)

- **Fade-in/fade-out**: Smooth opacity transitions (attempted, too complex)
- **3D models**: Three.js skeleton-rigged models (too slow on mobile)
- **One-Euro filter**: Velocity-adaptive smoothing (tried, "kind of a mess")
- **Kalman filtering**: More sophisticated tracking (not attempted)

## Debugging Tips

### Console Logging
- Triangle distance calculation: Line 693
- Person detection count: Line 583-585
- Drawing info: Line 617

### Common Issues
- **No detection**: Check camera permissions, HTTPS
- **Costumes not showing**: Check console for asset loading errors
- **Jittery motion**: Adjust `EMA_ALPHA_POSITION` (line 131)
- **Scale jumping**: Adjust `EMA_ALPHA_SCALE` (line 132)

### Debug Mode
Click "Debug Mode" button to see:
- Skeleton overlay
- Keypoint positions
- Keypoint labels
- Confidence scores

## Key Learnings

From the user's perspective:
1. **"This has all been hilarious"** - Complexity can spiral quickly
2. **Git reset is okay** - Sometimes starting over is better than debugging
3. **Test on real devices** - iOS behaves differently than desktop
4. **Iterate on sizing** - Physical testing reveals what works
5. **Position-based keys fail** - Need stable IDs for tracking moving people

## Contact / Origin

Built as a last-minute Halloween costume solution using:
- Claude Code (primary development)
- ChatGPT 5 (planning, QR codes)
- Gemini (technical architecture)
- Claude Opus 4.1 (initial artifact)

See README.md for full story and credits.

---
> Source: [chaboud/costuma](https://github.com/chaboud/costuma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
