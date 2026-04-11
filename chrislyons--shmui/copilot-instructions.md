## shmui

> **Target Model:** Google Gemini 3 Pro (multimodal)

# shmui - Gemini 3 Pro Assistant Guide

**Target Model:** Google Gemini 3 Pro (multimodal)
**Repo:** shmui (Dual-Stack Audio & Agentic UI Components)
**Based On:** ElevenLabs UI / shadcn/ui (React) + JUCE C++ (Native)
**Tech Stack:** React 18+ + TypeScript + Tailwind CSS | JUCE C++ + OpenGL

Dual-stack component library for audio & agentic applications: React/TypeScript for web, JUCE C++ for native desktop.

**CRITICAL:** JUCE components are integrated into Orpheus SDK as `packages/shmui-juce/`. This repo maintains source of truth for JUCE components, synced to Orpheus SDK.

---

## Project Context

**shmui** is a dual-stack component library built on [shadcn/ui](https://ui.shadcn.com/) (React) with JUCE C++ ports for native audio applications.

## Integration with Orpheus SDK

**Status:** JUCE components integrated into `~/dev/orpheus-sdk/packages/shmui-juce/`

**Why Integration:**
- Orpheus SDK is a professional C++20 audio SDK with broadcast-safe, deterministic audio engine
- shmui JUCE components provide audio visualization for Orpheus applications (Clip Composer, Wave Finder, FX Engine)
- React components remain in shmui for web-based tools

**Threading Compatibility:**
- shmui `AudioAnalyzer` uses lock-free atomics → compatible with Orpheus broadcast-safe audio threads (ZERO allocations)
- shmui visualization components require JUCE message thread → matches Orpheus threading model
- Both follow: NO audio thread allocations, NO locks on audio thread

**Synchronization Model:**
- `juce/Source/` in this repo is **source of truth**
- Changes sync to `~/dev/orpheus-sdk/packages/shmui-juce/` manually via `rsync`
- Orpheus SDK apps consume from package

**See Orpheus SDK docs:**
- `~/dev/orpheus-sdk/CLAUDE.md`, `~/dev/orpheus-sdk/GEMINI.md`
- `~/dev/orpheus-sdk/docs/ARCHITECTURE.md` (threading model, broadcast-safe constraints)
- `~/dev/orpheus-sdk/packages/shmui-juce/` (integrated JUCE components)

**Dual Stack:**
1. **React/TypeScript** (web) - Registry-based shadcn/ui components
2. **JUCE C++** (native) - Ports of React components for desktop audio apps

**Component Categories:**
- **Audio Visualizers:** Orbs, waveforms, bar displays, matrix displays
- **Agent Components:** Voice agents, conversational UI (React only)
- **Audio Analysis:** FFT, RMS, frequency bands (JUCE only)
- **General UI:** Buttons, dialogs, forms (React only, from shadcn/ui)

**Distribution:**
- **React:** Registry-based (copy-paste, not npm)
- **JUCE:** Header-only library (`juce/Source/ShmUI.h`)

---

## Repository Structure

```
shmui/
├── GEMINI.md              # This file
├── .codex/AGENTS.md       # Codex CLI config
├── CLAUDE.md              # Claude Code config (reference)
├── README.md              # Installation, usage
├── CONTRIBUTING.md        # Contributing guidelines
├── registry/              # React component registry (JSON configs)
│   ├── components/        # React component definitions
│   └── blocks/            # Example blocks
├── juce/                  # JUCE C++ components
│   └── Source/
│       ├── ShmUI.h        # Main include header
│       ├── Audio/         # AudioAnalyzer (FFT, RMS)
│       ├── Components/    # Visualizers (Orb, Waveform, Bar, Matrix)
│       ├── Shaders/       # OpenGL shaders (Orb)
│       └── Utils/         # Interpolation, ColorUtils
├── apps/                  # Example apps
├── docs/                  # Documentation
└── tests/                 # Component tests
```

---

## React/TypeScript Components

**How It Works:**
```bash
# Add React component
npx @elevenlabs/cli@latest components add <component-name>

# Or via shadcn/ui CLI
npx shadcn@latest add https://ui.elevenlabs.io/r/<component>.json
```

**Design Principles:**
- Accessibility First (WCAG 2.2 AA)
- Responsive Design (Mobile-First)
- Theming Support (light/dark mode via CSS variables)
- Component Variants (Class Variance Authority)

---

## JUCE C++ Components

**Critical Translation Context:**
- React → JUCE Component ports
- TypeScript → C++20
- React state → JUCE message thread updates

**JUCE Components:**
- `AudioAnalyzer` - FFT, RMS, frequency band analysis
- `WaveformVisualizer` - Multiple waveform display variants
- `BarVisualizer` - Frequency band display with state animations
- `OrbVisualizer` - OpenGL shader-based 3D orb
- `MatrixDisplay` - LED-style matrix with animations

**Usage:**
```cpp
#include "ShmUI.h"

// Create visualizer
auto orb = std::make_unique<shmui::OrbVisualizer>();

// Create audio analyzer
shmui::AudioAnalyzer analyzer;

// Connect to audio source (audio thread safe)
analyzer.processAudioBlock(buffer, numSamples, sampleRate);

// Update UI (message thread)
juce::MessageManager::callAsync([&]() {
    orb->setAudioLevel(analyzer.getRMS());
});
```

**Threading Model (CRITICAL):**
- `AudioAnalyzer` - Thread-safe for audio/UI communication
- Visualization components - Message thread only
- Use `juce::MessageManager::callAsync()` for cross-thread updates
- NO allocations on audio thread

---

## Gemini 3 Pro-Specific Guidance

**Leverage multimodal capabilities:**
- Component screenshots → Analyze layout (React) or rendering (JUCE/OpenGL)
- Design mockups → Suggest React/Tailwind OR JUCE/Graphics implementation
- Waveform visualizations → Compare React canvas vs JUCE Graphics approaches
- C++ code → Analyze threading safety, JUCE patterns

**Network access for:**
- **React/Web:**
  - shadcn/ui: https://ui.shadcn.com/
  - Tailwind CSS: https://tailwindcss.com/docs
  - Web Audio API: https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API
- **JUCE/C++:**
  - JUCE documentation: https://juce.com/learn/documentation
  - JUCE tutorials: https://juce.com/learn/tutorials
  - OpenGL: https://www.khronos.org/opengl/wiki/
  - C++20: https://en.cppreference.com/

**Best practices:**
- ASK which stack (React or JUCE) before suggesting implementations
- FLAG audio thread violations in JUCE code immediately
- VERIFY accessibility for React components (WCAG 2.2 AA)
- SUGGEST thread-safe patterns for JUCE audio integration

---

## Quick Commands

**React/TypeScript:**
```bash
# Development
pnpm install
pnpm run dev
pnpm run type-check
pnpm run build
pnpm run test
pnpm run test:a11y  # Accessibility tests
```

**JUCE C++:**
```bash
# Include in your JUCE project:
# 1. Add juce/Source to header search paths
# 2. #include "ShmUI.h"
# 3. Link JUCE modules (juce_gui_basics, juce_opengl, juce_dsp)
```

---

## Dual-Stack Patterns

**Shared Concepts (Different Implementations):**

| Feature | React/TypeScript | JUCE C++ |
|---------|------------------|----------|
| Audio Analysis | Web Audio API | juce::dsp::FFT |
| Waveform Display | Canvas 2D API | juce::Graphics |
| Orb Visualizer | WebGL shaders | OpenGL shaders |
| State Management | React hooks | JUCE ValueTree |
| Threading | Web Workers | juce::MessageManager |
| Styling | Tailwind CSS | juce::LookAndFeel |

**When to Use Which:**
- **React:** Web apps, cross-platform web UIs, agent interfaces
- **JUCE:** Native desktop apps, DAWs, audio plugins, low-latency requirements

---

## Audio Component Patterns

### React Orb Component
```tsx
<Orb
  audioLevel={level}
  isActive={isPlaying}
  className="h-32 w-32"
/>
```

### JUCE Orb Component
```cpp
auto orb = std::make_unique<shmui::OrbVisualizer>();
orb->setAudioLevel(analyzer.getRMS());  // Message thread
orb->setBounds(0, 0, 128, 128);
```

**Both provide:**
- Audio-reactive animations
- Customizable colors
- Performance-optimized rendering

---

## Threading Considerations (JUCE)

**AudioAnalyzer (Thread-Safe):**
```cpp
// Audio thread (safe)
void processBlock(juce::AudioBuffer<float>& buffer) {
    analyzer.processAudioBlock(buffer.getReadPointer(0),
                                buffer.getNumSamples(),
                                sampleRate);
}

// Message thread (safe)
void timerCallback() override {
    auto rms = analyzer.getRMS();
    visualizer.setAudioLevel(rms);
}
```

**Visualization Components (Message Thread Only):**
```cpp
// WRONG: Called from audio thread
void processBlock(...) {
    visualizer.paint(g);  // CRASH! Not thread-safe
}

// CORRECT: Use message thread
juce::MessageManager::callAsync([this]() {
    visualizer.repaint();
});
```

---

## See `CLAUDE.md` and `CONTRIBUTING.md` for:

- Detailed workflow conventions
- React component creation process
- JUCE integration patterns
- Git commit format and PR templates
- Testing protocols
- Documentation standards

**Key external resources:**
- Upstream (React): https://ui.elevenlabs.io/ (ElevenLabs UI)
- Base system (React): https://ui.shadcn.com/ (shadcn/ui)
- Native framework: https://juce.com/ (JUCE)

---

**Version:** 2.0 (Lean, dual-stack aware)
**Last Updated:** 2025-11-25
**Model Target:** Google Gemini 3 Pro (multimodal, November 2025)
**Environment:** Sandboxed to repository, network access enabled

**Remember:** shmui is a DUAL-STACK component library. Always clarify whether the user is working with React/TypeScript (web) or JUCE C++ (native) components. For React, prioritize accessibility (WCAG 2.2 AA) and mobile-first design. For JUCE, prioritize thread safety (NO audio thread allocations) and real-time performance.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chrislyons)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chrislyons)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
