## contentflow

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **professional Remotion video component library** for creating tech tutorial videos with Apple-style aesthetics and FilmStorm-level quality.

**Goal**: Build a reusable component library for AI tech content creators, featuring:
- Apple-style minimal design (#000000 background + #007AFF accent)
- Organic, non-mechanical animations
- Professional SVG icons (no emoji)
- Code demonstrations with syntax highlighting
- Data visualization and metrics

---

## Development Commands

```bash
# Start Remotion Studio (live preview at http://localhost:3000)
npm run dev

# Run linting (ESLint + TypeScript)
npm run lint

# Bundle for rendering
npm run build

# Render specific composition
npx remotion render src/index.ts <CompositionID> out/video.mp4

# Upgrade Remotion
npm run upgrade
```

---

## Architecture

### Design System (src/design-system/)

**tokens.ts** - Centralized design constants:
- Colors: Apple-style palette (#000000 bg, #007AFF primary)
- Fonts: SF Pro Display/Text, JetBrains Mono
- Sizes: Large typography (hero: 120px, h1: 88px)
- Spacing, radius, animation durations

**animations.ts** - Animation presets:
- Spring presets: smooth, snappy, bouncy, heavy, playful
- Entrance animations: fade, slideUp, slideLeft, scale
- Decorative: lineExpand, glowSweep, breathe

### Component Library (src/components/new/)

All new components go here. Import via `src/components/new/index.ts`.

#### Core Components

| Component | Purpose | Key Features |
|-----------|---------|--------------|
| **HeroTitle** | Opening titles | Reveal animation, glow sweep, tags |
| **SectionTitle** | Chapter headers | Section number, progress bar, left indicator |
| **CodeTerminal** | Code display | macOS terminal style, line-by-line typing, syntax highlight |
| **AnimatedList** | Feature lists | Staggered entrance, icons, check animations |
| **FeatureCard** | Feature showcases | Icon + title + description, grid layout (isFallback) |
| **FeatureGrid** | Feature grid | Grid of FeatureCard components |
| **MetricCard** | Data metrics | Animated number counting, large typography |
| **MetricRow** | Metric row | Horizontal row of MetricCard components |
| **ComparisonCards** | Comparisons | Side-by-side comparison with highlight |
| **ProductIntro** | Product intro | Full-screen product showcase with features |
| **SubtitleOverlay** | Subtitles | Word-level subtitle sync with fade animations |
| **Transitions** | Scene transitions | Fade, Slide, LightSweep, ZoomBlur, CurtainReveal |

#### Data & Charts

| Component | Purpose | Key Features |
|-----------|---------|--------------|
| **BarChart** | Bar charts | Animated bars via Recharts, style-aware |
| **LineChart** | Line charts | Progressive reveal, style-aware |
| **PieChart** | Pie charts | Animated slices, style-aware |
| **DataTable** | Tables | Animated rows with zebra striping |
| **HighlightQuote** | Quotes | Highlighted quote with attribution |
| **DataHighlight** | Data callout | Emphasized data point display |

#### Visual & Effects

| Component | Purpose | Key Features |
|-----------|---------|--------------|
| **ThreeScene** | 3D scenes | Three.js canvas via @remotion/three |
| **MotionBlurWrapper** | Motion blur | CameraMotionBlur from @remotion/motion-blur |
| **LightLeakOverlay** | Light effects | Cinematic light leak from @remotion/light-leaks |
| **LottieAnimation** | Lottie | Lottie JSON animation playback |
| **CausalGraph** | Causal diagrams | Node-link diagram with animated edges |
| **EvolutionTree** | Evolution | Tree diagram showing version progression |
| **KnowledgeWeb** | Knowledge graph | Interactive web of concepts |
| **ProcessFlow** | Flow diagrams | Step-by-step process visualization |

#### Text Effects

| Component | Purpose | Key Features |
|-----------|---------|--------------|
| **TypewriterText** | Typing effect | Character-by-character reveal |
| **TypewriterScene** | Typing scene | Full scene with TypewriterText |
| **CommentBubble** | Comments | Chat/comment bubble with avatar |

#### Icon Library (src/components/new/Icons.tsx)

**50+ SVG icons** based on Heroicons (MIT) and Lucide (ISC) - free for commercial use.

Common icons:
- Tech: Computer, Bot, Terminal, Code, Database, Network
- Actions: Check, Plus, Minus, Close, ArrowRight, Download
- General: User, File, Folder, Calendar, Clock, Settings

**Usage**:
```tsx
import { Zap, Lock, Computer } from '../components/new/Icons';
<Zap size={24} color="#007AFF" strokeWidth={2} />
```

**Important**: Never use emoji. Always use icons from Icons.tsx.

---

## Component Usage Patterns

### HeroTitle
```tsx
<HeroTitle
  title="OpenClaw"
  subtitle="完全新手指南"
  tags={["AI 编码助手", "开源免费", "自托管部署"]}
/>
```

### CodeTerminal
```tsx
<CodeTerminal
  code={`npm install -g openclaw`}
  language="bash"
  filename="install.sh"
  typingSpeed={1}
  showLineNumbers={true}
/>
```

### AnimatedList (with icons)
```tsx
<AnimatedList
  items={[
    { title: '...', description: '...', icon: 'computer' },
    { title: '...', description: '...', icon: 'bot' },
  ]}
  variant="card"
/>
```

### FeatureGrid (with icons)
```tsx
<FeatureGrid
  features={[
    { icon: 'zap', title: '极速响应', description: '...' },
    { icon: 'lock', title: '隐私安全', description: '...' },
  ]}
  columns={3}
/>
```

---

## Animation Guidelines

### Do's
- Use `useCurrentFrame()` for all animations
- Use `spring()` for natural motion
- Use `interpolate()` with easing functions
- Stagger list items with frame delays

### Don'ts
- **Never use CSS animations** (they won't render)
- **Never use Tailwind animation classes**
- **Never use emoji** (use SVG icons instead)
- Avoid mechanical timing (use organic easing)

### Example Pattern
```tsx
const frame = useCurrentFrame();
const { fps } = useVideoConfig();

// Spring animation
const scale = spring({
  frame,
  fps,
  config: { damping: 25, stiffness: 100 },
});

// Interpolation with easing
const opacity = interpolate(frame, [0, 20], [0, 1], {
  extrapolateLeft: 'clamp',
  easing: Easing.out(Easing.quad),
});
```

---

## Styling Guidelines

### Dynamic Style Tokens (useStyleTokens)

All components use `useStyleTokens()` hook for dynamic theming, NOT static imports.

```tsx
import { useStyleTokens } from '../../compositions/SceneRenderer';

const { colors, fonts, sizes, durations, subtitle } = useStyleTokens();
```

Three style presets available:
- `apple-tech` — Apple-style dark theme (#000000 bg, #007AFF accent)
- `fast-code` — Developer-focused, different color palette
- `data-viz` — Data visualization optimized

Effective-value pattern for props that should fallback to style tokens:
```tsx
const color = propColor ?? colors.primary;
```

### Style Context

Styles are provided by `StyleContext` in `src/styles/`:
- `registry.ts` — StyleRegistry class + useStyleTokens() hook
- `apple-tech.ts`, `fast-code.ts`, `data-viz.ts` — 3 style presets
- `index.ts` — StyleContext provider

DynamicVideo wraps all compositions with StyleContext. The background color reads from the active style preset.

### Typography Scale
- hero: 120px (main titles)
- h1: 88px (section titles)
- h2: 56px (subsections)
- h3: 40px (card titles)
- h4: 28px (small headers)
- body: 20px (descriptions)

---

## File Structure

```
src/
├── design-system/          # Design tokens and animation utilities
│   ├── tokens.ts           # Default COLORS, FONTS, DURATIONS (fallback values)
│   └── animations.ts       # Spring presets, entrance animations
├── styles/                 # Style system (dynamic theming)
│   ├── registry.ts         # StyleRegistry + useStyleTokens() hook
│   ├── apple-tech.ts       # Apple-tech style preset
│   ├── fast-code.ts        # Fast-code style preset
│   ├── data-viz.ts         # Data-viz style preset
│   └── index.ts            # StyleContext provider
├── components/new/         # Component library (28+ components)
│   ├── index.ts            # Barrel exports
│   ├── componentCatalog.ts # Component registry with Zod schemas
│   ├── Icons.tsx            # 50+ SVG icons
│   ├── HeroTitle.tsx, SectionTitle.tsx, CodeTerminal.tsx, ...
│   ├── BarChart.tsx, LineChart.tsx, PieChart.tsx
│   ├── ThreeScene.tsx, MotionBlurWrapper.tsx, LightLeakOverlay.tsx
│   └── ... (see Component tables above)
├── compositions/           # Remotion compositions
│   ├── DynamicVideo.tsx    # Main composition with TransitionSeries
│   ├── SceneRenderer.tsx   # Scene component resolver
│   └── componentMap.ts     # componentName → React component map
├── pipeline/               # Video generation pipeline (7 stages)
│   ├── run.ts              # CLI entry point (--topic, --style, --output)
│   ├── VideoPipeline.ts    # Pipeline orchestrator with resume
│   ├── ScriptGenerator.ts  # LLM → VideoScript
│   ├── TtsService.ts       # VideoScript → TTS audio
│   ├── WhisperTranscriber.ts # Audio → word-level timestamps
│   ├── AudioMixer.ts       # Audio normalization + ducking
│   ├── ComponentSelector.ts # VideoScript → SceneList (with LLM)
│   ├── TimelineBuilder.ts  # SceneList → Timeline with timing
│   ├── benchmark.ts        # Render performance benchmarking
│   └── schemas/            # Zod schemas for pipeline data contracts
├── config/                 # Configuration
│   └── render-profiles.ts  # 4 render presets (1080p, 720p, 4K, portrait)
├── scenes/demo/            # Demo compositions
├── Root.tsx                # Composition definitions
└── index.ts                # Entry point
```

---

## Dependencies

Key packages (all @remotion/* locked to 4.0.448):
- `remotion` @ 4.0.448 - Video rendering engine
- `@remotion/transitions` - Scene transitions (fade, slide, wipe, iris, flip, clockWipe)
- `@remotion/renderer` + `@remotion/bundler` - Programmatic rendering
- `@remotion/three` + `three` + `@react-three/fiber` - 3D scenes
- `@remotion/motion-blur` - CameraMotionBlur effect
- `@remotion/light-leaks` - Cinematic light leak overlays
- `@remotion/lottie` - Lottie animation playback
- `@remotion/captions` - Subtitle/caption data
- `@remotion/google-fonts` - Font loading (Noto Sans SC for CJK)
- `@remotion/media-utils` - Audio analysis utilities
- `@remotion/install-whisper-cpp` - Whisper.cpp for transcription
- `tailwindcss` @ 4.0.0 - Styling (via @remotion/tailwind-v4)
- `react` @ 19.2.3
- `zod` @ 4.3.6 - Schema validation
- `recharts` - Chart components
- `vitest` - Testing (184 tests across 17 files)

---

## TTS / Voiceover

Implemented via Kokoro TTS (local, FastAPI on localhost:8000):
- `TtsService` generates audio from VideoScript segments
- `WhisperTranscriber` generates word-level timestamps for subtitle sync
- `AudioMixer` normalizes audio and applies background music ducking
- Supports Chinese and English TTS output

---

## Pipeline

### 7-Stage Video Generation Pipeline

```
ScriptGenerator → TtsService → WhisperTranscriber → AudioMixer → ComponentSelector → TimelineBuilder → VideoPipeline
```

### CLI Usage
```bash
# Generate video from a topic
npx tsx src/pipeline/run.ts --topic "Ollama 本地部署指南" --style apple-tech

# With options
npx tsx src/pipeline/run.ts \
  --topic "test topic" \
  --style fast-code \
  --output output/my-video.mp4 \
  --profile 720p \
  --benchmark
```

### Data Contracts (Zod schemas)
- `VideoScript` — Script with title, topic, segments (narration + visual descriptions)
- `SceneList` — Resolved scenes with componentName, props, transition
- `AudioTiming` — Word-level timestamps for subtitle sync
- `RenderProfile` — Resolution, FPS, codec settings

### Component Registry
- `componentCatalog.ts` — All 28+ components with Zod props schemas
- `componentMap.ts` — componentName → React component resolver
- `SubtitleOverlay` — Word-synced subtitles from AudioTiming data

---

## Testing

```bash
# Run all tests (Vitest)
npx vitest run

# Run specific test
npx vitest run src/pipeline/__tests__/script-generator.test.ts

# Test count: 184 tests across 17 files
```

---

## Contact

For questions about this component library, refer to:
- `src/components/new/` - Component implementations
- `src/design-system/` - Design tokens (fallback defaults)
- `src/styles/` - Style system and presets

<!-- GSD:project-start source:PROJECT.md -->
## Project

**CreatorOS**

CreatorOS 是一个基于 Remotion 的 AI 内容生产操作系统。输入一个技术主题，系统自动生成脚本、配音、视觉素材，最终通过 Remotion 渲染出 FilmStorm 级别的高质量视频（1080p+，酷炫动画，专业配音）。

首发场景为**技术教程视频（16:9）**，后续扩展到短视频（9:16）、科技宣传片、数据动画等。目标平台覆盖 B 站、YouTube、抖音、小红书等全平台。

**Core Value:** Remotion 能力的全面运用 — 把 Remotion 的动画、转场、3D、字幕、图表、文字特效、音频同步等全部能力拉满，封装成可组合的 Skills，建立多风格模板库，让 AI 一键生成专业级视频。

### Constraints

- **技术栈**: Remotion（核心）+ React + TypeScript + Tailwind — 视频渲染必须基于 Remotion
- **硬件**: M4 Pro 48G，本地渲染，需要合理控制渲染时间
- **TTS**: 优先本地模型（Ollama），不依赖付费云服务
- **图片素材**: 代码生成为主，AI 图片为辅
- **风格**: 视频必须有 FilmStorm 级别的视觉质量，不能看起来像模板
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Recommended Stack
### Core Framework
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **Remotion** | 4.0.448 | Video rendering engine | The only mature React-based programmatic video framework. Rich API, 25+ official packages, active development. Version 4.0.448 is current latest; ALL @remotion/* packages must be locked to the same version to avoid runtime incompatibilities. |
| **React** | 19.2.x | Component model | Required by Remotion 4.x. Already at 19.2.3 in project; latest is 19.2.5. Minor update, no breaking changes. |
| **TypeScript** | 5.9.x | Type safety | Already at 5.9.3 in project. Strict mode recommended for pipeline type contracts (Script JSON, SceneList JSON). |
| **Tailwind CSS** | 4.0.x | Styling utility | Already at 4.0.0. Use via `@remotion/tailwind-v4` for Remotion-compatible integration. Note: Tailwind `animate-*` classes are FORBIDDEN in Remotion -- they use CSS animations which flicker during render. Use Tailwind only for layout and static styles. |
| **Zod** | 4.3.x | Schema validation | Already at 4.3.6. Critical for validating data contracts between pipeline stages (Script JSON, SceneList JSON, AudioTiming JSON). Every skill boundary must have a Zod schema. |
### Remotion Official Packages (All Must Be Same Version: 4.0.448)
| Package | Status | Purpose | Priority |
|---------|--------|---------|----------|
| `remotion` | Installed (4.0.434) | Core framework | Phase 1 -- Upgrade immediately |
| `@remotion/cli` | Installed (4.0.434) | Studio + render commands | Phase 1 -- Upgrade |
| `@remotion/captions` | Installed (4.0.434) | Subtitle/caption data, SRT import/export | Phase 2 -- Upgrade |
| `@remotion/google-fonts` | Installed (4.0.434) | Font loading (includes Noto Sans SC for CJK) | Phase 1 -- Upgrade + add Noto Sans SC |
| `@remotion/media` | Installed (4.0.434) | Media handling primitives | Phase 1 -- Upgrade |
| `@remotion/media-utils` | Installed (4.0.434) | `getAudioData()`, `visualizeAudio()`, `getAudioDurationInSeconds()` | Phase 2 -- Upgrade |
| `@remotion/paths` | Installed (4.0.434) | SVG path operations | Phase 1 -- Upgrade |
| `@remotion/tailwind-v4` | Installed (4.0.434) | Tailwind 4 integration | Phase 1 -- Upgrade |
| `@remotion/transitions` | Installed (4.0.434) | `<TransitionSeries>`, scene transitions | Phase 1 -- Upgrade |
| `@remotion/zod-types` | Installed (4.0.434) | Zod types for Remotion props | Phase 1 -- Upgrade |
| `@remotion/renderer` | **Missing** | Programmatic render control, batch rendering | Phase 1 -- Install for pipeline |
| `@remotion/bundler` | **Missing** | Webpack bundling for programmatic render | Phase 1 -- Install (peer of @remotion/renderer) |
| `@remotion/three` | **Missing** | `<ThreeCanvas>`, `useVideoTexture()` for 3D scenes | Phase 4 -- Install when building 3D scenes |
| `@remotion/motion-blur` | **Missing** | Motion blur visual effect | Phase 3 -- Install for visual polish |
| `@remotion/shapes` | **Missing** | SVG shape primitives (`makePie()`, `makeRect()`, `makeTriangle()`) | Phase 3 -- Install for chart building |
| `@remotion/lottie` | **Missing** | Lottie animation integration | Phase 3 -- Install for complex animations |
| `@remotion/gif` | **Missing** | GIF playback in compositions | Phase 2 -- Install if GIF assets needed |
| `@remotion/install-whisper-cpp` | **Missing** | Whisper.cpp integration for transcription | Phase 2 -- Install for caption timing |
| `@remotion/light-leaks` | **Missing** | WebGL-based cinematic light leak overlays | Phase 3 -- Install for FilmStorm quality |
| `@remotion/skia` | **Missing** | Skia 2D graphics rendering | Deferred -- Only if canvas-based graphics needed |
| `@remotion/rive` | **Missing** | Rive animation integration | Deferred -- Only if Rive assets available |
### Animation Libraries (Compatible with Remotion)
| Library | Version | Compatible? | Purpose | When to Use |
|---------|---------|-------------|---------|-------------|
| **Remotion `spring()` + `interpolate()`** | Built-in | YES (native) | All animation. This is the primary animation system. | Always. This is the default. |
| **CSS `@keyframes` / `animation`** | N/A | **NO** | -- | **NEVER.** Flickers during render because Remotion renders frames out of order across tabs. |
| **Tailwind `animate-*`** | N/A | **NO** | -- | **NEVER.** Same reason as CSS animations. |
| **Framer Motion** | N/A | **NO** | -- | **NEVER.** Uses `requestAnimationFrame` and time-based logic. No official Remotion integration exists. |
| **react-spring** | N/A | **NO** | -- | **NEVER.** Same reason as Framer Motion. |
| **GSAP** | 3.12.x | YES | Complex timeline animations, scroll-linked effects. Official Remotion integration via `<GsapSyncedAnimation>`. | Only when Remotion's built-in `spring()`/`interpolate()` cannot express the desired motion. Most Remotion projects do NOT need GSAP. |
| **Lottie** (via `@remotion/lottie`) | 4.0.448 | YES | Pre-built complex animations from After Effects | When a designer provides Lottie assets. Use `lottie-web` 5.13.x alongside `@remotion/lottie`. |
| **Anime.js** | 3.2.x | YES | Lightweight JS animation. Official Remotion example exists. | Rarely needed. Prefer Remotion's built-in animation system. |
### Database / Storage
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **File system (JSON)** | N/A | Script storage, scene lists, audio timing, project artifacts | The pipeline produces JSON artifacts at each stage. `projects/{id}/script.json`, `scenes.json`, `audio/*.timing.json`. Simple, version-controllable, grep-able. |
| **File system (Media)** | N/A | Audio files, images, rendered video | `public/assets/audio/{project}/`, `output/{project}/video.mp4`. |
### Local AI Models
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| **Ollama** | Latest | LLM and TTS inference runtime | Already installed on M4 Pro 48G. Manages model loading, inference, and API serving. Node.js client: `ollama` package 0.6.x. | HIGH |
| **Qwen 3 (via Ollama)** | Latest | Script generation, Chinese-English bilingual content | Best bilingual (Chinese+English) LLM for this use case. Strong technical writing. Qwen 3 8B or 14B fits in M4 Pro 48G with room for TTS model. | HIGH |
| **DeepSeek (via Ollama)** | Latest | Alternative for technical content | Excellent at code-related explanations. Use as fallback when Qwen struggles with specific technical topics. | MEDIUM |
| **Kokoro TTS (via Ollama)** | 82M params | Lightweight local TTS | Very small model (82M), fast inference, good quality. Good candidate for rapid iteration. Test Chinese quality specifically. | MEDIUM |
| **Qwen3-TTS** | Latest | Chinese-English bilingual TTS | Recently open-sourced. Supports voice cloning. MLX-optimized for Apple Silicon. Best bilingual quality candidate. | MEDIUM -- newly released, needs evaluation |
| **Edge TTS** | N/A | Online TTS via Microsoft | **DO NOT USE as primary.** The project requires local TTS (no network dependency). Edge TTS is online-only and the project already has 3 redundant edge-tts packages installed -- remove them. | HIGH (that it should be removed) |
### Audio Processing
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **FFmpeg** | System install | Audio normalization, format conversion, video concatenation | Already a Remotion dependency. Use for: normalizing TTS output to 48kHz WAV, trimming silence, concatenating segments. |
| **@remotion/media-utils** | 4.0.448 | `getAudioDurationInSeconds()`, `getAudioData()`, `visualizeAudio()` | Official Remotion package. Essential for audio-driven scene timing and waveform visualization. |
| **@remotion/install-whisper-cpp** | 4.0.448 | Whisper.cpp for audio transcription | Generates word-level timestamps from TTS audio. Critical for subtitle sync. Runs locally on M4 Pro via Metal acceleration. |
| **@remotion/captions** | 4.0.448 | Caption data model, SRT import/export | Official Remotion caption handling. Provides `parseCaptions()`, `serializeCaptions()`, and caption data types. |
### 3D Capabilities
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **@remotion/three** | 4.0.448 | React Three Fiber integration with Remotion | Official package. Provides `<ThreeCanvas>` component that integrates Three.js rendering into Remotion's frame-based pipeline. Also provides `useVideoTexture()` for using video as a 3D texture. |
| **@react-three/fiber** | 9.5.x | React renderer for Three.js | Required peer dependency of `@remotion/three`. React-based 3D scene composition. |
| **three** | 0.183.x | 3D rendering engine | Required peer dependency. Full 3D capabilities: models, particles, lighting, materials. |
### Chart / Data Visualization
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **@remotion/shapes** | 4.0.448 | SVG shape primitives for custom charts | `makePie()` for pie charts, `makeRect()` for bar charts, `evolvePath()` for animated path transitions. Low-level but pixel-perfect control. |
| **Recharts** | 3.8.x | Declarative chart components (Bar, Line, Pie, Area) | Best React+D3 chart library. Declarative API means LLMs can generate chart configs easily. SVG-based output renders well in Remotion. 3.8.x is current stable. |
| **D3.js** | 7.9.x | Low-level data visualization primitives | Only if Recharts cannot express a specific chart type. D3 operates on SVG/Canvas, compatible with Remotion. Do NOT install unless Recharts proves insufficient. |
### Image Generation
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **SVG/CSS via Remotion components** | N/A | Primary visual asset generation | The project's core principle: "code-generated visuals as primary." SVG and CSS animations produced by Remotion components are the default visual approach. No external tools needed. |
| **FLUX via ComfyUI** | Latest | Supplementary AI image generation | FLUX with GGUF Q4 quantization runs on M4 Pro via MLX acceleration. Use only when a specific visual cannot be achieved with SVG/CSS. This is explicitly supplementary, not primary. |
| **ComfyUI** | Latest | Stable Diffusion / FLUX workflow manager | Node-based UI for AI image generation. Supports GGUF models, MLX backend, ControlNet. Run locally. |
### Build Tooling / Testing
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **Vite** (via Remotion) | Bundled | Remotion 4.x uses Vite internally for Studio and bundling | No separate Vite config needed. Remotion handles bundling via `@remotion/bundler`. |
| **Webpack** (via Remotion) | Bundled | Legacy bundler for `remotion render` | Remotion uses webpack for production rendering. Config is managed by Remotion, not manually. |
| **ESLint** | 9.x | Linting | Already at 9.19.0. Add custom rule to flag CSS animations and `setTimeout` in component files. |
| **Prettier** | 3.8.x | Code formatting | Already at 3.8.1. |
| **Jest / Vitest** | Latest | Unit testing for pipeline utilities | Test data contracts (Zod schemas), timing calculations, script parsing. Not for visual components (visual regression testing is impractical for video frames). |
| **Remotion Benchmark** | CLI | Render performance profiling | `npx remotion benchmark` measures render speed per frame. Essential for identifying GPU-bound components and optimizing concurrency. |
### Infrastructure
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **Node.js** | 22.x LTS | Runtime environment | Required by Remotion 4.x. LTS ensures stability. |
| **npm** | 10.x | Package management | Already in use. Do NOT switch to pnpm or yarn mid-project. |
| **Remotion Studio** | 4.0.448 | Development preview (localhost:3000) | Built-in with `npm run dev`. Live preview, scrubbing, parameter adjustment. No additional setup needed. |
| **Ollama** | Latest | Local AI model runtime | Already installed on the M4 Pro. REST API on localhost:11434. Node.js client: `ollama` package. |
### Skillhub / Community Skills
| Skill | Source | Purpose | When to Use |
|-------|--------|---------|-------------|
| **remotion** (skills repo) | `github.com/remotion-dev/skills` | 30+ rule files covering Remotion best practices: animation, transitions, audio, captions, 3D, performance, etc. | Install immediately. These are the official Remotion skill rules that Claude reads as context. This is the single most important skill installation. |
| **Skillhub CLI** | `skillhub.cn` | Search and install community skills | Already installed. Use to discover skills for: script writing, TTS integration, asset management. |
| **code2animation** | Existing | Code-to-animation pipeline reference | Already in project. Reference for animation patterns. |
| **video-cog** | Existing | CellCog video generation reference | Already in project. Reference for multi-agent video production patterns. |
| **ai-video-script** | Existing | AI video script generation | Already in project. Reference for structured script output. |
## Alternatives Considered
### Video Framework
| Recommended | Alternative | Why Not |
|-------------|-------------|---------|
| **Remotion 4.x** | FFmpeg + Canvas | No React component model, no Studio preview, no frame-accurate animation system. Would require building everything from scratch. |
| **Remotion 4.x** | Motion Canvas | Less mature, smaller ecosystem, fewer official packages. Remotion has 25+ official packages; Motion Canvas has fewer integrations. |
| **Remotion 4.x** | Puppeteer video capture | Not a video framework. Just screen recording of a web page. No programmatic control over timing, transitions, or rendering. |
### Animation
| Recommended | Alternative | Why Not |
|-------------|-------------|---------|
| **Remotion `spring()`/`interpolate()`** | Framer Motion | NOT compatible with Remotion. Uses `requestAnimationFrame` and wall-clock time. No official integration. Will produce flickering renders. |
| **Remotion native** | react-spring | NOT compatible with Remotion. Same reason as Framer Motion. |
| **Remotion native + optional GSAP** | Anime.js only | GSAP has an official Remotion integration (`<GsapSyncedAnimation>`). Anime.js has an example but not a dedicated package. GSAP's timeline API is more powerful. |
| **Remotion native** | CSS animations | FORBIDDEN in Remotion. Renders frames out of order, causing flickering. |
### TTS
| Recommended | Alternative | Why Not |
|-------------|-------------|---------|
| **Kokoro TTS / Qwen3-TTS (local)** | Edge TTS | Online-only. Requires internet. Project constraint: local TTS first. Also, 3 redundant edge-tts packages are already in package.json -- remove them. |
| **Kokoro TTS / Qwen3-TTS (local)** | ElevenLabs | Paid cloud service. Project constraint: no paid cloud services. |
| **Kokoro TTS / Qwen3-TTS (local)** | Voxtral TTS (Mistral) | Does NOT support Chinese language. This is a Chinese-English bilingual project. Ruled out. |
| **Kokoro TTS / Qwen3-TTS (local)** | Alibaba Cloud TTS | Good quality but cloud-based. Keep as fallback for final production renders, not for local development iteration. |
### Charts
| Recommended | Alternative | Why Not |
|-------------|-------------|---------|
| **Recharts + @remotion/shapes** | D3.js directly | Too low-level for LLM-generated configs. Recharts provides the same D3 power with a declarative React API. |
| **Recharts** | Chart.js / react-chartjs-2 | Chart.js renders to Canvas, which is harder to control frame-by-frame in Remotion. Recharts renders to SVG, which integrates cleanly. |
| **Recharts** | Victory Charts | Smaller community, fewer chart types. Recharts has broader adoption and better TypeScript support. |
### Styling
| Recommended | Alternative | Why Not |
|-------------|-------------|---------|
| **Tailwind 4 + @remotion/tailwind-v4** | Styled-components | Additional runtime overhead. Tailwind with the official Remotion integration is the standard approach. |
| **Tailwind 4** | CSS Modules | Works fine with Remotion, but Tailwind is already in the project. No reason to add another styling system. |
## Current Issues in package.json
### Must Fix Immediately
### Must Install Before Phase 2
### Can Defer
## Installation
# Step 1: Upgrade all Remotion packages to latest
# Step 2: Remove redundant edge-tts packages
# Step 3: Install missing critical packages (Phase 1)
# Step 4: Install Remotion official skills
# (Clone or install from github.com/remotion-dev/skills into skills/ directory)
# Step 5 (Phase 2): Audio pipeline packages
# Step 6 (Phase 3): Visual polish packages
# Step 7 (Phase 4): 3D packages
# Step 8: Install Ollama TTS models (when ready)
# ollama pull kokoro  (or equivalent TTS model)
# ollama pull qwen3:8b  (for script generation)
## What NOT to Use
| Technology | Why Not | What to Use Instead |
|------------|---------|---------------------|
| **Framer Motion** | Incompatible with Remotion rendering (uses rAF, time-based) | Remotion `spring()` + `interpolate()` |
| **react-spring** | Incompatible with Remotion rendering | Remotion `spring()` + `interpolate()` |
| **CSS @keyframes / animation** | Flickers during Remotion render (frames rendered out of order) | Remotion `spring()` + `interpolate()` |
| **Tailwind animate-*** | CSS animations, same flickering issue | Remotion `spring()` + `interpolate()` |
| **setTimeout / setInterval** | Non-deterministic in Remotion render | `useCurrentFrame()` for all timing |
| **Date.now()** | Different values per render tab, causes inconsistency | `useCurrentFrame()` + `fps` |
| **requestAnimationFrame** | Wall-clock based, not frame-based | `useCurrentFrame()` |
| **Edge TTS (any variant)** | Online-only, conflicts with local-first requirement | Kokoro TTS or Qwen3-TTS via Ollama |
| **Voxtral TTS** | No Chinese language support | Kokoro TTS or Qwen3-TTS |
| **Redux / Zustand / MobX** | No interactive UI state to manage in a pipeline tool | JSON files on disk + React props |
| **SQLite / IndexedDB** | No database needed; pipeline artifacts are files | File system (JSON + media files) |
| **Docker (for now)** | Local-only on M4 Pro, adds unnecessary complexity | Direct local rendering via `npx remotion render` |
| **Canvas API** | Harder to control frame-by-frame; SVG is cleaner in Remotion | SVG via React components |
## Confidence Assessment
| Area | Confidence | Reason |
|------|------------|--------|
| **Remotion core + official packages** | HIGH | Verified against npm registry (4.0.448 current). Official docs confirm compatibility matrix. |
| **Animation libraries (what works, what doesn't)** | HIGH | Verified via official Remotion docs: GSAP, Lottie, Anime.js are compatible; Framer Motion and react-spring are NOT. |
| **Local TTS on Apple Silicon** | MEDIUM | Kokoro TTS and Qwen3-TTS are both real and local-capable, but Chinese quality on M4 Pro has not been tested in this project. Needs evaluation with real scripts. |
| **Local LLM for scripts** | HIGH | Qwen 3 is well-established for bilingual content. Ollama runtime is proven on Apple Silicon. |
| **Audio processing stack** | HIGH | All tools are official Remotion packages with proven APIs. |
| **3D capabilities** | MEDIUM | `@remotion/three` exists and works, but render time impact on M4 Pro for 1080p 3D scenes has not been measured. Needs profiling. |
| **Chart / data viz** | HIGH | Recharts is the standard React chart library. `@remotion/shapes` provides SVG primitives. Both are well-documented. |
| **Image generation** | LOW | FLUX via ComfyUI on M4 is theoretically possible but has not been tested. This is explicitly supplementary and should not block the pipeline. |
## Sources
- npm registry (verified versions: remotion 4.0.448, @remotion/* 4.0.448, react 19.2.5, typescript 5.9.3, tailwindcss 4.2.2, recharts 3.8.1, @react-three/fiber 9.5.0, three 0.183.2) -- HIGH confidence
- [Remotion Official Docs - Third-Party Integrations](https://www.remotion.dev/docs/third-party-integration) -- HIGH confidence
- [Remotion Official Docs - GSAP Integration](https://www.remotion.dev/docs/gsap) -- HIGH confidence
- [Remotion Official Docs - Three.js](https://www.remotion.dev/docs/three) -- HIGH confidence
- [Remotion Official Docs - Captions](https://www.remotion.dev/docs/captions) -- HIGH confidence
- [Remotion Official Docs - Flickering](https://www.remotion.dev/docs/flickering) -- HIGH confidence
- [Remotion Official Skills Repository](https://github.com/remotion-dev/skills) -- HIGH confidence
- [Remotion template-prompt-to-video](https://github.com/remotion-dev/template-prompt-to-video) -- HIGH confidence
- [Remotion Performance Docs](https://www.remotion.dev/docs/performance) -- HIGH confidence
- [Kokoro TTS](https://huggingface.co/hexgrad/Kokoro-82M) -- MEDIUM confidence
- [Qwen3-TTS Open Source Announcement](https://www.reddit.com/r/LocalLLaMA/comments/1qjul5t/) -- MEDIUM confidence (newly released)
- [Ollama Node.js Client](https://www.npmjs.com/package/ollama) -- HIGH confidence
- Project package.json analysis (current state: remotion 4.0.434, 4 redundant edge-tts packages) -- HIGH confidence
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

Architecture not yet mapped. Follow existing patterns found in the codebase.
<!-- GSD:architecture-end -->

<!-- GSD:skills-start source:skills/ -->
## Project Skills

| Skill | Description | Path |
|-------|-------------|------|
| remotion-components |  | `.claude/skills/remotion-components/SKILL.md` |
<!-- GSD:skills-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd-quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd-debug` for investigation and bug fixing
- `/gsd-execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->

<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd-profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->

# ContentFlow Video Skills

ContentFlow 视频生成技能已安装。技能文件位于: /Users/clear2x/ai_ws/ContentFlow/.claude/skills
读取 /Users/clear2x/ai_ws/ContentFlow/.claude/skills/README.md 获取所有可用技能和命令。

---
> Source: [clear2x/ContentFlow](https://github.com/clear2x/ContentFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
