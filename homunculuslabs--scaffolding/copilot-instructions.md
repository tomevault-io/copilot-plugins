## scaffolding

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a scaffolding system for setting up new projects with multi-agent development workflows, documentation systems, and testing frameworks. The scaffolding includes templates and tools for:

- Multi-agent sprint management
- Context handoff systems
- Bug detection protocols
- Documentation organization
- Testing frameworks
- Development workflows

## Quick Start Commands

```bash
# Install scaffolding system
./docs/scaffolding/install-sprint-system.sh

# Initialize context handoff system
cp docs/scaffolding/context-handoff/*.sh .
chmod +x *.sh

# Create initial documentation structure
cp -r docs/scaffolding/docs/* docs/

# Set up bug detection system
cp -r docs/scaffolding/bug-detection-system/* .

# Initialize testing framework
cp -r docs/scaffolding/feature-testing/* tests/
```

## Architecture

### Scaffolding Components

- **Sprint System** (`docs/scaffolding/`): Multi-agent project management templates
- **Context Handoff** (`context-handoff/`): Session context preservation scripts
- **Bug Detection** (`bug-detection-system/`): Automated bug tracking system
- **Documentation System** (`docs-system/`): Organized documentation structure
- **Feature Testing** (`feature-testing/`): Comprehensive testing templates

### Directory Structure
```
docs/scaffolding/
├── README.md                    # Scaffolding overview
├── install-sprint-system.sh     # Main installation script
├── context-handoff/             # Context management scripts
│   ├── quick-handoff.sh
│   ├── restore-context.sh
│   └── README.md
├── bug-detection-system/        # Bug tracking templates
│   ├── BUG_INDEX.md
│   ├── templates/
│   └── README.md
├── docs-system/                 # Documentation organization
│   ├── templates/
│   ├── index.md
│   └── README.md
├── feature-testing/             # Testing framework
│   ├── templates/
│   ├── run-all-tests.sh
│   └── README.md
└── docs/                        # Example documentation structure
    ├── sprints/
    ├── features/
    ├── guides/
    └── reference/
```

## Common Tasks

### Initialize New Project
```bash
# 1. Copy scaffolding to new project
cp -r docs/scaffolding/* /path/to/new-project/

# 2. Run installation script
cd /path/to/new-project
./install-sprint-system.sh

# 3. Customize CLAUDE.md for your project
# Edit this file to match your project's architecture
```

### Set Up Multi-Agent Workflow
```bash
# Create sprint directories
mkdir -p docs/sprints/sprint-{1..10}

# Initialize agent task tracking
cp docs/scaffolding/docs/sprints/README.md docs/sprints/

# Set up context handoff
./quick-handoff.sh "project initialization complete"
```

### Configure Testing Framework
```bash
# Copy testing templates
cp -r docs/scaffolding/feature-testing/* tests/

# Set up test categories based on your project
# Edit tests/package.json scripts section
# Customize test templates in tests/templates/
```

## Development Workflow

### Sprint Management
1. Update sprint goals in CLAUDE.md (copy from scaffolding template)
2. Assign agents to focus areas
3. Use context handoffs for session management
4. Track completed tasks and confidence levels
5. Generate sprint reports

### Context Management
```bash
# Start work session
./restore-context.sh  # See available contexts

# During work (create handoffs when needed)
./quick-handoff.sh "current task description"

# End session
./quick-handoff.sh "work completed and next steps"
```

### Bug Detection
```bash
# Run bug detection on modified files
# Follow templates in bug-detection-system/

# Check for common issues:
# - Security vulnerabilities
# - Performance problems
# - Code quality issues
# - Integration problems
```

## Customization for New Projects

### 1. Update Project Information
- Replace project name throughout scaffolding files
- Update architecture section with your tech stack
- Modify service ports and endpoints
- Customize authentication methods

### 2. Adapt Development Commands
```bash
# Replace with your project's commands:
npm run build                    # Your build command
npm run dev                      # Your dev server
npm test                         # Your test command
npm run lint                     # Your linting
```

### 3. Configure Services
- Update service management (PM2, Docker, etc.)
- Configure database connections
- Set up environment variables
- Customize deployment scripts

### 4. Tailor Testing Strategy
- Define test categories for your project
- Set up testing environments
- Configure CI/CD pipelines
- Create project-specific test templates

## Integration Patterns

### For Web Applications
- API gateway patterns
- Frontend/backend separation
- Database migration systems
- Authentication flows

### For AI/ML Projects
- Model management
- Data pipeline testing
- LLM integration patterns
- Performance monitoring

### For Microservices
- Service discovery
- Inter-service communication
- Distributed testing
- Container orchestration

## Best Practices

### Documentation
- Follow the documentation organization system
- Keep CLAUDE.md updated with project changes
- Use consistent naming conventions
- Maintain central documentation index

### Multi-Agent Development
- Use clear agent role definitions
- Implement proper context handoffs
- Track sprint progress systematically
- Maintain confidence ratings on deliverables

### Quality Assurance
- Run comprehensive test suites
- Implement bug detection protocols
- Use automated quality checks
- Document known issues and workarounds

## Support and Resources

- **Scaffolding Documentation**: `docs/scaffolding/README.md`
- **Installation Guide**: `docs/scaffolding/INSTALL.md`
- **System Templates**: `docs/scaffolding/*/templates/`
- **Example Configurations**: `docs/scaffolding/docs/`

## Notes for Future Development

This scaffolding system is designed to be:
- **Adaptable**: Easy to customize for different project types
- **Comprehensive**: Covers development workflow, testing, and documentation
- **Scalable**: Supports multi-agent development teams
- **Maintainable**: Clear organization and documentation standards

When adapting this scaffolding:
1. Preserve the core workflow patterns
2. Customize technology-specific details
3. Maintain documentation organization
4. Keep multi-agent coordination features

---
> Source: [HomunculusLabs/scaffolding](https://github.com/HomunculusLabs/scaffolding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
