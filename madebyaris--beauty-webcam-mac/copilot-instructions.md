## project-structure

> BeautyWebcam project structure and organization guidelines


# BeautyWebcam Project Structure

## Project Overview
BeautyWebcam is a macOS application that provides real-time webcam enhancement through a virtual camera system. The project is built using Objective-C with native macOS frameworks.

## Key Project Files
- [PRD.md](mdc:PRD.md) - Product Requirements Document
- [todo.md](mdc:todo.md) - Development roadmap and task breakdown

## Core Architecture Components

### Main Application Structure
```
BeautyWebcam/
├── Sources/
│   ├── Application/          # App delegate and main application logic
│   ├── Core/                 # Core business logic and services
│   ├── Capture/              # Video capture and camera management
│   ├── Processing/           # Image processing and enhancement engine
│   ├── VirtualCamera/        # CoreMediaIO virtual camera implementation
│   ├── UI/                   # User interface components
│   └── Utils/                # Shared utilities and helpers
├── Resources/                # App resources, icons, assets
├── Frameworks/               # External frameworks and libraries
└── Tests/                    # Unit and integration tests
```

## Framework Usage
- **CoreMediaIO**: Virtual camera device creation
- **AVFoundation**: Video capture session management
- **Metal**: GPU-accelerated image processing
- **Core Image**: Real-time filter processing
- **AppKit**: Menu bar and UI components
- **IOKit**: Low-level hardware communication

## Performance Requirements
- CPU Usage: <15% on M1 MacBook Air
- Memory Usage: <150MB total footprint
- Latency: <50ms capture to output
- Frame Rate: 30fps minimum, 60fps target

## Code Organization Principles
1. Separate concerns: capture, processing, output
2. Use composition over inheritance
3. Implement proper error handling throughout
4. Follow Apple's memory management best practices
5. Use modern Objective-C features (nullable annotations, generics)

---
> Source: [madebyaris/beauty-webcam-mac](https://github.com/madebyaris/beauty-webcam-mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
