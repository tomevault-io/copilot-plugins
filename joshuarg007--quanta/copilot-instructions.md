## quanta

> > **FOR CLAUDE: Read the SESSION STATE, DESIGN PRINCIPLES, and DUAL-USE CONTEXT sections before doing anything.**

# QUANTA Platform & Project DRIFT

> **FOR CLAUDE: Read the SESSION STATE, DESIGN PRINCIPLES, and DUAL-USE CONTEXT sections before doing anything.**

---

## DESIGN PRINCIPLES (Immutable Priority Order)

**Every decision in this codebase must respect this hierarchy:**

| Priority | Principle | Rationale |
|----------|-----------|-----------|
| **1** | **Legal Protection** | Grants, tax status, IP, liability — non-negotiable |
| **2** | **Science** | DRIFT is the core mission; research integrity first |
| **3** | **Cost-Efficiency** | Prevent runaway costs; hard limits for external tenants |
| **4** | **Functionality** | Features that work correctly |
| **5** | **User-Friendliness** | Nice to have, never at expense of above |

### Cost Control Mandate

> **Preventing ridiculous costs must be hardwired into every aspect of this application.**

- External tenants (non-Axion) have **hard limits** on all resource-consuming operations
- Limits are enforced at the database, API, and UI layers (defense in depth)
- Exceeding limits = blocked (not just warned)
- Add-on purchases unlock higher limits (revenue driver)
- Axion Deep Labs tenant has elevated/unlimited access for DRIFT research

---

## SESSION STATE (Update before ending each session)
**Last Updated:** 2026-01-11

### Where We Left Off:
- Defined multi-tenancy architecture based on SLMS pattern
- Clarified business model: DRIFT = internal R&D (grant-funded), QUANTA = pro bono education SaaS
- Established design principles with priority order
- Removed AI tutor from scope

### Immediate Next Steps:
1. Implement multi-tenancy models (Organization, updated User, etc.)
2. Create plans system with hard limits for external tenants
3. Build signup flow with domain-based org assignment
4. Implement usage tracking and limit enforcement

### Current Blockers:
- None documented

### Key Decisions Made:
- **Tenancy**: Both educational institutions AND Axion Deep Labs as tenants
- **Limits**: Hard caps for non-Axion tenants; pay-to-upgrade model
- **Revenue**: Add-on sales for limit increases; base education is pro bono (tax write-off)
- **No AI features**: AI tutor removed from scope

---

## DUAL-USE CONTEXT (Critical for Classification)

### The Two Planes

This platform operates on TWO distinct planes. **Never conflate them.**

| Aspect | QUANTA (Education Plane) | Project DRIFT (Research Plane) |
|--------|--------------------------|-------------------------------|
| **What it is** | Educational SaaS platform | Scientific research program |
| **NAICS** | 541511 (Software Services) | 541715 (R&D Physical Sciences) |
| **Purpose** | Deliver curriculum | Generate knowledge |
| **Users** | Students, educators | Researchers, investigators |
| **Outputs** | Learning, competency | Data, findings, publications |
| **Status** | Commercial product | Active research |
| **Language** | "Learn," "practice," "master" | "Investigate," "observe," "characterize" |

### Classification Rule

> Activity classification depends on **intent**, **technical uncertainty**, and **outcome** — not on shared code or infrastructure.

### Governance Documentation

Full governance framework at:
```
~/Corporate/AxionDeepLabs/Quanta_Dual_Use_Governance/
```

Key documents to reference:
- `02_Policies/03_Dual_Use_Platform_Policy_Quanta.md` — Classification logic
- `02_Policies/06_Language_And_Labeling_Guide.md` — Approved terminology
- `03_Research_Charter/01_Research_Charter_Summary.md` — DRIFT focus

---

## PROJECT DRIFT (Research Plane)

### Full Name
**DRIFT**: Degradation Regimes In Iterated Field Transformations

### Research Focus
Investigating behavioral uncertainty in quantum system state evolution under repeated manipulation, focusing on gradual degradation of stability under variation in operator ordering and diversity, in regimes where closed-form analytical prediction is not feasible across all configurations.

### Scientific Basis
Supported by peer-reviewed literature:
- Tranter et al. (2019) — Ordering affects Trotter error
- Nakamura & Ankerhold (2024) — Non-Markovian sequence effects
- Huang et al. (2024) — Diversity as experimental variable

### Research Themes
1. State distribution dynamics under iteration
2. Operator ordering effects
3. Operator diversity effects
4. Predictability boundaries
5. Stability/instability characterization

### When Writing About DRIFT
Use language like:
- "Investigating behavioral uncertainty..."
- "Characterizing gradual degradation..."
- "Studying how state distributions evolve..."
- "Conducting hypothesis-driven experimentation..."

**Never** describe DRIFT as a product, service, or commercial offering.

---

## QUANTA EDUCATION (Education Plane)

### What It Is
An educational SaaS platform delivering structured curriculum and interactive simulation of established quantum computing concepts.

### Target Users
- Undergraduate/graduate students
- University educators
- Self-directed learners
- Institutions building QC programs

### When Writing About QUANTA Education
Use language like:
- "Learn foundational quantum computing principles"
- "Practice with interactive simulations"
- "Structured curriculum for universities"
- "Accessible learning experiences"

**Never** claim students are "participating in research" or describe educational features as "experimental" (unless actually part of documented experiments).

---

## ROUTE STRUCTURE (axiondeep.com)

### Implemented/Planned Routes

