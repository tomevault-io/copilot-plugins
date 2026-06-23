## cross-reference-guide

> Use this file for finding rules in project

# Cross-Reference Guide - Comprehensive Rule Navigation

## 🗺️ RULE ECOSYSTEM OVERVIEW

Your `.cursor` rules are organized into **3 tiers** with **cross-cutting concerns**:

```
🏢 PROJECT-WIDE (.cursor/rules/)
├── 🔥 sprint-lessons-learned.mdc         # Master reference - Sprint 10 failures
├── ⚡ quick-troubleshooting-index.mdc    # Emergency rapid resolution  
├── 🧪 sprint10-testing-patterns.mdc     # Prevention through testing
├── 🔧 mcp-browser-tools-setup.mdc       # 3-component MCP protocol
├── 📋 sprint-planning.mdc               # Sprint workflow & planning
├── 🌐 use-mcp-servers.mdc               # MCP server integration
├── 🐛 debugging.mdc                     # General debugging protocols
├── 💬 commentsoverwrite.mdc             # Documentation standards
└── 📖 cross-reference-guide.mdc         # This navigation guide

🎨 FRONTEND (frontend/.cursor/rules/)
├── 📋 frontend-development-index.mdc    # Master index - START HERE for frontend work
├── 💧 nextjs-hydration-prevention.mdc   # Server/client consistency
├── 🔄 react-query-api-integration.mdc   # Server state management
├── 🏗️ component-architecture-patterns.mdc # Component design & composition patterns
├── 🎨 ux-workflow-patterns.mdc          # User experience & workflow patterns
└── ⚡ nextjs-performance-optimization.mdc # Next.js performance optimization

⚙️ BACKEND (backend/.cursor/rules/)
├── 📋 backend-development-index.mdc     # Master index - START HERE for backend work
├── 🔗 fastapi-dependency-injection.mdc  # Circular import prevention
├── 🏗️ fastapi-microservice-patterns.mdc # Complete service architecture patterns
├── 🤖 ml-service-integration.mdc         # GPU-optimized ML service patterns
└── 🔌 api-design-patterns.mdc           # RESTful API design & data modeling
```

## 🎯 SCENARIO-BASED NAVIGATION

### 🚨 "I Have an Emergency Issue"
**START HERE:** `.cursor/rules/quick-troubleshooting-index.mdc`
- **30-second fixes** for common Sprint 10 problems
- **Diagnostic flowchart** to route you to detailed solutions
- **Severity levels** to prioritize fixes

**Then go to specific rule:**
- Hydration errors → `frontend/.cursor/rules/nextjs-hydration-prevention.mdc`
- Import errors → `backend/.cursor/rules/fastapi-dependency-injection.mdc`
- MCP failures → `.cursor/rules/mcp-browser-tools-setup.mdc`

### 🛠️ "I'm Starting New Development Work"
**1. Check Sprint Context:** `.cursor/rules/sprint-lessons-learned.mdc`
   - Review applicable lessons from Sprint 10 failures
   - Check mandatory pre-work checklists

**2. Choose Development Track:**
   - **Frontend work (any)** → `frontend/.cursor/rules/frontend-development-index.mdc` (comprehensive guide)
   - **Frontend components** → `frontend/.cursor/rules/component-architecture-patterns.mdc`
   - **Frontend UX** → `frontend/.cursor/rules/ux-workflow-patterns.mdc`
   - **Frontend performance** → `frontend/.cursor/rules/nextjs-performance-optimization.mdc`
   - **Hydration issues** → `frontend/.cursor/rules/nextjs-hydration-prevention.mdc`
   - **Backend work (any)** → `backend/.cursor/rules/backend-development-index.mdc` (comprehensive guide)
   - **Backend API work** → `backend/.cursor/rules/api-design-patterns.mdc`
   - **Backend ML work** → `backend/.cursor/rules/ml-service-integration.mdc`
   - **Backend services** → `backend/.cursor/rules/fastapi-microservice-patterns.mdc`
   - **Import issues** → `backend/.cursor/rules/fastapi-dependency-injection.mdc`
   - **MCP integration** → `.cursor/rules/mcp-browser-tools-setup.mdc`

**3. Set Up Testing:** `.cursor/rules/sprint10-testing-patterns.mdc`
   - Pre-commit hooks for your development area
   - Continuous validation patterns

### 🧪 "I Want to Prevent Issues Through Testing"
**START HERE:** `.cursor/rules/sprint10-testing-patterns.mdc`
- **Component-level tests** for hydration safety
- **Import validation** for backend architecture
- **MCP integration tests** for tool reliability
- **CI/CD workflows** for continuous prevention

**Related rules:**
- Testing patterns reference specific prevention rules
- Each test maps to a Sprint 10 lesson learned

### 📋 "I'm Planning a Sprint"
**START HERE:** `.cursor/rules/sprint-planning.mdc`
- Sprint workflow integration with MCP tools
- PRD generation and documentation standards

**Integration points:**
- Health check: `.cursor/rules/sprint10-testing-patterns.mdc`
- MCP setup: `.cursor/rules/mcp-browser-tools-setup.mdc`
- Lessons: `.cursor/rules/sprint-lessons-learned.mdc`

### 🔧 "I Need to Set Up MCP Tools"
**START HERE:** `.cursor/rules/mcp-browser-tools-setup.mdc`
- 3-component verification protocol
- Troubleshooting for common setup issues

