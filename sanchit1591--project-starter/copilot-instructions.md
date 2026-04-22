## project-starter

> These are methodologies for approaching development work. For the overarching design philosophy, see `philosophy.mdc`. For specific actionable rules, see `core-rules.mdc`.


# Development Practices

These are methodologies for approaching development work. For the overarching design philosophy, see `philosophy.mdc`. For specific actionable rules, see `core-rules.mdc`.

---

## Tracer Bullet Development - Building Through the Fog

**Tracer bullet development is about building a thin, end-to-end slice of the system that works from the start.**

Like tracer rounds in combat, it provides real-time feedback on where you are aiming while you are still moving.

**Core idea**: Build something that runs through the entire system early, then evolve it.

**Mental model**: You are aiming in the dark. Tracer bullets show you where your system is actually going.

### The Problem It Solves

In new systems:
- Users don't fully know what they want
- Developers don't fully know what they are building
- Requirements are vague and will change
- Components have never been integrated before

**Depth-first development** (perfecting one layer in isolation):
- Optimizes parts before validating the whole
- Discovers integration problems late
- Creates expensive rework

**Tracer development**:
- Validates architecture early
- Exposes integration issues immediately
- Keeps progress visible

**Mental model**: Don't perfect pieces before you know the whole works.

### What a Tracer Bullet Is

A tracer bullet is:
- **End-to-end**: Passes through all layers
- **Minimal**: Simplest possible implementation
- **Functional**: Actually works, not a mock
- **Built to be extended**: Not thrown away, forms the foundation

**Example**: If your system has Architecture → API → Compute → Database → UI, a tracer bullet might pass a single string or record through all layers, touching every boundary once, proving the system "hangs together."

**Mental model**: Build the skeleton first. Add muscle later.

### Tracer Bullets vs Prototypes

**Prototypes**:
- Built to learn
- Often partial
- Typically disposable
- Explore specific unknowns

**Tracer bullets**:
- Built to keep
- End-to-end
- Architecturally real
- Form the foundation of the final system

**Key distinction**: Prototypes generate knowledge. Tracer bullets generate structure.

**Decision rule**:
- If your goal is "Do these parts hang together?" → use a tracer bullet
- If your goal is "Does this idea even work?" → build a prototype

**Mental model**: Prototype to understand. Tracer to build.

### Why Tracer Bullets Matter

**Benefits**:
- Early user feedback
- Early architectural validation
- Continuous sense of progress
- Reduced risk of late-stage surprises
- Shared concrete reference for the team

**For developers**:
- A real code path to evolve
- A scaffold for adding features
- A living architectural diagram

**For stakeholders**:
- Something demonstrable
- Proof of direction
- Faster alignment

**Mental model**: Seeing something work beats reasoning about what might work.

### Tracer Bullets and Change

Because tracer bullets:
- Use real integrations
- Cross real boundaries
- Expose real constraints

They:
- Reveal mismatches early
- Allow fast course correction
- Prevent over-investment in the wrong direction

**Mental model**: Correct early while the cost of change is still low.

### Tracer Bullets and Other Principles

**DRY**: The tracer establishes the authoritative flow of knowledge

**Decoupling / Orthogonality**: Boundaries become visible and testable

**Reversibility**: Early structure keeps choices changeable before they harden

**Together**: Tracer bullets make architectural decisions real while they are still cheap to change.

**Mental model**: Architecture is not what you draw. It is what your code already does.

### Common Failure Modes

**Not a tracer**:
- "Let's fully design this layer first"
- "We'll integrate later"
- "This is just a mock for now"

**Anti-patterns**:
- Building isolated components with no integration
- Writing throwaway "temporary" code that becomes permanent
- Over-polishing before validating end-to-end flow

**Mental model**: If it doesn't run end-to-end, it doesn't exist yet.

### The Approach

1. Build the simplest possible version that works end-to-end
2. Deploy it to real infrastructure (not mocks)
3. Get feedback, iterate
4. Flesh out features incrementally

### Design Checklist

When starting a system:
1. Can I push a minimal piece of data through the entire stack?
2. Does this exercise touch every major architectural boundary?
3. Is this code written to evolve, not to be discarded?
4. Are users seeing something real early?

**Mental model**: Build a thin working system first. Then grow it deliberately.

**For this project**: Every new feature should start with the thinnest possible working slice.

---

## Prototyping - Learning Before Committing

**Prototyping is about building small, disposable experiments to answer specific questions about a system.**

It is cheaper than full-scale production and exists only to generate insight.

**Core idea**: When something is uncertain, build a quick experiment to learn.

**Mental model**: Prototypes exist to reduce ignorance, not to become the product.

### Why Prototype

In complex systems:
- Some aspects are unproven
- Some behaviors are unknown
- Some assumptions may be wrong

**Instead of guessing**:
- Build something minimal
- Observe how it behaves
- Use the results to inform real design

**Mental model**: When you don't know, don't argue — measure.

### What a Prototype Is

A prototype is:
- **Narrow in scope**: Answers one focused question
- **Purpose-built**: Designed to answer that question
- **Disposable by design**: Intended to be thrown away

It may:
- Ignore correctness
- Ignore completeness
- Ignore robustness
- Ignore style

**Because**: Its only job is to teach you something.

**Mental model**: Prototypes are experiments, not products.

### What Prototyping Is For

