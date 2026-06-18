## dsa-pro

> Plan, design, and ship production-grade software end-to-end. Project planning - discovery, PRD, ADRs, work breakdown, roadmap, definition-of-done. SDLC playbooks - Agile, Lean/MVP, DevOps/CI-CD, TDD/property-test-first, Waterfall for regulated medical and finance, DDD/hexagonal. Principles - SOLID, DRY, YAGNI, KISS, parse-don't-validate, boring-tech-first. Data structures - balanced trees (AVL/RB/treap), B/B+/LSM, segment/Fenwick/persistent, heaps, hash tables (Robin Hood, cuckoo), graphs, tries (HAMT, radix), Bloom and Cuckoo filters, Count-Min, HyperLogLog, union-find, persistent, lock-free. Algorithms - sorts, graph (Dijkstra, Bellman-Ford, A*, Tarjan, Dinic), strings (KMP, Aho-Corasick, suffix automaton), DP, number theory, geometry, bit hacks, streaming. Use when planning a project, picking an SDLC, drafting a PRD or ADR, architecting a system, choosing between Map/Set/PriorityQueue, picking in-memory vs on-disk indexes, or designing code that stores, indexes, searches, sorts, or streams.


# dsa-pro: Plan, design, and ship the right structures and algorithms

A senior-engineer reference for taking a software project from fuzzy intent to production-grade code, then keeping it production-grade as it evolves. Combines a complete project-planning + SDLC playbook with the original dsa-pro data-structure and algorithm catalogue. Optimized for "translate a fuzzy ask into a deliverable plan and a correct, fast implementation the first time."

## When to invoke

Invoke when *any* of the following are true:

**Planning / architecture / process:**
- "Help me plan / scope / architect / design [a system]" — anything where the user needs to go from intent to plan.
- "What's the right SDLC for [domain]?" — picking between Agile, Lean, DevOps, TDD-first, Waterfall, DDD-style. Regulated domains (medical, finance), greenfield startup, enterprise migration, brownfield refactor — each has a default playbook.
- "Write me a PRD / ADR / spec / RFC / work breakdown / roadmap" — producing planning artifacts.
- "I have a vague idea and I don't know where to start" — discovery + decomposition.
- "How do I split this into modules / services / bounded contexts?" — DDD-style decomposition.
- "How do we test / verify / harden this?" — verification strategy across unit, property, integration, fuzz, contract, chaos.
- "Review my plan / architecture / design" — critique with the same principled lens used to make picks.

**Data-structure / algorithm choice (the original dsa-pro scope):**
- Storing, indexing, searching, sorting, traversing, batching, ranking, joining, deduplicating, or streaming data — and the answer is not a one-liner.
- Choosing between `Map` / `Set` / `PriorityQueue` and the right *shape* (insertion-order? sorted? concurrent? on-disk? approximate?) isn't obvious from the call site.
- Designing an index, cache, queue, batcher, scheduler, or hot data path.
- "Make it faster" lands without a named culprit and the suspect is a container or traversal pattern.
- The problem name maps to a known algorithm (shortest path, k-th element, top-K, substring search, range sum, count distinct, etc.).
- Reviewing code that holds a quadratic loop, naive scan, growing memory, or O(n) operation on a hot path.

If the task is `arr.push(x)` or `Object.keys(o)`, this skill is overkill.

## Core principles (the floor)

Read [`references/principles.md`](references/principles.md) for the full treatment. The shortlist Claude leans on by default:

