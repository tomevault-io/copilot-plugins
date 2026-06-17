## caloriecop

> A personal health management iOS app for calorie tracking and weight management.

# CalorieCop

A personal health management iOS app for calorie tracking and weight management.

## Version History

- **v0.0.0** - First MVP version (stable baseline)
- **v0.0.1** - UI redesign WIP (simplified Dashboard, mascot avatar)

To switch versions: `git checkout v0.0.0` or `git checkout v0.0.1`

## Tech Stack

- **UI**: SwiftUI (iOS 17+)
- **Storage**: SwiftData
- **AI Text**: MiniMax API (M2.7-highspeed for fast parsing)
- **AI Vision**: Qwen VL Plus (image food recognition)
- **Health**: HealthKit (Apple Watch integration)
- **Charts**: Swift Charts

## Project Structure

```
CalorieCop/
├── App/
│   ├── CalorieCopApp.swift      # Entry point, ModelContainer setup
│   └── ContentView.swift        # Main TabView (今日/记录/历史/目标)
├── Models/
│   ├── FoodEntry.swift          # Food record with nutrition
│   ├── NutritionInfo.swift      # Parsed nutrition data
│   ├── UserGoal.swift           # Calorie/weight targets
│   ├── UserSettings.swift       # Preferences (weight unit)
│   ├── WeightEntry.swift        # Weight records
│   ├── ChatMessage.swift        # AI advisor chat history
│   └── FoodPreference.swift     # User food habits for AI
├── Services/
│   ├── AIService/
│   │   ├── MiniMaxService.swift     # API calls (text & vision)
│   │   └── FoodParsingPrompt.swift  # System prompts
│   ├── HealthKitService.swift   # Apple Watch data
│   ├── APIKeyManager.swift      # API key handling
│   └── Secrets.swift            # API keys (gitignored)
├── Views/
│   ├── Dashboard/               # 今日 tab
│   ├── FoodInput/               # 记录 tab (camera, text input)
│   ├── History/                 # 历史 tab (calendar, AI advisor)
│   └── Goals/                   # 目标 tab (weight chart, goals)
└── Resources/
    └── Assets.xcassets/
```

## Key Patterns

### SwiftData Models
All models use `@Model` macro. Schema registered in `CalorieCopApp.swift`:
```swift
.modelContainer(for: [FoodEntry.self, UserGoal.self, ...])
```

### API Key Setup
Copy `Secrets.swift.template` to `Secrets.swift` and add your keys:
```swift
enum Secrets {
    static let miniMaxAPIKey = "your_minimax_key"
    static let qwenAPIKey = "your_qwen_key"  // For image recognition
}
```

### MiniMax API
- **Text parsing**: `MiniMax-M2.7-highspeed` for fast response
- **AI Advisor**: `MiniMax-M2.7-highspeed` for chat
- Endpoint: `https://api.minimax.io/v1/text/chatcompletion_v2`
- Note: MiniMax vision models (M1, VL-01) are NOT available via API

### Qwen VL Plus (Image Recognition)
- Used for food image parsing (since MiniMax vision unavailable)
- Model: `qwen-vl-plus`
- Endpoint: `https://dashscope-intl.aliyuncs.com/compatible-mode/v1/chat/completions` (Singapore)
- OpenAI-compatible format
- Supports multiple foods in single image

### Food Parsing
AI returns JSON **array** with: `food_name`, `grams`, `calories`, `protein`, `carbohydrates`, `fat`, `confidence`, `notes`, `days_ago`

User preferences are injected into system prompt for personalized parsing.

## Learnings & Gotchas

### SwiftData
- **Always call `modelContext.save()`** after insert/update/delete for immediate UI refresh
- `@Query` won't update until context is saved
- Example: `modelContext.insert(item); try? modelContext.save()`

### SwiftUI Sheets
- **Use `sheet(item:)` instead of `sheet(isPresented:)` with separate state**
- Avoids blank sheet on first open due to state timing issues
- Create an `Identifiable` wrapper if needed:
```swift
struct EditingItem: Identifiable {
    let id: Int
    let data: SomeType
}
@State private var editingItem: EditingItem?
.sheet(item: $editingItem) { item in ... }
```

### API Performance Profiling
Test API response times directly before assuming app-side issues:
```bash
# Quick API timing test
swift -e 'let start = Date(); /* API call */; print(Date().timeIntervalSince(start))'
```

### Multiple Food Parsing
- Prompt must specify JSON **array** format, even for single items
- `extractJSON()` should check for `[` before `{` to handle arrays first

## Build & Run

1. Open `CalorieCop.xcodeproj`
2. Set up Secrets.swift with MiniMax API key
3. Select your device/simulator
4. Cmd+R to run

For physical device: configure Signing & Capabilities with your Apple ID.

## Language

- UI: Chinese (Simplified)
- Code: English

## Development Rules

**Always verify builds before asking user to test:**
```bash
xcodebuild -scheme CalorieCop -destination 'platform=iOS Simulator,name=iPhone 15 Pro' build 2>&1 | grep -E "(error:|warning:|BUILD SUCCEEDED|BUILD FAILED)"
```

- Run this command after making code changes
- Fix any errors or warnings before telling user to build/run
- Only ask user to test after BUILD SUCCEEDED with no warnings

## Common Commands

```bash
# Build (with error/warning filter)
xcodebuild -scheme CalorieCop -destination 'platform=iOS Simulator,name=iPhone 15 Pro' build 2>&1 | grep -E "(error:|warning:|BUILD SUCCEEDED|BUILD FAILED)"

# Open project
open CalorieCop.xcodeproj

# View git tags
git tag -l

# Switch to a version
git checkout v0.0.0
```

## Known Issues

- **MiniMax Vision API**: Image recognition may not work reliably. The API sometimes returns "纯文本AI助手" error. May need alternative API (OpenAI, Claude) for vision.
- **HealthKit**: Requires physical device with Apple Watch for real activity data. Simulator uses estimated values.

## Design Notes

### Dashboard (今日概览)
- v0.0.0: Simple layout with calorie balance card, metabolism breakdown, nutrition stats
- v0.0.1: Attempted redesign with mascot, circular progress ring (reverted to simpler design)

### Calorie Calculation
- **缺口 (deficit)** = 总消耗 (BMR + active) - 摄入
- Positive deficit = good for weight loss (green)
- Negative deficit = 超出, over budget (red)
- Toggle between 实时 (Apple Watch) and 估算 (calculated) data

### Mascot
- Asset: `mascot_avatar` in Assets.xcassets
- Police officer character (inspired by Judy Hopps)
- Used in empty states and encouragement messages

---
> Source: [kaka-jun/CalorieCop](https://github.com/kaka-jun/CalorieCop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
