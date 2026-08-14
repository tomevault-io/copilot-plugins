## quickshell

> This document provides guidelines for agentic coding agents working in this Quickshell/QML codebase.

# AGENTS.md

This document provides guidelines for agentic coding agents working in this Quickshell/QML codebase.

## Project Overview

This is a Quickshell-based desktop shell configuration written in QML. Quickshell is a Qt-based shell renderer for Wayland/X11.

## Quickshell Documentation

When working with Quickshell-specific APIs (`Quickshell.*` imports), fetch documentation from:
- Main docs: https://quickshell.org/docs/master
- Sitemap: https://quickshell.org/sitemap-0.xml

## How to Run and Test

### Basic Run
```bash
timeout 1 qs
```

The `qs` command may not return on success, so always use a timeout.

### IPC Testing (for components with IpcHandler)
First, check `shell.qml` for available IpcHandler targets.

```bash
# Start qs in background, then send IPC message
timeout 3 qs & sleep 1; qs msg <target> <func> <args...>

# Examples:
timeout 3 qs & sleep 1; qs msg reload true
timeout 3 qs & sleep 1; qs msg launcher toggle
timeout 3 qs & sleep 1; qs msg launcher standalone Clip
```

The `sleep 1` ensures qs is fully initialized before receiving messages.

### Testing Single Components
For standalone component testing via IPC:
```bash
timeout 3 qs & sleep 1; qs msg launcher standalone <ComponentName>
```
Where `<ComponentName>` matches a file in `popups/launcher/components/` (e.g., `Clip`, `Emoji`, `Calc`).

### No Automated Tests or Linting
There are no automated tests or linting tools configured for this project.

## Code Style Guidelines

### Imports

1. **No version numbers** in QML import statements:
   ```qml
   // Correct
   import QtQuick
   import Quickshell
   
   // Incorrect
   import QtQuick 2.15
   import Quickshell 1.0
   ```

2. **Order**: Place all imports at the top of each QML file.

3. **Local imports**: Use directory imports or qmldir-based imports:
   ```qml
   import "bar"
   import "popups/launcher/components"
   import qs.components    // via qmldir
   import qs.config        // via qmldir
   ```

4. **JavaScript utilities**: Use the `as` keyword:
   ```qml
   import "../utils/filename.js" as Name
   ```

### Formatting

1. **Properties and signals**: Place at the top of QML objects, before any other content:
   ```qml
   IRect {
       id: root
       property bool standalone: false
       property var state: SelectionState
       property string name
       
       signal close
       
       // Then behaviors, children, etc.
   }
   ```

2. **Property declarations**: Always specify types:
   ```qml
   property int count: 0
   property string name: ""
   property bool active: false
   property var items: []
   property color bgColor: Colors.background1
   property MprisPlayer trackedPlayer: null
   ```

### Naming Conventions

1. **QML types and components**: UpperCamelCase
   ```qml
   Bar {}
   Launcher {}
   IText {}
   SelectionState {}
   ```

2. **Properties, functions, and variables**: lowerCamelCase
   ```qml
   property int selectedIndex: 0
   property string predictiveCompletion: ""
   
   function updateTrack() { ... }
   function setActivePlayer(player: MprisPlayer) { ... }
   ```

3. **IDs**: lowerCamelCase
   ```qml
   id: root
   id: leftInnerRow
   id: scaleAnim
   ```

4. **Private/internal properties**: Prefix with underscore or double underscore:
   ```qml
   property bool __reverse: false
   property var _input: state.input
   ```

### Pragma Directives

1. **Singletons**: Use `pragma Singleton` for service/config files:
   ```qml
   pragma Singleton
   import Quickshell
   
   Singleton {
       readonly property color background1: WallustColors.color8
   }
   ```

2. **ComponentBehavior**: Use for type-safe component bindings:
   ```qml
   pragma ComponentBehavior: Bound
   ```

### QML Object Patterns

1. **Base components**: Custom base components are prefixed with `I`:
   - `IText` - Custom text with animations
   - `IRect` - Rectangle with color/opacity behaviors
   - `IWindow` - Window wrapper
   - `IPopup` - Popup window base
   - `ISpringAnimation` - Spring animation wrapper

2. **Singleton services**: Located in `services/`, provide global state:
   ```qml
   // services/Mpris.qml
   pragma Singleton
   Singleton {
       property MprisPlayer activePlayer: ...
       signal trackChanged(reverse: bool)
   }
   ```

3. **Config singletons**: Located in `config/`, provide theme/configuration:
   ```qml
   // config/Colors.qml
   pragma Singleton
   Singleton {
       readonly property color background1: ...
       readonly property color foreground1: ...
   }
   ```

4. **Launcher components**: Extend `IComponent` pattern (see `popups/launcher/components/IComponent.qml`):
   - Must have `standalone`, `state`, `prefix`, `valid`, `priority` properties
   - Implement `process()` function returning `{ valid, priority, answer, preview, predictiveCompletion }`
   - Implement navigation functions: `up()`, `down()`, `prev()`, `next()`, `exec()`

### Animations

1. **Behavior on property**: For smooth transitions:
   ```qml
   Behavior on color {
       ColorAnimation {
           duration: General.animationDuration
           easing.type: Easing.InOutQuad
       }
   }
   ```

2. **Spring animations**: Use `ISpringAnimation`:
   ```qml
   Behavior on x {
       ISpringAnimation {}
   }
   ```

3. **Animation durations**: Reference `General.animationDuration` or `Style.anim.durations.*`.

### Error Handling

1. **JavaScript utilities**: Use exceptions for errors:
   ```javascript
   function riskyOperation() {
       if (!condition) {
           throw new Error("Operation failed");
       }
   }
   ```

2. **QML**: Use console.log for debugging, null checks for safety:
   ```qml
   property var activePlayer: Mpris.players.values[0] ?? null
   
   function exec() {
       if (this.canTogglePlaying)
           this.activePlayer.togglePlaying();
   }
   ```

### Comments

1. Use comments for TODOs and known issues:
   ```qml
   // TODO: use visible instead of opacity for perf
   // FIXME: currently relying on compositor animation
   ```

2. Document complex logic with inline comments.

3. Use JSDoc-style comments in JavaScript files:
   ```javascript
   /**
    * Formats a string according to the args that are passed in
    * @param { string } str 
    * @returns { string }
    */
   function format(str, ...args) { ... }
   ```

## Common Tasks

### Adding a new launcher component

1. Create `popups/launcher/components/NewComponent.qml`
2. Extend `IComponent` and implement required properties/functions
3. Add to `Launcher.widgets` list in `popups/launcher/ILauncher.qml`

### Adding a new bar module

1. Create `bar/NewModule.qml`
2. Import and add to `Bar.qml` in the appropriate RowLayout
3. Use `IText`, `IRect`, `Icon` components for consistent styling

### Adding IPC commands

1. Add `IpcHandler` to `shell.qml`:
   ```qml
   IpcHandler {
       target: "newTarget"
       function doSomething(arg: string): void {
           // implementation
       }
   }
   ```

2. Test with: `qs msg newTarget doSomething "argument"`

---
> Source: [littleblack111/quickshell](https://github.com/littleblack111/quickshell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