1. **Specify before naming.** State operations + access pattern + N + constraints *before* naming a structure, a framework, or a methodology. Most wrong answers happen because the question was named wrong.
2. **Stdlib first, boring tech first.** Beating the stdlib, replacing the database, building a custom framework — these are projects, not functions. Reach for them only when you can *name* what the standard option lacks.
3. **Constants and the memory hierarchy dominate Big-O at the sizes you actually run.** L1 ≈ 4 cycles, DRAM ≈ 100–300, NVMe ≈ 10⁵, network ≈ 10⁶⁺. Optimize for the hottest layer the data lives in.
4. **Encode invariants as property tests, not comments.** Every non-trivial structure, module, or workflow has invariants. If you can't write the invariant down as code, you don't fully understand it yet. Property tests against a reference catch refactor regressions that example-based tests miss.
5. **Make invalid states unrepresentable.** Push correctness into types and parsers; avoid validation-as-comment. (See `principles.md` → Parse, don't validate.)
6. **Measure in the target environment.** Big-O sets the lower bound; constants determine whether you ship. Same structure can run 10× faster or slower with different key distributions, cardinalities, working-set sizes, allocators, or JITs.
7. **Match process to risk.** Regulated medical / finance ≠ early-stage MVP ≠ infra refactor. An honest SDLC pick saves more time than any framework choice. See [`references/sdlc.md`](references/sdlc.md).

## Workflow

There are two entry points. Most non-trivial requests start at **Planning** and pass through **Implementation** for each non-trivial module. Trivial drop-ins (one structure, one function, one bug fix) can start at Implementation directly.

### A. Planning workflow (use when the user is going from idea → buildable plan)

Produce artifacts via guided conversation — *not* a 20-page document up front. Ask only the questions whose answers actually change the plan; default to producing a v0 draft and iterating.

1. **Discovery.** What problem, for whom, with what success measure. Existing constraints (compliance, latency, team, language). Surface unknowns explicitly; flag risks. Use `references/templates.md` → *Discovery brief* as the prompt skeleton.
2. **Pick SDLC.** Match project to methodology using `references/sdlc.md` → decision table (regulated → phased + audit trail; greenfield MVP → Lean + discovery-driven; platform refactor → trunk-based + feature flags; algorithm-heavy correctness work → TDD/property-first; complex domain → DDD/hexagonal). The choice drives cadence, ceremony, artifacts, and *which* of the next steps even apply.
3. **Decompose.** Bounded contexts (DDD) → modules → public interfaces. Name invariants at the seam, not just inside each module. State which modules are CRUD, which are algorithm-heavy, which are I/O-heavy — they get different treatment downstream.
4. **For each non-trivial module, run Implementation (B) at design fidelity.** This is where dsa-pro's original DSA workflow lives: restate as operations + access patterns + N + constraints, consult `decision-tree.md`, look up `catalogue.md`, find algorithm in `algorithms.md`. The output is a *named* structure + algorithm pairing per module, with the fallback and trap noted — not a wishlist.
5. **Verification plan.** Per module: invariants (catalogue.md), property tests (verification.md), oracle, fuzz harness, benchmark targets. Tie cadence to SDLC: TDD-first writes the tests first; Agile writes them per story; regulated writes a traceable test matrix. See `references/sdlc.md` for the mapping.
6. **Roadmap + work breakdown.** Translate modules → milestones → tickets. Honest estimates (T-shirt sizes, not story-point theatre). Definition of Done per ticket. Risk register with mitigations. Templates in `references/templates.md`.
7. **Produce the artifacts.** PRD, ADRs (one per non-obvious decision), work breakdown, roadmap, risk register, verification plan. Filed via `scripts/scaffold_project.py` (writes the markdown scaffolds; you fill them in conversationally, not in a vacuum).

Steps 5–7 run in parallel as the project moves through phases; revisit decisions when reality disagrees with the plan (ADR supersedes).

### B. Implementation workflow (DSA selection — the original dsa-pro core)

1. **Restate the problem as `<operations> + <access pattern> + <N> + <constraints>`.** Example: "read-heavy ordered point + range lookups, ~10 M 64-bit keys, in-process, single writer many readers, must survive process crash" — not "I need a tree." This step kills half the wrong answers before they're written.
2. **Consult [`references/decision-tree.md`](references/decision-tree.md)** for the canonical pick. Note the *fallback* and the *trap*.
3. **Look up the structure in [`references/catalogue.md`](references/catalogue.md)** for invariants, complexities, language-specific stdlib names, and production gotchas.
4. **Find the algorithm in [`references/algorithms.md`](references/algorithms.md)** when traversing, transforming, searching, ranking, or aggregating.
5. **Implement.** Start from a stdlib container. Roll custom only when you can name what the stdlib lacks.
6. **Verify** with property tests against a reference implementation. See [`references/verification.md`](references/verification.md) and `scripts/proptest.{ts,py,rs}`.
7. **Benchmark** with the harnesses in `scripts/bench.{ts,py,rs}`. Always report the access pattern alongside numbers — same structure ten times faster or slower depending on key distribution / working set.

## File map

| When | Open |
| --- | --- |
| User is planning a project / asking about SDLC | [`references/sdlc.md`](references/sdlc.md) — the six SDLC traditions, when to use each, anti-patterns |
| Going from fuzzy idea → buildable plan | [`references/project-planning.md`](references/project-planning.md) — full discovery → decomposition → roadmap walkthrough |
| Need PRD / ADR / WBS / risk register / verification plan template | [`references/templates.md`](references/templates.md) |
| Picking principles for a tough trade-off (SOLID, YAGNI, parse-don't-validate, etc.) | [`references/principles.md`](references/principles.md) |
| Picking the right structure for a stated workload | [`references/decision-tree.md`](references/decision-tree.md) |
| Need invariants / Big-O / language stdlib name / production traps for a specific structure | [`references/catalogue.md`](references/catalogue.md) |
| Need an algorithm template (sort, search, graph, string, DP, geometry, bit, stream) | [`references/algorithms.md`](references/algorithms.md) |
| About to ship a custom structure — what to test and how | [`references/verification.md`](references/verification.md) |
| Need a property-test or microbench scaffold | `scripts/proptest.{ts,py,rs}`, `scripts/bench.{ts,py,rs}` |
| Generate planning artifact stubs (PRD, ADR, WBS, etc.) into a project directory | `scripts/scaffold_project.py` |

Pseudocode in references uses a single canonical form; language-specific notes call out stdlib names and gotchas (e.g., Python `heapq` is min-only — negate for max; C++ `std::priority_queue` is max by default; Rust `BinaryHeap` is max; Java `PriorityQueue` is min by natural order). Skip past anything obvious; the references are dense by design.

## Output style

When invoked for planning, prefer **guided conversation that produces artifacts**, not a wall-of-text up front:

- Ask only the questions whose answers actually change the plan. Default to a v0 draft and iterate.
- When the answer is a decision, produce a short ADR (decision + context + alternatives + consequences) rather than a paragraph that buries it.
- When the answer is a structure / algorithm pick, name the pick + the fallback + the trap (the decision-tree format).
- When the user asks for a "complete plan," save artifacts to a project directory via `scripts/scaffold_project.py` and then walk through them inline — don't make the user open and read each one before the conversation can continue.
- Match formality to the SDLC pick: a Lean MVP plan is a one-pager; a regulated medical device plan needs the traceability matrix.

## What this skill is not

- **Not a substitute for product judgment.** The skill can structure a discovery conversation and challenge underspecified asks, but it doesn't know your users.
- **Not a replacement for the type checker, linter, or formatter.** Encode invariants in types where the language allows; this skill complements, not replaces, that work.
- **Not a license to ignore constants.** Big-O reasoning sets the lower bound. The bench in the prod-shaped environment is the final word.

---
> Source: [ananyakaushik48/dsa-pro](https://github.com/ananyakaushik48/dsa-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
