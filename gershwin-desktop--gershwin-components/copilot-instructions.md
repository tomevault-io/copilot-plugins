## gershwin-components

> - Write GNUstep compatible code. Don't use features unavailable in GNUstep such as CoreFoundation.

- Write GNUstep compatible code. Don't use features unavailable in GNUstep such as CoreFoundation.
- Use commands from `$PATH`, not hardcoded paths.
- Use `gmake`, not `make`.
- We may be running on Linux or on FreeBSD.
- Avoid bashisms; use POSIX sh.
- Build the application until it works, even with multiple attempts.
- Use -Wall -Wextra -Werror -O2, fix all compiler warnings, regardless of severity.
- Use `sudo -A -E` for commands requiring root privileges.
- Compile with `clang19`, never `gcc`.
- Use extensive logging for debugging with NSLog.
- If you build a preference pane, test with `/System/Applications/SystemPreferences.app/SystemPreferences`.
- Before running any shell commands, check which shell is being used.
- We may not have `strace`, in that case use `truss` for tracing system calls.
- Use Model-View-Controller (MVC) architecture where appropriate.
- Use KVO (Key-Value Observing) for observing changes in properties where needed.
- Always use a `GNUmakefile` to compile anything involving Foundation or GNUstep.
- Never "create a simple test to verify" anything. Verify using /System/Applications/LoginWindow.app or other system applications.
- For all UI elements, use 24px spacing from the window edges but 20 px from the top edge and at the bottom.
- Buttons are 24 px high and neighboring ones are 12 px apart from each other.
- Use `NSLocalizedString` for all user-facing strings to ensure they are localizable.
- Unless specifically asked to do so never write test cases; instead, use the existing real applications to verify functionality.
- NEVER write any .md or other summary files, unless specifically requested to do so. Never summarize successes.
- The GCD dispatch functions are available and should be used.
- ARC should be used.

---
> Source: [gershwin-desktop/gershwin-components](https://github.com/gershwin-desktop/gershwin-components) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-27 -->
