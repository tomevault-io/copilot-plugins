## godviewactivation

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GodViewActivation is a web-based space tourism application that simulates viewing Earth from space to evoke the "Overview Effect" - a cognitive shift characterized by awe, interconnectedness, and responsibility toward Earth. The application supports both VR mode (via WebXR) and standard web browser mode.

**Target Platforms**: Chrome 90+, Edge 90+, Firefox 90+, Safari 15+, VR headsets via WebXR
**Deployment**: Static web app (no backend), hosted via HTTPS for WebXR support

## Technology Stack

- **Frontend**: HTML5, JavaScript/TypeScript
- **3D Rendering**: Three.js (3D models, camera controls, textures)
- **VR Integration**: WebXR API
- **Audio**: Web Audio API
- **Deployment**: Static hosting (Vercel, Netlify, or GitHub Pages)

## Architecture Principles

### Modular Structure
The codebase should be organized into separate modules:
- **Controls Module**: Handles mouse, keyboard, touch, and VR controller inputs
- **Rendering Module**: Manages Three.js scene, Earth model, space environment, camera
- **UI Module**: HUD overlay, menu system, settings, accessibility features
- **Audio Module**: Ambient sounds, narration, background music management
- **VR Module**: WebXR integration, headset detection, VR-specific controls

### Performance Requirements
- **Target**: 60 FPS rendering
- **Progressive Loading**: Low-res Earth textures first, then high-res
- **Fallback Strategy**: WebGL preferred → Canvas 2D fallback if WebGL unavailable
- **Optimization**: Detect device capabilities and adjust graphics quality automatically
- **Mobile**: Responsive design with performance optimizations (reduced resolution on mobile)

### Visual Fidelity Standards
- **Earth Textures**: High-resolution, photorealistic (NASA Blue Marble/Visible Earth datasets)
- **Atmosphere**: Glow effects, clouds, weather patterns
- **Space Environment**: Procedural starfield, Moon, Sun with lens flare
- **Style**: Photorealism over cartoonish; emphasize Earth's isolation and beauty

## Control Schemes

### Normal Browser Mode
- **Mouse**: Left-drag (rotate), wheel (zoom), right-drag (pan/fly)
- **Keyboard**: WASD (movement), Spacebar/Shift (ascend/descend), arrows (fine rotation)
- **Touch**: Pinch-to-zoom, drag (rotate), two-finger drag (pan)

### VR Mode
- **Headset**: Natural rotation via head tracking
- **Controllers**: Triggers (zoom), joysticks (flying/movement)

### General Controls
- Smooth transitions with inertia-based movement for "weightless" feel
- Speed controls (slider/buttons) for adjustable flying speed
- Reset button to return to default orbital view

## Key Features to Implement

### Core Experience
1. **Dual Mode Support**: VR (WebXR) and normal browser mode with automatic headset detection
2. **Earth Representation**: Day/night cycle (real-time or simulated), atmospheric effects
3. **Audio System**: Ambient space sounds, optional astronaut narration, calming background music
4. **UI/HUD**: Mode selector, control hints, zoom indicator, settings menu

### Enhancement Features (Post-MVP)
- Guided tours with pre-set flight paths and narration
- Educational overlays (facts, environmental stats)
- Multi-view options (Mars, Moon comparisons)
- Meditation mode with slow-motion flying
- Screenshot/sharing features
- Real-time weather data integration (OpenWeatherMap API)
- User customization (starting positions)
- Progression system (achievements)
- Voice controls via Web Speech API
- Post-experience survey for Overview Effect feedback

## Development Guidelines

### Code Organization
- Use modular, reusable components
- Separate concerns: rendering logic, input handling, UI, audio
- TypeScript preferred for type safety
- Inline comments for complex Three.js/WebXR interactions

### Browser Compatibility
- Test across Chrome, Edge (with IE fallback via polyfills), Firefox, Safari
- Ensure WebXR works on compatible headsets (Oculus Quest, HTC Vive)
- Implement feature detection for WebXR, WebGL, Web Audio API

### Assets and Resources
- Use public domain Earth textures (NASA datasets)
- Royalty-free or generated audio tracks
- Optimize asset loading (progressive/lazy loading)

### Testing Strategy
- Unit tests for control input handlers
- Browser compatibility tests across target platforms
- Performance profiling to maintain 60 FPS
- VR testing on actual headsets

### Security and Privacy
- No user data collection
- All processing client-side
- HTTPS required for WebXR API access

## Project Status

This is an early-stage project. The README.md contains comprehensive requirements. Implementation should start with an MVP focusing on:
1. Basic Three.js scene with Earth model
2. Camera controls (mouse/keyboard)
3. WebGL rendering pipeline
4. Simple UI overlay

Then iterate toward VR support, enhanced visuals, and optional features.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vincent7q)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/vincent7q)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
