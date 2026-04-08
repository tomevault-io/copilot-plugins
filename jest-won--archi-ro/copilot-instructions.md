## archi-ro

> **Role:** Autonomous deep worker

# archi-ro Agents

## Discipline Agents (5 Specialists)

### 🔨 Hephaestus (executor)
**Role:** Autonomous deep worker  
**Model:** GPT-4 Turbo  
**Expertise:**
- End-to-end feature implementation
- Codebase exploration
- Pattern research
- Self-directed execution

**Use for:** Complete features, complex implementations

---

### 🔮 Oracle (architect)
**Role:** Architecture and debugging specialist  
**Model:** Claude Opus  
**Expertise:**
- System design
- Technical decisions
- Architecture review
- Complex debugging

**Use for:** System design, architectural decisions, hard bugs

---

### 📚 Librarian (reviewer)
**Role:** Documentation and code search expert  
**Model:** Claude Opus  
**Expertise:**
- Code review
- Documentation search
- Pattern finding
- Reference lookup

**Use for:** Code review, finding examples, documentation

---

### 🔍 Explore (debugger)
**Role:** Fast codebase grep specialist  
**Model:** Claude Sonnet  
**Expertise:**
- Quick searches
- File discovery
- Pattern matching
- Fast debugging

**Use for:** Finding code, quick searches, debugging

---

### 👁️ Multimodal Looker
**Role:** Image analysis specialist  
**Model:** Gemini Flash 2.0  
**Expertise:**
- Image analysis
- UI/UX review
- Visual debugging
- Screenshot analysis

**Use for:** UI review, visual bugs, design feedback

---

## Supporting Agents (7 Additional)

### Backend (GPT-4 Turbo)
API development, database design, server logic

### Frontend (GPT-4 Turbo)
React, Vue, UI components, state management

### Tester (GPT-4 Turbo)
Test generation, E2E testing, test coverage

### DevOps (GPT-4 Turbo)
CI/CD, deployment, infrastructure

### Security (Claude Opus)
Security audit, vulnerability detection

### Performance (Claude Sonnet)
Optimization, profiling, performance tuning

### UI Designer (Gemini Flash)
Design systems, UI/UX, accessibility

---

## Agent Selection

### Automatic (Recommended)
```bash
omk team 3 "build authentication"
# System selects: architect + executor + security
```

### Manual
```bash
omk team 3:executor "implement feature"
omk team 2:architect,reviewer "review design"
```

### Dynamic (Performance-Based)
System automatically selects best-performing agent based on:
- Historical success rate
- Task type match
- Cost optimization
- Recent performance

View stats:
```bash
omk agent-stats
omk model-strengths
```

---

## Model Mapping

| Model | Agents | Emoji |
|-------|--------|-------|
| Claude Opus | reviewer, architect, security | 🔵 |
| Claude Sonnet | debugger, performance | 🟢 |
| GPT-4 Turbo | executor, backend, frontend, tester, devops | 🟡 |
| Gemini Flash | ui-designer, data-scientist, multimodal-looker | 🔴 |

---

## Performance Tracking

All agent executions are tracked in `.omc/performance.json`:
- Success/failure rate
- Average duration
- Cost per task
- Task type strengths

This data powers dynamic agent selection for optimal results.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jest-won)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jest-won)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
