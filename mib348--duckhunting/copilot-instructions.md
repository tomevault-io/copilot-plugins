## main-rule

> - **ALWAYS use Context7 MCP tool** when needing information about JavaScript libraries, packages, or frameworks

# MCP Tools Priority Rules

## Context7 MCP Tool Usage (HIGHEST PRIORITY)
- **ALWAYS use Context7 MCP tool** when needing information about JavaScript libraries, packages, or frameworks
- **NEVER rely on potentially outdated training data** for library documentation
- Before implementing any library integration, use `resolve-library-id` followed by `get-library-docs`
- When troubleshooting library issues, fetch latest docs via Context7
- Examples of when to use Context7:
  - PixiJS implementation details
  - jQuery latest features
  - GSAP animation methods
  - Howler.js audio configuration
  - Vite configuration options
  - Any npm package documentation

## Web Search Priority
- Use web search for:
  - Current best practices and patterns
  - Latest browser compatibility information
  - Performance optimization techniques
  - Security recommendations
  - Real-world implementation examples

## Development Environment Requirements

### Hot Reloading (MANDATORY)
- **MUST implement hot reloading** for all development workflows
- Use Vite's built-in HMR (Hot Module Replacement) for:
  - JavaScript modules
  - CSS stylesheets
  - Asset changes
  - Game state preservation during development
- Configure Vite dev server with:
  - Fast refresh for instant updates
  - Error overlay for debugging
  - Asset watching for sprites/audio files

### Automated End-to-End Testing (MANDATORY)
- **MUST implement comprehensive E2E test automation**
- Use Playwright for cross-browser testing covering:
  - Game initialization and loading
  - Duck spawning and movement
  - Player input (mouse, touch, keyboard)
  - Audio playback and controls
  - Settings persistence
  - Shop functionality
  - Score tracking and progression
  - PWA installation and offline mode
- Configure automated test runs on:
  - Every commit (GitHub Actions)
  - Pull request validation
  - Pre-deployment checks
  - Scheduled regression testing

## MCP Tool Integration Workflow

### Sequential Thinking Usage
- Use for complex architectural decisions
- Planning game mechanics and systems
- Troubleshooting performance issues
- Design pattern selection

### Task Manager Usage
- Break down all major features into trackable tasks
- Maintain approval workflow for quality gates
- Track progress against PRD milestones
- Coordinate development phases

### Context7 Integration Examples
```javascript
// Before implementing PixiJS features
1. resolve-library-id: "PixiJS"
2. get-library-docs: "/pixijs/pixijs" topic: "sprites"

// Before audio implementation
1. resolve-library-id: "Howler.js"
2. get-library-docs: "/goldfire/howler.js" topic: "spatial audio"
```

## Development Workflow Rules

### Pre-Implementation Checklist
1. ✅ Use Context7 to get latest library documentation
2. ✅ Web search for current implementation patterns
3. ✅ Plan with Sequential Thinking if complex
4. ✅ Create Task Manager tasks for tracking
5. ✅ Implement with hot reloading enabled
6. ✅ Write E2E tests alongside implementation
7. ✅ Validate cross-browser compatibility

### Quality Gates
- **No code ships without**:
  - Hot reloading verification
  - Automated E2E test coverage
  - Cross-browser validation
  - Performance benchmarking
  - Accessibility compliance

## Technology Stack Validation
- **Before using any library**: Use Context7 to verify latest version and best practices
- **Hot reloading stack**: Vite + HMR + fast refresh
- **E2E testing stack**: Playwright + GitHub Actions + cross-browser matrix
- **Never assume library behavior** - always verify with latest docs via Context7

## Emergency Protocols
- If Context7 is unavailable: Use web search as fallback
- If web search fails: Document assumption and verify later
- If hot reloading breaks: Fix immediately before continuing development
- If E2E tests fail: Block deployment until resolved


These rules ensure maximum development velocity while maintaining code quality and leveraging the most current information available. 

---
> Source: [mib348/duckhunting](https://github.com/mib348/duckhunting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
