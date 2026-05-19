## rokid-apps

> 1. **Read README.md** - Understand the project scope

# Operating Instructions for Agents

## What To Do First

1. **Read README.md** - Understand the project scope
2. **Run `./tools/device_info.sh`** - Check if device is connected
3. **Verify ADB connection**: `adb devices -l`
4. **Check current app status** in CLAUDE.md "Current Status" section
5. **Read relevant app spec** in docs/APP_IDEAS.md before implementing

## How to Add Features Without Breaking Deployment

1. **Always test on device** before committing
2. **Use shared components** from `android/shared/` when available
3. **Follow UI guidelines** in docs/UI_GUIDELINES.md
4. **Run install script** after changes: `./tools/install.sh <AppName>`
5. **Check logcat** for crashes: `./tools/logcat.sh <AppName>`
6. **Keep APK size small** - avoid unnecessary dependencies

## Code Style & Conventions

### Kotlin/Compose Patterns

```kotlin
// Use Compose for all UI
@Composable
fun MyScreen(viewModel: MyViewModel = viewModel()) {
    val state by viewModel.state.collectAsState()
    // ...
}

// Use StateFlow for reactive state
class MyViewModel : ViewModel() {
    private val _state = MutableStateFlow(MyState())
    val state: StateFlow<MyState> = _state.asStateFlow()
}

// Use suspend functions for async operations
suspend fun fetchData(): Result<Data>
```

### File Organization

```
android/<AppName>/
├── app/
│   └── src/main/
│       ├── java/com/rokid/<appname>/
│       │   ├── MainActivity.kt       # Entry point
│       │   ├── ui/
│       │   │   ├── screens/          # Composable screens
│       │   │   ├── components/       # Reusable UI components
│       │   │   └── theme/            # App-specific theme overrides
│       │   ├── data/
│       │   │   ├── api/              # Network clients
│       │   │   └── repository/       # Data sources
│       │   └── viewmodel/            # ViewModels
│       ├── res/
│       │   └── values/
│       │       └── strings.xml
│       └── AndroidManifest.xml
├── build.gradle.kts
└── settings.gradle.kts
```

### Naming Conventions

- **Packages**: `com.rokid.<appname>` (lowercase)
- **Classes**: PascalCase (`NowCardViewModel`)
- **Functions**: camelCase (`fetchCurrentTask`)
- **Constants**: SCREAMING_SNAKE_CASE (`MAX_RETRY_ATTEMPTS`)
- **Composables**: PascalCase (`TaskCard`)
- **State classes**: Suffix with `State` (`NowCardState`)

## Testing Approach

### Focus Areas
- **End-to-end reliability** - App launches, displays correctly, responds to input
- **Network resilience** - Handles disconnections, timeouts, errors
- **UI readability** - Text visible on actual glasses display

### What to Test
1. App installs without errors
2. App launches to expected screen
3. DPAD navigation works (up/down/select/back)
4. Network operations have loading/error states
5. UI is readable on glasses (large text, high contrast)

### Testing Commands
```bash
# Install and verify launch
./tools/install.sh <AppName>

# Check for crashes
./tools/logcat.sh <AppName> | grep -E "(FATAL|Exception|Error)"

# Manual testing checklist
# - Press up/down: does focus move?
# - Press select/enter: does action trigger?
# - Press back: does it navigate back?
# - Disconnect network: does it show error state?
```

### Unit Tests
- Optional for MVP phase
- Add when logic is complex or bug-prone
- Focus on ViewModels and data layer

## Definition of Done for PRs

- [ ] Code compiles without warnings
- [ ] App installs on RV101 via `./tools/install.sh`
- [ ] App launches without crash
- [ ] New UI is readable on glasses (tested on device)
- [ ] DPAD navigation works for new screens
- [ ] Network operations have loading/error states
- [ ] No hardcoded secrets (use DataStore/config)
- [ ] Relevant docs updated (if behavior changed)
- [ ] CLAUDE.md status updated (if milestone completed)

## How to Update Docs When Behavior Changes

1. **New feature added**: Update APP_IDEAS.md if it differs from spec
2. **Device behavior discovered**: Update DEPLOY_RV101.md and CLAUDE.md
3. **New gotcha found**: Add to TROUBLESHOOTING.md
4. **API change**: Update EVENT_SCHEMA.md
5. **Milestone completed**: Check box in CLAUDE.md "Current Status"

## Debug Playbook

### ADB Shows "unauthorized"
```bash
# Reconnect cable
adb kill-server
adb start-server
adb devices -l

# If still unauthorized:
# 1. Open Hi Rokid app on phone
# 2. Go to Settings > Developer Options
# 3. Toggle ADB Debugging off and on
# 4. Accept prompt on glasses if shown
```

### Device Not Detected
```bash
# Check cable is 5-pin (not 3-pin charging cable)
# Try different USB port
adb kill-server
adb start-server
adb devices -l

# Check USB debugging enabled
# May need to enable in Hi Rokid app first
```

### App Installs But Won't Launch
```bash
# Check package name and activity
aapt dump badging app/build/outputs/apk/debug/app-debug.apk

# Check logcat for crash
./tools/logcat.sh <AppName>

# Verify intent-filter in AndroidManifest.xml
# Must have MAIN action and LAUNCHER category
```

### No UI / Unreadable UI
```kotlin
// Increase font sizes - minimum 24sp for body
Text(
    text = "Hello",
    fontSize = 28.sp,
    color = Color.White
)

// Use black background
Box(modifier = Modifier.background(Color.Black))

// Check actual resolution
adb shell wm size
adb shell wm density
```

### Router Unreachable (AgentHUD)
```bash
# Check same Wi-Fi network
# On glasses: Settings > Wi-Fi
# On dev machine: check network

# Check router is bound to all interfaces
# In agent-router, bind to 0.0.0.0 not localhost

# Check firewall
# macOS: System Preferences > Security > Firewall

# Fallback: use ADB port forwarding
./tools/reverse_ports.sh
# Then configure app to use ws://localhost:8787/ws
```

### WebSocket Drops Frequently
```kotlin
// Ensure reconnect logic with exponential backoff
private fun scheduleReconnect() {
    val delay = minOf(30_000L, 1000L * (1 shl reconnectAttempts))
    reconnectAttempts++
    // Schedule reconnect after delay
}

// Show connection status banner in UI
if (connectionState == Disconnected) {
    Text("Reconnecting...", color = Color.Yellow)
}
```

## Safety Guardrails

### Non-Destructive by Default
- Prefer soft-delete over hard-delete
- Confirm destructive actions with user
- Keep undo state where possible

### Token Security
- Store API keys in encrypted DataStore
- Never log tokens: `Log.d(TAG, "Token: $token")` ❌
- Use placeholder in logs: `Log.d(TAG, "Token: [REDACTED]")` ✓

### Test Before Commit
- Always deploy to device before pushing
- Don't commit code that crashes on launch
- Check logcat for errors after install

### Code Review Checklist
- No hardcoded URLs (use BuildConfig)
- No hardcoded API keys
- Error states handled
- Loading states shown
- Network timeouts configured

---
> Source: [cursive-team/rokid-apps](https://github.com/cursive-team/rokid-apps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
