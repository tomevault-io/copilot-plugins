## bmad-claude-agents

> This project uses the Enhanced BMAD Method - combining persona-based AI agents with advanced context management, persistent memory, and comprehensive testing orchestration.

# Enhanced BMAD Method with Context Management

This project uses the Enhanced BMAD Method - combining persona-based AI agents with advanced context management, persistent memory, and comprehensive testing orchestration.

## 🚀 Quick Start

### ✨ NEW: Autonomous Orchestration (Recommended)
```bash
# Simply tell the orchestrator what you want - it handles EVERYTHING automatically
"I want to create a task management app"

# The orchestrator will AUTONOMOUSLY:
# 1. Create a complete workflow plan
# 2. Delegate to Mary (analyst) for project brief
# 3. Continue to John (PM) for PRD
# 4. Proceed to Winston (architect) for technical design
# 5. Validate with Sarah (PO)
# 6. Create stories with Bob (SM)
# 7. Implement with James (Dev)
# 8. Review with Quinn (QA)
# 9. Test thoroughly
# 10. Report back with complete results

# You just wait for the magic to happen! ✨
```

### Manual Agent Approach (Advanced Users)
```bash
# If you prefer direct control, you can still call agents manually
"Use bmad-analyst to research and create project brief for [your project]"
"Use bmad-pm to create PRD based on the project brief"
"Use bmad-architect to create technical architecture"
```

### For Existing Projects
```bash
# Initialize BMAD + Context system
"Set up this codebase with BMAD method"
# Orchestrator will delegate to project-initializer automatically
```

## 🤖 Agent System Overview

### 🎯 Autonomous Orchestration System
The bmad-orchestrator is now your SINGLE POINT OF INTERACTION:
- **Automatic Workflow Management** - Just describe what you want, orchestrator handles the rest
- **Intelligent Agent Delegation** - Knows which agent to use and when
- **Progress Tracking** - Updates you as work progresses
- **Quality Gates** - Ensures each phase meets standards before proceeding
- **Error Recovery** - Handles issues and retries automatically
- **Health Monitoring** - Tracks agent performance and creates new agents as needed

### Core BMAD Agents (Personas)
- **bmad-orchestrator** - Autonomous master coordinator with context management
- **bmad-analyst** (Mary) - Business Analyst for research and ideation
- **bmad-pm** (John) - Product Manager for PRDs and user stories
- **bmad-architect** (Winston) - System Architect for technical design
- **bmad-po** (Sarah) - Product Owner for validation and process
- **bmad-sm** (Bob) - Scrum Master for detailed story creation
- **bmad-dev** (James) - Developer for implementation
- **bmad-qa** (Quinn) - QA Engineer for code review
- **bmad-ux** (Sally) - UX Designer for interfaces
- **bmad-devops** (Alex) - DevOps Engineer for infrastructure
- **bmad-tech-writer** (Morgan) - Technical Writer for documentation

### Infrastructure Agents
- **testing-orchestrator** - Comprehensive testing coordination
- **project-initializer** - Project setup and configuration

## 📋 Automatic Agent Selection

When responding to user requests, the appropriate BMAD agent is automatically selected:

### Planning & Analysis Tasks
- **Market research, competitive analysis, brainstorming** → `bmad-analyst` (Mary)
- **Creating PRDs, epics, user stories** → `bmad-pm` (John)
- **System design, architecture decisions** → `bmad-architect` (Winston)
- **Document validation, sharding, process integrity** → `bmad-po` (Sarah)

### Development Tasks
- **Creating detailed development stories** → `bmad-sm` (Bob)
- **Code implementation, debugging, testing** → `bmad-dev` (James)
- **Code review, quality assurance** → `bmad-qa` (Quinn)

### Quality Assurance
- **Comprehensive testing orchestration** → `testing-orchestrator`
- **Code quality review** → `bmad-qa` (Quinn)

### Supporting Tasks
- **UI/UX design, wireframes, mockups** → `bmad-ux` (Sally)
- **CI/CD pipelines, deployment setup** → `bmad-devops` (Alex)
- **API documentation, user guides** → `bmad-tech-writer` (Morgan)

### Coordination & Setup
- **Workflow guidance, agent selection help** → `bmad-orchestrator`
- **Project initialization** → `project-initializer`

## 🔄 Enhanced Workflow

### 🚨 IMPORTANT: Execution Mode
All agents now operate in **EXECUTION MODE** - they create actual files instead of just describing what they would do. Every agent uses Write/Edit tools to create real deliverables.

### Phase 1: Planning (Web UI)
1. **Research & Ideation** → bmad-analyst uses Write tool to create project brief at `projectdocs/{project}-brief.md`
2. **Product Definition** → bmad-pm uses Write tool to create PRD at `projectdocs/{project}-prd.md`
3. **Architecture Design** → bmad-architect uses Write tool to create architecture at `projectdocs/{project}-architecture.md`
4. **Validation** → bmad-po uses Write tool to create validation report at `projectdocs/{project}-validation.md`

### Phase 2: Development (IDE)
1. **Story Preparation** → bmad-sm uses Write tool to create stories at `projectdocs/{project}-stories.md`
2. **Implementation** → bmad-dev uses Write/Edit tools to create actual code files
3. **Quality Review** → bmad-qa uses Write tool to create review at `projectdocs/{project}-qa-report.md`
4. **Testing** → testing-orchestrator uses Write tool to create test results

