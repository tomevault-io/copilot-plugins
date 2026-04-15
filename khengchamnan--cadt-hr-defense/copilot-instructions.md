## cadt-hr-defense

> - Organize project around these folders:

---
description: 
globs: 
alwaysApply: false
---
# Palm HR Design System and Coding Conventions

## Project Structure
- Organize project around these folders:
  - `/model`: Data models
  - `/service`: Service layer code
  - `/theme`: Theming and styling constants
  - `/utils`: Utility functions and helper classes
  - `/widgets`: Reusable app widgets
  - `/screens`: UI screens and related components

## Models
- Place in `/model` folder
- Make immutable when possible
- Include `copyWith()`, `==`, `hashCode`, and `toString()`
- No persistence, Flutter, or networking code
- Group into subfolders by topic (e.g., `/model/users`, `/model/attendance`)

## Services
- Place in `/service` folder
- Currently provide static test data

## Theme
- Define in `/theme` folder
- Use `theme.dart` for:
  - `ColorsPalmColors`
  - `TextStylesPalmTextStyles`
  - `SpacingPalmSpacings`
  - `IconsPalmIcons`
- Reference theme constants instead of hardcoding styles

## Utils
- Place in `/utils` folder
- Use for static methods for common tasks

## App Widgets
- Place in `/widgets` folder
- Group into subfolders by UI category:
  - `actions/` (buttons)
  - `inputs/` (text fields)
  - `display/` (cards, lists)
  - `notifications/` (snackbars, alerts)
- Prefix with app's short name (e.g., `palm_button.dart`)

## Screen Widgets
- Place in `/screens/{screen_name}/` folder
- Screen-specific widgets in `/screens/{screen_name}/widgets/`
- Prefix with screen name (e.g., `leave_request_history_tile.dart`)

## Comments
Three required types:
1. Class explanation:
```dart
/// This screen allows users to:
/// - Enter leave requests and submit them.
/// - Select previous leave requests and reuse them.
```

2. Statement explanation:
```dart
startDate = null;                   // User must select the start date
```

3. Step clarification:
```dart
// 1 - Notify the listener
widget.onRequestChanged(newText);
```

## Naming Conventions
- Classes/Enums/Typedefs: UpperCamelCase
- Methods/Variables: lowerCamelCase
- File Names: lowercase_with_underscores.dart
- Use getters for computed values
- Always specify types explicitly
- Use terms consistently

## Design System
- Refer to Figma Design System in `/doc` folder
- Use for widget settings (color, spaces, icons) 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KhengChamnan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/KhengChamnan)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