```
/                       → Home (Axion Deep Labs)
/solutions              → All offerings overview
/quanta                 → QUANTA Education Platform (commercial)
/quanta/pricing         → Subscription tiers
/quanta/demo            → Interactive demo
/research               → Project DRIFT (R&D)
/research/publications  → Papers, findings
/research/methodology   → How we conduct research
```

### Separation Principle
- `/quanta/*` routes = Education Plane language only
- `/research/*` routes = Research Plane language only
- Cross-linking allowed but with correct framing:
  - QUANTA page can say "Built on insights from our research"
  - Research page can say "Using QUANTA as our research instrument"

---

## PROJECT STRUCTURE

```
quanta/
├── backend/                 # FastAPI Python backend
│   ├── app/
│   │   ├── main.py         # FastAPI app entry
│   │   ├── curriculum/     # Lesson content
│   │   └── simulation/     # Quantum simulation engine
│   └── venv/
├── frontend/               # React + TypeScript + Vite
│   ├── src/
│   │   ├── components/
│   │   │   └── lessons/
│   │   │       └── sections/
│   │   └── pages/
│   └── package.json
├── docker-compose.yml
└── CLAUDE.md              # This file
```

---

## COMMANDS

```bash
# Backend (development)
cd backend && source venv/bin/activate && uvicorn app.main:app --reload

# Frontend (development)
cd frontend && npm run dev

# Build frontend
cd frontend && npm run build

# Full stack with Docker
docker-compose up
```

---

## IMPLEMENTATION ROADMAP (High-Level)

### Phase 1: Foundation (Current → 2 weeks)

**Objective:** Establish clear separation on axiondeep.com

1.1. **Create `/research` route**
- Project DRIFT overview page
- Research focus and themes
- Link to publications (placeholder)
- "QUANTA as research instrument" section

1.2. **Refine `/quanta` or `/solutions/quanta` route**
- Education platform messaging
- Feature overview (curriculum, simulations, assessments)
- Target audiences
- University partnership CTA

1.3. **Language audit**
- Review all existing QUANTA mentions on site
- Apply Language and Labeling Guide
- Remove any research claims from product pages

### Phase 2: Research Presence (2-4 weeks)

**Objective:** Establish credibility for DRIFT

2.1. **Research methodology page**
- How experiments are designed
- Documentation standards
- Instrument capabilities

2.2. **Publications/findings section**
- Structure for future papers
- Blog-style research updates
- Preprint links when available

2.3. **Scientific advisory context**
- Literature foundation
- Academic partnership opportunities
- Grant/funding positioning

### Phase 3: Education Platform Polish (4-6 weeks)

**Objective:** Commercial-ready education offering

3.1. **Curriculum showcase**
- Module/lesson previews
- Learning outcomes
- Difficulty progression

3.2. **Pricing and access**
- Institutional tiers
- Individual subscriptions
- Trial/demo access

3.3. **University partnership materials**
- Integration guides
- Educator resources
- Case studies (when available)

### Phase 4: Platform Development (Ongoing)

**Objective:** Build the actual QUANTA platform capabilities

4.1. **Core simulation engine**
- Quantum state visualization
- Gate operations
- Measurement simulation

4.2. **Curriculum delivery**
- Lesson framework
- Progress tracking
- Assessment engine

4.3. **Research instrumentation**
- Experiment configuration
- Data collection
- Export/analysis tools

---

## CODE CONVENTIONS

### General
- **Git commits:** Standard messages, no Claude attribution unless requested
- **Documentation:** Update this CLAUDE.md at end of significant sessions

### TypeScript (Frontend)
- Strict types, avoid `any`
- Functional components with hooks
- Components in `/components`, pages in `/pages`

### Python (Backend)
- PEP 8 compliance
- Type hints required
- Async where beneficial

### File Organization
- Lesson sections: `frontend/src/components/lessons/sections/`
- Curriculum content: `backend/app/curriculum/lessons/`
- Research-specific code: Tag with `# DRIFT:` comment for clarity

---

## TERMINOLOGY QUICK REFERENCE

### Use for Education (QUANTA)
- "Educational platform"
- "Learn," "practice," "master"
- "Curriculum," "courses," "modules"
- "Students," "educators," "institutions"
- "Subscription," "access"

### Use for Research (DRIFT)
- "Research instrument"
- "Investigate," "characterize," "observe"
- "Experiments," "hypotheses," "findings"
- "Technical uncertainty," "knowledge creation"
- "Principal investigator," "research program"

### Never Mix
- Don't call education "experimental" (unless it literally is)
- Don't call research "a product" or "service"
- Don't say students are "participating in research"
- Don't use marketing superlatives in research docs

---

## KEY FILES TO KNOW

### Governance (Reference)
```
~/Corporate/AxionDeepLabs/Quanta_Dual_Use_Governance/
├── 02_Policies/06_Language_And_Labeling_Guide.md    # Approved terms
├── 03_Research_Charter/01_Research_Charter_Summary.md  # DRIFT focus
├── 08_Glossary/01_Glossary.md                        # Definitions
```

### Website (axiondeep.com)
```
~/AllProjects/axiondeep/
├── src/pages/                    # Add /research, update /quanta
├── src/components/               # Shared components
└── CLAUDE.md                     # Site-specific guidance
```

### Platform (This Project)
```
~/AllProjects/quanta/
├── backend/app/                  # API and simulation
├── frontend/src/                 # React UI
└── CLAUDE.md                     # This file
```

---

## SESSION CHECKLIST

Before ending a session:
- [ ] Update SESSION STATE with where we left off
- [ ] Note any blockers or decisions made
- [ ] Verify language separation in any content created
- [ ] Commit changes with descriptive messages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joshuarg007)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/joshuarg007)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
