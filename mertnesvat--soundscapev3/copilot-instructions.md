## soundscapev3

> Next Sleep is an iOS wellness audio app that helps users create personalized ambient soundscapes for sleep, relaxation, focus, and meditation. Users can mix multiple sounds together, adjust individual volumes, save favorite combinations, and set sleep timers.

# Next Sleep - Claude Code Context

## Project Overview

Next Sleep is an iOS wellness audio app that helps users create personalized ambient soundscapes for sleep, relaxation, focus, and meditation. Users can mix multiple sounds together, adjust individual volumes, save favorite combinations, and set sleep timers.

## Tech Stack

- **Platform**: iOS 17.0+, iPhone only
- **Language**: Swift 5.0
- **UI Framework**: SwiftUI
- **Architecture**: Clean Architecture (Domain, Data, Presentation layers)
- **State Management**: @Observable, @Environment
- **Audio**: AVFoundation, AVAudioEngine (for binaural beats)
- **Persistence**: UserDefaults, JSON files
- **Build System**: Xcode project (not SPM)

## Project Structure

```
SoundScape/
├── SoundScape.xcodeproj/
├── Sources/
│   ├── App/
│   │   ├── SoundScapeApp.swift      # App entry point, service initialization
│   │   └── ContentView.swift         # Main tab view (7 tabs)
│   ├── Domain/
│   │   ├── Entities/                 # Sound, ActiveSound, Alarm, Story, etc.
│   │   ├── Protocols/                # AudioPlayerProtocol
│   │   └── Repositories/             # SoundRepositoryProtocol
│   ├── Data/
│   │   ├── DataSources/              # LocalSoundDataSource, LocalStoryDataSource
│   │   ├── Repositories/             # SoundRepository
│   │   └── Services/                 # AudioEngine, SleepTimerService, InsightsService, etc.
│   └── Presentation/
│       ├── Sounds/                   # Main sound library view
│       ├── Mixer/                    # Active sounds mixing (sheet modal)
│       ├── Timer/                    # Sleep timer (sheet modal)
│       ├── SavedMixes/               # Saved combinations (sheet modal)
│       ├── Favorites/                # Favorited sounds
│       ├── BinauralBeats/            # Brainwave entrainment
│       ├── Stories/                  # Sleep stories
│       ├── Alarms/                   # Smart wake-up alarms
│       ├── Discover/                 # Community mixes
│       ├── Adaptive/                 # Context-aware soundscapes
│       ├── Insights/                 # Sleep analytics
│       └── Components/               # Shared UI (NowPlayingBarView)
└── Resources/
    ├── Sounds/                       # 27 MP3 audio files
    ├── Assets.xcassets/
    ├── Info.plist
    └── GoogleService-Info.plist
```

## Key Services

| Service | Purpose |
|---------|---------|
| `AudioEngine` | Multi-sound playback, volume control, session tracking |
| `SleepTimerService` | Countdown timer with fade-out |
| `InsightsService` | Usage analytics, session recording |
| `FavoritesService` | Persist favorite sounds |
| `SavedMixesService` | Save/load sound combinations |
| `BinauralBeatEngine` | Generate binaural/isochronic tones |
| `AlarmService` | Schedule wake-up alarms |
| `AdaptiveSessionService` | Context-aware sound transitions |

## Sound Categories

| Category | Color | Sounds |
|----------|-------|--------|
| Noise | Purple | White Noise, Pink Noise, Brown Noise, Deep Brown Noise |
| Nature | Green | Morning Birds, Winter Forest, Serene Morning, Spring Birds, Meadow, Night Wildlife, Calm Ocean |
| Weather | Blue | Rain Storm, Wind Ambient, Rainforest, Thunder, Heavy Thunder, Castle Wind |
| Fire | Orange | Campfire, Bonfire |
| Music | Pink | Creative Mind, Midnight Calm, Ocean Lullaby, Deep Focus Flow, Starlit Sky, Forest Sanctuary, Cinematic Piano, Ambient Melody |

