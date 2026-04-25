## tower-of-infinity-pt

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Tower of Infinity** is a Unity-based turn-based RPG prototype using feature-based architecture. The project follows SOLID principles where practical but prioritizes Unity conventions and performance when needed.

## Development Commands

This is a Unity project - use Unity Editor for building and testing. No external build commands are currently configured.

**Testing:** No dedicated test runner is configured. Use Unity's built-in Test Runner if needed.

**Linting:** No automated linting is configured. Follow C# conventions and Unity best practices.

## Architecture & Code Structure

### Core Architecture Pattern
- **Feature-Based Architecture**: Each major game system is isolated in `Assets/Project/Features/`
- **Singleton Pattern**: Core managers use singleton pattern for global access (CombatManager, TurnManager)
- **ScriptableObject Data**: Game data stored as ScriptableObjects with `SO_` prefix

### Folder Structure
```
Assets/Project/Features/
├── Combat/          # Combat mechanics, damage calculation, turn management
├── Core/            # Shared systems (LogManager, etc.)
├── UI/              # All UI components and managers
└── Units/           # Character stats, unit data, resource systems
```

### Key System Dependencies
- **TurnManager**: Controls combat flow, manages unit turns via coroutine system
- **CombatManager**: Central combat coordinator, holds elemental matchup table
- **DamageCalculator**: Static utility class handling all damage calculations
- **LogManager**: Centralized logging system used throughout codebase

### Design Patterns in Use
- **Singleton**: CombatManager, TurnManager, LogManager
- **Strategy Pattern**: SkillEffect hierarchy with DamageEffect, ResourceCostEffect
- **Observer Pattern**: Event system for combat state changes

### Code Conventions
- **Naming**: PascalCase for all classes, methods, and public fields
- **Prefixes**: 
  - `SO_` for ScriptableObjects
  - `PF_` for Prefabs (when applicable)
- **Unity Integration**: MonoBehaviour lifecycle methods (Awake, Start) used appropriately
- **Logging**: Use `LogManager.Log()` for debug output, `LogManager.LogError()` for errors

### Known Technical Debt
Based on existing code reviews, be aware of:
1. **Performance Issues**: TurnManager uses Update() calls that should be converted to coroutines
2. **SOLID Violations**: DamageCalculator has too many responsibilities, should be decomposed
3. **Dependency Issues**: Heavy reliance on static classes makes unit testing difficult
4. **Null Safety**: Missing null checks in GetComponent calls and array access

### Current Refactoring Priorities
1. Convert TurnManager Update() to coroutine-based system
2. Decompose DamageCalculator into smaller, focused classes
3. Add interface abstractions for better testability
4. Implement proper null checking throughout

### Unity-Specific Considerations
- Project uses Universal Render Pipeline (URP)
- Input System package is integrated
- TextMeshPro is used for UI text
- Features are designed to work with Unity's serialization system

## Development Guidelines

### When Adding New Features
1. Create new feature folder under `Assets/Project/Features/`
2. Use ScriptableObjects for data configuration
3. Follow existing singleton patterns for managers
4. Integrate with LogManager for debugging
5. Consider event system for loose coupling

### Code Quality Standards
- Maintain Unity MonoBehaviour patterns
- Use appropriate access modifiers (prefer private/protected)
- Document complex algorithms with comments
- Test in Unity Editor before committing changes
- Follow existing error handling patterns

### Working with Combat System
- All damage calculations go through DamageCalculator
- Turn management is handled by TurnManager singleton
- Unit interactions use InGameUnit as base class
- Skills use SkillEffect inheritance hierarchy

This codebase balances architectural principles with Unity development pragmatism. Focus on maintainability and extensibility while respecting Unity's component-based paradigms.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KORWOT) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
