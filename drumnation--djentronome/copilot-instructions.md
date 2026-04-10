## djentronome

> 1. Use vitest for testing

1. Use vitest for testing
2. Use pnpm for package manager
3. We use pnpm workspaces for node monorepo management
4. Optimize script processes using turborepo

# --- AGENT-SPECIFIC RULES BELOW ---
# Cursor rules for Agent Tony Stark

## Scenarios

### Create New React Component
- **Scenario:** "Create a new reusable [ComponentName] component using Mantine."
- **Skill-Jack:** `Read [".brain/1-agent-tony-stark/d-skill-jacks/react-component-standards.skill-jack.ts"]`
- **Skill-Jack:** `Read [".brain/1-agent-tony-stark/d-skill-jacks/mantine-ui-advanced.skill-jack.ts"]`
- **Skill-Jack:** `Read [".brain/1-agent-tony-stark/d-skill-jacks/frontend-state-management-ui.skill-jack.ts"]`
- **Rules:** `react-component-standards.rules.mdc`, `mantine-ui.rules.mdc`
- **Commands:** 
    - `mkdir -p apps/game-ui/src/components/[ComponentName]`
    - `touch apps/game-ui/src/components/[ComponentName]/{[ComponentName].tsx,[ComponentName].styles.ts,[ComponentName].types.ts,[ComponentName].stories.tsx,index.ts}`

### Implement Gameplay Visualization
- **Scenario:** "Implement the basic note highway visualization."
- **Skill-Jack:** `Read [".brain/3-agent-q/d-skill-jacks/web-audio-game-synchronization.skill-jack.ts"]` (for timing)
- **Skill-Jack:** `Read [".brain/1-agent-tony-stark/d-skill-jacks/canvas-2d-rendering.skill-jack.ts"]` (if using Canvas 2D)
- **Skill-Jack:** `Read [".brain/1-agent-tony-stark/d-skill-jacks/r3f-threejs-fundamentals.skill-jack.ts"]` (if using R3F)
- **Rules:** `r3f-usage-core.rules.mdc` (if using R3F), `game-ui-ux.rules.mdc`
- **Commands:**
    - `cd apps/game-ui`
    - (Potentially `pnpm add @react-three/fiber three @react-three/drei --filter game-ui` if starting R3F)

### Create UI Screen Component
- **Scenario:** "Create the [ScreenName] screen component (e.g., Song Selection, Results)."
- **Skill-Jack:** `Read [".brain/1-agent-tony-stark/d-skill-jacks/react-component-standards.skill-jack.ts"]`
- **Skill-Jack:** `Read [".brain/1-agent-tony-stark/d-skill-jacks/mantine-ui-advanced.skill-jack.ts"]`
- **Skill-Jack:** `Read [".brain/1-agent-tony-stark/d-skill-jacks/frontend-state-management-ui.skill-jack.ts"]`
- **Rules:** `react-component-standards.rules.mdc`, `mantine-ui.rules.mdc`, `game-ui-ux.rules.mdc`
- **Commands:** 
    - `mkdir -p apps/game-ui/src/screens/[ScreenName]`
    - `touch apps/game-ui/src/screens/[ScreenName]/{[ScreenName].tsx,[ScreenName].styles.ts,[ScreenName].types.ts,[ScreenName].stories.tsx,index.ts}`

### Add Storybook Story/Test
- **Scenario:** "Add Storybook stories/tests for the [ComponentName] component."
- **Skill-Jack:** `Read [".brain/1-agent-tony-stark/d-skill-jacks/react-component-standards.skill-jack.ts"]`
- **Skill-Jack:** `Read [".brain/1-agent-tony-stark/d-skill-jacks/storybook-interaction-testing.skill-jack.ts"]`
- **Rules:** `react-component-standards.rules.mdc`
- **Commands:**
    - `cd apps/game-ui`
    - `pnpm storybook` 

# Cursor rules for Agent Q

## Scenarios

### Implement Audio Playback
- **Scenario:** "Implement audio loading and synchronized playback for local files."
- **Skill-Jack:** `Read [".brain/3-agent-q/d-skill-jacks/web-audio-game-synchronization.skill-jack.ts"]`
- **Rules:** `core-audio.rules.mdc` (if exists, else `sound-and-music.rules.mdc`), `bpm-sync.rules.mdc`
- **Commands:** 
    - `cd packages/core-audio` (or relevant package)

### Develop Transcription Module
- **Scenario:** "Develop the offline kick/snare transcription module using [ChosenStrategy]."
- **Skill-Jack:** `Read [".brain/3-agent-q/d-skill-jacks/audio-transcription-strategies.skill-jack.ts"]`
- **Skill-Jack:** `Read [".brain/3-agent-q/d-skill-jacks/dsp-fundamentals.skill-jack.ts"]`
- **Skill-Jack:** `Read [".brain/3-agent-q/d-skill-jacks/python-node-integration.skill-jack.ts"]` (if using Python tools)
- **Skill-Jack:** `Read [".brain/3-agent-q/d-skill-jacks/ffmpeg-cli.skill-jack.ts"]` (if using FFmpeg)
- **Rules:** `typescript-standards.rules.mdc`
- **Commands:** 
    - (May involve setting up Python environment, installing libs like demucs/madmom)
    - `cd packages/transcription` (assuming a dedicated package)

### Investigate Advanced Transcription
- **Scenario:** "Research and prototype using [ModelName] for transcription."
- **Skill-Jack:** `Read [".brain/3-agent-q/d-skill-jacks/audio-transcription-strategies.skill-jack.ts"]`
- **Skill-Jack:** `Read [".brain/3-agent-q/d-skill-jacks/[ModelName]-api.skill-jack.ts"]` (To be created)
- **Commands:** 
    - (Likely involves running Python scripts, reading research papers/docs)

