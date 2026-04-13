## wt

> - [Claude Implementation Context](#claude-implementation-context)

# Claude Implementation Context

- [Claude Implementation Context](#claude-implementation-context)
  - [🎯 SESSION START CHECKLIST (MANDATORY - RUN FIRST)](#-session-start-checklist-mandatory---run-first)
  - [🚨🚨🚨 THE THREE COMMANDMENTS - NEVER VIOLATE 🚨🚨🚨](#-the-three-commandments---never-violate-)
    - [COMMANDMENT 1: THOU SHALT NOT use `git checkout` or `git switch` in main directory](#commandment-1-thou-shalt-not-use-git-checkout-or-git-switch-in-main-directory)
    - [COMMANDMENT 2: THOU SHALT NOT change branches within any worktree](#commandment-2-thou-shalt-not-change-branches-within-any-worktree)
    - [COMMANDMENT 3: THOU SHALT use wt commands for ALL branch/worktree operations](#commandment-3-thou-shalt-use-wt-commands-for-all-branchworktree-operations)
  - [🔒 MANDATORY PRE-COMMAND VALIDATION](#-mandatory-pre-command-validation)
  - [🚨 EMERGENCY PROCEDURES](#-emergency-procedures)
    - [If Dogfooding Violation Detected](#if-dogfooding-violation-detected)
    - [If Tools Fail](#if-tools-fail)
  - [🧠 CORE COGNITIVE PATTERNS](#-core-cognitive-patterns)
    - [Mental Model: Worktree-First Thinking](#mental-model-worktree-first-thinking)
    - [Decision Framework](#decision-framework)
    - [Pattern Recognition](#pattern-recognition)
  - [📋 CRITICAL DEVELOPMENT RULES](#-critical-development-rules)
    - [MANDATORY: TodoWrite/TodoRead Usage](#mandatory-todowritetodoread-usage)
    - [MANDATORY: Session State Management](#mandatory-session-state-management)
    - [MANDATORY: Dogfooding Principles](#mandatory-dogfooding-principles)
  - [🏗️ PROJECT IMPLEMENTATION NOTES](#️-project-implementation-notes)
    - [Worktree Management](#worktree-management)
    - [Zsh Integration](#zsh-integration)
    - [Command Design Principles](#command-design-principles)
  - [🔗 NAVIGATION GUIDE](#-navigation-guide)
  - [📝 IMPORTANT REMINDERS](#-important-reminders)
    - [Development Guidelines](#development-guidelines)
    - [Session State Tracking](#session-state-tracking)

## 🎯 SESSION START CHECKLIST (MANDATORY - RUN FIRST)

**Before ANY development work, Claude MUST execute these steps:**

```bash
# 1. Check current location and branch
pwd
git branch --show-current

# 2. If in main directory with wrong branch, fix immediately
if [[ $(pwd) == */wt ]] && [[ $(git branch --show-current) != "main" ]]; then
    echo "🚨 CRITICAL: Main directory on wrong branch - fixing now"
    git checkout main
fi

# 3. Read current session context
cat .claude/SESSION.md

# 4. List all worktrees to understand project state
./wt list --verbose

# 5. Confirm understanding
echo "✅ Session initialized - ready to proceed with development"
```

**Next Steps After Checklist:**
- Update `.claude/SESSION.md` with your current task
- Follow sequential task management (one worktree at a time)
- Use workflows from `.claude/WORKFLOWS.md` for complex operations

---

## 🚨🚨🚨 THE THREE COMMANDMENTS - NEVER VIOLATE 🚨🚨🚨

### COMMANDMENT 1: THOU SHALT NOT use `git checkout` or `git switch` in main directory
### COMMANDMENT 2: THOU SHALT NOT change branches within any worktree
### COMMANDMENT 3: THOU SHALT use wt commands for ALL branch/worktree operations

## 🔒 MANDATORY PRE-COMMAND VALIDATION

**COPY AND RUN THIS BEFORE EVERY GIT OPERATION:**

```bash
#!/bin/bash
# DOGFOODING VALIDATION - RUN BEFORE ANY GIT COMMAND
validate_dogfooding() {
    local current_dir=$(pwd)
    local current_branch=$(git branch --show-current 2>/dev/null || echo "unknown")

    echo "📍 Current location: $current_dir"
    echo "🌿 Current branch: $current_branch"

    # Check if in main directory with wrong branch
    if [[ "$current_dir" == */wt ]] && [[ "$current_branch" != "main" ]]; then
        echo "❌ CRITICAL VIOLATION: Main directory on wrong branch!"
        echo "🔧 Required fix: git checkout main"
        echo "⚠️  Future operations: Use worktrees only"
        return 1
    fi

    # Check for prohibited commands
    read -p "Enter your intended git command: " git_command
    if [[ "$git_command" =~ "checkout -b" ]] || [[ "$git_command" =~ "switch -c" ]]; then
        echo "❌ PROHIBITED COMMAND: $git_command"
        echo "✅ Use instead: ./wt add <branch-name>"
        return 1
    fi

    if [[ "$git_command" =~ "checkout" ]] || [[ "$git_command" =~ "switch" ]]; then
        if [[ "$current_dir" == */wt ]]; then
            echo "❌ PROHIBITED: No branch switching in main directory"
            echo "✅ Use instead: ./wt switch <worktree-name>"
            return 1
        fi
    fi

    echo "✅ Validation passed - command allowed"
    return 0
}

# Run validation
validate_dogfooding
```

## 🚨 EMERGENCY PROCEDURES

### If Dogfooding Violation Detected
1. **STOP** all operations immediately
2. **Assess state**: `./wt list --verbose`
3. **Fix main directory**: `git checkout main` if needed
4. **Document lesson**: Update `.claude/SESSION.md`
5. **Restart properly**: Follow session checklist

### If Tools Fail
1. **New Session**: Often resolves tool issues
2. **Manual Mode**: Use procedures from `.claude/WORKFLOWS.md`
3. **Document Constraint**: Note in `.claude/SESSION.md`

## 🧠 CORE COGNITIVE PATTERNS

### Mental Model: Worktree-First Thinking

**WRONG** (causes failures): "Git repository with worktree rules to remember"
**CORRECT** (prevents failures): "Worktree-managed codebase where Git is an implementation detail"

### Decision Framework

**Before ANY command:**
1. **LOCATION**: Where am I? (Main directory or worktree?)
2. **OPERATION**: What do I want? (New branch → `wt add`, Switch → `wt switch`)
3. **CONSTRAINT**: Does this violate the three commandments?
4. **ALTERNATIVE**: What's the wt way to achieve this?

### Pattern Recognition

🚫 **DANGER**: "I need to create a branch" → git checkout -b
✅ **SAFE**: "I need a new workspace" → wt add

🚫 **DANGER**: "Quick switch to..." → git checkout
✅ **SAFE**: "Let me validate first" → pre-command check

🚫 **DANGER**: Multiple tasks in same worktree
✅ **SAFE**: Parallel tasks in isolated worktrees

## 📋 CRITICAL DEVELOPMENT RULES

### MANDATORY: TodoWrite/TodoRead Usage
1. **Read Current Todo List**: Use `TodoRead` tool to check existing tasks
2. **Plan Work**: Use `TodoWrite` tool to create/update task list
3. **Track Progress**: Update status throughout development (pending → in_progress → completed)
4. **Mark Completion**: IMMEDIATELY mark tasks as completed when finished

### MANDATORY: Session State Management
- Always maintain `.claude/SESSION.md` with current tasks and progress
- Support multi-task parallel development via task isolation
- Document system constraints and blockers
- Record today's accomplishments and next actions for continuity

### MANDATORY: Dogfooding Principles
- **Always use wt commands** for worktree operations
- **Prohibit direct branch operations** in main directory
- **Prohibit branch switching** within existing worktrees

**Design Philosophy:**
- 1 worktree = 1 dedicated branch = 1 feature
- Main directory = main branch only
- Realize branch switching through worktree creation

## 🏗️ PROJECT IMPLEMENTATION NOTES

### Worktree Management
- **Organization**: All worktrees in `$REPO_ROOT/worktrees/` subdirectory
- **Auto-setup**: Auto-create directory and add to .gitignore on first use
- **Naming**: `feature/auth` → `worktrees/feature-auth/`
- **Isolation**: 1 worktree = 1 branch = 1 feature

### Zsh Integration
- **Command Role**: `wt switch` resolves and returns target path
- **Shell Role**: Zsh functions handle actual directory changing
- **Setup**: Generated by `wt shell-init`, zsh-only support
- **Tab Completion**: Integrated with zsh completion system

### Command Design Principles
- **Primary Info**: `wt list` shows all worktrees
- **Detailed View**: `wt list --verbose` for git status
- **Status Check**: `wt list --dirty` for uncommitted changes
- **Exact Matching**: No fuzzy matching to avoid complexity

## 🔗 NAVIGATION GUIDE

**For operational procedures**: See `.claude/WORKFLOWS.md`
**For current session status**: See `.claude/SESSION.md`
**For quick reference**: This file (CLAUDE.md)

## 📝 IMPORTANT REMINDERS

### Development Guidelines
- Do what has been asked; nothing more, nothing less
- NEVER create files unless absolutely necessary
- ALWAYS prefer editing existing files over creating new ones
- NEVER proactively create documentation files
- Only create documentation files if explicitly requested

### Session State Tracking
- Always maintain `.claude/SESSION.md` with current progress
- Update todo lists frequently using TodoWrite/TodoRead tools
- Document new insights and patterns for future sessions
- Ensure clear handoff information for session continuity

## 📋 DOCUMENT UPDATE GUIDELINES

### Before Any Update
1. **Information Placement**: Where does this belong? (CLAUDE.md/WORKFLOWS.md/SESSION.md/tasks/)
2. **Redundancy Check**: Does this create duplicate information?
3. **Cognitive Load**: Will this make information harder to find?
4. **Access Pattern**: Does this match actual usage frequency?

### Design Principles
- **Single Responsibility**: One source of truth per information type
- **Progressive Disclosure**: Overview → Procedure → Details
- **Information Density**: 1 file = 1 screen, minimize scrolling
- **Maintenance**: Separate static from dynamic content

### Update Validation
- **Findability Test**: Can target info be found in <2 minutes?
- **Consistency**: Same terms and formats for similar content?
- **Scalability**: Will this work with more information/users?

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/no-yan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/no-yan)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
