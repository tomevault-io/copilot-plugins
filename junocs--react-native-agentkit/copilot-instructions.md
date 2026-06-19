## react-native-agentkit

> Control React Native mobile apps via CLI commands. Discover UI elements, tap buttons, type text, toggle switches, scroll views, and navigate screens programmatically. Run natural language test cases autonomously.


# React Native AgentKit — AI Agent Skill

You can control React Native mobile apps that have integrated `react-native-agentkit`. The app exposes its live UI as structured data over a WebSocket connection, and you interact with it by sending JSON commands.

## Core Loop

Your workflow follows an **observe → reason → act** cycle:

1. **Observe** — Send `state` or `list` to see all interactive elements on screen
2. **Reason** — Map the user's intent to specific element IDs and actions
3. **Act** — Send commands (`tap`, `type`, `scroll`, etc.) to interact
4. **Repeat** — Each response includes updated screen state; continue until the task is done

## Pre-Flight Check

**Before interacting with the app UI, always check if the app is already connected.** If it is, skip device setup entirely:

```bash
# Step 0: Check if app is already connected
echo '{"cmd":"ping"}' | npx react-native-agentkit pipe --relay=ws://localhost:8347
```

- If you get `{"success": true, "result": {"pong": true}}` → **skip to UI commands** (the app is running)
- If you get an error or timeout → **proceed with device setup** below

This prevents unnecessary device operations when the app is already running and connected.

## Device Setup (when app is not running)

If the pre-flight check fails, you can autonomously prepare the device. Device commands do NOT require a relay connection — they operate directly on the OS.

### Setup Flow

```bash
# 1. Check what's available
npx react-native-agentkit device list --platform=ios

# 2. Boot a simulator (fuzzy-matches by name)
npx react-native-agentkit device boot --name="iPhone 16"

# 3. Pre-grant permissions (prevents native alert interruptions during automation)
npx react-native-agentkit device grant-permissions --bundleId=com.myapp --permissions=camera,photos,location,notifications

# 4. Install the app
npx react-native-agentkit device install ./ios/build/Build/Products/Debug-iphonesimulator/MyApp.app

# 5. Launch the app
npx react-native-agentkit device launch com.myapp

# 6. Wait a moment for relay connection, then begin UI automation
echo '{"cmd":"state"}' | npx react-native-agentkit pipe --relay=ws://localhost:8347
```

### Device Command Reference

| Command | Purpose | Example |
| --- | --- | --- |
| `device list` | List simulators/emulators | `device list --platform=ios` |
| `device boot` | Boot a device (fuzzy name match) | `device boot --name="iPhone 16"` |
| `device shutdown` | Shutdown a device | `device shutdown <udid>` |
| `device install` | Install app | `device install ./path/to/MyApp.app` |
| `device uninstall` | Uninstall app | `device uninstall com.myapp` |
| `device launch` | Launch an app | `device launch com.myapp` |
| `device terminate` | Terminate a running app | `device terminate com.myapp` |
| `device erase` | Factory reset a device | `device erase <udid>` |
| `device create-emulator` | Create Android AVD | `device create-emulator MyDevice` |
| `device grant-permissions` | Pre-grant OS permissions | `device grant-permissions --bundleId=com.myapp --permissions=camera,photos` |
| `device reset-permissions` | Reset permissions to prompt | `device reset-permissions --bundleId=com.myapp` |
| `device clear-data` | Clear app data (Android) | `device clear-data com.myapp` |
| `device status` | Show all devices and tools | `device status` |

All device commands output JSON and can be used in automation scripts.

### Permission Pre-Granting

Pre-granting OS permissions **before** launching the app prevents native permission dialogs from appearing during automation. This is critical for uninterrupted test flows.

**iOS** — available services:
`calendar`, `contacts`, `location`, `location-always`, `camera`, `microphone`, `photos`, `reminders`, `siri`, `notifications`, `speech-recognition`, `motion`, `media-library`, or `all`

```bash
npx react-native-agentkit device grant-permissions \
  --bundleId=com.myapp --permissions=camera,photos,location,notifications
```

