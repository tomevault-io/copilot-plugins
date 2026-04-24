## aiphonereminderproject

> Android AI companion that monitors device activity through automated screenshots and provides contextual interventions through character-driven dialogue.

# Ralsei AI Screentime Coach - Agent Instructions

Android AI companion that monitors device activity through automated screenshots and provides contextual interventions through character-driven dialogue.

## Build Verification ⚠️ CRITICAL

**ALWAYS build the project after making code changes to verify compilation success.**

After editing Kotlin/Java source files, manifest, or Gradle files:
1. Run: `$env:JAVA_HOME = "C:\Program Files\Android\Android Studio\jbr"; .\gradlew assembleDebug`
2. Check for compilation errors in the output
3. Fix any errors before presenting results to the user
4. Only report success after BUILD SUCCESSFUL confirmation

**Do not wait for the user to report compilation errors - catch them yourself!**

## Architecture Overview

Check root/README.md for the latest architecture

## Common Workflows

### Build & Install (Windows PowerShell)
```powershell
# Set JAVA_HOME for JDK 11+
$env:JAVA_HOME = "C:\Program Files\Android\Android Studio\jbr"

# Build and install
.\gradlew assembleDebug; .\gradlew installDebug

# Clear data before testing
adb shell pm clear com.example.myapplication
```

### Debugging with Logcat
```bash
# Core pipeline (screenshot → analysis → dialogue)
adb logcat -s "MainForegroundService:D" "ScreenshotController:D" "AnalyzerAgent:D" "ChatManager:D" "*:E"

# Warning system (pattern detection → response)
adb logcat -s "PatternAgent:D" "PersonalityAgent:D" "WarningCheckWorker:D" "*:E"

# Memory system
adb logcat -s "EnhancedMemoryManager:D" "*:E"
```

### Permission Debugging
```bash
# Check overlay permission
adb shell appops get com.example.myapplication SYSTEM_ALERT_WINDOW

# Manually grant (testing only)
adb shell pm grant com.example.myapplication android.permission.SYSTEM_ALERT_WINDOW
```

## API Key Configuration

Resolution chain (first found wins):
1. SharedPreferences (`PrefsHelper.getOpenAIApiKey()`)
2. Asset file (`app/src/main/assets/openai.env`)
3. Hardcoded fallback (dev only)

**Never commit** `openai.env` (see `openai.env.example`).

## Testing Strategy

```kotlin
// Enable mock mode for offline deterministic testing
LLMClientFactory.setMockMode(true)

// Run test scenarios
TestAgent.runScenario(context, scenario, clearFirst = true)
TestAgent.runAllScenarios(context)  // Full test suite

```

## Activity Exclusions

`MyApplication.kt` lifecycle tracking auto-excludes internal screens:
- `MainActivity`, `ChatActivity`, `AdvancedActivity`, `MemoryLogActivity`, `ResponseLogActivity`, `DebugActivity`

Implementation: `MainForegroundService` checks `MyApplication.isExcludedActivity()` before each screenshot.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JaclyNolan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
