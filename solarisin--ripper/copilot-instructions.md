## ripper

> This document lists coding style guidelines to be applied at all times


# Style guide
This document lists coding style guidelines to be applied at all times

## General style
- avoid using global variables, prefer passing parameters to functions
- avoid using mutable default arguments in function definitions
- use f-strings for string formatting
- avoid using print statements for debugging, use the loguru module instead
- avoid imports inside functions, prefer placing them at the top of the file
- ensure no blank lines contain any whitespace
- ensure there is no whitespace at the end of lines

## Documentation
- use descriptive names for functions and variables
- ensure that all functions have docstrings explaining their purpose and usage

## Unit tests
- newly added code should always be accomapnied by a comprehensive unit test
- update unit tests for any modified production code


## Type Hints
- Use modern python type hints, especially for parameters and return values
- prefer | over Union for unioned types
- prefer typing constructs from the beartype.typing module

## Qt Related Style
- Use PySide6 (Qt6) instead of PyQt5 or other Qt bindings
- Follow Qt best practices for widget creation and layout management
- Include proper signal-slot connections where appropriate
- Create clean, modular widget classes that can be easily tested
- Include proper error handling for GUI operations
- Use Qt's resource system when dealing with images or icons
- Follow Qt styling conventions and use Qt Style Sheets (QSS) when needed

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/solarisin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