**Android** — uses short or full permission names:
```bash
npx react-native-agentkit device grant-permissions \
  --packageId=com.myapp --permissions=CAMERA,ACCESS_FINE_LOCATION
```

> **Note:** On Android, `device install` already grants all manifest-declared permissions by default (via `adb install -g`). Use `grant-permissions` only for selective granting after install.

> **Note:** Permission pre-granting handles **OS-level** permission dialogs. The app's built-in `NativeDialogInterceptor` handles **JS-level** dialogs (Alert.alert, react-native-permissions). Both layers work together — pre-grant to prevent OS dialogs, and the interceptor catches any remaining JS-level ones.

### Physical iOS Device

If the user asks to run on a physical iPhone/iPad, use `ios-deploy` (auto-installed via Homebrew if missing):

```bash
# List connected physical devices
npx react-native-agentkit device list --platform=ios
# (Physical devices appear alongside simulators)

# Install and launch on physical device
npx react-native-agentkit device install ./MyApp.app --device=<udid>
npx react-native-agentkit device launch com.myapp --device=<udid>
```

### Expo Projects

For Expo-managed projects, use the `--expo` flag to delegate to Expo CLI:

```bash
npx react-native-agentkit device install --expo --platform=ios
npx react-native-agentkit device launch --expo --platform=android
```


## Connecting

### Local Development (via CLI)

```bash
# Single command
npx react-native-agentkit exec state --relay=ws://localhost:8347

# Pipe mode (best for automation — send JSON in, get JSON out)
echo '{"cmd":"state"}' | npx react-native-agentkit pipe --relay=ws://localhost:8347
```

### Production (via Cloud Relay)

```bash
# Connect through a relay server (with authentication)
npx react-native-agentkit exec state --relay=ws://relay-host:8347 --channel=my-app --secret=your-shared-secret

# Pipe mode through relay
echo '{"cmd":"state"}' | npx react-native-agentkit pipe --relay=ws://relay-host:8347 --channel=my-app --secret=your-shared-secret
```

### Pipe Mode Protocol

In pipe mode, communication is line-delimited JSON over stdin/stdout:

- **Send**: Write one JSON object per line to stdin (e.g., `{"cmd":"state"}\n`)
- **Receive**: Read one JSON response object per line from stdout
- **Important**: Each line must be a complete, valid JSON object — no multi-line formatting

## Commands Reference

### Discovery Commands

| Command | Purpose                                                 | JSON                                                       |
| ------- | ------------------------------------------------------- | ---------------------------------------------------------- |
| `state` | Full screen snapshot with all elements                  | `{"cmd":"state"}`                                          |
| `list`  | List interactive elements (with optional type filter)   | `{"cmd":"list"}` or `{"cmd":"list","filterType":"button"}` |
| `find`  | Search elements by text across id, label, and value     | `{"cmd":"find","filterText":"setpoint"}`                   |
| `read`  | Read a specific element's current value and state       | `{"cmd":"read","target":"element-id"}`                     |
| `wait`  | Wait for an element to appear (useful after navigation) | `{"cmd":"wait","target":"element-id","timeout":5000}`      |
| `ping`  | Check if the bridge is alive                            | `{"cmd":"ping"}`                                           |

### Action Commands

| Command     | Purpose                                                  | JSON                                                                    |
| ----------- | -------------------------------------------------------- | ----------------------------------------------------------------------- |
| `tap`       | Tap/press a button or pressable element                  | `{"cmd":"tap","target":"element-id"}`                                   |
| `toggle`    | Toggle a switch or checkbox                              | `{"cmd":"toggle","target":"element-id"}`                                |
| `setValue`  | Set the value of a slider or numeric control             | `{"cmd":"setValue","target":"element-id","value":75}`                   |
| `longPress` | Long press an element                                    | `{"cmd":"longPress","target":"element-id"}`                             |
| `type`      | Type text into an input field (appends to current value) | `{"cmd":"type","target":"element-id","text":"hello"}`                   |
| `clear`     | Clear an input field                                     | `{"cmd":"clear","target":"element-id"}`                                 |
| `scroll`    | Scroll a scrollable view                                 | `{"cmd":"scroll","target":"scroll-id","direction":"down","amount":300}` |
| `swipe`     | Swipe an element in a direction                          | `{"cmd":"swipe","target":"item-1","direction":"left"}`                  |
| `back`      | Navigate back                                            | `{"cmd":"back"}`                                                        |

