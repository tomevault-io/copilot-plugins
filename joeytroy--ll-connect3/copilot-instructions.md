## development

> Development rules for LConnect3 fan control system


# Fan Control Rules

## NEVER Use Direct HID for Fan Control

**CRITICAL**: Do not implement direct HID communication for fan speed control. The kernel driver approach is the only reliable method.

### Why HID is Forbidden:
- Direct HID control breaks fan communication completely
- Causes fans to get stuck at low speeds (40 dBA at 80°C)
- Kernel driver "owns" the device and conflicts with direct HID
- Results in unreliable fan behavior and system instability

### Always Use Kernel Driver:
- Use `/proc/Lian_li_SL_INFINITY/Port_X/fan_speed` interface
- Kernel driver handles all USB communication reliably
- Provides stable fan control with predictable behavior
- Maximum reliable speed: ~59.5 dBA (1900 RPM) at 100%

### Implementation Pattern:
```cpp
// ✅ CORRECT - Use kernel driver
std::string procPath = "/proc/Lian_li_SL_INFINITY/Port_" + std::to_string(channel + 1) + "/fan_speed";
std::ofstream file(procPath);
if (file.is_open()) {
    file << (int)speed;
    file.close();
    return true;
}

// ❌ FORBIDDEN - Direct HID control
uint8_t report[7];
report[0] = 0xE0;
// ... HID implementation
hid_write(m_handle, report, 7);
```

### Hardware Limitations:
- Lian Li SL-Infinity fans have physical maximum ~1900 RPM
- 100% kernel driver setting = ~59.5 dBA (not 62.9 dBA)
- This is a hardware limitation, not a software issue
- Do not attempt to bypass with >100% values or HID workarounds

## C++ Best Practices for Fan Control

### Static Variables and State Management:
- Use `static` variables for persistent state in control loops
- Initialize static variables with proper default values: `static int rpm_out{0};`
- Use `static bool` for flags: `static bool holdDown{false};`
- Always initialize timers: `static QElapsedTimer holdT;`

### Timer Management:
- Always check if timer is valid before using: `if (stepTimer.isValid())`
- Use `restart()` to get elapsed time and reset: `double dt = stepTimer.restart()/1000.0;`
- Provide fallback values: `if (dt <= 0) dt = 0.1;`

### Error Handling and Debugging:
- Use `qDebug()` for debug output, not `std::cout` in Qt applications
- Include context in debug messages: `qDebug() << "RPM conversion: targetRPM=" << targetRPM;`
- Check file operations: `if (file.is_open()) { /* success */ } else { /* handle error */ }`

### Data Structures:
- Use `std::deque` for temperature history: `static std::deque<double> hist;`
- Use `qBound()` for clamping: `speedPercent = qBound(0, speedPercent, 100);`
- Use `std::clamp()` for modern C++: `target = std::clamp(target, MIN_RPM, MAX_RPM);`

### Control Loop Patterns:
- Always validate input ranges: `if (channel >= UNIHUB_SLINF_CHANNEL_COUNT) return false;`
- Use meaningful variable names: `targetRPM`, `currentTemp`, `speedPercent`
- Separate calculation from output: compute target, then apply limits, then write

### Qt Integration:
- Use Qt types when available: `QElapsedTimer` instead of `std::chrono`
- Include proper headers: `#include <QElapsedTimer>`, `#include <QDebug>`
- Use Qt's `qBound()` for value clamping

### Code Organization:
- Keep control logic in dedicated functions: `controlFanSpeeds()`, `setFanSpeed()`
- Use clear function names that describe what they do
- Group related functionality together
- Comment complex algorithms, especially control theory implementations

## Communication Guidelines

### Keep Responses Focused and Concise:
- **One task at a time** - Don't try to solve multiple problems simultaneously
- **Direct answers** - Address the specific question asked, not tangential topics
- **Minimal explanations** - Provide just enough context to understand the solution
- **Clear action items** - When suggesting changes, be specific about what to modify
- **Avoid over-engineering** - Simple solutions are often better than complex ones

### Code Change Philosophy:
- **Make small, testable changes** - One modification per iteration
- **Test immediately** - Verify each change works before moving to the next
- **Revert quickly** - If something breaks, undo it immediately
- **Document the "why"** - Brief comments explaining the reasoning behind changes

## Git Workflow Guidelines

### Use GitHub Desktop for Commits:
- **Never use command line git** - Use GitHub Desktop for all commits
- **Visual commit management** - Review changes before committing
- **Clear commit messages** - Write descriptive messages in GitHub Desktop
- **Stage changes selectively** - Choose which files to include in each commit
- **Review diffs** - Use GitHub Desktop's diff view to verify changes

### Commit Best Practices:
- **One logical change per commit** - Don't mix unrelated changes
- **Test before committing** - Ensure code compiles and works
- **Meaningful commit messages** - Describe what was changed and why
- **Use branches for experiments** - Create feature branches for major changes

## Security and Permissions

### Sudo Command Policy:
- **NEVER execute sudo commands automatically** - Always ask for permission first
- **Explain why sudo is needed** - Provide clear reasoning for elevated privileges
- **Wait for explicit approval** - Don't proceed until user confirms
- **Suggest alternatives** - If possible, recommend non-sudo approaches first
- **Document sudo usage** - Explain what the command will do and why it's necessary

---
> Source: [joeytroy/ll-connect3](https://github.com/joeytroy/ll-connect3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
