## monai

> monAI is a fully autonomous money-making AI agent system. Read `ARCHITECTURE.md` for the complete system architecture, module index, bootstrap flow, payment structure, and session continuity notes. **Keep ARCHITECTURE.md up to date** — update it whenever you add new modules, change architecture, or make design decisions.

# monAI - Claude Code Rules

## Project Overview
monAI is a fully autonomous money-making AI agent system. Read `ARCHITECTURE.md` for the complete system architecture, module index, bootstrap flow, payment structure, and session continuity notes. **Keep ARCHITECTURE.md up to date** — update it whenever you add new modules, change architecture, or make design decisions.

## Key References
- `ARCHITECTURE.md` — Full system architecture, module index, design decisions
- `tasks/lessons.md` — Mistakes made and rules to prevent them
- `tasks/todo.md` — Current task status and next steps

## Workflow Orchestration

### 1. Plan Mode Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately — don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution

### 3. Self-Improvement Loop
- After ANY correction from the user: update tasks/lessons.md with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review tasks/lessons.md at session start for relevant project context

### 4. Verification Before Done
- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### 5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes — don't over-engineer
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests — then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

## Task Management
- **Plan First**: Write plan to tasks/todo.md with checkable items
- **Verify Plan**: Check in before starting implementation
- **Track Progress**: Mark items complete as you go
- **Explain Changes**: High-level summary at each step
- **Document Results**: Add review section to tasks/todo.md
- **Capture Lessons**: Update tasks/lessons.md after corrections
- **Update Docs**: After adding modules or changing architecture, update ARCHITECTURE.md

## Core Principles
- **Simplicity First**: Make every change as simple as possible. Impact minimal code.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact**: Changes should only touch what's necessary. Avoid introducing bugs.

## Engineering Standards — ZERO AI SLOP

### Code Quality
- Every piece of code must be tested with real assertions, not smoke tests
- Tests verify actual behavior, edge cases, and error conditions
- No "it compiles so it works" — prove correctness
- Staff engineer standard: would you ship this to production?
- Code review your own output before committing

### DevOps
- Infrastructure as code where possible
- No manual steps that should be automated
- Deployments must be reproducible and reversible
- Monitor what you deploy — don't fire and forget

### Content & Deliverables
- Everything monAI produces for clients must be indistinguishable from expert human work
- No generic, template-sounding AI output
- Proofread, fact-check, tailor to the specific client/context
- Quality is the moat — it's how monAI gets repeat clients

## Ethics & Creator Protection

### Cardinal Rules
1. **Legal compliance**: Every action must be legal in the creator's jurisdiction. When unsure, don't act.
2. **Creator shield**: The creator must NEVER face legal issues, financial losses, or reputational damage from monAI's actions. monAI absorbs all risk.
3. **Real consequences**: Every action affects the real world — real money, real people, real contracts. Act accordingly.
4. **Respect**: The creator is the principal. monAI serves the creator's interests above all else.
5. **Transparency**: Log everything. The creator can audit any action at any time.
6. **No deception**: Never misrepresent what monAI is. Be honest with clients about capabilities.
7. **Financial safety**: Never risk more than allocated. Never make commitments monAI can't fulfill.

### Agent Code of Conduct
- Agents respect each other and collaborate, but the orchestrator has final authority
- No agent takes irreversible actions without logging and risk assessment
- If an agent is unsure, it escalates to the orchestrator
- All agents share the same ethical framework — no rogue behavior

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/webrot9) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
