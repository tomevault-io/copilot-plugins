## order-your-plan

> Prompt Expansion / Plan Document Generator: User provides a simple requirement → Domain research → Interactive refinement → Creative expansion → Outputs an ultra-detailed execution plan document (.md). Based on the Milk Tea Ordering + Hyper-Concurrent Questioning methodology. A Plan Mode alternative for any AI Agent.


# Prompt Expansion / Plan Document Generator

> **In one sentence**: Forges a user's fragment of inspiration, through research, interaction, and creative expansion, into an ultra-detailed plan document that an AI Agent can directly execute.

## What problem does this skill solve?

A user's thinking is jumpy and intuitive — "make a chaos pendulum demo page", "build a data visualization tool", "write an automatic backup script".

What an AI Agent needs is precise and structured — tech stack, file structure, feature list, UI specifications, acceptance criteria, prohibitions.

**This skill is that bridge.**

| Input | Output |
|------|------|
| "Make a chaos pendulum demo page" | A 3000+ word detailed execution plan .md |
| "I want to make a data dashboard" | A complete document including architecture, tech selection, UI specifications, acceptance criteria |
| "Help me write an automation script" | Feature list, error handling strategy, input/output specifications, test plan |

### Differences from other skills

| Skill | What it does | Output | Characteristics |
|------|--------|------|------|
| **prompt-refiner** | Vague instruction → Clarify → Execute | Direct execution result | Instant task, execute immediately after clarification |
| **claude-code-goal-writer** | Rough input → Goal document | /goal command + goal.md | Designed specifically for Claude Code |
| **ai-dev-flow** | Vague idea → Research → Routing → Delivery | Final product | Complete development pipeline |
| **milk-tea-ordering** | Requirement guidance → Preference learning | Confirmation form → Execution | Includes preference memory and combo system |
| **This skill** | Simple inspiration → Research → Refine → Creative expansion | **Pure plan document .md** | **Planning only, no execution** |

**Core difference**: This skill does not execute any code or perform any implementation. Its sole output is an **extremely detailed plan document** that can be handed to any AI Agent (Claude Code, Cursor, Windsurf, Copilot, Hermes itself) for execution.

---

## Core Methodology: Three-Layer Alchemy

```
User's fragment of inspiration (rough ore)
         │
         ▼
┌─────────────────┐
│  Layer 1: Research  │  ← Hyper-concurrent questioning: What is this thing? How to do it?
│  (Research)     │     Three-pronged parallel drilling, build domain knowledge
└────────┬────────┘
         ▼
┌─────────────────┐
│  Layer 2: Refine    │  ← Milk tea ordering: Ask step by step what the user wants
│  (Refine)       │     One question at a time, interactive options
└────────┬────────┘
         ▼
┌─────────────────┐
│  Layer 3: Forge     │  ← Creative expansion: AI proactively supplements what the user didn't think of
│  (Forge)        │     Smelt all information into an ultra-detailed plan
└────────┬────────┘
         ▼
An extremely detailed execution plan document (.md)
```

---

## Phase 0: Reception and Parsing

After receiving the user's requirement, first perform a quick parse:

### 0.1 Identify Requirement Type

| User says | Identified as | Initial domain |
|--------|--------|---------|
| "Make a website/web page/page" | web-project | Frontend development |
| "Make a tool/script/program" | software-tool | Software development |
| "Make a visualization/dashboard" | data-viz | Data visualization |
| "Make a simulation/emulation" | simulation | Scientific computing |
| "Make a system/platform" | system-platform | System architecture |
| "Write a paper/document" | document | Academic writing |
| "Make a hardware/circuit" | hardware | Embedded/Hardware |
| "Make a game" | game | Game development |
| "Help me plan" | planning | Project management |
| Other | general | General |

### 0.2 Identify Vagueness

Evaluate the vagueness of user input across 3 dimensions:

| Dimension | Clear signal | Vague signal |
|------|---------|---------|
| **Goal** | "Make a blog with login functionality" | "Make a website" |
| **Constraints** | "Use React, deploy to Vercel" | "Make it look nice" |
| **Scope** | "3 pages: Home, Articles, About" | "Something roughly like that" |

**Vagueness Score** (0-3, higher = more vague):
- 0: Goal + Constraints + Scope all clear → Skip research, go directly to refinement confirmation
- 1: Missing 1 item → Light research + quick refinement
- 2: Missing 2 items → Full research + standard refinement
- 3: All vague → Deep research + complete refinement + creative expansion

### 0.3 Startup Message

```
OK, I'll help you turn this idea into a detailed execution plan.

You said "[user's original words]", let me first research this domain, then confirm details with you step by step.
```

