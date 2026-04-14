## audio-library-claude

> **Project**: Audio Library Manager with Multi-View Support

# Audio Library Claude - Project Memory

**Project**: Audio Library Manager with Multi-View Support
**Main Branch**: `main`
**Current Status**: V29 Refactoring Complete + Clean Directory Migration
**Localhost**: http://localhost:5500/index.html
**Last Updated**: 2025-10-18

---

## 🎯 PROJECT OVERVIEW

Audio library management web app with:
- **File upload** with auto-processing (BPM/key detection, stem separation, instrument detection)
- **Multi-stem player** with independent controls (vocals, drums, bass, other)
- **Advanced playback** (loop controls, cycle mode, markers, metronome, rate/pitch control)
- **Tag-based filtering** (can-have, must-have, exclude modes)
- **Multi-view architecture** (Library View working, Galaxy View planned, Sphere View planned)
- **Responsive design** (works on desktop and mobile browsers)

---

## 📁 DIRECTORY STRUCTURE

```
audio-library-claude/
├── index.html                 # Main HTML (production)
├── CLAUDE.md                  # This file - project memory
├── CHANGELOG.txt              # Version history
├── MIGRATION_COMPLETE.md      # Latest migration details
│
├── public/                    # Public assets
│   ├── lib/                   # External libraries (signalsmith-stretch)
│   ├── media/                 # Media files
│   └── icon/                  # App icons & favicons
│
├── styles/                    # CSS files
│   ├── main.css               # Main styles
│   ├── playerBar.css          # Player bar styles
│   ├── stems.css              # Stem player styles
│   └── views.css              # View-specific styles
│
├── src/                       # Source code
│   ├── audio/                 # Audio processing
│   ├── components/            # 14 component modules
│   ├── core/                  # Core application logic
│   ├── services/              # Service classes (FileLoader, ActionRecorder)
│   ├── state/                 # State managers (Player, Loop, Stem)
│   ├── utils/                 # Utility functions
│   └── views/                 # View renderers (Library, Galaxy, Sphere)
│
├── docs/                      # Documentation
│   ├── architecture/          # Architecture analysis docs
│   ├── refactoring/           # Refactoring progress docs
│   ├── handoffs/              # Session handoff docs
│   └── claude-code-chat-session-logs/  # Chat logs
│
└── experiments/               # Experimental/legacy code
    ├── modal-stem-icon-implementation/
    └── modular-multi-stem/
```

---

## ✅ COMPLETED REFACTORING (V29)

### State Management
- ✅ **PlayerStateManager** (`src/state/playerStateManager.js`) - Centralized player state
- ✅ **LoopStateManager** (`src/state/loopStateManager.js`) - Centralized loop state
- ✅ **StemStateManager** (`src/state/stemStateManager.js`) - Centralized stem state

### Component Modules
- ✅ **LoopControls** - Loop UI and logic
- ✅ **AdvancedRateMode** - Speed/pitch controls
- ✅ **StemLegacyPlayer** - Legacy stem waveforms (deprecated)
- ✅ **ActionRecorder** - Loop action recording
- ✅ **FileLoader** - File loading service
- ✅ **WaveformGenerator** - Waveform rendering

### Architecture Pattern
- ✅ **Hybrid State**: Local cache + centralized state managers
- ✅ **Dependency Injection**: Modules accept state/callbacks
- ✅ **Thin Wrappers**: app.js coordinates, modules contain logic
- ✅ **No God Object**: Business logic extracted from app.js

**Result**: app.js reduced from ~6500 lines → ~1900 lines

---

## 🚨 CRITICAL RULES FOR NEW CODE

### 1. FOLLOW MODULAR ARCHITECTURE
**DO NOT add large functions to app.js!**

If adding new functionality:
1. Create a new module in appropriate directory:
   - `src/components/` for UI components
   - `src/state/` for state management
   - `src/utils/` for utility functions
   - `src/services/` for service classes
2. Use dependency injection (pass state/callbacks)
3. Add thin wrapper in app.js (3-7 lines max)
4. Expose to window if needed by HTML

### 2. MULTI-VIEW REQUIREMENT
**App has 3 views** (Library working, Galaxy/Sphere planned):
- State must persist across view switches
- Player must work in all views
- Use state managers for shared state

### 3. USE EXISTING PATTERNS
- **State**: Use PlayerStateManager, LoopStateManager, StemStateManager
- **Components**: Follow existing component patterns
- **Coordination**: app.js coordinates, doesn't implement

---

## 📖 DOCUMENTATION LOCATIONS

### For Architecture Understanding:
- `docs/architecture/APP_JS_AUDIT_2025-10-18.md` - Complete app.js breakdown
- `docs/architecture/COMPREHENSIVE_CODEBASE_ANALYSIS_V30.md` - Full codebase analysis
- `MIGRATION_COMPLETE.md` - Latest directory structure

### For Refactoring Context:
- `docs/refactoring/REFACTORING_V29_COMPLETE.md` - V29 refactoring summary
- `docs/refactoring/STEM_STATE_EXTRACTION_COMPLETE.md` - Stem state details
- `docs/refactoring/LOOP_STATE_EXTRACTION_COMPLETE.md` - Loop state details
- `docs/refactoring/PLAYER_STATE_EXTRACTION_COMPLETE.md` - Player state details

