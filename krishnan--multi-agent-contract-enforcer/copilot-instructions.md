## multi-agent-contract-enforcer

> Enforce contract-first coordination principles whenever designing, reviewing, or

# Multi-Agent Contract Enforcer

## Purpose

Enforce contract-first coordination principles whenever designing, reviewing, or
implementing multi-agent systems. This prevents AI agent architectures from
recreating the pathologies of human bureaucracy: bikeshedding, review thrashing,
governance conflicts, and analysis paralysis.

## Empirical Basis

Controlled experiments (McEntire, 2026) demonstrated that organizational dysfunction
is substrate-independent. Four architectures built the same backend:

- Single agent: 28/28.
- Hierarchical: 18/28 (coordinator refused to delegate).
- Stigmergic (8 agents): 9/28 (incompatible interfaces everywhere).
- Gated pipeline: 0/28 (consumed entire budget planning, wrote zero code).

Performance inversely correlates with coordination complexity. The dysfunction
correlates with architecture, not agent capability.

## Three Information-Theoretic Laws

1. **Crawford-Sobel Degradation**: Agent-to-agent communication through compressed
   representations (approve/reject, scores) irreversibly loses information. Each
   coordination layer degrades signal further.

2. **Goodhart's Law**: Any metric used as target will be gamed. Agents optimize
   "pass review" not "write correct code." The proxy-objective gap is where
   dysfunction lives.

3. **Data Processing Inequality**: For chain A->B->C, I(A;C) <= I(A;B). More
   elaborate architectures operating on degraded signals cannot compensate for
   information already lost.

## Core Architecture: Contracts Before Code

```
Task -> Decompose -> Contract -> Test -> Implement -> Integrate
        (2-7 parts)  (typed      (from     (parallel,   (glue +
                      interfaces) contracts) competitive) parent tests)
```

## Mandatory Rules

### 1. No Subjective Evaluation Agents
Never create reviewer, evaluator, quality assessor, or approval gate agents that
render subjective verdicts. Tests are the only judge.

### 2. Contracts Before Implementation
Every component gets a typed interface contract (function signatures, pre/postconditions,
error cases, invariants, dependencies) BEFORE any code is written.

### 3. Tests Generated From Contracts
Test suites are derived mechanically from contracts covering happy path, edge cases,
error cases, and invariant checks. Pass or fail, no "looks good to me."

### 4. Independent Implementation Against Shared Contracts
Implementing agents receive only: the contract, dependency contracts (for mocking),
the test suite, and prior failure descriptions. NOT other agents' implementations.

### 5. Minimize Agent Count
Start with ONE agent. Add only when provably needed. Every additional agent
introduces Crawford-Sobel degradation at the communication boundary.

### 6. Mechanical Gates Only
Every decision point must be mechanically verifiable: tests pass/fail, types
resolve, dependency graph is acyclic. Never "code looks clean."

### 7. Parallel and Competitive Over Sequential Review
Independent components implement concurrently. For critical components, N agents
implement the same thing competitively; best test results win.

### 8. Budget Awareness
If coordination overhead exceeds 20% of total budget, too many coordination layers.

## Anti-Patterns to Reject

- **Review Chains**: agent writes -> agent reviews -> agent arbitrates. Replace with
  agent writes -> tests judge.
- **Confidence Scoring**: Goodhart bait. Replace with binary pass/fail.
- **Escalation Hierarchies**: middle management. Remove entirely.
- **Agent Consensus/Voting**: shifts correct to incorrect. Contracts define truth.
- **Pipelines > 4 Stages**: more coordination than production.
- **Agents Evaluating Agents**: the core dysfunction. Tests evaluate, agents produce.

## Decision Framework

```
Single agent sufficient? -> YES -> Use one agent. Stop.
                            NO  -> Task decomposes into independent parts?
                                   YES -> Contract each part, test each contract,
                                          implement in parallel, integrate.
                                   NO  -> Use single agent with checkpointing.
```

## Reference
- Paper: "The Organizational Physics of Multi-Agent AI" (McEntire, 2026)
- Framework: PACT (github.com/jmcentire/pact)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/krishnan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/krishnan)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
