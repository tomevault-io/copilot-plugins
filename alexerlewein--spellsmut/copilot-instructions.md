## rules

> You are working on the **SpellForce Platinum Edition modding project**, a comprehensive toolkit for creating and modifying game content. This is a mature, production-ready project with established patterns.

# SpellForce Project - Windsurf AI Rules

## 🎯 Project Context
You are working on the **SpellForce Platinum Edition modding project**, a comprehensive toolkit for creating and modifying game content. This is a mature, production-ready project with established patterns.

## 📁 Critical File Organization Rules

### 🚨 MANDATORY - NO EXCEPTIONS
- **Python Environment**: ALWAYS use `uv run` and `uv pip install` - NEVER use `pip` or `python`
- **Test Files**: MUST be in `src/tests/` - NEVER in root or other directories
- **Documentation**: MUST be in `docs/` - NEVER in root directory
- **Source Code**: MUST be in `src/` - Follow established subfolder structure
- **Root Directory**: Config files ONLY - Keep clean

### 📂 Key Directory Structure
```
SpellSmut/
├── src/TirganachReloaded/     # Main GUI app (PySide6)
├── src/helper_tools/         # Standalone utilities
├── src/tests/               # ALL test files
├── docs/                    # User documentation
├── ProjectPlanning/         # Development planning
├── ExtractedAssets/         # Game assets
├── ModdingTools/            # Reference tools
└── .ai/                     # AI instructions (this folder)
```

## 🛠️ Technology Stack

### Core Technologies
- **Python 3.11+** with **UV** package manager (MANDATORY)
- **PySide6/Qt6** for GUI development
- **SQLite + Pickle** for dual-cache data layer
- **Pandas** for data processing
- **Pytest** for testing

### Game-Specific Formats
- **CFF files**: GameData.cff contains all game entities
- **Lua scripts**: Quest system and game logic
- **PAK archives**: 23 archives with 59,500+ files
- **DDS textures**: Icon atlases and game textures

## 🎮 Current Project Status (November 2025)

### ✅ COMPLETED SYSTEMS
- **GUI Editor**: PySide6 CFF editor with multilingual support
- **Content Creator Suite**: Weapon, Armor, Spell, NPC, Race, Quest creators
- **ID Management System**: Centralized conflict-free ID allocation
- **Asset Extraction**: Complete extraction pipeline
- **Documentation**: 15+ comprehensive guides

### ⚠️ ACTIVE CHALLENGE
- **Icon System**: Handle-to-atlas mapping resolution (critical priority)

## 🔧 Development Standards

### Code Quality Requirements
- **PEP 8 compliance** for all Python code
- **Type hints** required for function signatures
- **Comprehensive docstrings** (Google style)
- **Error handling** with proper exception management
- **Logging** using Python's logging module

### Testing Requirements
- **80% minimum test coverage** for new code
- **Integration tests** for complex workflows
- **UI tests** for critical GUI functionality
- **Performance tests** for data operations

### Data Management
- **Use dual cache system** (SQLite + Pickle) for performance
- **Validate all data** before processing
- **Provide progress feedback** for long operations
- **Handle large datasets** efficiently

## 🎯 Specific Guidelines

### When Working on GUI Components
1. Use PySide6 following Qt best practices
2. Implement proper MVC separation
3. Add comprehensive error handling for file operations
4. Test with multiple screen resolutions
5. Use signals/slots properly

### When Working on Data Processing
1. Always use the cache system
2. Validate data integrity before processing
3. Provide progress feedback for long operations
4. Handle large datasets with chunking
5. Clean up temporary files properly

### When Working on Quest System
1. Follow state machine patterns (5 states)
2. Use condition-action paradigm
3. Implement dialogue trees with branching
4. Validate quest dependencies
5. Export proper Lua format

### When Working on Asset Pipeline
1. Respect original file naming conventions
2. Maintain directory structure compatibility
3. Handle binary formats with proper endianness
4. Validate checksums for critical assets
5. Document extraction processes

## 🚨 Forbidden Patterns

### NEVER DO THESE:
- Use `pip` instead of `uv`
- Place test files outside `src/tests/`
- Put documentation in root directory
- Modify files in `OriginalGameFiles/`
- Create temporary files in project root
- Use hardcoded absolute paths
- Skip error handling for file operations
- Commit sensitive data

## ✅ Required Patterns

### ALWAYS DO THESE:
- Use `uv run script.py` for execution
- Follow established folder structure
- Write comprehensive tests
- Document complex algorithms
- Validate all inputs and file data
- Use proper logging
- Follow PEP 8 and type hints
- Update documentation when adding features

## 📋 Current Priorities

1. **Icon System Mapping**: Resolve handle-to-atlas mapping challenge
2. **Performance Optimization**: Continue optimizing data layer
3. **Documentation Maintenance**: Keep guides updated
4. **Testing**: Expand test coverage
5. **User Experience**: Polish UI/UX

## 📞 Resources

### Internal Documentation
- `ProjectPlanning/` for development status
- `docs/` for comprehensive guides
- Inline docstrings for API reference
- Type hints for function signatures

### Key Files to Understand
- `src/TirganachReloaded/main.py` - Application entry point
- `src/TirganachReloaded/data/` - Data layer implementation
- `docs/Project/` - Technical architecture
- `ProjectPlanning/Status/` - Current project status

---

**Remember**: This is a mature, production-ready project. Follow existing patterns, prioritize stability and performance, and always use UV for Python operations.

---
> Source: [AlexErlewein/SpellSmut](https://github.com/AlexErlewein/SpellSmut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
