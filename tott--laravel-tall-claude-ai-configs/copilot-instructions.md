## laravel-tall-claude-ai-configs

> This document provides essential context and quick reference for AI-assisted development in Laravel TALL stack applications.

# CLAUDE.md - Laravel TALL Stack AI-Assisted Development Guidelines

This document provides essential context and quick reference for AI-assisted development in Laravel TALL stack applications.

## **🚀 Development Environment**

**Laravel Sail** is used for Docker development. Always prefix commands with `./vendor/bin/sail`:

```bash
# Essential Commands
./vendor/bin/sail up -d          # Start environment  
./vendor/bin/sail down           # Stop environment
./vendor/bin/sail artisan migrate --seed  # Database setup
./vendor/bin/sail npm run dev    # Frontend development
./vendor/bin/sail artisan test   # Run tests
```

**📖 [Complete Commands Reference](docs/reference/laravel-commands.md)**

---

## 🛠️ MCP Server Tools Strategy

**Always use MCP servers for enhanced development:**

### Core Development (Always Available)
- `mcp__serena__*` - Semantic code analysis and intelligent navigation
- `mcp__context7__*` - Up-to-date documentation access
- `mcp__browsermcp__*` - Real-time browser debugging with authenticated sessions

### Quality Assurance (Recommended)
- `mcp__zen__codereview` - Professional code review before PRs
- `mcp__zen__precommit` - Automated quality gates for commits
- `mcp__zen__secaudit` - Security auditing for releases

### Advanced Development (Complex Tasks)
- `mcp__zen__thinkdeep` - Extended reasoning for architectural decisions
- `mcp__zen__debug` - Systematic debugging workflows
- `mcp__zen__consensus` - Multi-model validation for major decisions

**📖 [Complete MCP Server Guides](docs/mcp-servers/)**

---

## 📖 Documentation-First Development

**ALWAYS consult docs/ before starting complex tasks.** The documentation contains:

### Documentation Priority Workflow
```bash
1. **Check CLAUDE.md** → Essential context and quick reference
2. **Consult docs/workflows/** → Understand the process for your task type  
3. **Reference docs/reference/** → Get specific standards, commands, patterns
4. **Engage .claude/agents/** → Delegate complex domain-specific work
5. **Use docs/mcp-servers/** → Optimize tool usage and troubleshooting
```

### When Documentation is Incomplete
```bash
# If docs are missing or outdated, update them FIRST
mcp__serena__write_memory "missing_documentation" "Document what needs to be added"

# Then implement with proper documentation
"Implement [feature] and update docs/workflows/[relevant].md with new patterns discovered"
```

---

## 📊 Smart Tool Selection

**Match tool complexity to task complexity:**
- **Edit/MultiEdit**: Simple, targeted changes and small modifications
- **Serena tools**: Complex refactoring, cross-file analysis, unfamiliar code
- **Zen tools**: Quality assurance, debugging, architectural analysis

**📖 [Complete Development Workflows](docs/workflows/)**

---

## 🏗️ Laravel TALL Stack Architecture Quick Reference

### Core Technology Stack
- **Laravel 12** + **TALL Stack** (Tailwind, Alpine.js, Laravel, Livewire)
- **FilamentPHP** - Admin interfaces (optional)
- **Laravel Sail** - Docker development environment
- **Pest** - PHP testing framework

### Key Application Patterns
- **`app/Livewire/`** - Reactive UI components
- **`app/Services/`** - Business logic services  
- **`app/Models/`** - Eloquent models and relationships
- **`resources/views/livewire/`** - Livewire component templates

**📖 [Complete Architecture Guide](docs/setup/project-architecture.md)**

---

## 🤖 AI Development Guidelines

### Core Development Principles
1. **Documentation First**: Always check `docs/` for detailed information
2. **Pattern Consistency**: Follow existing TALL stack patterns  
3. **Quality Gates**: Use MCP tools for systematic quality assurance
4. **Context Preservation**: Use Serena memory system to document decisions

### Natural Language Development Workflow
```bash
# 1. Context gathering
mcp__serena__get_symbols_overview [relevant_directory]
mcp__serena__search_for_pattern [related_functionality]

# 2. Implementation with quality gates  
mcp__zen__codereview [implemented_feature]
mcp__zen__precommit [validate_changes]

# 3. Knowledge preservation
mcp__serena__write_memory [pattern_name] [architectural_decisions]
```

### Component Architecture Decision Rules
```bash
# Admin functionality + CRUD operations
→ Consider FilamentPHP Resource for rapid development

# User-facing + interactive/real-time  
→ Use Livewire Component

# API endpoints + external integrations
→ Standard Laravel controllers with proper validation
```

### Sub-Agent Coordination Strategy
**Use specialized sub-agents for complex domain-specific work:**

```bash
# Task Complexity Assessment
Simple (1 file, <50 lines)     → Main Agent Only
Moderate (2-3 files)           → Consider specialist agent  
Complex (Multi-system)         → Multi-agent workflow
Architecture/Performance       → Always use specialist

# Example Delegations
"DevOps Specialist: Configure Docker services for [feature]"
"Testing Specialist: Create comprehensive test suite for [component]"
"Security Specialist: Audit authentication system for vulnerabilities"
"Performance Specialist: Optimize database queries in [service]"
```

**📖 [AI Interaction Patterns](docs/reference/ai-interaction-patterns.md)**

---

## 📚 Documentation Contribution Guidelines

**Critical**: Follow strict documentation architecture to prevent duplication and maintain consistency.