### For Session Continuity:
- `docs/handoffs/` - Session handoff documents
- `CHANGELOG.txt` - Version history
- `docs/CLAUDE.md` - Copy of this file

---

## 🔧 DEVELOPMENT WORKFLOW

### Session Start
```bash
# 1. Check current state
pwd
git branch --show-current
git status

# 2. Pull latest (if working on shared branch)
git pull origin main

# 3. Create feature branch (recommended)
git checkout -b feature-name
```

### During Development
```bash
# Commit frequently with clear messages
git add .
git commit -m "feat: Add feature description

Detailed explanation if needed.

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>"
```

### Testing
```bash
# Start local server (required for CORS)
python3 -m http.server 5500

# Test in browser
open http://localhost:5500/index.html

# Check console for errors
# Test: upload, play, stems, tags, filters
```

### Session End
```bash
# Push branch to GitHub
git push origin feature-name

# Create PR if ready for review
gh pr create --title "Feature name" --body "Description"
```

---

## 🎵 CURRENT FEATURES

### Working Features ✅
- File upload with drag-and-drop
- Auto-processing (BPM, key, instruments, chords, beatmap)
- Stem separation (vocals, drums, bass, other)
- Tag filtering (can-have, must-have, exclude)
- Multi-stem player with independent controls
- Loop controls with cycle mode
- Markers (bar/beat) with shift start
- Metronome with sound selection
- Rate/pitch control (basic + advanced mode)
- Action recording for loops
- BPM lock across file changes
- Preserve loop across file changes
- Volume control with mute
- Shuffle and old-style loop
- Responsive mobile design

### In Progress 🚧
- Galaxy View (visual graph of audio files)
- Sphere View (3D visualization)

### Known Issues 🐛
- Legacy stem player (file list expansion) not working (deprecated, not fixing)
- [Add other known issues here]

---

## 📝 NEXT STEPS

### Immediate Priority
1. **Galaxy View Integration** - Add visual graph view
2. **Bug Fixes** - Address critical issues
3. **Sphere View** - 3D visualization (future)

### Future Enhancements
- PWA support (offline mode, add to home screen)
- Playlist management
- Export/share functionality
- Social features (collaborative libraries)

---

## 🔍 FINDING INFORMATION

### "Where is X functionality?"
1. Check `docs/architecture/APP_JS_AUDIT_2025-10-18.md` - Complete breakdown
2. Search `src/` for the module name
3. Check `app.js` for window exposures

### "How does X work?"
1. Check relevant state manager (`src/state/`)
2. Check component module (`src/components/`)
3. Read associated docs in `docs/refactoring/`

### "What changed recently?"
1. Check `CHANGELOG.txt` for version history
2. Check `git log --oneline` for recent commits
3. Check `docs/refactoring/` for refactoring details

---

## 🚀 COMMON TASKS

### Adding a New Feature
1. Create module in appropriate `src/` subdirectory
2. Add tests for complex logic
3. Create thin wrapper in `app.js`
4. Expose to `window` if needed by HTML
5. Update this CLAUDE.md with feature description
6. Add to CHANGELOG.txt

### Fixing a Bug
1. Identify which module contains the bug
2. Fix in the module (not app.js!)
3. Test thoroughly
4. Commit with descriptive message
5. Update CHANGELOG.txt if user-facing

### Refactoring Code
1. Read `docs/refactoring/REFACTORING_LESSONS_LEARNED.md`
2. Follow existing patterns (state managers, components)
3. Use dependency injection
4. Keep app.js as thin coordinator
5. Document changes in `docs/refactoring/`

---

## ⚠️ PITFALLS TO AVOID

1. **Adding logic to app.js** - Extract to modules instead
2. **Direct state mutation** - Use state managers
3. **Global variables** - Use state managers
4. **Monolithic functions** - Break into smaller modules
5. **Skipping documentation** - Update CLAUDE.md and CHANGELOG.txt
6. **Not testing** - Always test in browser before committing

---

## 📞 GETTING HELP

### Documentation Not Clear?
- Read related docs in `docs/architecture/` and `docs/refactoring/`
- Check git history: `git log -- path/to/file`
- Check previous session handoffs in `docs/handoffs/`

### Stuck on Architecture Decision?
- Follow existing patterns (look at similar features)
- Consult `docs/architecture/APP_JS_AUDIT_2025-10-18.md`
- When in doubt: extract to module, use dependency injection

### Breaking Changes?
- Always work on feature branch
- Commit working state before experiments
- Can revert: `git reset --hard [commit-hash]`

---

## 🎓 UNDERSTANDING THE CODEBASE

### Key Files to Understand
1. `src/core/app.js` - Application coordinator (~1900 lines)
2. `src/state/playerStateManager.js` - Player state
3. `src/state/stemStateManager.js` - Stem state
4. `src/components/loopControls.js` - Loop functionality
5. `src/views/fileListRenderer.js` - Library view

### Key Concepts
- **Hybrid State**: Local vars + state managers
- **Dependency Injection**: Modules receive dependencies
- **Thin Wrappers**: app.js delegates to modules
- **Multi-View Architecture**: State persists across views
- **Component Modules**: Reusable, testable, isolated

---

**Remember**: This is a clean, modular codebase. Keep it that way! When in doubt, extract to a module.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/John-Cunningham)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/John-Cunningham)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