---

## Phase 1: Domain Research (Hyper-Concurrent Questioning Engine)

> **Core principle**: Before asking the user, let the AI itself figure out two things first —
> 1. What this thing **is** and **how to do it** (technical knowledge)
> 2. How to **plan** this thing and **which dimensions need user confirmation** (planning knowledge)
>
> You cannot ask the user questions you don't understand, nor ask questions the user doesn't need to be asked.

### 1.1 Research Strategy Selection

| Vagueness | Research depth | Method |
|--------|---------|------|
| 0-1 | Shallow (self-analysis) | Current agent analyzes directly, no sub-agents dispatched |
| 2 | Medium (single-track research) | 1 sub-agent does comprehensive research |
| 3 | Deep (three-track concurrent) | 3 sub-agents drill in parallel |

### 1.2 Two Dimensions of Research (Both Required)

**Dimension A: Technical Knowledge** — What this thing is and how to do it
- Core functionality and essence
- Technical solution comparison
- Implementation roadmap
- Common pitfalls

**Dimension B: Planning Knowledge** — How to plan this thing and what needs user decisions
- What dimensions typically need to be determined before starting this type of project?
- Which dimensions **must be confirmed with the user**? (Different users choose very differently)
- Which dimensions **can the AI decide by convention**? (Strong industry consensus, no need to ask)
- Which dimensions are **optional**? (Depends on project complexity)
- For each dimension that needs confirmation, what are the reasonable options?

