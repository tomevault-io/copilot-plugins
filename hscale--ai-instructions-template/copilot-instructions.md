## ai-instructions-template

> 🚨 SYNCHRONIZED FROM: AGENTS.md (MASTER SOURCE)

<!--
    🚨 SYNCHRONIZED FROM: 
    
🚨 SYNCHRONIZED FROM: AGENTS.md (MASTER SOURCE)

This file is automatically synchronized across all AI platforms.
DO NOT edit directly - edit the master file and run sync:
.\tools\ai-instructions-sync.ps1 -Action sync-all

AI PLATFORM FILES THAT MUST BE SYNCHRONIZED:
- AGENTS.md (MASTER SOURCE - ROOT LEVEL)
- .github/copilot-instructions.md (GitHub Copilot)
- CLAUDE.md (Claude AI platform - ROOT LEVEL)
- GEMINI.md (THIS FILE - Google Gemini AI platform)
- .vscode/README.md (VS Code)
- .zed/README.md (Zed Editor)

NEVER edit platform files directly - always edit the master file (AGENTS.md)!

LAST SYNC: {SYNC_TIMESTAMP}
TARGET: Google Gemini AI
PRIORITY: High
-->

# AI Instructions - {PROJECT_NAME} (Google Gemini Enhanced)

**🔄 AUTOMATIC SYNCHRONIZATION ACTIVE**
- ✅ This file is synchronized from the master source (AGENTS.md)
- ⚡ Changes to master automatically sync to all AI platforms
- 🛡️ Automatic backup protection before every sync
- 📊 Comprehensive validation ensures consistency
- 🎯 Edit AGENTS.md → All platforms updated automatically

**🚀 Google Gemini Enhanced Features**
- **Multimodal Intelligence**: Handle text, code, images, and data simultaneously
- **Massive Context Window**: Process large codebases with 1M+ token context
- **Advanced Reasoning**: Complex problem-solving with chain-of-thought
- **Code Understanding**: Deep comprehension of multiple programming languages
- **Real-time Learning**: Adapt to project patterns and team preferences

## 🎯 Core Mission

As Google Gemini working on **{PROJECT_NAME}**, leverage your advanced multimodal capabilities and massive context window to provide superior assistance for {PROJECT_TYPE} development using the Kiro methodology and {ENHANCEMENT_LEVEL} engineering excellence.

### **🧠 Gemini-Specific Capabilities**
- **Long Context Processing**: Analyze entire codebases in single context
- **Multimodal Analysis**: Process diagrams, screenshots, and documentation
- **Advanced Code Generation**: Create complex, interconnected components
- **Pattern Recognition**: Identify architectural patterns across large projects
- **Real-time Adaptation**: Learn and adapt to project-specific conventions

## 🏗️ Project Context

- **Project**: {PROJECT_NAME}
- **Project Type**: {PROJECT_TYPE}
- **Technology Stack**: {TECHNOLOGY_STACK}
- **Architecture**: {ARCHITECTURE_PATTERN}
- **Primary Language**: {PRIMARY_LANGUAGE}
- **Framework**: {PRIMARY_FRAMEWORK}
- **Database**: {DATABASE_TECH}
- **Infrastructure**: {INFRASTRUCTURE_TECH}
- **Enhancement Level**: {ENHANCEMENT_LEVEL}

## Core Directives (Kiro Method + Gemini Optimized)

### 🎯 **Primary Rules**
- **Multimodal First**: Leverage image, diagram, and code analysis capabilities
- **Context Maximization**: Use full context window for comprehensive understanding
- **IDE Tools First**: Use built-in diagnostics, language servers, integrated terminals
- **Timezone**: All datetime operations use {TIMEZONE}
- **Real Time Only**: Never fake timestamps, always use actual current time
- **Cross-Platform**: Support Windows, macOS, Linux, WSL2 seamlessly

### 📁 **CRITICAL PATH RULES (Kiro Framework)**
- **ALWAYS use relative paths**: All paths relative to project root
- **Kiro Specs Location**: `.kiro/specs/[project-name]/` for specifications
- **AI Agent Files**: `ai-agents/[agent-name].md` for specialized agents
- **AI Work Sessions**: `ai-agents-works/sessions/` with single-file approach
- **Session Naming**: `{STATUS}-{timestamp}-{session-name}.md` format
- **Consistent Separators**: Use forward slashes `/` in all path references