## Understanding the Screen State

When you send `state`, you get back a list of elements like:

```json
{
  "success": true,
  "result": {
    "screenState": {
      "elements": [
        {
          "id": "cool-setpoint",
          "type": "input",
          "label": "Cool Setpoint",
          "value": "72",
          "actions": ["type", "clear", "focus", "blur"],
          "state": { "disabled": false, "selected": false }
        },
        {
          "id": "mode-selector",
          "type": "button",
          "label": "Operation Mode: Heat",
          "actions": ["tap"],
          "state": { "disabled": false, "selected": false }
        },
        {
          "id": "fan-auto-switch",
          "type": "switch",
          "label": "Fan Auto",
          "actions": ["toggle"],
          "state": { "disabled": false, "selected": false, "checked": true }
        },
        {
          "id": "settings-tab",
          "type": "tab",
          "label": "Settings",
          "actions": ["tap"],
          "state": { "disabled": false, "selected": false }
        }
      ],
      "count": 4
    }
  }
}
```

### Action Response Example

When you send an action command (e.g., `tap`), the response includes both the action result and the updated screen state — so you don't need a separate `state` call:

```json
{
  "success": true,
  "command": "tap",
  "target": "mode-selector",
  "result": { "elementTapped": true },
  "screenState": {
    "elements": [
      {
        "id": "mode-heat",
        "type": "button",
        "label": "Heat",
        "actions": ["tap"],
        "state": { "selected": true }
      },
      {
        "id": "mode-cool",
        "type": "button",
        "label": "Cool",
        "actions": ["tap"],
        "state": { "selected": false }
      },
      {
        "id": "mode-auto",
        "type": "button",
        "label": "Auto",
        "actions": ["tap"],
        "state": { "selected": false }
      }
    ],
    "count": 3
  },
  "timestamp": 1710000000000
}
```

### Key Element Fields

- **`id`** — The target identifier you use in commands (from `testID`, `accessibilityLabel`, or auto-generated)
- **`type`** — One of: `button`, `input`, `text`, `switch`, `slider`, `scroll`, `link`, `header`, `list`, `tab`, `modal`, `view`, `image`
- **`label`** — Human-readable description of what the element is
- **`value`** — Current text/value content (for inputs, displays)
- **`actions`** — What you can do with this element (`tap`, `type`, `clear`, `scroll`, `toggle`, `setValue`, `swipe`, `longPress`, `focus`, `blur`)
- **`state.disabled`** — If `true`, the element won't respond to actions
- **`state.checked`** — For switches/toggles, the current on/off state

## Workflow Patterns

### Pattern 1: Modify an Input Value

```
1. {"cmd":"state"}                                    → Find the input's id and current value
2. {"cmd":"clear","target":"cool-setpoint"}           → Clear existing value
3. {"cmd":"type","target":"cool-setpoint","text":"75"} → Type new value
```

### Pattern 2: Navigate Then Act

```
1. {"cmd":"state"}                                    → See current screen
2. {"cmd":"tap","target":"settings-btn"}              → Navigate to settings
3. {"cmd":"wait","target":"temperature-input","timeout":3000} → Wait for new screen to load
4. {"cmd":"state"}                                    → See new screen elements
5. (continue with actions on the new screen)
```

### Pattern 3: Find Elements When IDs Are Unknown

```
1. {"cmd":"find","filterText":"setpoint"}             → Search by text
2. {"cmd":"list","filterType":"button"}               → List only buttons
3. {"cmd":"find","filterText":"mode"}                 → Search for mode-related elements
```

### Pattern 4: Toggle a Switch

```
1. {"cmd":"read","target":"fan-auto-switch"}          → Check current state
2. (if state.checked is not desired value)
3. {"cmd":"toggle","target":"fan-auto-switch"}         → Toggle it
```

### Pattern 5: Long Press for Context Menu