## Tab Structure (7 tabs)

1. **Sounds** - Main library with Mixer/Timer/Saved as toolbar sheet modals
2. **Binaural** - Brainwave entrainment tones
3. **Wind Down** - Sleep content and stories
4. **Sleep Rec** - Sleep recording
5. **Discover** - Community-curated mixes
6. **Adaptive** - Context-aware soundscapes
7. **Insights** - Sleep analytics dashboard

## Build Commands

```bash
# Build
cd SoundScape && xcodebuild -scheme SoundScape -destination 'platform=iOS Simulator,name=iPhone 17 Pro' build

# Run in simulator
xcrun simctl install "iPhone 17 Pro" build/Build/Products/Debug-iphonesimulator/SoundScape.app
xcrun simctl launch "iPhone 17 Pro" com.StudioNext.SoundScape
```

## Adding New Sounds

1. Add MP3 file to `Resources/Sounds/`
2. Add entry to `LocalSoundDataSource.swift` with id, name, category, fileName
3. Add file reference to `SoundScape.xcodeproj/project.pbxproj`:
   - PBXBuildFile section
   - PBXFileReference section
   - Sounds group children
   - PBXResourcesBuildPhase files
4. Update `SoundCardView.swift` and `MixerSoundRowView.swift` if new category

## Key Patterns

- Services are `@Observable` classes injected via `.environment()`
- Views access services via `@Environment(ServiceName.self)`
- Sound playback is managed by `AudioEngine.activeSounds`
- Sessions recorded to `InsightsService` when timer ends or sounds stop

## Design Context

### Users
Bedtime users seeking relief from the day — people who want to fall asleep faster, focus better, or decompress. They open the app when they're tired, often in a dark room. Speed and calm matter: they need to get their soundscape running with minimal friction, then put the phone down.

### Brand Personality
**Modern, Sleek, Technical** — a precision instrument for sleep, not a whimsical toy. Think Endel's generative minimalism: abstract, dark, flowing. The interface should feel like a well-engineered tool that disappears into the background.

### Emotional Goals
- **Instant calm & relief** — tension dissolves the moment the app opens; the day is over
- **Cozy comfort** — warmth and safety, like crawling into fresh sheets

### Aesthetic Direction
- **Dark-first**: OLED black backgrounds with purple accent (#7F6FD8) and category-coded colors
- **Generative & abstract**: Flowing liquid visualizations, particle systems, wave layers — not literal nature photos
- **Minimal chrome**: Let content and audio breathe; reduce visual noise
- **Reference**: Endel — abstract, generative, minimal dark UI with flowing visuals
- **Anti-pattern**: Avoid cluttered dashboards, bright whites, or playful/cartoonish elements

### Design Principles
1. **Calm by default** — Every pixel should lower cortisol. No jarring transitions, no competing elements, no visual anxiety.
2. **Dark is home** — OLED black is the primary canvas. Color is intentional and category-coded. Light is used sparingly as accent and glow.
3. **Disappearing UI** — The best interaction is the shortest one. Get to playback fast, then get out of the way.
4. **Contrast matters** — Enhanced contrast ratios in dark mode. Text and interactive elements must be clearly legible against dark backgrounds without being harsh.
5. **Motion with purpose** — Animations serve comprehension (state changes, feedback) or atmosphere (visualizations). Never decorative jank.

### Design Tokens (Current)
| Token | Values |
|-------|--------|
| Accent | #7F6FD8 (purple) |
| Corner Radius | 16 (cards), 12 (buttons), 8 (badges) |
| Primary Padding | 24 (edges), 16 (standard), 12 (compact) |
| Shadow (OLED) | category color glow, radius 8-16 |
| Spring Animation | response 0.3-0.6, damping 0.6-0.7 |
| Font Weights | .thin (display), .medium (body), .bold (headers) |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mertnesvat) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