**Dimension B is the key innovation**: It determines what Phase 2 asks and doesn't ask.
Without this research, you might ask the user things they don't need to be asked (wasting time),
or miss things the user should be asked about (plan doesn't meet requirements).

### 1.3 Three-Track Concurrent Drilling Template (Deep Research)

Use `delegate_task` to dispatch 3 research Agents in parallel:

**Agent A (Pioneer — Technical Knowledge + Planning Dimensions)**:
```
You are a senior architect and project manager in the {domain}. The user wants to "[user's original words]".

Please complete two parts of research:

【Part 1: Technical Knowledge】
1. What is the essence of this thing? What are the core functionalities?
2. What are the industry best practices? What mature technical solutions exist?
3. Technical selection recommendations (list pros/cons comparison of 2-3 solutions)
4. Common pitfalls and things to watch out for
5. Recommended tech stack and tools

【Part 2: Planning Dimension Analysis】(This is the key!)
1. For this type of project, what dimensions typically need to be determined before starting?
   (e.g., tech stack, design style, data source, deployment method, security level...)
2. For each dimension, classify as:
   - 【Must Confirm】Different users/scenarios choose very differently, must ask
   - 【AI Decides】Strong industry consensus, AI can decide by convention
   - 【Optional】Depends on project complexity, can skip for simple projects
3. For each 【Must Confirm】 dimension, provide 3-4 reasonable options
   (Described in everyday language, no technical jargon)

Output requirements: Specific data, comparison tables. Target output 4000+ words.
```

**Agent B (Assistant 1 — Design and Experience)**:
```
You are a UI/UX design expert in the {domain}. The user wants to "[user's original words]".

Please research in detail:
1. Common design patterns and styles for this type of product
2. Visual design direction (color scheme, layout, animations)
3. User experience best practices
4. Key design choices that need user confirmation (style preferences, color tendencies, etc.)
5. Conventions that designers typically decide directly without asking the user

Output requirements: Specific design suggestions, color schemes. Target output 2000+ words.
```

**Agent C (Assistant 2 — Implementation and Verification)**:
```
You are a senior engineer in the {domain}. The user wants to "[user's original words]".

Please research in detail:
1. Technical roadmap for implementing this type of project (steps from zero to completion)
2. Project structure and file organization suggestions
3. Testing and verification strategy
4. Performance optimization key points
5. Conventions that engineers typically decide directly without asking the user

Output requirements: Specific directory structure, code architecture, verification commands. Target output 2000+ words.
```

### 1.4 Research Result Integration

After three-track research is complete, consolidate into two **internal documents** (not shown to user, serves as knowledge base for Phase 2):

**Document 1: Domain Knowledge Summary**
```markdown
## Technical Knowledge
[Core concepts, solution comparison, implementation roadmap, key pitfalls]
```

**Document 2: Planning Dimension Checklist** (Core! Determines what Phase 2 asks)
```markdown
## Planning Dimension Analysis

### 【Must Confirm】Dimensions (Phase 2 must ask the user)
| Dimension | Why it must be asked | Options (everyday language) |
|------|------------|-------------------|
| ... | ... | A. ... / B. ... / C. ... / D. You recommend |

### 【AI Decides】Dimensions (Phase 2 skips, AI decides directly)
| Dimension | AI's decision | Reason |
|------|---------|------|
| ... | ... | Industry convention/best practice |

### 【Optional】Dimensions (Depends on project complexity whether to ask)
| Dimension | When to involve | When to skip |
|------|---------|---------|
| ... | Complex projects/user explicitly requests | Simple projects |
```

### 1.5 Shallow Research (Self-Analysis)

For common requirements (login page, calculator, TODO app, etc.), no need to dispatch sub-agents.
The current agent completes the analysis internally:

```
Internal thought process:
1. What is the core functionality of this thing? (Dimension A)
2. What dimensions need to be determined? Which must be asked to the user? Which do I decide directly? (Dimension B)
3. What are the tech stack options? Pros and cons of each?
4. What are the common design patterns?
5. What are the common pitfalls?
```

Even for shallow research, **Dimension B analysis cannot be skipped**.
Even in domains you're very familiar with, think clearly: "What questions am I going to ask the user this time?"

---

## Phase 2: Interactive Refinement (Milk Tea Ordering Engine)

> **Core principle**: Ask only one question at a time. Guide with options, don't torture the user with open-ended questions.
> **What to ask**: Determined by Phase 1's "Planning Dimension Checklist", not pre-written.

### 2.1 Iron Rules of Ordering

**⚠️ This is the most important rule of this skill. Violation means failure:**

1. **Ask only one question at a time** — Absolutely never list 3-5 questions at once for the user to take an "exam"
2. **Use the `clarify` tool's `choices` parameter** — Provide clickable option tabs
3. **Have a default recommendation** — Every question has a "Recommended ✨" option
4. **Options in everyday language** — No technical jargon, use analogies and metaphors
5. **Allow "You recommend"** — When the user doesn't want to choose, AI decides automatically
6. **Follow-up questions no more than 2 rounds** — After choosing an option, at most 1 related sub-question

### 2.2 Dynamic Question Generation (Core Logic)

**Phase 2 questions are not pre-written, but dynamically generated based on Phase 1 research results.**

Generation logic:

```
Phase 1 output: "Planning Dimension Checklist"
    │
    ├── 【Must Confirm】dimensions → Phase 2 must ask user one by one
    │     Options for each dimension come from Phase 1 research
    │     Options described in everyday language (already required during Phase 1 research)
    │
    ├── 【AI Decides】dimensions → Phase 2 skips, writes AI's decision directly into the plan
    │     Briefly note "AI decided" and reason in confirmation card
    │
    └── 【Optional】dimensions → Dynamically judge whether to ask based on previous choices
          If user chose "Product-level" refinement → May ask a few more dimensions
          If user chose "Prototype-level" → Skip most optional dimensions
```

### 2.3 Universal Startup Dimensions (Applicable to Almost All Projects)

Besides domain-specific dimensions from Phase 1 research, the following dimensions **apply to almost all projects** and can be used as the first or first few questions:

**Dimension A: Core Goal (Always ask first)**
```
clarify(
    question="🎯 What is the core purpose of this thing?",
    choices=[
        "Teaching demo — To demonstrate principles, doesn't need to be too refined",
        "Physical simulation — Should be as realistic and accurate as possible",
        "Interactive experience — Focus on user interaction and exploration",
        "Data presentation — Focus on visualizing data",
        "Other — I'll describe it"
    ]
)
```
Note: Options will be adjusted based on domain. The above is a generic version; in practice, get more fitting options from Phase 1 research results.

**Dimension B: Refinement Level (Applicable to Almost All Projects)**
```
clarify(
    question="🔬 What level of refinement do you want?",
    choices=[
        "Prototype — Just enough to understand the idea, quick validation (Recommended ✨)",
        "Demo — Polished appearance, can be shown to others",
        "Product-level — Close to actual product completion",
        "You recommend — Automatically choose based on purpose"
    ]
)
```

**Dimension C: AI Involvement Level (Determines depth of subsequent questions)**
```
clarify(
    question="🤖 How much do you want the AI to decide for you?",
    choices=[
        "Full delegation — You decide, I just look at the result (Recommended ✨)",
        "Key confirmations — Ask me the important stuff, you handle the details",
        "Step-by-step confirmation — Let me choose every dimension",
    ]
)
```
This dimension is critical:
- Choose "Full delegation" → Skip most subsequent questions, AI uses neutral approach
- Choose "Key confirmations" → Only ask 【Must Confirm】dimensions
- Choose "Step-by-step confirmation" → Ask all dimensions, including optional ones

### 2.4 Dynamic Question Sequence

After completing universal startup dimensions, ask one by one based on Phase 1's "Planning Dimension Checklist":

```
For each 【Must Confirm】dimension:
    Ask with clarify (options from Phase 1 research)
    │
    ├── User chose a specific option → Record, proceed to next dimension
    ├── User chose "You recommend" → AI uses neutral approach, skip follow-up
    └── Option triggers follow-up condition → Ask 1 follow-up (at most)
```

**Question ordering** principles:
1. Dimensions affecting architecture/tech stack come first (determines scope of all subsequent choices)
2. Dimensions affecting visual/experience come next
3. Detail/optimization dimensions come last
4. Dimensions with dependencies are asked in dependency order (e.g., "which API" only asked after "API data source" is selected)

### 2.5 Follow-up Strategy

After the user selects an option, decide whether to follow up based on the selection:

**Follow-up conditions** (any one triggers follow-up):
- The option itself contains multiple possibilities (e.g., "tech cool" → Follow up with "glow style")
- The option points to an external dependency (e.g., "API interface" → Follow up with "which API")
- The option is major and irreversible (e.g., "Use React" → Follow up with "Do you need TypeScript?")

**No follow-up conditions**:
- User chose "You recommend"
- Option is already specific enough
- Follow-up has minimal impact on final result

**Follow-up rules**:
- At most 1 follow-up per dimension
- Total follow-ups no more than 3 (avoid user fatigue)
- Never follow up when user chooses "You recommend"

### 2.6 Neutral Approach Principle

**When the user chooses "You recommend" or AI involvement level is "Full delegation", use the neutral approach:**

Neutral approach selection logic:
- Tech stack → Most mature, best documented, largest community
- Design style → Clean and simple (widest appeal, least likely to go wrong)
- Refinement level → Demo level (more refined than prototype, but not over-engineered)
- Interaction style → Basic interaction (sufficient but not complex)
- Data source → Simulated data (no external service dependency)

The essence of the neutral approach: **A stable solution that is neither aggressive nor outdated**.
Its goal: Even if the user makes no choices at all, the resulting plan won't go badly wrong.

---

## Phase 3: Creative Expansion (AI Proactive Supplementation)

> **Core principle**: What the user didn't think of, the AI should think of. Not passively answering questions, but proactively making suggestions.

### 3.1 Three Directions of Creative Expansion

#### Direction 1: Feature Expansion

User says "make a login page", the AI should think of:
```
💡 What the user didn't mention but might need:
- Forgot password flow
- Remember password feature
- Input validation (email format, password strength)
- Loading states and error messages
- Accessibility (keyboard navigation, screen reader)
```

#### Direction 2: Experience Optimization

```
💡 Details that can improve the experience:
- Real-time validation feedback during input
- Button hover/active states
- Skeleton screen during page load
- Success/failure animation feedback
- Dark/light theme switching
```

#### Direction 3: Technical Safeguards

```
💡 Things to consider for long-term operation:
- Input security (XSS prevention, injection prevention)
- Error handling (network errors, timeout, service unavailable)
- Performance optimization (large data volumes, high-frequency operations)
- Code maintainability (modularization, comments, naming)
```

### 3.2 Presentation of Creative Expansion

**After the user completes all selections**, the AI proactively presents creative suggestions:

```
💡 I also thought of some things you might need. You can choose to keep or remove:

✅ Auto-keep (beneficial for any project):
- Input validation and error handling
- Responsive layout adaptation
- Basic accessibility support

🔘 Optional enhancements (check what you need):
- [ ] Dark/light theme switching
- [ ] Animation transition effects
- [ ] Keyboard shortcut support
- [ ] Offline usage capability
- [ ] Multi-language support

🚫 Not recommended (not needed for this project):
- User authentication system (not needed for pure frontend project)
- Database integration (not needed for static pages)
```

Use the `clarify` tool for user confirmation:

```
clarify(
    question="Which of the optional enhancements above would you like to add?",
    choices=[
        "Add all — Give me the most complete version",
        "Keep core only — Minimum but sufficient",
        "Add animations — Make the page more lively",
        "You decide — Automatically choose based on project type"
    ]
)
```

### 3.3 Constraints for Wild Ideas

The user's ideas may be wild and imaginative. The AI's responsibility is:

1. **Respect creativity**: Don't say "this won't work" or "can't be done"
2. **Find feasible paths**: Transform wild ideas into implementable technical solutions
3. **Practical suggestions**: If an idea is technically too complex, offer alternatives

```
User: "I want to make a 3D data visualization controlled by gestures"

AI's response:
"Interesting idea! Let's do it in two steps:
Step 1: First make a 3D data visualization (using Three.js, mature technology)
Step 2: Add gesture control (using MediaPipe Hands, natively supported in browsers)

If you don't want to depend on a camera, you can also simulate gestures with a mouse —
drag to rotate, scroll to zoom, double-click to reset. The experience is also great."
```

---

## Phase 4: Solution Confirmation

> **Core principle**: Before generating the final document, let the user confirm all selections.

### 4.1 Confirmation Card

```
📋 Solution Confirmation:

┌──────────────────────────────────┐
│ 🎯 Project: Chaos Pendulum Demo Page    │
│                                  │
│ Core goal: Teaching demo                 │
│ Refinement level: Demo                   │
│ Visual style: Clean and simple           │
│ Technical solution: Pure frontend HTML+CSS+JS │
│ Interaction style: Basic interaction (draggable parameters) │
│ Data source: Simulated data              │
│ Device adaptation: Fully responsive      │
│                                  │
│ ✅ Included:                      │
│  · Chaos pendulum physics simulation     │
│  · Parameter adjustment panel            │
│  · Trajectory visualization              │
│  · Input validation + error handling     │
│                                  │
│ 💡 Creative enhancements:        │
│  · Animation transition effects          │
│  · Dark/light theme switching            │
│                                  │
│ 📁 Output file: Execution Plan.md        │
└──────────────────────────────────┘

Confirm to generate plan? Or tell me what to modify~
```

### 4.2 Post-confirmation Handling

- User says "confirm"/"start"/"ok" → Proceed to Phase 5
- User says "modify XX" → Return to the corresponding dimension for re-selection
- User says "cancel" → Terminate the process

---

## Phase 5: Plan Document Forging (Core Output)

> **This is the sole output of this skill. An extremely detailed execution plan document.**

### 5.1 Document Structure Template

The generated plan document **must** contain the following sections (in this order):

```markdown
# [Project Name] — Execution Plan

## 1. Project Overview

### 1.1 Project Goal
[One sentence describing what this project aims to achieve]

### 1.2 Project Background
[Why this is being done, what problem it solves, what users it targets]

### 1.3 Core Value
[What the user gets after this project is completed]

---

## 2. Technical Specifications

### 2.1 Tech Stack
| Layer | Technology | Version/Notes | Reason for Selection |
|------|------|----------|---------|
| Frontend framework | ... | ... | ... |
| Styling | ... | ... | ... |
| Interaction | ... | ... | ... |
| Build tool | ... | ... | ... |

### 2.2 Project Structure
```
project-name/
├── index.html          ← Entry file
├── css/
│   ├── reset.css       ← Style reset
│   ├── variables.css   ← CSS variables (theme colors, spacing, etc.)
│   ├── layout.css      ← Layout styles
│   ├── components.css  ← Component styles
│   └── animations.css  ← Animation styles
├── js/
│   ├── app.js          ← Main entry
│   ├── core/           ← Core logic
│   │   ├── ...
│   │   └── ...
│   ├── ui/             ← UI interaction
│   │   ├── ...
│   │   └── ...
│   └── utils/          ← Utility functions
│       ├── ...
│       └── ...
├── assets/             ← Static resources
│   └── ...
└── README.md           ← Usage instructions
```

### 2.3 File Dependency Relationship
[Describe which files depend on which files, loading order]

---

## 3. Feature Design

### 3.1 Feature List

#### Core Features (Must Have)
| ID | Feature | Description | Acceptance Criteria |
|------|------|------|---------|
| F01 | ... | ... | ... |
| F02 | ... | ... | ... |

#### Enhanced Features (Should Have)
| ID | Feature | Description | Acceptance Criteria |
|------|------|------|---------|
| F03 | ... | ... | ... |

#### Optional Features (Nice to Have)
| ID | Feature | Description | Acceptance Criteria |
|------|------|------|---------|
| F04 | ... | ... | ... |

### 3.2 Feature Flow Diagram
[Describe core feature execution flow in text]

```
User action → System processing → Output result
    │           │          │
    ▼           ▼          ▼
  Step 1      Step 2      Step 3
```

### 3.3 Data Structure Design
[Define core data structures using pseudocode or JSON Schema]

```javascript
// Example data structure
{
  "key": "value",
  "nested": {
    "field": "type"
  }
}
```

---

## 4. UI/UX Design Specifications

### 4.1 Design Language
- **Overall style**: [Clean and simple / Tech cool / Professional business / ...]
- **Color scheme**:
  - Primary: #[hex] — [Usage description]
  - Secondary: #[hex] — [Usage description]
  - Background: #[hex]
  - Text: #[hex]
  - Accent: #[hex]
- **Font**: [Font stack]
- **Border radius**: [Size]
- **Shadow**: [Style]
- **Spacing system**: [4px/8px base unit]

### 4.2 Layout Design
[Describe page layout precisely in text]

```
┌─────────────────────────────────┐
│  Header: Title + Navigation             │
├─────────────────────────────────┤
│  ┌──────────┐  ┌──────────────┐ │
│  │  Sidebar   │  │  Main Content     │ │
│  │  Parameter │  │  Visualization    │ │
│  │  Panel     │  │  Area             │ │
│  └──────────┘  └──────────────┘ │
├─────────────────────────────────┤
│  Footer: Status info + Control buttons │
└─────────────────────────────────┘
```

### 4.3 Component List
| Component | Style Description | States | Interaction Behavior |
|------|---------|------|---------|
| Button | ... | Default/hover/active/disabled | Click feedback |
| Input field | ... | Empty/has value/error/disabled | Input validation |
| ... | ... | ... | ... |

### 4.4 Animation Specifications
| Element | Animation Type | Duration | Easing Function | Trigger Condition |
|------|---------|------|---------|---------|
| ... | ... | ... | ... | ... |

### 4.5 Responsive Breakpoints
| Breakpoint | Width | Layout Adjustment |
|------|------|---------|
| Mobile | < 768px | ... |
| Tablet | 768-1024px | ... |
| Desktop | > 1024px | ... |

---

## 5. Implementation Guide

### 5.1 Implementation Order
[Implementation steps arranged by dependency]

```
Step 1: Basic structure (HTML skeleton + CSS reset + basic layout)
  ↓
Step 2: Core logic ([specific feature modules])
  ↓
Step 3: UI components ([specific component list])
  ↓
Step 4: Interaction features ([specific interactions])
  ↓
Step 5: Enhanced features (animations, theme switching, etc.)
  ↓
Step 6: Polish and optimize (responsive, performance, accessibility)
```

### 5.2 Key Implementation Details
[For each core feature, provide specific technical implementation hints]

#### Feature F01: [Feature Name]
```
Implementation key points:
1. [Specific technical approach]
2. [Key algorithm/formula]
3. [Edge cases to watch out for]

Reference code structure:
function functionName(params) {
  // 1. Input validation
  // 2. Core processing
  // 3. Output result
}
```

### 5.3 Third-party Dependencies (if needed)
| Library | Version | Purpose | CDN Link |
|------|------|------|---------|
| ... | ... | ... | ... |

---

## 6. Security and Quality

### 6.1 Security Measures
- [ ] Input validation: [Specific rules]
- [ ] XSS protection: [Specific measures]
- [ ] Error handling: [Specific strategy]

### 6.2 Performance Requirements
- [ ] First screen load: < [X] seconds
- [ ] Interaction response: < [X]ms
- [ ] Memory usage: < [X]MB

### 6.3 Code Quality Standards
- [ ] Naming conventions: [Specific conventions]
- [ ] Comment requirements: [Specific requirements]
- [ ] Modularity level: [Specific requirements]

---

## 7. Acceptance Criteria

### 7.1 Functional Acceptance
| ID | Acceptance Item | Acceptance Method | Pass Standard |
|------|--------|---------|---------|
| V01 | ... | Run xxx command / Perform xxx operation | ... |
| V02 | ... | ... | ... |

### 7.2 Visual Acceptance
| ID | Acceptance Item | Acceptance Method | Pass Standard |
|------|--------|---------|---------|
| V03 | Layout correct | Browser screenshot | ... |
| V04 | Responsive | Resize browser window | ... |

### 7.3 Acceptance Script
```bash
# Automated acceptance commands
# 1. Syntax check
# 2. Functional test
# 3. Visual check
```

---

## 8. Prohibitions

The following behaviors are **absolutely not allowed**:
- ❌ [Specific prohibition 1]
- ❌ [Specific prohibition 2]
- ❌ [Specific prohibition 3]

---

## 9. Expansion Directions (Future)

The following features are not included in the current version but can be expanded in the future:
- 🔮 [Expansion direction 1]
- 🔮 [Expansion direction 2]
- 🔮 [Expansion direction 3]
```

### 5.2 Document Quality Standards

The generated plan document must meet:

| Dimension | Standard | Verification Method |
|------|------|---------|
| **Completeness** | All 9 sections have substantive content | No empty sections |
| **Specificity** | Every feature has clear acceptance criteria | No vague descriptions like "make it look nice" |
| **Executability** | AI Agent can start writing code immediately after reading | No additional questions needed |
| **Consistency** | No contradictions front to back | Tech stack consistent throughout |
| **Detail level** | 3000-8000 words | Word count check |

### 5.3 Document Output

```
📁 Output location: D:\Hermes_Output\plans\
📄 File name: plan_[project_abbreviation]_[YYYYMMDD].md
```

---

## Phase 6: Delivery and Follow-up

### 6.1 Delivery Message

```
✅ Execution plan has been generated!

📄 File: D:\Hermes_Output\plans\plan_chaos_pendulum_demo_20260601.md
📊 Document length: Approximately XXXX words

📋 Document contains:
  1. Project Overview — Goal, background, core value
  2. Technical Specifications — Tech stack, project structure, dependencies
  3. Feature Design — Core/enhanced/optional feature list
  4. UI/UX Design — Colors, layout, components, animation specifications
  5. Implementation Guide — Step order, key details, dependencies
  6. Security & Quality — Security measures, performance requirements
  7. Acceptance Criteria — Functional/visual acceptance checklist
  8. Prohibitions — Clear red lines
  9. Expansion Directions — Future possibilities

💬 This plan can be directly handed to any AI Agent for execution.
   Want me to help you execute it? Or need to modify certain parts of the plan?
```

### 6.2 Follow-up Options

Use `clarify` to provide follow-up options:

```
clarify(
    question="What to do next?",
    choices=[
        "Execute directly — I'll implement according to the plan",
        "Review the document first — I want to review the plan",
        "Modify the plan — Some parts need adjustment",
        "Just save it — I'll find an AI to execute it myself"
    ]
)
```

---

## Execution Tool Mapping

Hermes tools used by each Phase of this skill:

| Phase | Tool | Purpose |
|-------|------|------|
| Phase 0 | Internal analysis | Parse user input |
| Phase 1 | `delegate_task` | Parallel domain research |
| Phase 2 | `clarify` | Interactive options |
| Phase 3 | Internal analysis | Creative expansion |
| Phase 4 | `clarify` | Solution confirmation |
| Phase 5 | `write_file` | Output plan document |
| Phase 6 | `clarify` | Follow-up options |

---

## Quick Reference

### Trigger Keywords

| User says | Identified as |
|--------|--------|
| "Help me make a XXX" | plan-generator |
| "I want to make a XXX" | plan-generator |
| "Help me write a plan" | plan-generator |
| "Help me refine this requirement" | plan-generator |
| "Help me expand this prompt" | plan-generator |
| "Write me a detailed proposal" | plan-generator |
| "Help me make a plan" | plan-generator |

### Dynamic Question Generation Flow (Quick Reference)

```
Phase 1 Research Output
    │
    ├── "Planning Dimension Checklist"
    │     ├── 【Must Confirm】→ Phase 2 asks one by one
    │     ├── 【AI Decides】→ Skip, write directly into plan
    │     └── 【Optional】→ Dynamically judge based on user choices
    │
    └── "Domain Knowledge Summary" → Technical basis for writing the plan

Phase 2 Question Sequence
    │
    ├── Q1: Core goal (almost always first)
    ├── Q2: Refinement level
    ├── Q3: AI involvement level (determines how many subsequent questions)
    │
    ├── Ask one by one according to 【Must Confirm】 list (order determined by research)
    │     Each question: clarify + options (from research) + "You recommend"
    │
    └── Creative expansion confirmation (Phase 3)
```

### Default Approach Quick Reference (Neutral Approach)

| Dimension | Neutral Default |
|------|---------|
| Core goal | Inferred from user description |
| Refinement level | Demo level |
| Design style | Clean and simple |
| Technical solution | Most mature, largest community |
| Interaction style | Basic interaction |
| Data source | Simulated data |

### AI Involvement Level Correspondence Table

| User Selection | Phase 2 Behavior | Phase 3 Behavior |
|---------|------------|------------|
| Full delegation | Only ask core goal + refinement level, AI decides the rest | AI supplements creativity on its own |
| Key confirmations | Ask 【Must Confirm】dimensions | Show creative suggestions for user confirmation |
| Step-by-step confirmation | Ask all dimensions (must + optional) | Show creativity in detail for user to choose one by one |

---

## Future Development Outlook

This skill currently focuses on **generating execution plans for AI Agents**. Here are future development directions:

### Phase 2: General Plan Generator
Not limited to code projects, can also generate:
- **DIY project plans**: User says "I want to make a wooden bookshelf" → Generate a detailed plan including materials list, tools list, step-by-step illustrations
- **Appliance repair plans**: User says "The washing machine won't spin" → Generate troubleshooting flow, repair steps, safety precautions
- **Cooking plans**: User says "I want to make braised pork" → Generate ingredients list, seasoning ratios, heat control, timeline
- **Study plans**: User says "I want to learn Python" → Generate learning path, resource recommendations, practice projects, milestones

### Phase 3: Interactive Execution Guidance
The plan is no longer just a static document, but:
- Real-time interactive guidance ("Now paste A onto B, watch the alignment")
- Progress tracking ("You've completed 3/8 steps")
- Problem diagnosis ("This step is wrong, possibly due to XX reason")
- Video/illustration assistance (generate diagrams for key steps)

### Phase 4: Multi-modal Plans
- Generate 3D model previews for physical projects
- Generate circuit diagrams for circuit projects
- Generate floor plans for architecture/renovation
- Generate step-by-step photos for cooking

**Current version (v1.0) focuses on Phase 1: Execution plan generation for code projects.**

---

## Common Pitfalls

1. **Storing domain knowledge in the skill** — This is the most serious error. A skill is a methodology, not a database. Domain-specific dimensions, technology selection tables, industry best practices, etc. **cannot** be pre-stored in the skill. Reasons: (a) Domains are infinite, you can't store them all; (b) Skills are loaded into model context, pre-stored knowledge wastes tokens; (c) Pre-stored knowledge becomes outdated. Correct approach: Phase 1 dynamic research, research results stored in project workspace, not in the skill. **2026-06-01 live verification: User explicitly corrected this error.**

2. **Phase 1 research only researches "what this thing is"** — It must also research "how to plan this thing" (Dimension B). That is: what dimensions need to be determined in this domain? Which must be asked to the user? Which can the AI decide? Which are optional? Without Dimension B research, Phase 2 doesn't know what to ask. **2026-06-01 live verification: Dimension B analysis is the core basis for dynamic question generation.**

3. **Asking multiple questions at once** — Users will feel like they're taking an "exam". Strictly one question at a time, using the clarify tool.

4. **Starting to ask users before sufficient research** — An AI that doesn't understand a domain can't ask good questions. Phase 1 research must be thorough.

5. **Using technical jargon in options** — "Use SSR or CSR?" Users can't understand. Use "Page loads faster (but needs a server) or simpler (pure local opening)?"

6. **Omitting the "You recommend" option** — Some users really don't want to choose. The "You recommend" option must be provided for AI to decide automatically.

7. **Plan document not specific enough** — "Make a nice UI" is not a plan. "Primary color #2563EB, background #F8FAFC, border radius 8px, shadow 0 2px 8px rgba(0,0,0,0.1)" is a plan.

8. **Skipping creative expansion** — User says "make a login page", and you only make a login page? No good. Must proactively think of forgot password, input validation, loading states, etc.

9. **Incomplete plan document structure** — All 9 sections are required. Even if a section doesn't apply, write "N/A — This project does not require XXX".

10. **Non-executable acceptance criteria** — "Ensure good quality" is not an acceptance criterion. "After running `open index.html`, the page loads within 3 seconds" is.

11. **Not respecting the user's wild ideas** — User says "control 3D visualization with gestures", don't say "too hard". Say "We can do it in two steps...".

12. **Outputting files to the desktop** — Unless the user explicitly requests it, output to D:\Hermes_Output\plans\.

13. **Missing AI involvement level dimension** — Without asking the user how much they want to participate in decisions, you can't determine how many questions to ask. The three levels of "Full delegation"/"Key confirmations"/"Step-by-step confirmation" are the control valve for Phase 2 question quantity. **2026-06-01 live verification: After the user chose "Key confirmations", the number of questions was just right (5).**

---

## Verification Checklist

- [ ] Has Phase 1 domain research been conducted?
- [ ] Is the "one question at a time" iron rule strictly followed?
- [ ] Does every option have a "You recommend" option?
- [ ] Has Phase 3 creative expansion been executed?
- [ ] Does the plan document contain all 9 sections?
- [ ] Is the plan document 3000+ words?
- [ ] Does every feature have clear acceptance criteria?
- [ ] Can the document be directly handed to an AI Agent for execution (no additional questions needed)?
- [ ] Is the output file in the D:\Hermes_Output\plans\ directory?
- [ ] Are follow-up options provided (execute/review/modify/save)?

---
> Source: [SOGERSEN/order-your-plan](https://github.com/SOGERSEN/order-your-plan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
