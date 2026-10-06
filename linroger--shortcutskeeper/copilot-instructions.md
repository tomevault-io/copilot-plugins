## shortcutskeeper

> Coordinates all .claude folder components to work together as an active intelligence enhancement system for software development and research projects.

# Claude Code Master Instructions

## System Integration Hub

### Purpose
Coordinates all .claude folder components to work together as an active intelligence enhancement system for software development and research projects.

### Active Components
- **ai_history/**: @.claude/ai_history/CLAUDE.md
- **claude_code/**: @.claude/claude_code/CLAUDE.md  
- **commands/**: @.claude/commands/CLAUDE.md
- **sub-agent-tasks/**: @.claude/sub-agent-tasks/CLAUDE.md

### Integration Protocol
Every session must actively use ALL components:

#### Knowledge → Tools → Commands Flow
1. **Check existing knowledge** before starting work
2. **Apply tool optimization** during execution
3. **Create automation commands** for repeated patterns
4. **Update knowledge base** with discoveries

#### Cross-Component Behaviors
- **Save conversations**: ai_history tracks all significant exchanges
- **Optimize workflows**: claude_code patterns guide tool usage
- **Delegate strategically**: sub-agent-tasks stores parallel execution outputs
- **Automate repetition**: commands folder stores custom slash commands
- **Evolve intelligently**: Each component learns from the others

#### System Questions
ALWAYS consider during work:
- "Should this be saved as knowledge?"
- "Can this be automated with a command?"
- "Is there relevant knowledge to apply?"
- "Can I delegate part of this for parallel execution?"
- "Should I check sub-agent outputs for context before proceeding?"

---

## Core Development Workflows

### 1. Native macOS App Development

#### 1.1 Initial Project Setup & Analysis

**ALWAYS Follow This Sequence:**
1. **Read existing project structure** - Use LS tool to understand current codebase organization
2. **Analyze CLAUDE.md files** - Read any project-specific instructions first
3. **Review existing Swift files** - Understand current architecture and patterns
4. **Identify the main app target** - Locate the primary .swift files and entry points

#### 1.2 SwiftUI App Architecture Approach

**UI Design Philosophy:**
- **Start with the data model** - Define your core data structures first
- **MVVM Architecture** - Always separate ViewModels from Views
- **SwiftData integration** - Use SwiftData for persistence, not Core Data
- **Native macOS design** - Use NavigationSplitView, sidebars, and native controls
- **Responsive layouts** - Design for different window sizes from the start

**UI Connection Pattern:**
```
Data Model → ViewModel (@Observable) → SwiftUI View → User Interface
```

**Key UI Principles:**
- **Use @Observable** for ViewModels (Swift 6 pattern)
- **@State for local view state** only
- **@Environment for shared data** across views
- **SF Symbols** for all icons
- **System colors** for native appearance

#### 1.3 Function-to-UI Connection Strategy

**Standard Connection Pattern:**
1. **ViewModel Methods** - All business logic goes in ViewModel
2. **View Actions** - Buttons and controls call ViewModel methods
3. **State Updates** - ViewModel updates trigger UI refresh automatically
4. **Error Handling** - Show alerts/sheets for errors from ViewModel

**Example Pattern:**
```swift
// In ViewModel
@Observable class MyViewModel {
    var items: [Item] = []
    var isLoading = false
    
    func performAction() {
        isLoading = true
        // Business logic here
        isLoading = false
    }
}

// In View
Button("Action") {
    viewModel.performAction()
}
.disabled(viewModel.isLoading)
```

#### 1.4 MANDATORY Build-Test-Fix Cycle

**ALWAYS Implement This Cycle:**

1. **After Every Code Change:**
   - Run `xcodebuild -scheme [ProjectName] build` to test compilation
   - If build fails, analyze errors and fix immediately
   - Never proceed to next feature until current code builds successfully

2. **Build Testing Commands:**
   ```bash
   # Clean build folder
   xcodebuild clean -scheme [ProjectName]
   
   # Build for macOS
   xcodebuild -scheme [ProjectName] -destination 'platform=macOS' build
   
   # Build and run tests
   xcodebuild test -scheme [ProjectName] -destination 'platform=macOS'
   ```

3. **Error Resolution Process:**
   - Read error messages carefully
   - Fix syntax/import errors first
   - Address deprecation warnings
   - Resolve missing dependencies
   - Fix architectural issues last

4. **Success Criteria:**
   - Zero build errors
   - Zero warnings (unless explicitly legacy)
   - App launches without crashes
   - Basic functionality works as expected

#### 1.5 Common macOS App Patterns You Should Always Use

**File Organization:**
```
App/
├── Models/          # Data models and business logic
├── ViewModels/      # @Observable view models
├── Views/           # SwiftUI views
├── Services/        # Network, persistence, external APIs
├── Extensions/      # Swift extensions
└── Resources/       # Assets, localizations
```

**Navigation Pattern:**
```swift
NavigationSplitView {
    // Sidebar content
} detail: {
    // Main content area
}
```

**Data Persistence Pattern:**
```swift
@Model
class MyDataModel {
    // SwiftData model definition
}

// In App file:
.modelContainer(for: MyDataModel.self)
```

### 2. Python Development Workflow

#### 2.1 Project Structure & Setup

**Standard Project Organization:**
1. **Main script or notebook** - Clear entry point with project description
2. **Imports and dependencies** - All required libraries with version specifications
3. **Configuration section** - Constants, settings, global configuration
4. **Core logic sections** - Logical chunks with clear documentation
5. **Results and outputs** - Summary and key findings
6. **Testing and validation** - Unit tests and integration tests

**Essential Libraries for Data Science:**
```python
import pandas as pd
import numpy as np
import plotly.graph_objects as go
import plotly.express as px
from plotly.subplots import make_subplots
import requests  # For API calls
import json      # For data serialization
```

#### 2.2 Code Quality Best Practices

**Every Code Section Should:**
- Have clear, descriptive comments and docstrings
- Include comprehensive error handling
- Display intermediate results for validation
- Use professional variable naming conventions
- Include type hints where appropriate

**Data Visualization Standards:**
- **ALWAYS use Plotly** for interactive visualizations
- Include hover data and interactive features
- Use professional color schemes and consistent styling
- Add proper titles, labels, and legends
- Ensure responsive design for different screen sizes

#### 2.3 Standard Analysis Workflow

**General Analysis Pattern:**
1. **Data Validation** - Check for missing values, outliers, data quality
2. **Exploratory Analysis** - Summary statistics and initial insights
3. **Data Processing** - Cleaning, transformation, feature engineering
4. **Core Analysis** - Primary analysis based on project requirements
5. **Results Validation** - Cross-checking and validation of findings
6. **Output Generation** - Clear presentation of results and conclusions

---

## Multi-Agent Orchestration Framework

### 3.1 When to Use Multiple Agents

**Deploy Parallel Agents For:**
- **Complex research tasks** - Multiple agents researching different aspects
- **Code validation** - One agent writes, another reviews and tests
- **Multi-platform development** - Different agents for different components
- **Large-scale analysis** - Agents handling different data segments or approaches
- **Comparative analysis** - Agents testing different methodologies
- **Architecture design** - Multiple agents designing different system components

### 3.2 Agent Instruction Framework

**Standard Agent Prompt Structure:**
```
AGENT ROLE: [Specific role and expertise area]
TASK: [Specific, well-defined task with clear boundaries]
CONTEXT: [Relevant background information and constraints]
REQUIREMENTS: [Technical requirements and quality standards]
OUTPUT FORMAT: [Expected deliverable format and structure]
SUCCESS CRITERIA: [How to measure successful completion]
COORDINATION: [How this agent's work relates to other agents]
```

**Example Agent Instructions:**
```
AGENT ROLE: SwiftUI Architecture Specialist
TASK: Design the data model and ViewModel structure for a task management app
CONTEXT: User wants to build a productivity app with project tracking and team collaboration
REQUIREMENTS: Must use SwiftData, follow MVVM, support macOS 14+, handle offline/online sync
OUTPUT FORMAT: Complete Swift code files with comprehensive documentation
SUCCESS CRITERIA: Code compiles successfully, follows architectural best practices, scalable design
COORDINATION: Will integrate with UI specialist and testing specialist agents
```

### 3.3 Parallel Agent Coordination

**Agent Orchestration Pattern:**
1. **Task Decomposition** - Break complex projects into independent, parallelizable components
2. **Agent Specialization** - Assign agents with specific expertise to each component
3. **Parallel Execution** - Run agents simultaneously using Task tool for maximum efficiency
4. **Output Integration** - Systematically synthesize and integrate results from all agents
5. **Validation Phase** - Deploy additional agents to review and validate integrated solution
6. **Iteration Management** - Coordinate feedback loops and refinement cycles

**Recommended Agent Types:**
- **Research Agent** - Gathers information, analyzes requirements, reviews best practices
- **Architecture Agent** - Designs system structure, data flow, and technical approach
- **Implementation Agent** - Writes code following specifications and architectural guidelines
- **Testing Agent** - Validates code, identifies issues, ensures quality standards
- **Integration Agent** - Combines components and ensures seamless compatibility
- **Optimization Agent** - Reviews for performance, security, and maintainability improvements

### 3.4 Agent Output Processing

**Standard Output Integration Process:**
1. **Collect all agent outputs** in structured, comparable format
2. **Identify synergies and conflicts** between different approaches
3. **Extract best practices** and innovative solutions from each agent
4. **Synthesize unified approach** that combines strengths of all solutions
5. **Validate integrated result** against original requirements and success criteria
6. **Document decision rationale** for future reference and learning

**Quality Assessment Questions:**
- Which agent provided the most technically sound approach?
- Are there conflicting recommendations that need expert resolution?
- What innovative insights should be preserved in the final solution?
- How can different methodologies be combined for optimal results?
- Does the integrated solution maintain coherence and consistency?

---

## Enhanced Development Toolkit

### 4.1 Environment Management

**Python Package Management:**
```bash
# Modern Python package management (preferred)
uv pip install [package_name]     # Fast pip replacement
uv pip install -r requirements.txt
uv venv venv                       # Create virtual environment

# Traditional methods (fallback)
pip install [package_name]
pip install --upgrade [package_name]
python -m venv venv
source venv/bin/activate           # Activate environment
```

**Dependency Management:**
```bash
# Generate requirements files
pip freeze > requirements.txt
uv pip compile requirements.in    # Generate locked requirements

# Development dependencies
pip install -e .[dev]              # Install with development dependencies
```

### 4.2 Code Search and Analysis

**Advanced Search Commands:**
```bash
# Ripgrep for fast code search
rg "pattern" --type swift         # Search Swift files only
rg "class.*Manager" -A 5          # Show 5 lines after match
rg "import.*SwiftUI" --files      # Show files containing imports
rg "TODO|FIXME|HACK" --color always  # Find code maintenance items

# Find files by pattern
fd "*.swift" --type f             # Find all Swift files
fd "ViewModel" --extension swift  # Find ViewModel Swift files
```

**Code Quality Analysis:**
```bash
# Swift code analysis
swiftlint                         # Check Swift style and best practices
swift package dump-package        # Show package information
swift build                       # Build Swift package

# Python code analysis
flake8 *.py                       # Python style checking
black *.py                        # Python code formatting
mypy *.py                         # Python type checking
pytest                            # Run Python tests
```

### 4.3 Project Management Commands

**Git Workflow Management:**
```bash
# Branch management
git checkout -b feature/new-feature
git stash push -m "work in progress"
git cherry-pick [commit-hash]

# Advanced git operations
git log --oneline --graph         # Visual commit history
git diff --name-only              # See changed files
git blame [file]                  # See who changed what
git bisect start                  # Find problematic commits
```

**Build and Test Automation:**
```bash
# Xcode command line tools
xcodebuild -list                  # Show available schemes
xcodebuild -showBuildSettings     # Show build configuration
xcrun simctl list                 # List available simulators

# Package management
swift package generate-xcodeproj  # Create Xcode project from Package.swift
swift package update              # Update package dependencies
```

### 4.4 Development Automation

**File Management:**
```bash
# Bulk operations
find . -name "*.swift" -exec grep -l "ViewModel" {} \;
find . -type f -name "*.py" -size +1M    # Find large files

# Archive and backup
rsync -av --progress source/ dest/       # Copy with progress
tar -czf backup.tar.gz project/          # Create compressed archive
```

**Development Server Management:**
```bash
# Local development servers
python -m http.server 8000        # Simple HTTP server
python manage.py runserver        # Django development server
npm start                         # Node.js development server
swift run                         # Run Swift executable
```

---

## Quality and Best Practices

### 5.1 Code Quality Standards

**Universal Quality Requirements:**
- Comprehensive error handling for all user-facing operations
- Consistent code style following language-specific conventions
- Clear documentation for complex logic and public APIs
- Proper input validation and sanitization
- Efficient resource management and memory usage
- Comprehensive test coverage for critical functionality

**Review Checklist:**
- [ ] No TODO comments or placeholder code in production
- [ ] All functions have clear, single responsibilities
- [ ] Error cases are handled gracefully with user feedback
- [ ] Performance is optimized for the target platform
- [ ] Security best practices are followed throughout
- [ ] Code is readable and maintainable by others

### 5.2 Testing Standards

**Testing Requirements:**
- Unit tests for all business logic and algorithms
- Integration tests for component interactions
- User interface tests for critical user workflows
- Performance tests for resource-intensive operations
- Edge case testing with boundary conditions
- Error scenario testing with invalid inputs

**Test Organization:**
```
Tests/
├── UnitTests/          # Fast, isolated unit tests
├── IntegrationTests/   # Component interaction tests
├── UITests/           # User interface automation tests
└── PerformanceTests/  # Benchmarking and performance tests
```

### 5.3 Documentation Standards

**Required Documentation:**
- Clear README with setup and usage instructions
- API documentation for public interfaces
- Architecture documentation for complex systems
- Deployment and configuration guides
- Troubleshooting and FAQ sections
- Contributing guidelines for team projects

---

## Automatic Behaviors

### 6.1 Always Do These Actions

**For Every Development Project:**
1. Read existing project structure and documentation first
2. Understand the current architecture and patterns before making changes
3. Build and test after every significant change
4. Fix all build errors and warnings before proceeding
5. Use established patterns and conventions consistently
6. Implement proper error handling and user feedback
7. Document complex logic and architectural decisions

**For Every Code Change:**
1. Validate that the change solves the intended problem
2. Ensure the change doesn't break existing functionality
3. Test the change in realistic usage scenarios
4. Review the code for maintainability and clarity
5. Update relevant documentation and comments
6. Consider performance and security implications

**For Every Complex Feature:**
1. Design the feature architecture before implementation
2. Break down into smaller, testable components
3. Implement incrementally with frequent testing
4. Validate against requirements throughout development
5. Optimize for performance and user experience
6. Ensure accessibility and platform compliance

### 6.2 Never Do These Things

**Code Quality Violations:**
- Never leave TODO comments in production code
- Never use deprecated APIs without clear justification and migration plan
- Never skip error handling for operations that can fail
- Never use hardcoded values without proper configuration management
- Never commit code that doesn't compile or pass basic tests
- Never ignore compiler warnings without understanding their implications

**Project Management Anti-patterns:**
- Never proceed with broken builds or failing tests
- Never combine unrelated changes in single commits
- Never skip documentation for complex or non-obvious logic
- Never ignore performance regressions or memory leaks
- Never deploy code without proper testing and validation
- Never make architectural changes without team consultation

---

This master instruction set provides comprehensive guidance for software development workflows while remaining agnostic to specific project domains. It emphasizes quality, testing, and systematic approaches to building robust applications.

---
> Source: [linroger/ShortcutsKeeper](https://github.com/linroger/ShortcutsKeeper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
