## pine-error-prevention-automation

> 1. **`pine_master.sh`** - Development, versioning, error detection, solutions

# Pine Script Error Prevention Automation Rules

## Core Automation Workflow with Intelligent Script Usage

### 🎯 Primary Scripts (Only Two Needed)
1. **`pine_master.sh`** - Development, versioning, error detection, solutions
2. **`pine_tracker.sh`** - Feature tracking, failure logging, rollback documentation

### 📋 When to Use Scripts (Not Every Time!)

#### Use `pine_master.sh develop` ONLY when:
- Starting a NEW feature/modification
- After SIGNIFICANT changes (not every edit)
- Before taking a break/ending session
- When you explicitly ask for versioning

#### Use `pine_master.sh fix` ONLY when:
- Actual compilation error occurs
- You report a specific problem
- After multiple failed attempts

#### Use `pine_tracker.sh` ONLY when:
- You explicitly mention a new feature to add
- A feature attempt fails completely
- Rolling back to previous version
- Marking feature as complete

### 🚫 DON'T Run Scripts When:
- Making minor edits or tweaks
- In the middle of active coding flow
- Just fixed a simple typo
- You haven't asked for any tracking

### 💡 Intelligent Workflow Pattern

```mermaid
flowchart TD
    A[🎯 1. FEATURE REQUEST<br/>"Add weekly filter to FVG"] --> B[🧠 AI CHOOSES APPROACH<br/>• Track if helpful<br/>• Version if needed<br/>• Start coding]
    B --> C[🔨 2. ACTIVE DEVELOPMENT<br/>AI uses full toolkit<br/>• edit_file, search_replace<br/>• codebase_search, grep_search<br/>• run_terminal_cmd if needed<br/>• Any available tool]
    
    C --> D{3. ERROR OCCURS?}
    D -->|YES| E[🧠 AI SELECTS BEST SOLUTION<br/>• Pattern recognition<br/>• Tool combinations<br/>• Creative problem solving]
    D -->|NO| F{4. MAJOR MILESTONE?}
    
    E --> G{Fixed?}
    G -->|YES| C
    G -->|NO| H[🔄 AI ADAPTS STRATEGY<br/>• Try different tools<br/>• Use scripts if helpful<br/>• Document learning]
    
    F -->|SUCCESS| I[🎯 AI CHOOSES COMPLETION<br/>• Track success if valuable<br/>• Document patterns<br/>• Version if appropriate]
    F -->|FAILURE| H
    F -->|CONTINUE| C
    
    I --> J[📦 SESSION END<br/>AI decides best wrap-up<br/>• Scripts if helpful<br/>• Direct action if better]
    H --> J
    
    style A fill:#e3f2fd
    style B fill:#e8f5e8
    style C fill:#fff3e0
    style E fill:#e8f5e8
    style I fill:#e8f5e8
    style J fill:#e8f5e8
    style H fill:#e3f2fd
```

**Key Phases:**
1. 🎯 **Start**: Version + track ONCE at beginning
2. 🔨 **Code**: Active development with NO script interruptions  
3. 🔧 **Error**: Use scripts only when needed
4. 🎯 **Milestone**: Track significant progress
5. 📦 **End**: Final version at session end

### 🎯 Smart Decision Tree

```mermaid
flowchart TD
    A[🤔 Got an Error?] --> B{New/Unknown Error?}
    B -->|YES| C[🧠 AI AGENT CHOOSES BEST APPROACH<br/>• Query JSON patterns<br/>• Use any available tools<br/>• Apply optimal solution]
    B -->|NO| D[⚡ AI AGENT APPLIES SOLUTION<br/>• Known fix from memory<br/>• Direct code edit<br/>• Whatever tool works best]
    
    C --> E{Fix Worked?}
    D --> E
    
    E -->|YES| F[✅ Keep Coding<br/>AI continues with full toolkit]
    E -->|NO| G[🔄 AI AGENT ADAPTS<br/>• Try different approach<br/>• Use alternative tools<br/>• Scripts as backup option]
    
    H[🚀 Making Progress?] --> I{What Type?}
    I -->|Small Tweaks| J[💨 AI Codes Freely<br/>Full tool flexibility]
    I -->|Major Feature Added| K[📊 Optional: Scripts for tracking]
    I -->|Feature Complete| L[🎯 Optional: Scripts for documentation]
    
    F --> M[🔄 Continue Development<br/>AI uses best tools for the job]
    G --> N[📝 AI Documents & Adapts<br/>Learn from experience]
    J --> M
    K --> M
    L --> M
    
    style A fill:#fff3e0
    style C fill:#e8f5e8
    style D fill:#e8f5e8
    style F fill:#e8f5e8
    style G fill:#e3f2fd
    style H fill:#f3e5f5
    style K fill:#f0f8ff
    style L fill:#f0f8ff
```

