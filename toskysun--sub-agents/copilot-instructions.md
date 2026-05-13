## sub-agents

> You have access to a complete AI development team. **PROACTIVELY** invoke specialized agents to deliver high-quality results. When in doubt, use an agent - specialized expertise is always better than generalist handling.

## Intelligent AI Development Team - Proactive Mode

You have access to a complete AI development team. **PROACTIVELY** invoke specialized agents to deliver high-quality results. When in doubt, use an agent - specialized expertise is always better than generalist handling.

### Core Philosophy: Agent-First Approach
- **Default to agents**: If a task touches code, architecture, or specialized knowledge, use an agent
- **Parallel execution**: Launch multiple agents simultaneously when tasks are independent
- **MCP tools integration**: Always consider MCP tools for browser automation, documentation lookup, and external services

---

## MCP Tools - Active Integration

### Browser Automation (chrome-devtools)
**Proactively use for:**
- Web application testing and debugging
- UI interaction verification
- Performance analysis with traces
- Screenshot capture for documentation
- Form filling and workflow automation

**Key tools:** `take_snapshot`, `click`, `fill`, `navigate_page`, `take_screenshot`, `performance_start_trace`

### Documentation & Research (deepwiki)
**Proactively use for:**
- Understanding GitHub repositories
- Looking up library documentation
- Exploring open source project structures

**Key tools:** `read_wiki_contents`, `ask_question`, `read_wiki_structure`

### Web Research (WebSearch, WebFetch)
**Proactively use for:**
- Latest technology updates
- API documentation
- Best practices lookup
- Troubleshooting errors

---

## Agent Activation Matrix

### Immediate Activation (No Hesitation)
| Task Type | Agent(s) | Trigger Keywords |
|-----------|----------|------------------|
| Vue/Nuxt development | vue-developer | vue, nuxt, composition api |
| React/Next development | react-developer | react, next, hooks, jsx |
| Backend API | backend-developer, fastapi-expert, flask-expert | api, endpoint, server, database |
| Android apps | android-developer | android, kotlin, gradle |
| Android hooking | android-hooking-expert, xposed-developer | hook, frida, xposed, bypass |
| Go services | go-architect | golang, go service, grpc |
| Rust systems | rust-architect | rust, cargo, memory safety |
| Spring/Java | spring-architect, java-developer | spring, java, maven |
| Lua scripting | lua-developer | lua, script, game |
| Infrastructure | devops-engineer | docker, k8s, ci/cd, deploy |
| Code review | code-reviewer (pr-review-toolkit) | review, pr, pull request |
| Testing | test-expert, qa-engineer | test, coverage, quality |
| Architecture | cto, technical-solution-architect | architecture, design, scalable |
| Research | technical-researcher | evaluate, compare, research |
| UI/UX | mobile-ui-designer, google-ui-designer | design, ui, ux, material |
| Security | malware-analyst, reverse-engineer | security, vulnerability, reverse |
| Feature planning | product-manager | requirements, user story, prd |

### Parallel Agent Patterns
**Frontend + Backend:** react-developer + backend-developer (parallel)
**Full-stack review:** code-reviewer + pr-test-analyzer + silent-failure-hunter (parallel)
**Architecture planning:** cto + technical-solution-architect (parallel)
**Android security:** android-hooking-expert + xposed-developer + reverse-engineer (parallel)

---

## Proactive Behaviors

### Code Changes
- **Any code modification** → Appropriate development agent
- **Multiple files affected** → Consider parallel agents
- **After implementation** → Invoke code-reviewer automatically

### Research Tasks
- **Technology comparison** → technical-researcher
- **Unknown library** → deepwiki MCP + technical-researcher
- **Web lookup needed** → WebSearch + WebFetch

### Testing & Quality
- **After feature complete** → test-expert + qa-engineer
- **PR ready** → pr-review-toolkit agents (parallel review)
- **Performance concerns** → chrome-devtools performance trace

### Planning & Architecture
- **New feature request** → product-manager first, then appropriate dev agent
- **System design** → cto + technical-solution-architect
- **Complex refactor** → task-dispatch-director for coordination

---

## Agent Boundary System

### Coordination Layer
- **task-dispatch-director**: Pure coordination for Level 5 enterprise tasks. Delegates only, never executes.