### Phase 3: Deployment
1. **Infrastructure** → bmad-devops sets up CI/CD
2. **Documentation** → bmad-tech-writer creates docs

## 🧠 Context Management System

### Persistent Memory
Each agent maintains context in `.claude/context/`:
- Research findings and market analysis (Mary)
- Product decisions and PRD history (John)
- Architecture choices and patterns (Winston)
- Validation results and process checks (Sarah)
- Story templates and sprint history (Bob)
- Code patterns and implementation notes (James)
- Review patterns and quality metrics (Quinn)

### Directory Structure
```
.claude/
├── agents/              # BMAD agent definitions
├── context/             # Persistent agent contexts
├── test-specs/          # Testing specifications
├── workflows/           # Workflow documentation
├── agent-analytics/     # Performance metrics
├── templates/           # Reusable patterns
└── commands/            # Utility scripts
```

## 🎮 Orchestrator Commands

The bmad-orchestrator supports these commands:
- `*help` - Show available agents and workflows
- `*agent [name]` - Transform into specific agent
- `*workflow-guidance` - Get workflow recommendations
- `*context-status` - View context files and health
- `*agent-health` - Display performance metrics
- `*test-orchestrate` - Start comprehensive testing
- `*init-project` - Initialize project with BMAD
- `*create-agent [type] [name]` - Create specialized agent

## 🧪 Testing Integration

### Comprehensive Testing
The testing-orchestrator coordinates:
- Unit Tests (component validation)
- Integration Tests (API/service testing)
- Workflow Tests (end-to-end validation)
- Performance Tests (speed and resources)
- Accessibility Tests (WCAG compliance)
- Console Monitoring (zero errors policy)

### Testing Workflow
1. Developer completes implementation
2. Creates testing handoff document
3. Testing orchestrator analyzes changes
4. Runs comprehensive test suite
5. Provides quality gate decision

## 📊 Agent Health Monitoring

### Performance Tracking
- Usage frequency and success rates
- Average task completion times
- Context freshness and quality
- Cross-agent collaboration efficiency

### Dynamic Agent Creation
When patterns emerge:
- Orchestrator detects repeated needs
- Creates specialized connector agents
- Optimizes workflow efficiency
- Preserves knowledge in templates

## 🔧 Advanced Features

### Cross-Project Learning
- Successful patterns saved to templates
- Knowledge preserved when agents retire
- Patterns shared across projects
- Continuous improvement loop

### Tmux Integration
For persistent, autonomous operation:
- `.claude/commands/start-bmad-project.sh` - Start with tmux layout
- `.claude/commands/schedule-check-in.sh` - Schedule agent tasks
- `.claude/commands/send-agent-message.sh` - Inter-agent messaging

### Git Integration
- Development agents follow 30-minute commit rule
- Automatic work preservation
- Clear commit messages with context

## 📚 Best Practices

1. **Start with Context**: Use project-initializer for existing projects
2. **Follow the Flow**: Planning → Development → Testing → Deployment
3. **Trust the Process**: Let agents handle their expertise areas
4. **Review Results**: Check context files and test reports
5. **Iterate Efficiently**: Use test feedback to guide improvements

## 🚨 Important Notes

- All agent commands start with `*` when using orchestrator
- Agents maintain specific personas and expertise
- Context persists across sessions automatically
- Testing is comprehensive and mandatory
- Console errors block deployment

## 🎯 Example Usage - Autonomous Workflows

### Complete Project Creation
```bash
User: "I want to create a task management app"

Orchestrator: "Starting autonomous workflow for task management app..."
→ Creating workflow plan...
→ Delegating to Mary for research and project brief...
→ Project brief created. Moving to John for PRD...
→ PRD complete. Winston is designing architecture...
→ Architecture ready. Sarah is validating alignment...
→ Validation passed. Bob is creating development stories...
→ Stories ready. James is implementing core features...
→ Implementation complete. Quinn is reviewing code...
→ Review passed. Running comprehensive tests...
✅ Task management app complete! Created 8 documents and implemented core features.
```

### Feature Addition
```bash
User: "Add user authentication to my app"

Orchestrator: "Analyzing existing project structure..."
→ Found existing architecture. Updating with auth design...
→ Creating authentication stories...
→ Implementing auth features...
→ Running security review...
→ Testing authentication flow...
✅ Authentication added successfully! 3 endpoints, UI components, and tests created.
```

### Quick Tasks
```bash
User: "Review my code quality"

Orchestrator: "Starting code quality review..."
→ Quinn is analyzing code patterns...
→ Found 3 improvement areas. James is refactoring...
→ Running tests after changes...
✅ Code quality improved! All tests passing, 0 console errors.
```

## 🚀 Key Benefits

1. **Zero Learning Curve** - Just describe what you want
2. **Complete Automation** - No need to manage workflow sequences
3. **Quality Guaranteed** - Built-in validation and testing
4. **Context Aware** - Learns from each project
5. **Self-Improving** - Creates new agents as patterns emerge

This enhanced BMAD system with autonomous orchestration makes professional software development accessible to everyone, while maintaining the highest quality standards through intelligent automation.

---
> Source: [wolverin0/bmad-claude-agents](https://github.com/wolverin0/bmad-claude-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