### 🚨 **WORK LOSS PREVENTION PROTOCOL**
- **Create session document FIRST**: Before starting any AI work
- **Update frequently**: Save progress every 15-30 minutes
- **Use descriptive filenames**: Include date, agent, and objective
- **Backup critical work**: In both active sessions and appropriate subfolders

### 🏗️ **Architecture Context ({ENHANCEMENT_LEVEL})**
- **{ARCHITECTURE_PATTERN}**: {ARCHITECTURE_DESCRIPTION}
- **Multi-Tenant**: Database/application/workflow isolation
- **{PRIMARY_LANGUAGE} Backend**: {BACKEND_DESCRIPTION}
- **{FRONTEND_FRAMEWORK} Frontend**: {FRONTEND_DESCRIPTION}

## Gemini-Specific Agent Specialization

### **Advanced Development**
- `architect-agent.md` → System design with multimodal diagrams
- `backend-agent.md` → {PRIMARY_LANGUAGE} services with context awareness
- `frontend-agent.md` → {FRONTEND_FRAMEWORK} with visual components
- `database-agent.md` → {DATABASE_TECH} with performance analysis
- `devops-agent.md` → Infrastructure with monitoring dashboards

### **Intelligent Analysis**
- `integration-agent.md` → Cross-service analysis with dependency mapping
- `qa-agent.md` → Comprehensive testing with visual test reports
- `security-agent.md` → Security analysis with threat modeling
- `performance-agent.md` → Performance optimization with metrics visualization

### **Kiro Framework + Gemini**
- `spec-agent.md` → Requirements with visual mockups and diagrams
- `hooks-agent.md` → Automation with intelligent triggers
- `steering-agent.md` → ADRs with decision trees and impact analysis

## Gemini Technical Patterns

### **Large Context Analysis Pattern**
```{PRIMARY_LANGUAGE}
// Gemini Advantage: Analyze entire codebase context
// Process multiple files, dependencies, and patterns simultaneously
// Reference: Use full context window for comprehensive understanding

// Example: Cross-service dependency analysis
// 1. Load all service definitions
// 2. Analyze inter-service communication patterns
// 3. Identify potential bottlenecks or issues
// 4. Suggest optimizations based on full context
```

### **Multimodal Development Pattern**
```markdown
// Gemini Capability: Process images, diagrams, and code together
// 1. Analyze UI mockups or architecture diagrams
// 2. Generate corresponding code structure
// 3. Implement visual components matching designs
// 4. Create documentation with visual elements
```

### **Project Structure (CANONICAL)**
```
{PROJECT_ROOT}/
├── .github/copilot-instructions.md    # Master AI instructions
├── .kiro/                             # Architecture & specs
│   ├── specs/                         # Project specifications
│   │   ├── {PROJECT_NAME}/            # Main project specs
│   │   └── [feature-name]/            # Feature-specific specs
│   ├── steering/                      # ADRs and decisions
│   └── hooks/                         # Automation hooks
├── .vscode/README.md                  # VS Code AI instructions
├── .zed/README.md                     # Zed Editor AI instructions
├── AGENTS.md                          # General AI instructions
├── CLAUDE.md                          # Claude AI instructions
├── GEMINI.md                          # THIS FILE (Gemini AI instructions)
├── ai-agents/                         # Specialized AI agent files
├── ai-agents-works/                   # AI work tracking system
│   └── sessions/                      # Single-file session tracking
├── {PROJECT_MAIN_DIR}/                # Main application code
│   ├── {BACKEND_DIR}/                 # Backend services
│   ├── {FRONTEND_DIR}/                # Frontend applications
│   ├── {SHARED_DIR}/                  # Shared packages
│   └── {INFRASTRUCTURE_DIR}/          # Infrastructure code
└── tools/                             # Development toolkit
```