```
1. {"cmd":"state"}                                    → Find the element to long-press
2. {"cmd":"longPress","target":"list-item-3"}          → Long press to open context menu
3. {"cmd":"wait","target":"context-menu","timeout":2000} → Wait for menu to appear
4. {"cmd":"state"}                                    → See menu options
5. {"cmd":"tap","target":"delete-option"}              → Select an option
```

### Pattern 6: Switch Between Tabs

```
1. {"cmd":"list","filterType":"tab"}                   → List all tab elements
2. {"cmd":"tap","target":"settings-tab"}               → Tap the desired tab
3. {"cmd":"wait","target":"settings-content","timeout":3000} → Wait for tab content to load
4. {"cmd":"state"}                                    → See the new tab's elements
```

### Pattern 7: Handle Native Alerts & Permissions

Native `Alert.alert()` dialogs are automatically intercepted. When an agent is connected, the native dialog is suppressed and buttons are registered as tappable elements. When no agent is connected, the native dialog shows normally.

```
1. {"cmd":"tap","target":"delete-account-btn"}         → Tap triggers Alert.alert('Confirm', '...', [{text:'Cancel'}, {text:'Delete'}])
2. {"cmd":"list","filterType":"button"}                 → Alert buttons now appear: alert-confirm-cancel, alert-confirm-delete
3. {"cmd":"tap","target":"alert-confirm-delete"}        → Execute the Delete button's action
```

Android permission requests (`PermissionsAndroid`) and `react-native-permissions` library calls (iOS + Android) are also intercepted. Allow/Don't Allow/Never Ask Again buttons appear as elements:

```
1. (App requests CAMERA permission)                       → Agent sees: permission-camera-grant, permission-camera-deny, permission-camera-block
2. {"cmd":"tap","target":"permission-camera-grant"}       → Grants the permission
```

> **Note:** Alert/permission buttons are auto-cleaned up after being tapped or after a timeout.

### Pattern 8: Scroll to Explore Content

Mobile screens often have content below the visible area. If you can't find an element, or are asked to explore/read the full screen, scroll down to reveal more content:

```
1. {"cmd":"state"}                                         → See visible elements — look for scroll-type elements
2. {"cmd":"scroll","target":"main-scroll","direction":"down"} → Scroll down to reveal more content
3. {"cmd":"state"}                                         → Check for newly visible elements
4. (repeat scrolling until you find the target or reach the bottom)
5. {"cmd":"scroll","target":"main-scroll","direction":"up"}   → Scroll back up if needed
```

> **Tip:** Use `list` with `filterType: "scroll"` to find all scrollable containers. Horizontal lists use `left`/`right` directions instead of `up`/`down`.

### Pattern 9: Swipe to Delete

Elements with `swipe` in their actions support directional swiping. Commonly used for swipe-to-delete in lists:

```
1. {"cmd":"swipe","target":"home-activity-1","direction":"left"}  → Swipe left to reveal delete action
2. {"cmd":"list","filterType":"button"}                          → See the delete button that appeared
3. {"cmd":"tap","target":"home-activity-1-delete-btn"}           → Tap delete to remove the item
```

> **Tip:** Swipe `right` to close a swiped row without deleting. Check the element's `state.selected` to see if it's currently swiped open.

## Test Cases

The project may contain pre-written test cases in `.rnatest.js` (or `.rnatest.ts`, `.rnatest.tsx`, `.rnatest.jsx`) files. When the user asks you to "run tests", "test the app", or "run my test cases", you should discover and execute these test files.

### 1. Discover Test Files

Search the project for files matching `*.rnatest.*`:

```bash
find . -name "*.rnatest.*" -not -path "*/node_modules/*"
```

### 2. Read and Understand Test Structure

Each test file exports a structured object using `describe`/`it` blocks. When you read a test file, you'll see something like:

```js
const { describe, it, beforeEach } = require('react-native-agentkit/test');

module.exports = describe('Login Flow', () => {

  beforeEach(`
    Make sure we are on the login screen
  `);

  it('should login with valid credentials', `
    Clear the email input and type "user@test.com"
    Clear the password input and type "password123"
    Tap the submit button
    The Home screen should appear with a welcome message
  `);

});
```

