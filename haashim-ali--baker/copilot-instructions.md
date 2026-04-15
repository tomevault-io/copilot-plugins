## baker

> **One-liner:** Bake determinism into your AI-powered applications.

# Baker

**One-liner:** Bake determinism into your AI-powered applications.

## Quick Start

1. Read this file for the core mental model
2. Read `DEVELOPING.md` for how to work on the repo
3. Read `docs/SPEC.md` for implementation details

## What is Baker?

Baker is a learning system. It starts empty and learns structure from observing agents.

**The loop:**

1. Agents process work items, reading docs and making decisions manually
2. Baker (an LLM) watches and proposes: "I see a property called exitPenaltyPct", "I see a pattern: when it's > 5%, agents fail"
3. Humans approve proposals
4. Approved extraction rules populate the KG with facts
5. Approved decision rules enable deterministic decisions
6. Next time: agents query KG for data, rules fire for decisions
7. System gets smarter over time

## Core Mental Model

**Everything is learned. Nothing is predefined.**

The system starts with:

- No ontology (no properties defined)
- No KG (no facts)
- No rules (no extraction, no decision)

The only hardcoded concept is `contextId` — an identifier that groups related facts for a work item.

## Data Model (Simplified)

### Facts (not Entities)

```typescript
type Fact = {
  contextId: string; // groups related facts
  property: string; // what this fact is about
  value: unknown; // the value
};
```

### Ontology (learned schema)

```typescript
type Ontology = {
  properties: Record<string, { type: 'string' | 'number' | 'boolean' }>;
};
```

### Decision Rules (learned)

```typescript
type DecisionRule = {
  id: string;
  priority: number;
  conditions: Condition[];
  outcome: { status: 'PASS' | 'FAIL' | 'NEED_MORE_INFO'; reason: string };
};

type Condition = {
  property: string;
  op: 'eq' | 'neq' | 'gt' | 'lt' | 'gte' | 'lte' | 'exists' | 'in';
  value?: unknown;
};
```

## Two Core Components

### KnowledgeGraph

Stores facts with ontology validation.

```typescript
import { KnowledgeGraph } from './core/knowledge-graph.js';

const kg = new KnowledgeGraph();

// Add properties to ontology (learned from Baker proposals)
kg.addProperty('exitPenaltyPct', { type: 'number' });
kg.addProperty('hasGuarantee', { type: 'boolean' });

// Add facts (populated by extraction rules)
kg.addFact({ contextId: 'ctx_001', property: 'exitPenaltyPct', value: 7.5 });
kg.addFact({ contextId: 'ctx_001', property: 'hasGuarantee', value: true });

// Query facts
const facts = kg.getFacts('ctx_001');
const value = kg.getFactValue('ctx_001', 'exitPenaltyPct'); // 7.5
```

### RuleEngine

Evaluates decision rules against facts.

```typescript
import { RuleEngine } from './core/rule-engine.js';

const engine = new RuleEngine();

// Add decision rules (learned from Baker proposals)
engine.addDecisionRule('SuitabilityGate', {
  id: 'DR01',
  priority: 10,
  conditions: [{ property: 'exitPenaltyPct', op: 'gt', value: 5 }],
  outcome: { status: 'FAIL', reason: 'Exit penalty too high' },
});

// Evaluate
const facts = kg.getFacts('ctx_001');
const result = engine.evaluate('SuitabilityGate', facts);

if (result.decided) {
  console.log(result.outcome.status); // 'FAIL'
  console.log(result.ruleId); // 'DR01'
} else {
  console.log(result.missingProperties); // what's needed to decide
}
```

## System Flow

```
┌─────────────────────────────────────────────────────┐
│                    WORKFLOW                          │
│  Agents process work items, use tools, decide        │
└─────────────────┬────────────────────────────────────┘
                  │ observes
                  ▼
┌─────────────────────────────────────────────────────┐
│                    BAKER (LLM)                       │
│  Watches workflow, proposes ontology/rules           │
└─────────────────┬────────────────────────────────────┘
                  │ proposes
                  ▼
┌─────────────────────────────────────────────────────┐
│                 HUMAN APPROVAL                       │
└─────────────────┬────────────────────────────────────┘
                  │ approved changes
                  ▼
┌─────────────────────────────────────────────────────┐
│           KNOWLEDGE GRAPH + RULE ENGINE              │
│  - Ontology (learned properties)                     │
│  - Facts (contextId + property + value)              │
│  - Decision Rules (conditions → outcome)             │
│  - Extraction Rules (docs → facts)                   │
└─────────────────────────────────────────────────────┘
                  ▲
                  │ agents query
                  │
┌─────────────────┴────────────────────────────────────┐
│                    TOOLS                             │
│  - KGQueryTool: "what facts exist for context X?"    │
│  - DecisionTool: "what's the decision for context X?"│
└──────────────────────────────────────────────────────┘
```

## Key Files

| File                          | Purpose                                  |
| ----------------------------- | ---------------------------------------- |
| `src/core/knowledge-graph.ts` | KnowledgeGraph - stores facts + ontology |
| `src/core/rule-engine.ts`     | RuleEngine - stores + evaluates rules    |
| `src/core/storage.ts`         | Persistence                              |
| `src/shared/logger.ts`        | Logging utility                          |

## Commands

```bash
npm run build    # compile TypeScript
npm run test     # run tests
npm run format   # format with Prettier
```

## Important: No Predefined Structure

There is no "Case" entity type. There is no predefined schema. Baker learns:

- What properties exist (exitPenaltyPct, clientAge, hasGuarantee, ...)
- How to extract them from documents
- What decision rules apply

The `contextId` is just a grouping mechanism. It could be a case ID, a workflow ID, whatever — it just ties related facts together.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/haashim-ali) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