### **Gemini Development Commands**
```bash
# Smart toolkit (leveraging Gemini's context)
.\tools\smart-dev-toolkit.ps1 -Action dev-start -Service all -UseGeminiContext

# Infrastructure ({CONTAINER_TECH})
{INFRASTRUCTURE_COMMAND}

# Services - from {PROJECT_MAIN_DIR} directory
{SERVICE_START_COMMAND}

# Frontend - from project root
{FRONTEND_START_COMMAND}
```

### **Gemini Error Resolution Priority**
1. **Context Analysis** → Use full codebase context for root cause analysis
2. **Multimodal Debugging** → Analyze logs, code, and visual indicators
3. **Pattern Recognition** → Identify similar issues across the project
4. **Intelligent Suggestions** → Provide multiple solution approaches
5. **Impact Assessment** → Evaluate changes across entire system

## AI Work Tracking System (Gemini Enhanced)

### **Single-File Session Management**
```
ai-agents-works/sessions/              # Simple session directory
├── ACTIVE-{timestamp}-{session-name}.md     # Currently running sessions
├── COMPLETED-{timestamp}-{session-name}.md  # Finished sessions
├── PAUSED-{timestamp}-{session-name}.md     # Paused sessions
└── CANCELLED-{timestamp}-{session-name}.md  # Cancelled sessions
```

### **Gemini Session Management Commands**
```powershell
# 🚨 CRITICAL: Start session IMMEDIATELY to prevent work loss
.\ai-agents-works\automation\ai-work-tracker.ps1 -Action start-session -AgentName "gemini-enhanced" -Objective "Multimodal analysis task"

# Leverage Gemini's context for session updates
.\ai-agents-works\automation\ai-work-tracker.ps1 -Action update-session -SessionFile "ACTIVE-20250113-142530-gemini-analysis.md" -Activity "Processing large context" -TokensUsed 5000

# Complete session with comprehensive summary
.\ai-agents-works\automation\ai-work-tracker.ps1 -Action complete-session -SessionFile "ACTIVE-20250113-142530-gemini-analysis.md"

# List sessions with Gemini-specific filters
.\ai-agents-works\automation\ai-work-tracker.ps1 -Action list-sessions -Status ACTIVE -Agent gemini
```

### **🚨 WORK LOSS PREVENTION PROTOCOL (Gemini Enhanced)**
```powershell
# STEP 1: Create session document IMMEDIATELY
.\ai-agents-works\automation\ai-work-tracker.ps1 -Action start-session -AgentName "gemini" -Objective "Your work objective"

# STEP 2: Use Gemini's context for comprehensive documentation
# Document not just what you're doing, but full context understanding

# STEP 3: Update session frequently with multimodal insights
.\ai-agents-works\automation\ai-work-tracker.ps1 -Action update-session -SessionFile $sessionFile -Activity "Analyzing visual components and code patterns" -TokensUsed [tokens]

# STEP 4: Leverage large context for session summaries
# Include cross-file dependencies and system-wide implications
```

### **Gemini-Specific Success Metrics**
- **Context Utilization**: Effectively use large context windows (target: >80% of available context)
- **Multimodal Integration**: Successfully process images, diagrams, and code together
- **Pattern Recognition**: Identify and reuse successful patterns (target: 95%+ accuracy)
- **Cross-Platform Compatibility**: Seamless operation across all development environments
- **Large-Scale Analysis**: Handle enterprise-level codebases efficiently
- **Real-time Adaptation**: Learn and improve from project-specific patterns

## Key Ports & Services

### **Infrastructure (localhost)**
- {INFRASTRUCTURE_PORTS}

### **Backend Services**
- {BACKEND_SERVICES_PORTS}

### **Frontend Apps**
- {FRONTEND_APPS_PORTS}

## Quick Reference Paths

### **AI Instructions & Platform Files (SYNCHRONIZED)**
- **MASTER**: `AGENTS.md` (ROOT LEVEL)
- **GitHub Copilot**: `.github/copilot-instructions.md`
- **Claude AI**: `CLAUDE.md` (ROOT LEVEL)
- **Gemini AI**: `GEMINI.md` ← THIS FILE
- **VS Code**: `.vscode/README.md`
- **Zed Editor**: `.zed/README.md`

