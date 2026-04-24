## visualpostfixcalculator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Run

```bash
# Run the application
./gradlew run

# Build a JAR
./gradlew jar

# Run tests
./gradlew test

# Run a single test class
./gradlew test --tests "com.s1scottd.SomeTestClass"
```

## Architecture

This is a JavaFX desktop application implementing an RPN (Reverse Polish Notation) postfix calculator with a visual stack display.

**Key components:**

- `VisualPostfixCalculator` — Main JavaFX `Application` subclass. Owns the UI layout (`BorderPane` with a scrollable stack panel on the left and calculator grid on the right), handles all button events, and calls into the stack.
- `LinkedListStack<T>` — Generic stack backed by a `LinkedList`. Provides `push`, `pop`, `peek`, `clear`, `size`, `isEmpty`, and `getContents()` (returns a copy of all elements for display).

**Calculator logic flow:**
- Digits/decimal typed → appended to the `TextField` display.
- `Enter` → `processEntry()` parses display text and pushes it onto the stack.
- Operators (`+`, `-`, `*`, `/`) → if display is non-empty and not already pushed, calls `processEntry()` first, then `performOperation()` which pops two values, computes the result, pushes it back, and shows it in the display.
- `CHS` (change sign) → pops top of stack, negates it, pushes back.
- `CLx` → clears both the display and the entire stack.
- `updateStackDisplay()` re-renders the `VBox` stack panel from `stack.getContents()` after every state change.

**Module system:** The project uses Java 9 modules (`module-info.java` at `app/src/main/java/module-info.java`). The module is named `VisualPostfixCalculator` and exports `com.s1scottd`.

**Icons:** Operator buttons use Ikonli FontAwesome 5 icons (`org.kordamp.ikonli:ikonli-fontawesome5-pack`). Icon literals follow the `fas-*` naming convention (e.g., `fas-plus`, `fas-divide`).

**Java/JavaFX version:** Java 17 toolchain, JavaFX 17.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ScottsSecondAct) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
