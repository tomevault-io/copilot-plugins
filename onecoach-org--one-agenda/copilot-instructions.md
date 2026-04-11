## one-agenda

> > **Meta**: Proactive planning engine. Takes tasks, events, and preferences as input; generates optimized daily plans as output.

# OneAgenda - Agent Guide

> **Meta**: Proactive planning engine. Takes tasks, events, and preferences as input; generates optimized daily plans as output.

## Quick Reference

| Attribute   | Value                                      |
| :---------- | :----------------------------------------- |
| **Package** | `@onecoach/one-agenda`                     |
| **Purpose** | Algorithmic + AI-powered schedule planning |
| **Pattern** | Pure/stateless planning functions          |

---

## Directory Structure

```
submodules/one-agenda/
├── src/
│   ├── domain/        # Domain types (Task, Event, Plan)
│   │   └── types.ts   # Zod schemas - SOURCE OF TRUTH
│   ├── planner/       # Deterministic planning algorithm
│   │   └── plan-day.ts
│   ├── agentic/       # LLM-powered planning (Vercel AI SDK)
│   │   └── OneAgendaAgenticPlanner.ts
│   ├── connectors/    # External data (Google/Outlook calendars)
│   └── index.ts       # Public API
```

---

## Key Concepts

### Input (`PlannerInput`)

```typescript
interface PlannerInput {
  tasks: Task[]; // With estimates, priority, deadline
  events: Event[]; // Fixed calendar events
  preferences: Preference[]; // Work hours, timezone
  constraints: Constraint[]; // Hard/soft constraints
}
```

### Output (`Plan`)

```typescript
interface Plan {
  summary: PlanSummary; // Focus minutes, slack time
  blocks: PlanBlock[]; // Time-ordered schedule
  decisions: Decision[]; // Why this was scheduled here
  risks: Risk[]; // Potential issues detected
}
```

---

## Usage

### Deterministic Planning

For rule-based scheduling:

```typescript
import { planDay } from '@onecoach/one-agenda';

const plan = await planDay(input);
```

### Agentic Planning (LLM)

For reasoning, "what-if" scenarios, or unstructured input:

```typescript
import { OneAgendaAgenticPlanner } from '@onecoach/one-agenda';

const agent = new OneAgendaAgenticPlanner(options);
const plan = await agent.plan('Move my workout to evening');
```

---

## Development Guidelines

> [!IMPORTANT]
>
> - **Zod**: All types validated at runtime. Update schemas in `src/domain/types.ts`.
> - **Stateless**: `planDay` is pure. Input in, output out. No DB writes.
> - **Dates**: Use `date-fns` with ISO strings.

---

## Navigation Guide

| Task                      | Location                  |
| :------------------------ | :------------------------ |
| Add/modify types          | `src/domain/types.ts`     |
| Modify planning algorithm | `src/planner/plan-day.ts` |
| Add AI reasoning          | `src/agentic/`            |
| Add calendar connector    | `src/connectors/`         |
| Check public API          | `src/index.ts`            |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OneCoach-org)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/OneCoach-org)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