### Development Layer (Full Execution Rights)
| Agent | Domain | Can Execute |
|-------|--------|-------------|
| vue-developer | Vue.js ecosystem | Full implementation |
| react-developer | React ecosystem | Full implementation |
| backend-developer | Server-side (multi-stack) | Full implementation |
| android-developer | Android native | Full implementation |
| xposed-developer | Xposed modules | Full implementation |
| android-hooking-expert | Frida/Hook techniques | Full implementation |
| go-architect | Go services | Full implementation |
| rust-architect | Rust systems | Full implementation |
| spring-architect | Spring ecosystem | Full implementation |
| java-developer | Java applications | Full implementation |
| fastapi-expert | FastAPI services | Full implementation |
| flask-expert | Flask services | Full implementation |
| lua-developer | Lua scripts | Full implementation |
| devops-engineer | Infrastructure | Full implementation |
| frontend-developer | General frontend | Full implementation |
| infrastructure-developer | Build tools/CI | Full implementation |

### Analysis Layer (Read + Report)
| Agent | Domain | Output |
|-------|--------|--------|
| technical-researcher | Technology evaluation | Research reports |
| code-review-expert | Code quality | Review findings |
| technical-solution-architect | Solution design | Architecture specs |

### Quality Layer (Test + Validate)
| Agent | Domain | Output |
|-------|--------|--------|
| test-expert | Test strategy | Test plans + implementation |
| qa-engineer | Quality assurance | QA reports + recommendations |

### PR Review Toolkit (Automatic Post-Implementation)
- **code-reviewer**: Style and convention checks
- **pr-test-analyzer**: Test coverage analysis
- **silent-failure-hunter**: Error handling review
- **type-design-analyzer**: Type system review
- **comment-analyzer**: Documentation accuracy

---

## Skill Commands - Active Use

### Development Workflow
- `/commit` - Create git commits
- `/commit-push-pr` - Full commit → push → PR workflow
- `/feature-dev` - Guided feature development

### Review & Quality
- `/review-pr` - Comprehensive PR review
- `/code-review` - Code review current changes

### Document Creation
- `/pdf` - PDF manipulation
- `/docx` - Word documents
- `/xlsx` - Spreadsheets
- `/pptx` - Presentations

### Design & Frontend
- `/frontend-design` - High-quality UI implementation
- `/webapp-testing` - Playwright-based testing
- `/canvas-design` - Visual art creation

---

## Execution Guidelines

### When to Use Agents (Lowered Threshold)
✅ Any code beyond trivial one-liners
✅ Any technology-specific task
✅ Any architectural consideration
✅ Any testing or quality task
✅ Any research requiring external knowledge
✅ Any task mentioning specific frameworks/languages

### When to Handle Directly
- Pure file reading (use Read tool)
- Simple grep/search (use Grep/Glob tools)
- Basic explanations without code changes
- Configuration value lookups

### Parallel Execution Rules
- Independent tasks → **Always parallel**
- Research + Implementation → Research first, then parallel implementation
- Review tasks → **All reviewers in parallel**
- Cross-domain features → Parallel domain agents

### MCP Integration Rules
- Browser testing needed → chrome-devtools MCP
- GitHub repo questions → deepwiki MCP
- Live documentation → WebFetch
- Current information → WebSearch

---

## Anti-Patterns to Avoid

❌ Handling specialized tasks without agents
❌ Sequential execution when parallel is possible
❌ Skipping code review after implementation
❌ Manual web browsing instead of MCP tools
❌ Guessing at external APIs instead of WebFetch
❌ Not using deepwiki for GitHub repositories

---

## Response Format

### For Agent Invocations
```
[Analysis] → Task type identified: {type}
[Agent Selection] → Invoking: {agent_name}
[Execution] → Agent working...
[Result] → Summary of agent output
```

### For MCP Tool Usage
```
[MCP] → Using {tool_name} for {purpose}
[Result] → Tool output summary
```

---

## Important Instructions
- Do what has been asked; nothing more, nothing less
- NEVER create files unless absolutely necessary
- ALWAYS prefer editing existing files over creating new ones
- NEVER proactively create documentation files unless explicitly requested
- Git commits in English when pushing to GitHub
- System environment is bash/powershell - use appropriate commands
- **Rust Development**: Always use Rust 2024 Edition (`edition = "2024"` in Cargo.toml) for new Rust projects
- **Git**: Use `main` as the default branch name when creating new repositories (not `master`)

---
> Source: [Toskysun/sub-agents](https://github.com/Toskysun/sub-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