**Integration with:**
- Testing: `.cursor/rules/sprint10-testing-patterns.mdc` (MCP testing section)
- Sprint work: `.cursor/rules/sprint-planning.mdc` (MCP-driven workflow)

## 📊 RULE DEPENDENCY MAP

### Core Dependencies (Must Read First)
```
sprint-lessons-learned.mdc (MASTER reference)
    ├── Referenced by: quick-troubleshooting-index.mdc
    ├── Referenced by: sprint10-testing-patterns.mdc
    ├── Referenced by: nextjs-hydration-prevention.mdc
    ├── Referenced by: react-query-api-integration.mdc
    └── Referenced by: fastapi-dependency-injection.mdc
```

### Technical Implementation Chain
```
1. sprint-lessons-learned.mdc (What went wrong?)
2. [specific technical rule] (How to do it right?)
3. sprint10-testing-patterns.mdc (How to verify it works?)
4. quick-troubleshooting-index.mdc (How to fix it when it breaks?)
```

### MCP Integration Chain
```
1. mcp-browser-tools-setup.mdc (Setup & verification)
2. use-mcp-servers.mdc (Integration patterns)
3. sprint-planning.mdc (Workflow integration)
4. sprint10-testing-patterns.mdc (MCP testing)
```

## 🔄 CROSS-CUTTING PATTERNS

### The "Sprint 10 Prevention Protocol"
Multiple rules implement this pattern:

**1. Identify the Problem** (from Sprint 10 lessons)
**2. Implement the Solution** (from technical rules)
**3. Test the Prevention** (from testing patterns)
**4. Enable Quick Recovery** (from troubleshooting index)

### The "3-Layer Defense Strategy"
Each critical issue has **3 layers of protection**:

#### Example: Hydration Errors
- **Layer 1:** Prevention rules (`nextjs-hydration-prevention.mdc`)
- **Layer 2:** Testing patterns (`sprint10-testing-patterns.mdc`)
- **Layer 3:** Emergency recovery (`quick-troubleshooting-index.mdc`)

#### Example: Circular Imports  
- **Layer 1:** Architecture rules (`fastapi-dependency-injection.mdc`)
- **Layer 2:** Import testing (`sprint10-testing-patterns.mdc`)
- **Layer 3:** Recovery protocol (`quick-troubleshooting-index.mdc`)

## 🎯 RULE UPDATE PROTOCOL

### When to Update Rules
- **New Sprint 10-type failure** → Update `sprint-lessons-learned.mdc`
- **New quick fix discovered** → Update `quick-troubleshooting-index.mdc`
- **New testing pattern** → Update `sprint10-testing-patterns.mdc`
- **New MCP setup issue** → Update `mcp-browser-tools-setup.mdc`

### How to Maintain Cross-References
When updating any rule, check these files for cross-references:
1. **This file** (`cross-reference-guide.mdc`) - Update navigation paths
2. **Main cursor rules** (`.cursor/cursor_rules.md`) - Update quick reference links
3. **Sprint lessons** (`sprint-lessons-learned.mdc`) - Add new failure patterns
4. **Troubleshooting index** (`quick-troubleshooting-index.mdc`) - Add new symptoms/fixes

## 🚀 ONBOARDING NEW TEAM MEMBERS

### Essential Reading Order for New Developers
1. **Start:** `.cursor/cursor_rules.md` (overview & quick patterns)
2. **Context:** `.cursor/rules/sprint-lessons-learned.mdc` (what not to do)
3. **Navigation:** `.cursor/rules/cross-reference-guide.mdc` (this file)
4. **Emergency:** `.cursor/rules/quick-troubleshooting-index.mdc` (when things break)
5. **Technical:** Choose frontend/backend specific rules
6. **Testing:** `.cursor/rules/sprint10-testing-patterns.mdc` (prevention)

### Quick Competency Check
New team members should be able to:
- [ ] Navigate to the right rule for their work area
- [ ] Apply Sprint 10 prevention patterns
- [ ] Use the troubleshooting index for common issues
- [ ] Set up MCP tools using the 3-component protocol
- [ ] Run testing patterns to validate their changes

## 📈 SUCCESS METRICS

### Rule Effectiveness Indicators
- **Zero hydration errors** in development console
- **Clean backend startup** without import warnings  
- **First-try MCP success** rate >90%
- **Testing protocol adoption** in daily workflow
- **Quick issue resolution** using troubleshooting index

### Knowledge Transfer Success
- New team members resolve issues **without** repeating Sprint 10 debugging sessions
- **Consistent implementation** of prevention patterns across features
- **Proactive testing** prevents issues rather than reactive debugging

---

**🧭 Navigation Quick Keys:**
- 🚨 Emergency → `quick-troubleshooting-index.mdc`
- 🔥 Sprint 10 lessons → `sprint-lessons-learned.mdc`  
- 🎨 Frontend → `frontend/.cursor/rules/`
- ⚙️ Backend → `backend/.cursor/rules/`
- 🔧 MCP → `mcp-browser-tools-setup.mdc`
- 🧪 Testing → `sprint10-testing-patterns.mdc`

*This cross-reference system ensures no Sprint 10 failure pattern is ever repeated.*

---
> Source: [rm2thaddeus/Pixel_Detective](https://github.com/rm2thaddeus/Pixel_Detective) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
