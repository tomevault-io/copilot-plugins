## kjump

> This package contains all the action classes for KJump functionality.

## Action Package

This package contains all the action classes for KJump functionality.

### Files

- `BaseAction.java` - Abstract base class for all KJump actions
- `KJumpAction.java` - Single character jump action
- `Char2Action.java` - Two character jump action
- `Word0Action.java` - Word jump action (0-based)
- `Word1Action.java` - Word jump action (1-based)
- `LineAction.java` - Line jump action
- `GotoDeclarationWord1Action.java` - Go to declaration word action
- `GlobalWord0Action.java` - Global word jump action across all editors

### Description

All actions extend `BaseAction` which provides common functionality for editor validation and jump handling. Each action implements a specific jump mode by overriding the `getMode()` method to return the appropriate mode constant from `JumpHandler` or `GlobalJumpHandler`.

---
> Source: [a690700752/KJump](https://github.com/a690700752/KJump) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
