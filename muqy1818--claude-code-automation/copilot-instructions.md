## claude-code-automation

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## System Architecture

This repository contains a **Python-based Claude Code automation framework** with a sophisticated dual-layer architecture for intelligent development workflows:

### **Layer 1: Native Claude Code Hooks**
- **Purpose**: Real-time prompt modification and intelligent response behavior control
- **Technology**: Python-based hook scripts with advanced flag processing
- **Configuration**: `.claude/settings.json` (Claude Code native format)
- **Scope**: UserPromptSubmit events with comprehensive flag system
- **Key Innovation**: Flag dispatcher for instant response mode switching

### **Layer 2: Advanced Command System** 
- **Purpose**: Sophisticated development workflows and context-aware automation
- **Technology**: Python framework with context management and intelligent detection
- **Configuration**: `.claude/settings/` folder with hierarchical JSON configuration
- **Scope**: 11 specialized slash commands, project intelligence, multi-level hook system
- **Key Innovation**: Context-aware execution adapting to project type and environment

### Core Components

- **Native Hook Scripts** (`hooks/UserPromptSubmit/`): Prompt modification for response behavior
- **Command System Core** (`core/`): Advanced automation framework
  - **Context Manager** (`core/context_manager.py`): Intelligent project detection
  - **Command Registry** (`core/command_registry.py`): Slash command execution engine  
  - **Hook Dispatcher** (`core/hook_dispatcher.py`): Command-level hook system
- **Configuration System**: Multi-layered settings with global defaults and project-specific overrides

## Hook System

The automation operates through Claude Code's native hook system that modifies prompts before they reach Claude:

### Hook Configuration
Hooks are configured in `.claude/settings.json` using Claude Code's native format:
```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python \".claude/hooks/UserPromptSubmit/flag_dispatcher.py\""
          },
          {
            "type": "command",
            "command": "python \".claude/hooks/UserPromptSubmit/append_ultrathink.py\""
          },
          {
            "type": "command",
            "command": "python \".claude/hooks/UserPromptSubmit/answer_in_short.py\""
          },
          {
            "type": "command",
            "command": "python \".claude/hooks/UserPromptSubmit/append_explain.py\""
          },
          {
            "type": "command",
            "command": "python \".claude/hooks/UserPromptSubmit/append_default.py\""
          }
        ]
      }
    ]
  }
}
```

### Active UserPromptSubmit Hooks (Execution Order)
1. **flag_dispatcher** - Central flag processing system for intelligent shortcuts (`-ut`, `-debug`, `-quick`, etc.)
2. **append_ultrathink** - Adds deep thinking prompts for complex architectural problems
3. **answer_in_short** - Enforces concise responses for simple queries  
4. **append_explain** - Adds explanation requests for learning scenarios
5. **append_default** - Activates with `-d` flag for project-aware digest mode

### Flag Dispatcher System
**NEW** Advanced shortcut system for intelligent response control:

#### **Core Performance Flags**
- **`-ut`** - Ultra Think mode: Deep analysis with comprehensive reasoning
- **`-debug`** - Debug mode: Systematic error analysis and troubleshooting  
- **`-quick`** - Quick mode: Brief, concise answers only
- **`-explain`** - Explain mode: Detailed explanations and teaching

#### **Development Focus Flags**
- **`-code`** - Code mode: Implementation focus with minimal explanation
- **`-review`** - Review mode: Code review with best practices
- **`-secure`** - Security mode: Focus on security implications
- **`-optimize`** - Optimization mode: Performance and efficiency focus
- **`-refactor`** - Refactor mode: Code restructuring suggestions
- **`-test`** - Test mode: Testing strategies and implementation

#### **Flag Conflict Resolution**
- **Automatic detection** of conflicting flags (e.g., `-quick` vs `-ut`)
- **Priority-based resolution** - first detected flag takes precedence
- **Intelligent combinations** - compatible flags can be used together

### Hook Execution Model
- Hooks execute **sequentially** in the order defined in settings.json
- Each hook receives the **original or previously modified prompt** from stdin
- Hooks output **modified prompts** to stdout
- **Exit code 0** indicates success, non-zero indicates error/skip
- Flags are **automatically stripped** from prompts after processing
- The `-d` flag is **stripped from prompts** and triggers digest mode instructions

## Custom Command System

The advanced automation layer provides **11 specialized commands** that operate independently of the native hook system:

### Core Development Commands
- `/security` - Multi-layered security analysis (secrets, dependencies, static analysis, config)
- `/create-PR` - Intelligent PR creation with context-aware titles and descriptions
- `/refactor` - Context-aware code refactoring (pattern, structure, performance, quality)
- `/file-review` - Comprehensive code analysis with focus areas
- `/push-to-github` - Validated git push with branch management
- `/header-comments` - Intelligent file header generation
- `/explain-pull-request` - Architectural documentation and analysis
- `/create-readme` - Generate intelligent bilingual README with deep project analysis
- `/update-readme` - Intelligently update existing README while preserving custom content
- `/git-commit` - Generate intelligent commit messages using ultra think analysis
- `/clean-project` - **NEW** Safely clean project files while preserving code and git integrity

