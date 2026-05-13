## speckit-tutorial

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a spec-kit tutorial repository containing three progressive tutorial projects that teach Spec-Driven Development using GitHub's spec-kit tool. The repository is primarily educational documentation, not executable code.

## Repository Structure

```
speckit-tutorial/
├── tutorial-1-todo-app/          # Beginner: Basic spec-kit workflow
│   ├── README.md                 # Step-by-step guide
│   └── .speckit/                 # Pre-written spec files as examples
│       ├── constitution.md
│       ├── specifications/
│       ├── plans/
│       └── tasks/
├── tutorial-2-api-service/       # Intermediate: Multi-spec management
│   ├── README.md
│   └── .speckit/
├── tutorial-3-microservices/     # Advanced: Microservices architecture
│   ├── README.md
│   └── .speckit/
└── README.md                     # Main index and learning guide
```

## Key Concepts

### Spec-Kit Workflow
The tutorials teach this standard workflow:
1. **Constitution** (`/speckit.constitution`) - Define project principles and constraints
2. **Specify** (`/speckit.specify`) - Define requirements and user stories
3. **Plan** (`/speckit.plan`) - Create technical implementation strategy
4. **Tasks** (`/speckit.tasks`) - Break down into actionable tasks
5. **Implement** (`/speckit.implement`) - Execute implementation

### Tutorial Progression
- **Tutorial 1**: Vanilla JavaScript ToDo app - Teaches basic workflow
- **Tutorial 2**: Node.js REST API - Introduces `/speckit.clarify` and `/speckit.analyze`
- **Tutorial 3**: Microservices with Docker - Adds `/speckit.checklist` and complex architecture

## Working with This Repository

### This is a Documentation Repository
- The `.speckit/` directories contain **example specification files**, not generated code
- Users are expected to use these as templates when learning spec-kit
- The tutorials guide users to generate actual implementations in their own workflow

### When Updating Tutorials

Each tutorial README.md follows this structure:
1. Goals and learning objectives
2. Step-by-step instructions using spec-kit commands
3. Example prompts for AI agents
4. Verification checklists
5. Troubleshooting section

When updating specifications in `.speckit/` directories:
- Maintain consistency with the tutorial's stated learning objectives
- Ensure specifications are detailed enough to be educational examples
- Keep technical stack choices aligned with the constitution.md
- Verify that tasks in `tasks/*.md` reference the correct specifications

### Spec File Conventions

**Constitution files** should include:
- Project vision and core principles
- Technical constraints and technology stack
- Development practices and standards
- Out of scope items
- Success criteria

**Specification files** should include:
- User stories with acceptance criteria
- Data models and schemas
- API endpoints (for API projects)
- Non-functional requirements
- Edge cases and error scenarios

**Plan files** should include:
- Architecture diagrams (ASCII art)
- File structure
- Component breakdown
- Implementation phases
- Technical decisions with rationale

**Task files** should:
- Break work into phases
- Include time estimates
- Specify dependencies
- Reference related specifications
- Be ordered logically

### Language Conventions
- All documentation is in Japanese (日本語)
- Code examples and technical terms use English
- Spec-kit command names remain in English (e.g., `/speckit.specify`)

### Tutorial Consistency

When modifying tutorials, ensure:
- Tutorial 1 uses simple, framework-free technologies (HTML/CSS/JS)
- Tutorial 2 uses Node.js + Express with JWT auth
- Tutorial 3 uses microservices with Docker Compose and RabbitMQ
- Each tutorial builds on concepts from previous ones
- Time estimates are realistic (30min / 60min / 90min)

## External Dependencies

This repository teaches usage of:
- **spec-kit**: `uv tool install specify-cli --from git+https://github.com/github/spec-kit.git`
- AI agents: Claude Code, Cursor, GitHub Copilot (all compatible)

The tutorials assume users have:
- An AI coding agent installed
- Basic understanding of software development
- Tools for their chosen tutorial (browser for T1, Node.js for T2, Docker for T3)

## Maintenance Notes

### When Adding New Tutorials
Follow the established pattern:
1. Create `tutorial-N-name/README.md` with step-by-step guide
2. Create `.speckit/` subdirectories: specifications, plans, tasks
3. Write example spec files that are educational and detailed
4. Update main README.md with tutorial description
5. Ensure Japanese language consistency

### When Updating Specs
If updating specification examples in `.speckit/` directories, also update:
- Corresponding tutorial README.md if workflow changes
- Task breakdowns to match new specifications
- Checklists to verify new features

### Quality Standards
Spec files should demonstrate:
- Clear, unambiguous requirements
- Realistic technical constraints
- Proper separation of concerns (constitution vs specification vs plan)
- Practical examples that users can learn from

---
> Source: [kkawailab/speckit-tutorial](https://github.com/kkawailab/speckit-tutorial) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