### Explore Streaming Service API
- **Scenario:** "Investigate the [Spotify/YouTube] API for playback control and metadata."
- **Skill-Jack:** `Read [".brain/3-agent-q/d-skill-jacks/streaming-api-usage.skill-jack.ts"]` (To be created)
- **Commands:**
    - (Web searches, potentially installing SDKs e.g., `pnpm add spotify-web-api-js --filter @kit/integration-spotify`)

### Test Audio Synchronization
- **Scenario:** "Write tests to verify audio playback timing against game events."
- **Skill-Jack:** `Read [".brain/3-agent-q/d-skill-jacks/web-audio-game-synchronization.skill-jack.ts"]`
- **Skill-Jack:** `Read [".brain/2-agent-spock/d-skill-jacks/tdd-vitest.skill-jack.ts"]`
- **Rules:** `functional-test-principles.rules`
- **Commands:**
    - `cd packages/core-audio` (or relevant package)
    - `pnpm test` 

# Cursor rules for Agent Spock

## Scenarios

### Setup New Package
- **Scenario:** "Create a new shared package named [PackageName]."
- **Skill-Jack:** `Read [".brain/2-agent-spock/d-skill-jacks/monorepo-tooling-pnpm-turbo.skill-jack.ts"]`
- **Rules:** `monorepo-library-setup.rules.mdc`, `agent-use-monorepo-docs.rules.mdc`
- **Commands:** 
    - `mkdir -p packages/[PackageName]/src`
    - `touch packages/[PackageName]/src/index.ts`
    - (Agent generates package.json, tsconfig.json based on rules/skill-jack)
    - `pnpm install`

### Implement MIDI Handling
- **Scenario:** "Implement the logic to detect kick and snare from Alesis Nitro."
- **Skill-Jack:** `Read [".brain/2-agent-spock/d-skill-jacks/web-midi-integration.skill-jack.ts"]`
- **Rules:** `midi-handler.rules.mdc`
- **Commands:** 
    - `cd packages/midi-handler` (or relevant package)

### Add New Dependency to Package
- **Scenario:** "Add the [DependencyName] library to the [PackageName] package."
- **Skill-Jack:** `Read [".brain/2-agent-spock/d-skill-jacks/monorepo-tooling-pnpm-turbo.skill-jack.ts"]`
- **Commands:** 
    - `pnpm add [DependencyName] --filter @kit/[PackageName]` (Adjust scope @kit/ if needed)
    - `pnpm add [DependencyName] --filter [appName]`

### Implement Core Game Logic (Game Loop, State, Scoring)
- **Scenario:** "Implement the [GameLogicFeature] logic (e.g., game loop, state update, scoring) in the [PackageName] package."
- **Skill-Jack:** `Read [".brain/2-agent-spock/d-skill-jacks/tdd-vitest.skill-jack.ts"]`
- **Skill-Jack:** `Read [".brain/2-agent-spock/d-skill-jacks/game-loop-architecture.skill-jack.ts"]` (for F6)
- **Skill-Jack:** `Read [".brain/2-agent-spock/d-skill-jacks/zustand-advanced.skill-jack.ts"]` (if using Zustand for state)
- **Skill-Jack:** `Read [".brain/2-agent-spock/d-skill-jacks/rhythm-game-scoring.skill-jack.ts"]` (for F8)
- **Skill-Jack:** `Read [".brain/2-agent-spock/d-skill-jacks/latency-compensation.skill-jack.ts"]` (for F12)
- **Skill-Jack:** `Read [".brain/2-agent-spock/d-skill-jacks/data-structures-game-patterns.skill-jack.ts"]` (for F5, F8)
- **Skill-Jack:** `Read [".brain/2-agent-spock/d-skill-jacks/error-handling-resilience.skill-jack.ts"]`
- **Rules:** `game-core-architecture.rules.mdc`, `functional-test-principles.rules`, `typescript-standards.rules.mdc`, `zustand.rules.mdc`
- **Commands:**
    - `cd packages/[PackageName]`
    - `pnpm test --watch`

### Implement Latency Calibration Logic
- **Scenario:** "Implement the core logic for measuring and applying latency offset."
- **Skill-Jack:** `Read [".brain/2-agent-spock/d-skill-jacks/latency-compensation.skill-jack.ts"]`
- **Skill-Jack:** `Read [".brain/3-agent-q/d-skill-jacks/web-audio-game-synchronization.skill-jack.ts"]` (for timing relation)
- **Rules:** `typescript-standards.rules.mdc`
- **Commands:**
    - `cd packages/[relevant-package]`

### Run Tests for Package
- **Scenario:** "Run the tests for the [PackageName] package."
- **Skill-Jack:** `Read [".brain/2-agent-spock/d-skill-jacks/monorepo-tooling-pnpm-turbo.skill-jack.ts"]`
- **Commands:** 
    - `turbo run test --filter @kit/[PackageName]` (Adjust scope @kit/ if needed)
    - `turbo run test --filter [appName]`

### Update Documentation
- **Scenario:** "Document the [Feature/Module] in the appropriate monorepo location."
- **Rules:** `agent-use-monorepo-docs.rules.mdc`, `monorepo-project-documentation-structure.rules.mdc`
- **Commands:**
    - (Agent navigates to correct docs folder, e.g., `packages/[PackageName]/docs` or `docs/features/[Feature]`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/drumnation) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
