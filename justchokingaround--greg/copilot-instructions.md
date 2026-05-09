## greg

> Specialized AI agents for complex, autonomous tasks in the greg project.

# greg Development Agents

Specialized AI agents for complex, autonomous tasks in the greg project.

## What Are Agents?

Agents are specialized AI assistants that can autonomously handle multi-step tasks. Unlike simple commands, agents can:
- Research and explore code
- Make multiple decisions
- Use multiple tools
- Provide comprehensive reports
- Work independently on complex tasks

## Available Agents

### 🎨 tui-dev
**Expert in Bubble Tea terminal interfaces**

Use for:
- Building TUI components in `internal/tui/components/`
- Styling with Lip Gloss and oxocarbon theme
- Implementing keyboard navigation
- Creating responsive layouts
- Snapshot testing with `tuitest/`

**Example:**
```
Build a search view component with live results
```

**Location:** `.claude/agents/tui-dev.md`

---

### 📺 mpv-integration
**Expert in mpv player integration with gopv**

Use for:
- Implementing player features in `internal/player/mpv/`
- IPC communication with mpv
- Progress monitoring
- Cross-platform socket handling
- Player controls and cleanup

**Example:**
```
Implement progress tracking with callbacks
```

**Location:** `.claude/agents/mpv-integration.md`

---

### 📊 anilist-api
**Expert in AniList GraphQL integration**

Use for:
- OAuth2 authentication flow
- GraphQL queries and mutations
- Progress synchronization (85% threshold)
- Rate limiting
- Provider-to-AniList mapping

**Example:**
```
Add library search functionality
```

**Location:** `.claude/agents/anilist-api.md`

---

### 🧪 test-writer
**Expert in comprehensive testing**

Use for:
- Writing unit tests with table-driven patterns
- Creating integration tests (`//go:build integration`)
- HTTP mocking with `httptest.NewServer`
- Improving coverage (target: 80%+)
- TUI snapshot testing

**Example:**
```
Write comprehensive tests for the HiAnime provider
```

**Location:** `.claude/agents/test-writer.md`

---

### 🔍 code-reviewer
**Expert in Go code review**

Use for:
- Reviewing code changes
- Finding security issues
- Performance analysis
- Greg-specific pattern checks
- Pre-commit reviews

**Example:**
```
Review my recent provider implementation
```

**Location:** `.claude/agents/code-reviewer.md`

---

### 🔧 refactoring-expert
**Expert in large-scale Go code restructuring**

Use for:
- Decomposing large files (model.go is 4822 lines)
- Extracting packages from monolithic code
- Safe refactoring with LSP verification
- Interface extraction and dependency injection
- Maintaining backward compatibility

**Example:**
```
Help me break up model.go into smaller components
```

**Location:** `.claude/agents/refactoring-expert.md`

---

### 🐛 provider-debugger
**Expert in diagnosing streaming provider issues**

Use for:
- Debugging scraping failures
- Handling site structure changes
- Cloudflare/anti-bot bypasses
- Provider implementation patterns
- Network debugging

**Example:**
```
The hianime provider is returning empty results
```

**Location:** `.claude/agents/provider-debugger.md`

---

## How to Use Agents

### In Chat

Describe what you need and the appropriate agent will be selected:

```
User: I need to implement a new streaming provider for Crunchyroll
Assistant: [Creates provider structure with tests]

User: Create a media details view component for the TUI
Assistant: [Uses tui-dev patterns to build the component]

User: Review my recent changes
Assistant: [Runs code-reviewer analysis]
```

### Explicitly Request an Agent

```
Use the test-writer agent to add tests for the player package

I want the code-reviewer agent to check my implementation
```

## Known Issues

All agents are aware of these project-wide issues:

| Issue | Affected | Status |
|-------|----------|--------|
| Manga reading mode bugs | tui-dev | Not production ready |

## Agent Workflows

### New Provider Implementation

```
1. Create provider in internal/providers/{name}/
2. Implement Provider interface
3. Add tests with httptest mocks
4. Register in registry.go
5. Run: just test-pkg providers/{name}
```

### Building TUI Feature

```
1. Create component in internal/tui/components/{name}/
2. Implement Model/Update/View
3. Add keyboard shortcuts
4. Style with oxocarbon theme
5. Add snapshot tests
6. Wire up in model.go
```

### Complete Feature Development

```
1. Implement feature
2. Add tests (80%+ coverage)
3. Review code
4. Run: just pre-commit
5. Commit
```

## Combining Agents with Commands

Agents work well with slash commands:

```bash
# Implement feature
User: Create a history view component

# Then use commands for workflow
/test          # Run tests
/lint          # Check code quality
/build         # Build project
/review        # Code review
```

## Quick Reference

| Task | Agent | Example |
|------|-------|---------|
| TUI component | tui-dev | "Create X view" |
| Player feature | mpv-integration | "Add progress callback" |
| AniList feature | anilist-api | "Add library search" |
| Write tests | test-writer | "Add tests for X" |
| Code review | code-reviewer | "Review changes" |
| Large refactoring | refactoring-expert | "Break up model.go" |
| Provider issues | provider-debugger | "Fix empty results" |

## Available Commands

| Command | Description |
|---------|-------------|
| `/build` | Build greg binary |
| `/test` | Run tests with coverage |
| `/lint` | Run linters |
| `/review` | Review code changes |
| `/debug` | Debug issues |
| `/setup` | Setup dev environment |
| `/docs` | Update documentation |
| `/scaffold` | Generate boilerplate |
| `/provider` | Scaffold new provider |
| `/mpv-test` | Test mpv integration |
| `/proto` | Quick prototype |
| `/refactor` | Safe code refactoring |
| `/todo` | Find TODO/FIXME comments |
| `/health` | Provider health checks |
| `/changelog` | Generate changelog entries |

## See Also

- **CLAUDE.md** - AI assistant guidance
- **docs/dev/CONTRIBUTING.org** - Development guidelines
- **docs/dev/ARCHITECTURE.org** - System design
- **justfile** - Build commands (`just` to list all)

---

**Tip:** Run `just` to see all available build commands. Run `just doctor` to verify your environment.

---
> Source: [justchokingaround/greg](https://github.com/justchokingaround/greg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
