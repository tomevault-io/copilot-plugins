## dexa

> A browser-based sandbox for editing and running Slang shaders in real-time on WebGPU.

# Dexa

A browser-based sandbox for editing and running Slang shaders in real-time on WebGPU.

## Quick Start

```bash
npm run dev      # Start dev server (http://localhost:3000)
npm run build    # Production build to dist/
npm run preview  # Preview build artifacts
```

## Architecture

```
src/
├── main.js                    # Application entry point (App class)
├── index.html                 # HTML (dev: uses import map, build: import map removed)
├── styles.css                 # All styles (uses CSS variables)
├── utils/
│   ├── editor.js              # CodeMirror 6 wrapper (ShaderEditor) with auto-compile
│   ├── slang-compiler.js      # slang-wasm wrapper (SlangCompiler)
│   ├── webgpu-renderer.js     # WebGPU renderer with multi-pass support
│   ├── texture-manager.js     # Texture loading, presets, camera input
│   ├── pass-manager.js        # Multi-pass state management (Buffer A-D)
│   ├── layout-manager.js      # Panel resize, layout, and fullscreen management
│   ├── capture-manager.js     # Screenshot and video recording
│   ├── ui-events.js           # DOM event wiring (UIEvents)
│   ├── error-display.js       # Error overlay management (ErrorDisplay)
│   ├── settings.js            # LocalStorage settings persistence
│   └── url-state.js           # URL state persistence (multi-pass support)
└── wasm/
    ├── slang-wasm.js          # Slang compiler (official v2026.2.2)
    └── slang-wasm.wasm        # 15MB - recommend git LFS
```

## UI Structure

- **Editor Panel**: Pass tabs (Main, A, B, C, D) with "+" button to add buffers
- **Preview Panel**: Canvas with texture slot buttons (iChannel0-3)
- **Floating Toolbar** (top-right): Play/Pause, Fullscreen, Download, Settings
- **Settings Modal**: Layout orientation, panel swap, editor theme, Copy URL
- **Download Modal**: Screenshot (PNG) and video recording (WebM)
- **Texture Modal**: Image upload, camera input, presets (noise/gradient/checker)
- **Auto-compile**: 500ms debounce after typing (no manual compile button)

## Key Patterns

### Module Structure
- ES Modules throughout (type: module)
- Dev: Dependencies fetched from esm.sh via import map
- Build: Bundled with Rollup, WASM kept as external module

### Slang → WebGPU Flow
1. SlangCompiler: Slang → WGSL conversion
2. WebGPURenderer: WGSL → GPU pipeline creation and execution

### Uniform & Binding Structure
```slang
// Binding 0: Uniforms (32 bytes)
struct Uniforms {
    float2 resolution;  // Canvas size (DPR aware)
    float time;         // Elapsed time (seconds)
    float _padding;
    float2 mouse;       // Mouse position (Y-flipped)
};

// Binding 1-5: Texture channels
[[vk::binding(1, 0)]] Texture2D<float4> iChannel0;
[[vk::binding(2, 0)]] SamplerState iSampler;
[[vk::binding(3, 0)]] Texture2D<float4> iChannel1;
[[vk::binding(4, 0)]] Texture2D<float4> iChannel2;
[[vk::binding(5, 0)]] Texture2D<float4> iChannel3;

// Binding 6-9: Buffer passes (previous frame output)
[[vk::binding(6, 0)]] Texture2D<float4> iBufferA;
[[vk::binding(7, 0)]] Texture2D<float4> iBufferB;
[[vk::binding(8, 0)]] Texture2D<float4> iBufferC;
[[vk::binding(9, 0)]] Texture2D<float4> iBufferD;
```

### Shader Entry Point
```slang
[shader("fragment")]
float4 fragMain(VertexOutput input) : SV_Target { ... }
```
- Outputs as `@fragment fn <name>` in WGSL
- Renderer auto-detects entry point name

### Multi-Pass Rendering
- **Execution order**: Buffer A → B → C → D → Main
- **Double buffering**: Each buffer has current/previous textures for feedback effects
- **Format**: rgba16float for high precision
- Buffer textures recreated on window resize

## Code Style

- Vanilla JavaScript (no TypeScript, no framework)
- Class-based design (ShaderEditor, SlangCompiler, WebGPURenderer, TextureManager, PassManager, LayoutManager, UIEvents, ErrorDisplay, CaptureManager)
- Singleton: SlangCompiler accessed via `getCompiler()`
- DOM elements cached in constructor or init()
- Async initialization uses `async init()` pattern

## Keyboard Shortcuts

- `Space`: Play/Pause
- `F`: Toggle fullscreen
- `1-5`: Switch pass (Main, A, B, C, D)
- `?`: Show help modal
- `Cmd/Ctrl + S`: Copy URL
- `Cmd/Ctrl + D`: Download modal
- `Cmd/Ctrl + ,`: Settings modal
- `Escape`: Close modal / Exit fullscreen

## Build

Rollup + Terser:
- `src/main.js` → `dist/main.js` (minified, sourcemap)
- slang-wasm treated as external (dynamic import preserved)
- Static files copied to dist via rollup-plugin-copy

## Browser Requirements

- WebGPU-capable browser required (Chrome 113+, Edge 113+)
- Feature detection via `navigator.gpu`

## Dependencies

Runtime:
- CodeMirror 6 (editor)
- slang-wasm (Slang compiler - from official playground)

Dev:
- Rollup, @rollup/plugin-node-resolve, @rollup/plugin-terser
- rollup-plugin-copy

---
> Source: [Nismit/dexa](https://github.com/Nismit/dexa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