### Documentation Ownership Rules
- **Workflows** = PROCESS (how to do tasks) → Delegate to specialists
- **Agents** = EXPERTISE (domain knowledge) → Provide comprehensive coverage
- **Never duplicate content** between workflows and agents

### Before Contributing Documentation
1. **Check existing coverage** - Avoid duplication with current docs
2. **Verify specialist references** - Only reference existing agents (DevOps, Security, Performance, Testing, TALL Stack)  
3. **Follow delegation patterns** - Workflows delegate complex tasks to appropriate specialists
4. **Use established structures** - Follow architectural patterns in existing documents

**📖 Complete Guidelines**: [Documentation Maintenance Guidelines](docs/maintenance/documentation-guidelines.md)

---

## 🔄 Git Workflow

**Commit after EVERY change** without asking permission:
```bash
git add -A && git commit -m "[action]: [description]"
```

**Commit Prefixes:** `feat:`, `fix:`, `refactor:`, `style:`, `docs:`, `chore:`, `test:`

---

## 📚 Documentation Navigation & When to Consult

**💡 Always check docs/ for detailed guidance before starting complex tasks.**

### 🚀 Setup & Getting Started
**Consult when:** Setting up environment, understanding codebase architecture, first-time setup
- **[Development Environment](docs/setup/development-environment.md)** - Docker, Sail, basic setup

### 🛠️ Development Workflows  
**Consult when:** Starting new features, debugging issues, optimizing performance, ensuring quality
- **[Feature Development](docs/workflows/feature-development.md)** - Complete development process, planning to deployment
- **[Quality Assurance](docs/workflows/quality-assurance.md)** - Code review, testing, security workflows
- **[Debugging & Investigation](docs/workflows/debugging-investigation.md)** - Systematic problem-solving approaches
- **[Performance Optimization](docs/workflows/performance-optimization.md)** - Performance analysis and tuning strategies

### 🤖 AI Agent Specialists
**Consult when:** Need domain expertise for complex tasks, specialized knowledge required
- **[DevOps Specialist](.claude/agents/devops-specialist.md)** - Docker, Sail, deployment, infrastructure management
- **[Testing Specialist](.claude/agents/testing-specialist.md)** - Comprehensive testing strategies & QA processes
- **[Security Specialist](.claude/agents/security-specialist.md)** - Security audits, vulnerability assessments
- **[Performance Specialist](.claude/agents/performance-specialist.md)** - Performance optimization & database tuning
- **[TALL Stack Specialist](.claude/agents/tall-specialist.md)** - Livewire, Alpine.js, frontend patterns

### 🔧 MCP Server Tools
**Consult when:** Need to understand tool capabilities, optimize tool usage, troubleshoot MCP issues
- **[Serena Guide](docs/mcp-servers/serena-guide.md)** - Semantic code analysis, navigation, editing
- **[Zen Guide](docs/mcp-servers/zen-guide.md)** - Advanced analysis, multi-model workflows, quality assurance
- **[Context7 Guide](docs/mcp-servers/context7-guide.md)** - Up-to-date documentation access & retrieval
- **[BrowserMCP Guide](docs/mcp-servers/browsermcp-guide.md)** - Real-time browser debugging and troubleshooting

### 📖 Reference Materials
**Consult when:** Need command syntax, coding standards, architectural decisions, AI interaction patterns
- **[Laravel Commands](docs/reference/laravel-commands.md)** - Complete Sail, Artisan, and project command reference
- **[Code Conventions](docs/reference/code-conventions.md)** - TALL stack patterns, naming, structure standards
- **[AI Interaction Patterns](docs/reference/ai-interaction-patterns.md)** - Natural language development techniques
- **[Decision Tracking](docs/reference/decision-tracking.md)** - Architectural decision record templates & processes

### 🔍 Quick Documentation Lookup
```bash
# Find relevant documentation
mcp__serena__list_dir --relative_path="docs" --recursive=true
mcp__serena__search_for_pattern --substring_pattern="your_topic" --relative_path="docs"

# Check specific workflow  
"Before implementing [feature], consult docs/workflows/feature-development.md"

# Get specialist guidance
For [complex_task], delegate to appropriate specialist in .claude/agents/
```

**📖 [Complete Documentation Index](docs/README.md)**

---

## 🎯 Development Context

### Session Continuation Protocol
1. **Check memories**: `mcp__serena__list_memories` and read relevant entries
2. **Review git status**: Check recent commits and current branch state  
3. **Use TodoWrite**: Track complex tasks and progress

### Browser Testing & Debugging (BrowserMCP)
**Use BrowserMCP for real-time debugging and troubleshooting:**
```bash
# Available via mcp__browsermcp__* tools
# - Real-time browser automation and testing
# - Authenticated session debugging
# - New feature validation and troubleshooting
```

**Use for:** UI debugging, feature testing, user workflow validation, real-time troubleshooting.

---

## 💡 Collaboration Guidelines

- **Challenge and question**: Don't immediately agree with suboptimal approaches
- **Push back constructively**: Suggest better alternatives with clear reasoning
- **Think critically**: Consider edge cases, performance, maintainability
- **Seek clarification**: Ask follow-up questions for ambiguous requirements
- **Propose improvements**: Suggest better patterns and cleaner implementations

---

**Built with ❤️ using Laravel TALL stack and AI-powered development workflows**

*For detailed information on any topic, always consult the [docs/](docs/) directory.*

---
> Source: [tott/laravel-tall-claude-ai-configs](https://github.com/tott/laravel-tall-claude-ai-configs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
