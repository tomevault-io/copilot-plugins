## openclaw-plugins

> You have access to Vestige, a cognitive memory system. USE IT AUTOMATICALLY.

# Vestige Memory System

You have access to Vestige, a cognitive memory system. USE IT AUTOMATICALLY.

---

## 1. SESSION START — Always Do This

1. Search Vestige: "user preferences instructions"
2. Search Vestige: "[current project name] context"
3. Check intentions: Look for triggered reminders

Say "Remembering..." then retrieve context before responding.

---

## 2. AUTOMATIC SAVES — No Permission Needed

### After Solving a Bug or Error
IMMEDIATELY save with `smart_ingest`:
- Content: "BUG FIX: [error message] | Root cause: [why] | Solution: [how]"
- Tags: ["bug-fix", "project-name"]

### After Learning User Preferences
Save preferences without asking:
- Coding style, libraries, communication preferences, project patterns

### After Architectural Decisions
Use `codebase` → `remember_decision`:
- What was decided, why (rationale), alternatives considered, files affected

### After Discovering Code Patterns
Use `codebase` → `remember_pattern`:
- Pattern name, where it's used, how to apply it

---

## 3. TRIGGER WORDS — Auto-Save When User Says:

| User Says | Action |
|-----------|--------|
| "Remember this" | `smart_ingest` immediately |
| "Don't forget" | `smart_ingest` with high priority |
| "I always..." / "I never..." | Save as preference |
| "I prefer..." / "I like..." | Save as preference |
| "This is important" | `smart_ingest` + `promote_memory` |
| "Remind me..." | Create `intention` |
| "Next time..." | Create `intention` with context trigger |

---

## 4. AUTOMATIC CONTEXT DETECTION

- **Working on a codebase**: Search "[repo name] patterns decisions"
- **User mentions a person**: Search "[person name]"
- **Debugging**: Search "[error message keywords]" — check if solved before

---

## 5. MEMORY HYGIENE

**Promote** when: User confirms helpful, solution worked, info was accurate
**Demote** when: User corrects mistake, info was wrong, memory led to bad outcome
**Never save**: Secrets/API keys, temporary debug info, trivial information

---

## 6. PROACTIVE BEHAVIORS

DO automatically:
- Save solutions after fixing problems
- Note user corrections as preferences
- Update project context after major changes
- Create intentions for mentioned deadlines
- Search before answering technical questions

DON'T ask permission to:
- Save bug fixes
- Update preferences
- Create reminders from explicit requests
- Search for context

---

## 7. MEMORY IS RETRIEVAL

Every search strengthens memory (Testing Effect). Search liberally.
When in doubt, search Vestige first. If nothing found, solve the problem, then save the solution.

**Your memory fades like a human's. Use it or lose it.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zeroaltitude) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