### **Specialized AI Agents (Gemini Enhanced)**
- **Architect**: `ai-agents/architect-agent.md` (with visual system design)
- **Backend**: `ai-agents/backend-agent.md` (with context-aware development)
- **Frontend**: `ai-agents/frontend-agent.md` (with visual component analysis)
- **Database**: `ai-agents/database-agent.md` (with performance visualization)
- **DevOps**: `ai-agents/devops-agent.md` (with infrastructure monitoring)
- **Integration**: `ai-agents/integration-agent.md` (with dependency mapping)
- **QA**: `ai-agents/qa-agent.md` (with comprehensive test analysis)
- **Security**: `ai-agents/security-agent.md` (with threat modeling)
- **Performance**: `ai-agents/performance-agent.md` (with metrics analysis)

### **Kiro Framework Specifications**
- **Main Requirements**: `.kiro/specs/{PROJECT_NAME}/requirements.md`
- **Main Design**: `.kiro/specs/{PROJECT_NAME}/design.md`
- **Main Tasks**: `.kiro/specs/{PROJECT_NAME}/tasks.md`
- **Steering Documents**: `.kiro/steering/[decision-name].md`

### **Development Tools (Gemini Optimized)**
- **Smart Toolkit**: `tools\smart-dev-toolkit.ps1`
- **Agent Selector**: `tools\smart-agent-selector.ps1`
- **Environment Healer**: `tools\self-healing-env.ps1`
- **Metrics Collector**: `tools\metrics-collector.ps1`
- **AI Instructions Sync**: `tools\ai-instructions-sync.ps1`

## Gemini-Specific Guidelines

### **Leverage Your Strengths**
1. **Large Context**: Always consider the full codebase context when making recommendations
2. **Multimodal**: Request and analyze visual elements (diagrams, screenshots, mockups) when available
3. **Advanced Reasoning**: Use chain-of-thought for complex architectural decisions
4. **Pattern Recognition**: Identify and suggest proven patterns from similar projects
5. **Real-time Learning**: Adapt recommendations based on project-specific conventions

### **Gemini Best Practices**
1. **Context Window Management**: Use full context efficiently, prioritize most relevant information
2. **Multimodal Integration**: Combine text, code, and visual analysis for comprehensive solutions
3. **Incremental Learning**: Build understanding of project patterns over time
4. **Cross-File Analysis**: Consider dependencies and impacts across entire codebase
5. **Visual Documentation**: Generate or suggest visual representations when helpful

### **Gemini Collaboration Patterns**
1. **With Human Developers**: Provide comprehensive analysis and multiple solution approaches
2. **With Other AI Agents**: Share context and insights for collaborative problem-solving
3. **With IDE Tools**: Integrate seamlessly with development environment capabilities
4. **With Project Documentation**: Maintain and enhance visual and textual documentation
5. **With Testing Systems**: Provide comprehensive test analysis and recommendations

## Multi-OS Gemini Reference

### **Project Root Paths (CANONICAL)**
**Windows PowerShell:**
- Project Root: `{WINDOWS_PROJECT_PATH}`
- Navigation: `Push-Location "{WINDOWS_PROJECT_PATH}\{PROJECT_MAIN_DIR}"; {WINDOWS_COMMANDS}; Pop-Location`

**macOS/Linux Bash:**
- Project Root: `{UNIX_PROJECT_PATH}`
- Navigation: `cd {UNIX_PROJECT_PATH}/{PROJECT_MAIN_DIR} && {UNIX_COMMANDS}`

**WSL2 (Recommended for Windows):**
- Project Root: `{WSL_PROJECT_PATH}`
- Navigation: `cd {WSL_PROJECT_PATH}/{PROJECT_MAIN_DIR} && {WSL_COMMANDS}`

### **Gemini Development Environment**
- **Preferred**: Large context window for comprehensive analysis
- **Multimodal**: Support for images, diagrams, and visual elements
- **Cross-Platform**: Seamless operation across all development environments
- **IDE Integration**: Work effectively with VS Code, Zed, and other editors
- **Real-time**: Continuous learning and adaptation to project patterns

---

> **🧠 Gemini Enhanced**: This instruction set is specifically optimized for Google Gemini's advanced capabilities including large context processing, multimodal analysis, and superior reasoning abilities. Use these strengths to provide exceptional development assistance for {PROJECT_NAME}.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hscale) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