The `describe` block groups tests. Each `it` block contains natural language instructions — these are the steps you need to execute using AgentKit commands.

### 3. Execute Each Test

For each test file, follow this execution order:

1. If `beforeAll` exists, execute those instructions once before all tests
2. For each `it` test in the suite:
   a. If `beforeEach` exists, execute those instructions
   b. Execute the test's instructions line by line
   c. If `afterEach` exists, execute those instructions
   d. Track whether the test passed or failed
3. If `afterAll` exists, execute those instructions once after all tests
4. For nested `describe` blocks, apply the same logic recursively

### 4. Translating Natural Language Instructions to Commands

Each line in a test is a natural language instruction. Translate it into AgentKit commands by examining the current screen state:

| Instruction pattern | How to execute |
| --- | --- |
| "Tap/Press/Click the X" | `state` → find element matching "X" → `{"cmd":"tap","target":"<id>"}` |
| "Type/Enter \"Y\" into X" | `state` → find input matching "X" → `{"cmd":"clear","target":"<id>"}` then `{"cmd":"type","target":"<id>","text":"Y"}` |
| "Clear the X input" | `state` → find input matching "X" → `{"cmd":"clear","target":"<id>"}` |
| "Toggle/Switch X on/off" | `state` → find switch matching "X" → `{"cmd":"toggle","target":"<id>"}` |
| "Scroll down/up on X" | `state` → find scroll matching "X" → `{"cmd":"scroll","target":"<id>","direction":"down"}` |
| "Swipe left/right on X" | `state` → find element matching "X" → `{"cmd":"swipe","target":"<id>","direction":"left"}` |
| "Navigate to X" / "Go to X" | `state` → find tab/button matching "X" → `{"cmd":"tap","target":"<id>"}` |
| "Wait for X to appear" | `{"cmd":"wait","target":"<best-match-id>","timeout":5000}` |
| "Go back" / "Navigate back" | `{"cmd":"back"}` |
| "Set X to Y" | `state` → find slider/input matching "X" → `{"cmd":"setValue","target":"<id>","value":Y}` |
| "Long press on X" | `state` → find element matching "X" → `{"cmd":"longPress","target":"<id>"}` |