### Command System Architecture
- **Context-Aware Execution** - Commands adapt behavior based on detected project type, languages, frameworks
- **Parameter validation** with type checking and choices
- **Prerequisite checking** (git repo, required tools)
- **Pre/post command hooks** for extended automation workflows
- **Intelligent error handling** with detailed validation messages

### Command-Level Hook System
Unlike native prompt hooks, the command system supports:
- **PreCommand hooks** - Execute before command runs (validation, setup)
- **PostCommand hooks** - Execute after command completes (cleanup, notifications)
- **OnFileChange hooks** - Trigger on file system changes
- **OnGitAction hooks** - Respond to git operations

This provides enterprise-grade automation capabilities beyond simple prompt modification.

## Context Detection System

The system automatically detects and adapts to different project environments:

### Project Type Detection
**Primary Focus**: Python-based automation framework with multi-language support
- **Python** (Primary): `requirements.txt`, `setup.py`, `pyproject.toml`, `Pipfile`
- **Web/Node.js**: `package.json`, `webpack.config.js`, `yarn.lock`
- **Java**: `pom.xml`, `build.gradle`, `gradlew`
- **Rust**: `Cargo.toml`
- **Go**: `go.mod`, `go.sum`
- **Docker**: `Dockerfile`, `docker-compose.yml`
- **Configuration**: `.claude/` directory structure with Python core

### Dynamic Context Injection
- **Language detection** via file extensions and patterns
- **Framework identification** from dependency analysis
- **Git state tracking** (branch, uncommitted changes, main branch detection)
- **Tool availability** checking (git, docker, npm, pip, etc.)

## Development Workflows

### Security Analysis (`/security`)
Performs comprehensive security audits:
- **Secret detection** - API keys, tokens, credentials
- **Dependency scanning** - Known vulnerabilities in packages
- **Static analysis** - Code-level security issues (SQL injection, XSS)
- **Configuration review** - Security misconfigurations

### Pull Request Creation (`/create-PR`)
Intelligent PR workflow:
- **Change categorization** (features, fixes, docs, tests)
- **Commit message analysis** for context
- **Auto-generated titles and descriptions** based on diff analysis
- **Support for draft PRs and custom base branches**

### Refactoring (`/refactor`)
Context-aware code improvement:
- **Pattern extraction** - Reduce duplication, improve abstractions
- **Structure optimization** - File organization, module design
- **Performance enhancement** - Algorithm optimization, complexity reduction
- **Quality improvement** - Readability, maintainability, documentation

### Bilingual README Generation (`/create-readme`, `/update-readme`)
**NEW** Intelligent documentation automation with bilingual support:

#### **Create README** (`/create-readme`)
- **Deep Project Analysis** - Automatic detection of project type, languages, frameworks
- **Bilingual Support** - Generate English (primary) and Chinese versions
- **Style Levels** - Minimal, standard, or comprehensive documentation
- **Smart Content** - Context-aware sections based on detected technologies
- **Technical Intelligence** - Automatic badge generation, installation instructions, usage examples

#### **Update README** (`/update-readme`) 
- **Content Preservation** - Maintains author's voice and custom content
- **Selective Enhancement** - Updates outdated information while preserving unique elements
- **Bilingual Synchronization** - Keeps English and Chinese versions consistent
- **Smart Gap Analysis** - Identifies missing sections and outdated information
- **Incremental Updates** - Section-specific updates with rollback capability

#### **Language Support**
- `--lang=en` (default) - English documentation
- `--lang=zh` - Chinese documentation  
- `--lang=both` - Generate both language versions
- **Technical Accuracy** - Proper localization while preserving technical terms
- **Cultural Adaptation** - Appropriate examples and references for target audience

### Intelligent Git Commit (`/git-commit`)
**NEW** Ultra think-powered commit message generation:

#### **Deep Change Analysis**
- **Multi-Type Detection** - Automatic classification (feat, fix, docs, refactor, test, chore)
- **Scope Intelligence** - File-path-based scope detection for modular projects
- **Breaking Change Detection** - Identifies API modifications and major changes
- **Historical Pattern Analysis** - Learns from existing commit conventions

#### **Context-Aware Generation**
- **Conventional Commits** - Standard `type(scope): description` format
- **Project Adaptation** - Respects existing team conventions and patterns
- **Branch Context** - Incorporates branch naming patterns for additional intelligence
- **Multi-Component Changes** - Handles complex changesets across multiple modules

#### **Style Support**
- `--style=conventional` (default) - Standard conventional commits
- `--style=semantic` - Emoji-enhanced commit messages
- `--style=angular` - Angular project conventions
- `--dry-run` - Preview without committing
- `--auto` - Skip confirmation for automated workflows

### Project Cleanup (`/clean-project`)
**NEW** Safe, intelligent project cleanup system:

#### **Safety-First Design**
- **Mandatory branch protection** - Cannot run on main/master branches
- **Dry-run mode** - Preview all changes before execution
- **Git integrity preservation** - Zero risk to repository state
- **Rollback capability** - All operations are reversible

#### **Cleanup Modes**
- **Safe mode** (default) - Conservative cleanup of temporary files only
- **Moderate mode** - Extended cleanup including build artifacts and cache
- **Custom targeting** - Selective cleanup with user-defined patterns

#### **Key Features**
- **Multi-language support** - Python, Node.js, Java, Rust, Go projects
- **Build system awareness** - Respects .gitignore and build configurations
- **Performance optimization** - Parallel processing for large projects
- **Detailed reporting** - Complete audit trail of all changes

## Configuration Management

### Settings Hierarchy
1. **Global settings** (`.claude/settings/global.json`) - System-wide defaults
2. **Project settings** (`.claude/settings/project.json`) - Project-specific overrides
3. **Hook configuration** (`.claude/settings/hooks.json`) - Hook priorities and conditions
4. **Command definitions** (`.claude/settings/commands.json`) - Command parameters and validation

### Key Configuration Areas
- **Security constraints** - Allowed tool patterns and safe execution modes
- **Performance tuning** - Parallel execution, caching, lazy loading
- **Integration settings** - Git, GitHub, package managers
- **User preferences** - Verbosity, auto-confirmation, learning modes

## Hook Development Guidelines

### Creating New Hooks for Claude Code
1. **Read from stdin** - Accept the prompt text as input from standard input
2. **Write to stdout** - Output the modified prompt to standard output
3. **Use exit codes** - Exit with 0 for success, non-zero for error/skip
4. **Handle edge cases** - Always output something to stdout (original prompt if no modification)
5. **Apply conditional logic** - Skip modification when not applicable

### Hook Script Structure
```python
#!/usr/bin/env python3
import sys

def main():
    try:
        # Read prompt from stdin
        prompt = sys.stdin.read().strip()
        
        # Apply your logic here
        if should_modify_prompt(prompt):
            modified_prompt = modify_prompt(prompt)
            print(modified_prompt)
        else:
            print(prompt)  # Output unchanged
            
        sys.exit(0)  # Success
    except Exception:
        sys.exit(1)  # Error

if __name__ == "__main__":
    main()
```

### Adding Hooks to Configuration
1. Add your script to `.claude/hooks/UserPromptSubmit/your_hook.py`
2. Update `.claude/settings.json` to include your hook (maintain execution order):
```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python \".claude/hooks/UserPromptSubmit/flag_dispatcher.py\""
          },
          {
            "type": "command",
            "command": "python \".claude/hooks/UserPromptSubmit/your_hook.py\""
          },
          {
            "type": "command",
            "command": "python \".claude/hooks/UserPromptSubmit/append_ultrathink.py\""
          }
        ]
      }
    ]
  }
}
```

**Important**: Place your hook after `flag_dispatcher` but consider its interaction with other hooks.

## Flag Usage Examples and Best Practices

### **Common Flag Combinations**
```bash
# Ultra think analysis for complex problems
"Analyze the authentication system architecture -ut"

# Quick debug for immediate issues
"Why is this API call failing? -debug"

# Concise answers for simple questions
"What does this function do? -quick"

# Code focus with security analysis
"Implement user login function -code -secure"

# Review with optimization suggestions
"Review this algorithm -review -optimize"
```

### **Flag Conflict Examples**
```bash
# Conflicting flags (auto-resolved to first)
"Explain this concept -quick -explain"  # → Uses -quick mode
"Debug this error -ut -debug"           # → Uses -ut mode

# Compatible flag combinations
"Review code security -review -secure"   # ✓ Compatible
"Optimize and refactor -optimize -refactor" # ✓ Compatible
```

### **Best Practices**
1. **Start with problem type**: Use `-debug` for errors, `-ut` for architecture
2. **Combine compatible flags**: `-review + -secure`, `-code + -optimize`
3. **Use `-quick` for rapid iteration** during development cycles
4. **Apply `-explain` for learning** complex concepts and patterns
5. **Leverage `-test`** when implementing testing strategies

### **Flag Selection Guide**
- **Complex Analysis**: `-ut` (Ultra Think)
- **Problem Solving**: `-debug` (Debug mode)
- **Fast Development**: `-quick` (Quick mode)
- **Learning**: `-explain` (Detailed explanations)
- **Implementation**: `-code` (Code focus)
- **Quality Assurance**: `-review`, `-test`, `-secure`
- **Performance**: `-optimize`, `-refactor`

## System Integration

### Git Integration
- **Automatic branch detection** and main branch identification
- **Uncommitted change tracking** for workflow decisions
- **Commit message analysis** for PR and documentation generation

### GitHub Integration
- **GitHub CLI integration** for PR creation and management
- **Issue linking** and automatic label assignment
- **Repository context** in command execution

### Package Manager Support
- **Multi-language support** - npm, pip, cargo, go mod
- **Dependency analysis** for security scanning
- **Tool availability detection** for prerequisite checking

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MuQY1818)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MuQY1818)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
