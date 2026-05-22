## adr

> description: Captures and tracks architectural decisions using the ADR format

---
description: Captures and tracks architectural decisions using the ADR format
globs: 
  - "**/*.md"
  - "**/*.ts"
  - "**/*.js"
  - "**/README.md"
alwaysApply: false
---

# Architectural Decision Records (ADR)

## 📋 ADR Template

### When to Create an ADR
```
Create ADR when decision:
✓ Affects system architecture
✓ Has long-term implications
✓ Involves significant tradeoffs
✓ Changes established patterns
✓ Requires team consensus
✓ Costs significant time/money
```

### ADR Format
```markdown
# ADR-{number}: {title}

## Status
{Proposed | Accepted | Deprecated | Superseded by ADR-XXX}

## Context
What is the issue that we're seeing that is motivating this decision?

## Decision
What is the change that we're proposing and/or doing?

## Consequences
What becomes easier or more difficult because of this change?

### Positive
- {benefit 1}
- {benefit 2}

### Negative  
- {drawback 1}
- {drawback 2}

### Neutral
- {neutral impact}

## Alternatives Considered
1. {Alternative 1}: {Why rejected}
2. {Alternative 2}: {Why rejected}
```

## 🏗️ Architecture Decision Triggers

### Technical Triggers
```javascript
// TRIGGER: Switching core technology
"Should we migrate from Express to Fastify?"
→ Create ADR: Performance requirements changed

// TRIGGER: Changing data flow
"Should we implement event sourcing?"
→ Create ADR: Audit requirements need full history
// TRIGGER: Infrastructure changes
"Should we move from monolith to microservices?"
→ Create ADR: Scaling requirements exceeded
```

### Business Triggers
```
TRIGGER: "New compliance requirement (GDPR)"
→ ADR: Data retention and deletion strategy

TRIGGER: "10x user growth expected"
→ ADR: Horizontal scaling approach

TRIGGER: "Need offline capability"
→ ADR: Local-first architecture
```

## 📊 Decision Tracking

### Active Decisions Index
```yaml
CURRENT_ADRS:
  ADR-001:
    title: "Use PostgreSQL as primary database"
    status: "Accepted"
    date: "2024-01-15"
    impact: ["All data models", "Deployment", "Backup strategy"]
    
  ADR-002:
    title: "Implement JWT authentication"
    status: "Superseded by ADR-005"
    date: "2024-02-01"
    superseded: "2024-03-10"
    
  ADR-003:
    title: "Use Redis for caching"
    status: "Accepted"
    date: "2024-02-15"
    review_date: "2024-08-15"
```

### Decision Dependencies
```
ADR-003 (Redis caching)
  ├─ Depends on: ADR-001 (PostgreSQL)
  ├─ Enables: ADR-004 (Session management)
  └─ Conflicts with: None

ADR-005 (OAuth2 authentication)
  ├─ Supersedes: ADR-002 (JWT auth)
  ├─ Depends on: ADR-003 (Redis)
  └─ Impacts: All auth endpoints
```

## 🔄 ADR Lifecycle

### Review Schedule
```
REVIEW_TRIGGERS:
- Every 6 months for "Accepted" status
- When assumptions change
- When better alternatives emerge
- When consequences prove incorrect
- Before major version releases
```

### Status Transitions
```
Proposed → Accepted
    ↓         ↓
Rejected  Deprecated
            ↓
        Superseded
```

## 💡 Quick Decision Framework

### Rapid ADR Creation
```markdown
## Quick ADR: {Decision needed}

### Option A: {Quick description}
Pros: {1-2 main benefits}
Cons: {1-2 main drawbacks}
Cost: {time/money/complexity}

### Option B: {Quick description}
Pros: {1-2 main benefits}
Cons: {1-2 main drawbacks}
Cost: {time/money/complexity}

### Recommendation: Option {A|B}
Because: {One sentence rationale}
```

## 🎯 Common Architecture Decisions

### Database Decisions
```markdown
COMMON_PATTERNS:
- SQL vs NoSQL → Consider consistency needs
- Single vs Multi-tenant → Consider isolation requirements
- Normalized vs Denormalized → Consider read/write ratio
- Sync vs Async processing → Consider user experience
```

### API Design Decisions
```markdown
TYPICAL_CHOICES:
- REST vs GraphQL vs gRPC
- Versioning strategy (URL vs Header)
- Authentication method (JWT vs Session)
- Rate limiting approach
- Pagination style (cursor vs offset)
```

### Frontend Architecture
```markdown
FRAMEWORK_DECISIONS:
- SPA vs SSR vs Static
- State management (Redux vs Context vs Zustand)
- Component library choice
- Build tool selection
- CSS approach (CSS-in-JS vs Tailwind)
```

## 📈 Decision Impact Analysis

### Measuring Decision Success
```yaml
ADR_METRICS:
  ADR-001 (PostgreSQL):
    predicted_benefits:
      - "ACID compliance" ✓ Achieved
      - "Complex queries" ✓ Achieved  
      - "Scaling to 1M users" ✓ Achieved
    predicted_drawbacks:
      - "Operational complexity" ✓ Manageable
      - "Higher costs" ✗ More than expected
    
    verdict: "Good decision, would repeat"
```

## 🚨 Decision Anti-Patterns

### Avoid These
```
❌ "We've always done it this way"
❌ "X company does it this way"
❌ "It's the newest technology"
❌ "We might need it someday"
❌ Making decisions without documenting
❌ Not revisiting old decisions
```

### Embrace These
```
✅ Data-driven decisions
✅ Clear success criteria
✅ Documented tradeoffs
✅ Regular reviews
✅ Team consensus
✅ Reversible decisions when possible
```

## 📝 ADR Best Practices

### Writing Effective ADRs
1. **Be concise**: 1-2 pages maximum
2. **Be specific**: Include concrete examples
3. **Be honest**: Document real tradeoffs
4. **Be timely**: Write while context is fresh
5. **Be collaborative**: Get team input

### ADR File Organization
```
docs/
├── adr/
│   ├── README.md (index of all ADRs)
│   ├── ADR-001-database-choice.md
│   ├── ADR-002-auth-strategy.md
│   └── ADR-003-caching-layer.md
└── architecture/
    └── diagrams/
```

## 🔍 Finding Relevant ADRs

### Quick Lookup Commands
```bash
# Find all accepted ADRs
grep -l "Status: Accepted" docs/adr/*.md

# Find ADRs about specific topic
grep -i "authentication" docs/adr/*.md

# Find superseded decisions
grep -l "Superseded" docs/adr/*.md
```

Remember: Good architectures are made of documented decisions!

---
> Source: [DVC2/cursor_prompts](https://github.com/DVC2/cursor_prompts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
