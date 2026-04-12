## shutter-hub

> For large features, use the spec-based development approach:

# Spec-Based Development Workflow

For large features, use the spec-based development approach:

## Step 1: Create Minimal Spec
Create a minimal specification file in `.cursor/plans/` or provide it directly:

```
新しい機能: [機能名]
基本的な要件: [最小限の要件]
```

## Step 2: Deep Interview
Ask in-depth questions about:
- **Technical Implementation**: Integration with existing systems, performance requirements, error handling, data consistency, security, scalability
- **UI/UX Details**: User journey, edge cases, accessibility, responsive design, animations, error states
- **Business Logic**: Business rules, exception handling, data validation, workflow branching, notification timing
- **Trade-offs**: Complexity vs completeness, performance vs development speed, flexibility vs simplicity, future extensibility vs current requirements

**Important**: Ask non-obvious questions. Don't ask about things that can be determined from existing code (e.g., "Will we use React?" when the project already uses React).

## Step 3: Generate Specification
After the interview is complete, generate a comprehensive specification document including:
- Functional and non-functional requirements
- Technical specifications (database design, API design, UI/UX design)
- Implementation plan with step-by-step procedures
- Important notes and constraints

Save the specification to `.cursor/plans/[feature-name]_spec_[timestamp].md`

## Step 4: Implementation in New Session
Use the generated specification to implement the feature in a new session.

## Example Usage in Claude Code

```
I want to build a new feature: User notification system
Basic requirements: Support both email and push notifications

Please read this CLAUDE.md file and interview me in detail about technical implementation, UI/UX, concerns, tradeoffs, etc.
Make sure the questions are not obvious and be very in-depth.
Continue interviewing until it's complete, then write the spec to a file.
```

## Example Usage in Cursor

```
@spec-interview.md .cursor/plans/notification-system.md
```

## Interview Checklist
When conducting the interview, ensure you cover:
- [ ] Technical implementation details (5+ deep questions)
- [ ] UI/UX details (3+ questions)
- [ ] Business logic clarification (3+ questions)
- [ ] Trade-off considerations (2+ alternative options with pros/cons)
- [ ] Edge cases (3+ cases)
- [ ] Security and permissions (2+ questions)
- [ ] Performance requirements (2+ questions)
- [ ] Future extensibility (1+ question)

## Question Examples (Non-Obvious)

❌ **Avoid obvious questions:**
- "Will we use React?" (project already uses React)
- "Will we use Supabase?" (already using Supabase)

✅ **Ask deep questions:**
- "What's the expected concurrent access for this feature? Do we need rate limiting?"
- "How much information should we show users when errors occur? Are there security concerns?"
- "Will this feature be extended to other user types (model, photographer) in the future?"
- "Do we need transaction processing to maintain data consistency?"
- "What are the performance targets for this feature? (response time, throughput)"
- "How should users recover from mistakes? Is there an undo mechanism?"

**Reference**: See `.cursor/commands/spec-interview.md` for detailed workflow, question templates, and best practices.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ShuheiHasegawa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ShuheiHasegawa)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