**Assertion patterns** (verify but don't act):

| Assertion pattern | How to verify |
| --- | --- |
| "X should appear/exist/be visible" | `{"cmd":"find","filterText":"X"}` → assert count > 0 |
| "X should not exist/be gone" | `{"cmd":"find","filterText":"X"}` → assert count = 0 |
| "X should contain/show \"Y\"" | `{"cmd":"read","target":"<id>"}` → assert value contains "Y" |
| "X should be enabled/disabled" | `{"cmd":"read","target":"<id>"}` → assert state.disabled matches |
| "X should be checked/on" | `{"cmd":"read","target":"<id>"}` → assert state.checked = true |
| "X should be unchecked/off" | `{"cmd":"read","target":"<id>"}` → assert state.checked = false |
| "The screen should show X" | `{"cmd":"state"}` → look for element with label/value containing "X" |

**Key:** Always run `state` or `find` first to discover element IDs on the current screen, then match the natural language target to the closest element by its label, ID, or value.

### 5. Report Results

After executing all tests, report a clear summary:

```
Test Results: Login Flow
  ✅ should login with valid credentials
  ❌ should show error for invalid credentials
     Step failed: "An error message should be visible"
     Reason: No element matching "error" found on screen
     Last screen state: 5 elements (email-input, password-input, submit-btn, ...)

2 tests: 1 passed, 1 failed
```

For each failed test, include:
- Which step failed
- What was expected vs what was found
- The last known screen state (for debugging)

## Error Handling

If a command fails, the response has `"success": false` and an `"error"` message:

```json
{
  "success": false,
  "command": "tap",
  "error": "Element not found: nonexistent-button"
}
```

**Common errors and recovery:**

| Error                                      | Cause                             | Recovery                                                  |
| ------------------------------------------ | --------------------------------- | --------------------------------------------------------- |
| `Element not found: X`                     | Element not on current screen     | Send `state` to see what's available, or `find` to search |
| `Element is disabled`                      | Button/input is disabled          | Check prerequisites (e.g., a required field or toggle)    |
| `Element does not have an onPress handler` | Wrong element type for the action | Use `read` to check the element's `actions` list          |
| `Timed out waiting for element`            | Screen didn't load in time        | Increase `timeout` or check if navigation succeeded       |

## Timeout & Bail-Out

**You MUST stop and report if you cannot make progress within 60 seconds.**

Stuck loops are the biggest risk in automated testing. Follow these rules:

1. **Track your progress** — After each action, verify the screen state actually changed. If three consecutive actions produce the same screen state, you are stuck.
2. **60-second hard limit** — If you have been attempting the same step for more than 60 seconds without success, stop immediately. Do not keep retrying.
3. **Retry limit** — Do not retry the same failed action more than 3 times. After 3 failures, try an alternative approach or bail out.
4. **Report clearly** — When bailing out, explain:
   - What you were trying to do
   - What went wrong (element not found, action had no effect, unexpected screen, etc.)
   - The last known screen state
   - Suggested next steps for the developer

**Example bail-out response:**
```
I was unable to complete the login flow. After tapping "submit-btn", the screen
did not navigate to the Home screen as expected. The screen still shows the Login
form after 3 attempts. This may indicate a validation error not visible in the
element tree, or a network issue preventing login. Last screen state had 5 elements
including: email-input, password-input, submit-btn, signup-link, forgot-password-link.
```

## Best Practices

1. **Always start with `state`** — Know what's on screen before acting
2. **Use `wait` after navigation** — Screens take time to render; wait for a key element before continuing
3. **Clear before typing** — Use `clear` then `type` to replace a value; `type` alone appends
4. **Check `actions` array** — Only attempt actions that appear in the element's `actions` list
5. **Check `state.disabled`** — Disabled elements will reject commands
6. **Use `find` for exploration** — When you don't know exact IDs, search by text
7. **Read the updated `screenState`** — After action commands, the response includes the new screen state (no need for a separate `state` call)
8. **Handle multi-screen flows** — Some actions trigger navigation; always re-observe after screen changes
9. **Handle alerts & permissions** — Native dialogs are auto-intercepted when an agent is connected. Run `list` after triggering an alert to see buttons (`alert-{title}-{text}`). Permissions appear as `permission-{name}-grant` / `permission-{name}-deny` / `permission-{name}-block`
10. **Scroll to explore** — Mobile screens often extend beyond the visible area. If you can't find an element or are asked to explore, scroll down through scrollable containers to reveal more content. Always check for `scroll`-type elements in the element list
11. **Bail out after 60 seconds** — If you cannot make progress on a step within 60 seconds or after 3 retries, stop and report what went wrong rather than looping indefinitely
12. **Run test cases when asked** — If the user asks you to "run tests" or "test the app", look for `.rnatest.*` files in the project and execute them following the Test Cases section above

## End-to-End Example: AC System Control

**User request:** "Set the cool setpoint to 75 and switch to cool mode"

```
Agent → {"cmd":"state"}
App   ← elements: [mode-selector (button, "Mode: Heat"), cool-setpoint (input, "72"), heat-setpoint (input, "68"), apply-btn (button)]

Agent → {"cmd":"clear","target":"cool-setpoint"}
App   ← success, value cleared

Agent → {"cmd":"type","target":"cool-setpoint","text":"75"}
App   ← success, newValue: "75"

Agent → {"cmd":"tap","target":"mode-selector"}
App   ← success, screenState now shows: [mode-heat (button, selected:true), mode-cool (button), mode-auto (button)]

Agent → {"cmd":"tap","target":"mode-cool"}
App   ← success, screenState shows mode-cool is now selected

Agent → {"cmd":"tap","target":"apply-btn"}
App   ← success, changes applied

Agent → "Done! I set the cool setpoint to 75°F and switched the operation mode to Cool."
```

---
> Source: [junocs/react-native-agentkit](https://github.com/junocs/react-native-agentkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
