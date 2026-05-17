## laws-of-software-engineering

> 56 laws and principles of software engineering (Conway, Brooks, Hyrum, Gall, Postel, CAP, YAGNI, DRY, SOLID, Pareto, Amdahl, Murphy, and more). Use when designing architecture, planning projects, debating scope, reviewing code, running postmortems, or making engineering decisions where an established principle applies.


# Laws of Software Engineering

56 recurring laws, principles, and rules of thumb across software engineering. Cite one only when it sharpens the current decision; skip it if it's just a name-drop.

Source: paraphrased from [lawsofsoftwareengineering.com](https://lawsofsoftwareengineering.com/).

## Teams
- **Conway's Law** — Systems mirror the communication structure of the org that builds them. Apply: org design, re-orgs, platform rewrites.
- **Brooks's Law** — Adding people to a late project makes it later. Apply: hiring/resourcing decisions on slipping projects.
- **Dunbar's Number** — Informal structure breaks past ~150 stable relationships. Apply: org-size planning, team-of-teams design.
- **Ringelmann Effect** — Per-person productivity drops as groups grow. Apply: team sizing, splitting a growing team.
- **Price's Law** — ~50% of output comes from ~√n contributors. Apply: key-person risk, retention prioritization.
- **Putt's Law** — Tech is often managed by those who don't understand it. Apply: escalation, leadership gaps.
- **Peter Principle** — People get promoted until they stop being good at the job. Apply: promotion criteria, IC tracks.
- **Bus Factor** — Min # of people whose loss would halt the project. Apply: knowledge-sharing, documentation priority.
- **Dilbert Principle** — Weak performers sometimes get promoted to limit their damage. Apply: promotion decision review.

## Planning
- **Premature Optimization (Knuth)** — Optimize after measuring, not before. Apply: code review, performance work.
- **Parkinson's Law** — Work expands to fill the time given. Apply: deadline setting, sprint scoping.
- **Ninety-Ninety Rule** — First 90% takes 90% of the schedule; last 10% takes the other 90%. Apply: estimation sanity-check.
- **Hofstadter's Law** — It always takes longer than you expect, even accounting for this law. Apply: padding estimates.
- **Goodhart's Law** — When a measure becomes a target, it stops being a good measure. Apply: KPI design, incentives.
- **Gilb's Law** — Anything worth quantifying can be measured imperfectly better than not at all. Apply: pushing back on "we can't measure that."

## Architecture
- **Hyrum's Law** — With enough users, every observable behavior becomes a contract. Apply: API/SDK/CLI evolution, versioning.
- **Gall's Law** — Complex systems that work evolved from simpler working systems. Apply: greenfield scoping, avoiding big-bang rewrites.
- **Law of Leaky Abstractions** — Non-trivial abstractions always leak. Apply: abstraction choices, cross-layer debugging.
- **Tesler's Law** — Inherent complexity can be moved, not removed. Apply: who owns the complexity — library, app, or user?
- **CAP Theorem** — Under partition, choose consistency or availability. Apply: distributed system design, DB selection.
- **Second-System Effect** — Second systems tend to be overreaching and bloated. Apply: resisting scope creep on v2.
- **Fallacies of Distributed Computing** — Eight false assumptions (network reliability, zero latency, etc.). Apply: reviewing distributed designs.
- **Law of Unintended Consequences** — Changes to complex systems produce unexpected effects. Apply: change review, rollout planning.
- **Zawinski's Law** — Every program expands until it can read mail. Apply: scope discipline, "feature" pushback.

## Quality
- **Boy Scout Rule** — Leave code cleaner than you found it. Apply: code review norms, refactoring discipline.
- **Murphy's Law** — Anything that can go wrong will, at the worst time. Apply: chaos testing, failure-mode design.
- **Postel's Law** — Strict in what you send; lenient in what you accept. Apply: API design, protocol handling.
- **Broken Windows Theory** — Small unrepaired defects signal that quality doesn't matter. Apply: prioritizing small cleanups, linter enforcement.
- **Technical Debt** — Shortcuts accrue interest paid as slower future work. Apply: tradeoff conversations, cleanup time.
- **Linus's Law** — Given enough eyeballs, bugs are shallow. Apply: open-source strategy, code review staffing.
- **Kernighan's Law** — Debugging is 2× harder than writing. Apply: style, reviewer gut-check.
- **Testing Pyramid** — Many fast unit tests, fewer integration, thin E2E top. Apply: test strategy, flaky-test triage.
- **Pesticide Paradox** — Same tests repeated find fewer new bugs. Apply: test refresh, mutation testing.
- **Lehman's Laws** — Software in use must evolve; complexity grows unless worked against. Apply: roadmap pacing.
- **Sturgeon's Law** — 90% of everything is mediocre. Apply: realistic expectations of libraries, tools, candidates.

## Scale
- **Amdahl's Law** — Parallel speedup is capped by the serial fraction. Apply: parallelism ROI, profiling.
- **Gustafson's Law** — More parallelism is typically spent on bigger problems at constant time. Apply: capacity planning.
- **Metcalfe's Law** — Network value grows roughly with n². Apply: network-effect products, platform strategy.

## Design
- **YAGNI** — Don't build it until a real requirement demands it. Apply: scope negotiation, code review.
- **DRY** — Every piece of knowledge has one authoritative representation. Apply: refactoring, duplication review.
- **KISS** — As simple as requirements allow, no simpler, no more elaborate. Apply: default preference during design.
- **SOLID** — SRP, OCP, LSP, ISP, DIP. Apply: OO class/module design, dependency direction.
- **Law of Demeter** — Objects talk only to immediate collaborators. Apply: reviewing coupling.
- **Principle of Least Astonishment** — Software should behave the way people expect. Apply: API ergonomics, UX defaults.

## Decisions
- **Dunning-Kruger** — Limited knowledge breeds overconfidence. Apply: weighting opinions, seeking second opinions.
- **Hanlon's Razor** — Don't attribute to malice what carelessness explains. Apply: postmortems, incident culture.
- **Occam's Razor** — Prefer the simplest explanation that fits. Apply: debugging, root-cause analysis.
- **Sunk Cost Fallacy** — Past investment isn't a reason to continue. Apply: deprecating projects, killing features.
- **The Map Is Not the Territory** — Diagrams and docs are not the running system. Apply: incident response, assumption-checking.
- **Confirmation Bias** — We favor evidence that confirms what we believe. Apply: debugging, code review, design review.
- **Hype Cycle & Amara's Law** — Short-term overestimation, long-term underestimation. Apply: tech adoption timing.
- **Lindy Effect** — The longer a tech has lasted, the longer it will. Apply: choosing between boring and novel stacks.
- **First Principles Thinking** — Strip to underlying facts; reason up. Apply: novel problems, breaking deadlocks.
- **Inversion** — Ask "how would this fail?"; design against it. Apply: threat modeling, design review.
- **Pareto Principle (80/20)** — ~80% of outcome from ~20% of causes. Apply: prioritization, bug triage.
- **Cunningham's Law** — Post a wrong answer; watch the right one appear. Apply: documentation drafts, shaking out opinions.

---

**Tradeoff:** these are tendencies, not theorems. Any law can be misapplied as a rhetorical trump card. Invoke them when they sharpen the decision; drop them when the situation has genuinely different dynamics.

---
> Source: [Y0lan/laws-of-software-engineering](https://github.com/Y0lan/laws-of-software-engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