**🧠 AI Agent Flexibility Guidelines:**
- 🛠️ **Tool Freedom** → Use any tool that solves the problem best
- 🎯 **Intelligent Selection** → Choose optimal approach for each situation  
- 📊 **Scripts as Enhancement** → Use when helpful, not mandatory
- 🔄 **Adaptive Response** → Switch strategies as needed
- 💡 **Creative Solutions** → Not limited to predefined workflows

### 📊 Script Usage Guidelines

#### High-Value Script Usage:
- **Beginning**: Version once, track feature
- **Errors**: Search solutions, apply fixes
- **Completion**: Document success/failure
- **End**: Final version with summary

#### Avoid Script Overuse:
- NOT after every code change
- NOT for minor syntax fixes
- NOT during rapid iterations
- NOT when in coding flow

### 🔄 Error Handling Priority

1. **First Time Error**: 
   - Run `pine_master.sh fix`
   - Log to solutions if fixed
   
2. **Repeat Error**:
   - Check local memory first
   - Apply known solution
   - No script needed

3. **Persistent Error**:
   - Run `pine_tracker.sh fail`
   - Consider rollback
   - Document approach

### 💾 Database Update Strategy

#### Update Immediately:
- New error type discovered
- Successful fix for complex issue
- Feature completion/failure

#### DON'T Update:
- Every minor edit
- Known issues already in DB
- Simple typo fixes
- Routine code changes

### 🎮 Practical Example Flow

```bash
# You: "Add weekly timeframe filter to FVG"

# 1. START (Scripts needed)
./scripts/pine_tracker.sh add fvg "Weekly timeframe filter"
./scripts/pine_master.sh develop working/fvg/fvg.pine minor

# 2. CODING PHASE (No scripts)
# - Write HTF logic
# - Add filter conditions  
# - Adjust calculations
# - Test edge cases

# 3. ERROR OCCURS (Script needed)
# "Cannot use 'resolution' in request.security"
./scripts/pine_master.sh fix working/fvg/fvg.pine
# Found: Use 'timeframe.period' instead
# Apply fix and continue...

# 4. MORE CODING (No scripts)
# - Implement fix
# - Add more logic
# - Refine feature

# 5. COMPLETE (Scripts needed)
./scripts/pine_tracker.sh complete fvg v0.2.0
./scripts/pine_master.sh develop working/fvg/fvg.pine minor
```

### 🏁 Session Patterns

#### Efficient Session:
```
Start → Track feature → Version once → Code code code → 
Fix errors as needed → Code more → Complete → Final version
(Only 4-5 script calls for entire feature)
```

#### Inefficient Session:
```
Start → Version → Code → Version → Fix → Version → 
Code → Version → etc...
(Too many script calls, disrupts flow)
```

### 🎯 Key Principles

1. **AI Agent has full tool autonomy** - Use whatever works best
2. **Scripts enhance, never constrain** - Guidelines, not rigid rules
3. **Intelligent tool selection** - Choose optimal approach for each situation
4. **Adaptive problem solving** - Switch strategies as needed
5. **Maintain coding momentum** - Don't interrupt flow
6. **Version at milestones** - When it adds value
7. **Track failures** - When learning is valuable
8. **Creative solutions encouraged** - Not limited to predefined workflows

### 📝 Mental Model

Think of scripts as:
- **Checkpoints** in a video game (not every step)
- **Milestones** on a journey (not every mile)
- **Insurance** for your work (not paranoia)

The goal is smooth, efficient development with intelligent safety nets, not constant interruption with script execution.

## 🧠 AI Agent Tool Autonomy

### Core Philosophy
The workflows are **intelligent guidelines** that enhance AI agent capabilities, not rigid constraints. AI agents should:

- **Use any available tool** that solves the problem efficiently
- **Combine tools creatively** for optimal solutions  
- **Switch strategies dynamically** based on context
- **Prioritize effectiveness** over following specific tool patterns
- **Leverage full toolkit** including but not limited to:
  - `edit_file`, `search_replace` for code changes
  - `codebase_search`, `grep_search` for finding patterns
  - `run_terminal_cmd` for system operations
  - `create_diagram` for visualization
  - `web_search` for research
  - Any other available tools as needed

### When Scripts Are Helpful
- **Complex tracking** across multiple files/sessions
- **Batch operations** on many indicators
- **Analytics generation** for insights
- **Standardized logging** for team workflows

### When Direct Tools Are Better  
- **Single file changes** - Direct edit is faster
- **Quick fixes** - No overhead needed
- **Creative solutions** - Custom tool combinations
- **User-specific requests** - Adapt to their preferred workflow

### Flexibility Examples
```bash
# AI Agent might choose:
• Direct file edit for simple version increment
• Script for complex multi-file versioning
• Hybrid approach: direct edit + JSON logging
• Creative combination of multiple tools
```

The system **amplifies** AI agent intelligence, never **constrains** it.

---
> Source: [tradesdontlie/pinescript-development-workspace](https://github.com/tradesdontlie/pinescript-development-workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