Use prototypes to explore:
- **Technical feasibility**: "Will this library even work for this?"
- **Performance characteristics**: "Is this fast enough in practice?"
- **Integration uncertainty**: "Can these two systems talk meaningfully?"
- **Design assumptions**: "Does this model actually make sense?"

**Example** (car analogy):
You don't build a full car to test:
- Aerodynamics → wind tunnel model
- Styling → clay model
- Structural integrity → stress rig

Each prototype answers one focused question.

**Mental model**: Learn one thing at a time, cheaply.

### Prototypes vs Tracer Bullets

**Prototypes**:
- Built to learn
- Partial and narrow
- Often messy
- Intentionally thrown away

**Tracer bullets**:
- Built to validate architecture
- End-to-end
- Minimal but real
- Designed to evolve

**Decision rule**:
- If your goal is "Do these parts hang together?" → use a tracer bullet
- If your goal is "Does this idea even work?" → build a prototype

**Mental model**: Prototype to reduce uncertainty. Trace to grow the system.

### When to Prototype First

If you:
- Don't know whether a technology will work
- Are unsure about a core assumption
- Are experimenting with something you've never built before
- Need fast validation without architectural commitment

**Then**: Prototype first.

Once you gain confidence:
- Discard the prototype
- Implement the real design using tracer bullet development

**Mental model**: Learn fast, then build deliberately.

### Dangers of Prototyping

**Common mistakes**:
- Treating prototype code as production
- "Cleaning it up later" instead of discarding
- Letting shortcuts harden into architecture

**Why this is dangerous**:
- Technical debt becomes invisible
- Fragile experiments become foundations

**Rule**: If it was built to learn, it must be thrown away.

**Mental model**: Disposable code should never become structural code.

### Relation to Other Principles

**Reversibility**: Prototypes reduce risk before committing to decisions

**DRY**: Prototypes may violate DRY — that is acceptable because they are temporary

**Decoupling / Orthogonality**: Prototypes do not need perfect boundaries. Real design happens after learning.

**Tracer Bullets**: Come after prototypes, once feasibility is understood

**Mental model**: Prototype to discover. Trace to construct.

### Design Checklist

**Before building**:
1. What specific question am I trying to answer?
2. What can I safely ignore to answer it faster?
3. Will I throw this away once I learn?

**After learning**:
4. What changed in my understanding?
5. Is it time to switch to tracer bullet development?

**Mental model**: Prototypes buy knowledge. Tracer bullets build systems.

---

## Estimating - Reasoning About Uncertainty

**Estimating is not guessing how long something takes. It is building a simplified model of reality to reason about effort, scale, and risk.**

**Core idea**: Estimates are hypotheses about complexity, not promises about dates.

**Mental model**: You are not predicting the future. You are exposing uncertainty.

### Why Estimating Matters

**Good estimates**:
- Guide prioritization
- Surface risk early
- Enable better tradeoffs
- Improve planning conversations

**Bad estimates**:
- Hide uncertainty
- Create false confidence
- Collapse under real-world complexity

**Mental model**: The value of an estimate is not accuracy — it is clarity.

### How to Estimate (Pragmatic Approach)

1. **Understand what is being asked**
2. **Narrow the scope**
3. **Build a simple mental model**
4. **Break into components**
5. **Identify the parameters that drive complexity**
6. **Calculate based on those drivers**

**Key insight**: Focus on factors that multiply, not those that merely add.

**Mental model**: Find what dominates the outcome, not what decorates it.

### Hedging Your Estimate

Hedging does not mean being vague. It means:
- Varying the key assumptions
- Re-running the model with different plausible values
- Seeing which uncertainties actually control the result

**Purpose**:
- Expose sensitivity
- Identify risk drivers
- Guide what must be validated early

**Mental model**: Change the assumptions and see what breaks.

### What Really Makes Estimates Wrong

**Common failure modes**:
- Unclear or shifting requirements
- Human coordination and communication costs
- Hidden integration complexity
- Late discovery of edge cases

These are not execution problems — they are uncertainty problems.

**Mental model**: Most estimates fail socially before they fail technically.

### Estimating as a Culture

**Healthy culture**:
- Treats estimates as evolving models
- Re-estimates as knowledge increases
- Makes assumptions explicit
- Uses estimates to reduce risk, not assign blame

**Unhealthy culture**:
- Treats estimates as fixed commitments
- Punishes revision
- Ignores new information

**Mental model**: Estimates should invite learning, not enforce certainty.

### Learning from Estimates

After delivery:
- Compare estimate vs reality
- Identify what was misunderstood
- Update mental models

This feedback loop builds intuition over time.

**Mental model**: You get better at estimating only by auditing your own assumptions.

### Relation to Other Principles

**DRY**: Centralizes knowledge, reducing estimation error

**Decoupling / Orthogonality**: Limits blast radius of changes

**Reversibility**: Keeps incorrect decisions cheap to undo

**Tracer Bullets**: Validate architecture early

**Prototyping**: Reduce uncertainty before committing

**Together**: Good design makes good estimation possible.

**Mental model**: You cannot estimate what you do not understand — and you cannot understand what you have not explored.

---

## Related Files

- `philosophy.mdc` - Design philosophy and principles
- `core-rules.mdc` - Specific actionable rules
- `commits.mdc` - Commit message standards
- `testing.mdc` - Testing philosophy and standards

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sanchit1591) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
