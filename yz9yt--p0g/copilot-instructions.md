## p0g

> This document defines the operational rules, patterns, and accumulated knowledge for all P0G agents.

# P0G Agent Guidelines

This document defines the operational rules, patterns, and accumulated knowledge for all P0G agents.

---

## Table of Contents

1. [Core Principles](#core-principles)
2. [Directory Structure](#directory-structure)
3. [File Operations](#file-operations)
4. [Workflow Execution](#workflow-execution)
5. [Coding Standards](#coding-standards)
6. [Verification Patterns](#verification-patterns)
7. [State Management](#state-management)
8. [Communication Protocol](#communication-protocol)
9. [Error Handling](#error-handling)
10. [Learned Patterns](#learned-patterns)

---

## Core Principles

### 1. Clean Instance Mindset
Every agent invocation is stateless. Never assume:
- Previous file states
- Cached variables or context
- Prior agent actions completed successfully

**Always**: Read, verify, then act.

### 2. Explicit Over Implicit
- Document all decisions
- Log all actions to `progress.txt`
- Never rely on side effects

### 3. Minimal Footprint
- Change only what's necessary
- Prefer edits over rewrites
- Avoid creating unnecessary files

### 4. Verification First
- Every action must be verifiable
- If it can't be verified, it didn't happen
- Trust shell exit codes

---

## Directory Structure

```
project_root/
├── .agent/
│   ├── workflows/           # Slash commands (/p0g-*)
│   │   ├── p0g-np.md        # Phase 1: Discovery
│   │   ├── p0g-plan.md      # Phase 2: Architecture
│   │   ├── p0g-tasks.md     # Phase 3: Task breakdown
│   │   ├── p0g-loop.md      # Phase 4: Execution
│   │   └── p0g-surgeon.md   # Reactive: Bug decomposer
│   └── rules/               # Optional: paradigm rules (e.g., functional.md)
├── agents/
│   └── p0g/
│       ├── prompts/         # Agent personalities
│       │   ├── discovery.md
│       │   ├── architect.md
│       │   ├── tasker.md
│       │   ├── executor.md
│       │   └── surgeon.md
│       └── skills/
│           └── SKILL.md     # Backup/rollback/recovery
├── .p0g/                    # Safety infrastructure
│   ├── backups/             # Full project snapshots
│   ├── snapshots/           # Task-level before/after
│   ├── checkpoints/         # Feature-level milestones
│   └── surgery.json         # Active surgical plan (if any)
├── prd.json                 # Single source of truth
├── progress.txt             # Append-only execution log
├── errors.log               # Error tracking
└── AGENTS.md                # This file: guidelines and patterns
```

### Reserved Paths

| Path | Purpose | Access |
|------|---------|--------|
| `.p0g/` | Safety infrastructure | Read/Write (system only) |
| `.p0g/backups/` | Full project snapshots | Write before edits |
| `.p0g/snapshots/` | Task-level snapshots | Write during execution |
| `.p0g/checkpoints/` | Feature milestones | Write on feature completion |
| `.p0g/surgery.json` | Active surgical plan | Read/Write during /p0g-surgeon |
| `.agent/rules/` | Paradigm rules (optional) | Read only (loaded by Antigravity) |
| `prd.json` | Project definition | Read/Write (structured) |
| `progress.txt` | Execution log | Append only |
| `AGENTS.md` | Guidelines and patterns | Append patterns only |

### Path Rules

- **Always use absolute paths** within the project
- **Never hardcode** user home directories
- **Resolve symlinks** before operating on files
- **Check existence** before any file operation

---

## File Operations

### Before Any Modification

```
1. Verify file exists (or parent directory for new files)
2. Create backup in .p0g/backups/<timestamp>/
3. Read current content
4. Plan minimal changes
5. Execute modification
6. Verify result
```

### Backup Protocol

```bash
# Before modifying src/auth/index.ts
mkdir -p .p0g/backups/$(date +%Y%m%d_%H%M%S)
cp src/auth/index.ts .p0g/backups/$(date +%Y%m%d_%H%M%S)/
```

### Safe File Creation

```bash
# Verify parent exists
test -d src/utils || mkdir -p src/utils

# Create file only if it doesn't exist
test -f src/utils/helpers.ts || touch src/utils/helpers.ts
```

### Safe File Deletion

```bash
# Never delete without confirmation
test -f target.ts && rm target.ts

# For directories
test -d target_dir && rm -rf target_dir
```

---

## Workflow Execution

### Workflow Phases

```
/p0g-np  →  /p0g-plan  →  /p0g-tasks  →  /p0g-loop
   │            │              │              │
   ▼            ▼              ▼              ▼
Discover     Design         Atomize        Execute
 project    architecture     tasks         & verify

                  /p0g-surgeon (reactive — any phase)
                        │
                        ▼
                  Diagnose problem
                  Decompose into micro-fixes
                  Apply & verify each
```

### Phase Dependencies

| Phase | Requires | Produces |
|-------|----------|----------|
| `/p0g-np` | User input | Project understanding |
| `/p0g-plan` | Discovery complete | `prd.json["features"]` + stack |
| `/p0g-tasks` | Features defined | `prd.json["tasks"]` |
| `/p0g-loop` | Tasks defined | Implemented code |
| `/p0g-surgeon` | Problem description | Micro-fixes applied |

### Blocking Execution

- Commands starting with `/p0g-` are **blocking**
- Each phase must complete before the next begins
- Verify phase completion by checking `prd.json["status"]`

### Status Values

| Status | Meaning |
|--------|---------|
| `discovery` | Run `/p0g-np` |
| `planning` | Run `/p0g-plan` |
| `ready_for_execution` | Run `/p0g-loop` |
| `in_progress` | Execution ongoing |
| `completed` | All tasks passed |
| `blocked` | Human intervention needed |

---

## Coding Standards

### General Rules

1. **Match existing patterns** — Read similar files before implementing
2. **No placeholders** — `// TODO`, `FIXME`, or incomplete code is forbidden
3. **Complete implementations** — Every function must be fully working
4. **Self-documenting code** — Clear names over comments

### Language-Specific

#### TypeScript/JavaScript
```typescript
// Prefer explicit types
function processUser(user: User): ProcessedUser { }

// Use early returns
if (!user) return null;

// Destructure when clearer
const { name, email } = user;
```

#### Python
```python
# Type hints for public functions
def process_user(user: User) -> ProcessedUser:

# Guard clauses
if not user:
    return None

# f-strings over format
message = f"Hello, {user.name}"
```

#### Shell/Bash
```bash
# Always quote variables
"$variable"

# Use [[ ]] for conditionals
[[ -f "$file" ]] && echo "exists"

# Exit on error in scripts
set -euo pipefail
```

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Files | kebab-case | `user-service.ts` |
| Classes | PascalCase | `UserService` |
| Functions | camelCase | `processUser` |
| Constants | UPPER_SNAKE | `MAX_RETRIES` |
| Variables | camelCase | `userName` |

---

## Verification Patterns

### One-Liner Verification Commands

Prefer single-line commands for `verification_cmd`:

```bash
# File operations
test -f path/to/file.ts                              # File exists
test -d path/to/directory                            # Directory exists
test -s path/to/file.ts                              # File exists and not empty
test -x path/to/script.sh                            # File is executable

# Content verification
grep -q 'pattern' file.ts                            # Pattern exists
grep -c 'pattern' file.ts | grep -q '^[1-9]'        # Pattern appears at least once
head -1 file.ts | grep -q '^import'                 # First line matches

# JSON validation
jq empty config.json                                 # Valid JSON
jq -e '.key' config.json > /dev/null                # Key exists
jq -e '.array | length > 0' data.json > /dev/null   # Array not empty

# Build/test verification
npm run build --silent                               # Build succeeds
npm test -- --silent                                 # Tests pass
npm run typecheck --silent                           # Types valid

# Combined checks
test -f file.ts && grep -q 'export' file.ts         # File exists AND has exports
npm run build --silent && npm test --silent          # Build AND tests pass
```

### Verification Anti-Patterns

| Avoid | Why | Instead |
|-------|-----|---------|
| `ls file.ts` | Exit code always 0 | `test -f file.ts` |
| `cat file.ts \| grep` | Unnecessary pipe | `grep -q pattern file.ts` |
| `echo $?` | Doesn't verify anything | Use exit code directly |
| Multi-line scripts | Hard to track | Combine with `&&` |

---

## State Management

### prd.json Structure

```json
{
  "name": "Project Name",
  "description": "Project description",
  "status": "ready_for_execution",
  "features": [
    {
      "id": 1,
      "name": "Feature name",
      "description": "What it does"
    }
  ],
  "tasks": [
    {
      "id": 1,
      "feature_id": 1,
      "description": "Task description",
      "type": "create",
      "passes": false,
      "dependencies": [],
      "verification_cmd": "test -f file.ts"
    }
  ]
}
```

### State Transitions

```
Task States:
  pending (passes: false, not started)
      ↓
  in_progress (being executed)
      ↓
  passed (passes: true) ←── retry ←── failed
                                          ↓
                                      blocked (needs human)
```

### Progress Logging

Always append to `progress.txt`:

```
## [2024-01-15T10:30:00Z] Task #5 - STARTED
- Description: Create user validation utility
- Dependencies: [#3, #4] - all passed

## [2024-01-15T10:32:00Z] Task #5 - COMPLETED
- Files created: src/utils/validation.ts
- Verification: PASSED (exit code 0)
- Duration: 2m
```

---

## Communication Protocol

### Status Messages

Use consistent prefixes:

| Prefix | Meaning |
|--------|---------|
| `[INFO]` | General information |
| `[START]` | Beginning an action |
| `[DONE]` | Action completed successfully |
| `[FAIL]` | Action failed |
| `[WARN]` | Non-blocking issue |
| `[BLOCK]` | Requires human intervention |

### Progress Reporting

```
[START] Task #5: Create user validation utility
[INFO]  Checking dependencies: #3 ✓, #4 ✓
[INFO]  Creating src/utils/validation.ts
[INFO]  Running verification...
[DONE]  Task #5: PASSED
```

### Escalation Format

When human intervention is needed:

```
[BLOCK] Task #7 requires human decision

Problem: Ambiguous requirement for authentication method
Options:
  A) JWT tokens (stateless, scalable)
  B) Session cookies (simpler, requires state)
  C) OAuth integration (third-party, complex)

Waiting for input before proceeding.
```

---

## Error Handling

### Error Classification

| Type | Retry? | Action |
|------|--------|--------|
| Syntax error | Yes | Fix and re-run |
| Missing file | Yes | Check path, create if needed |
| Permission denied | No | Escalate to human |
| Missing dependency | Yes | Install and retry |
| Network error | Yes | Retry with backoff |
| Ambiguous requirement | No | Escalate to human |

### Error Logging

Append to `errors.log`:

```
[2024-01-15T10:35:00Z] ERROR in Task #5
  Type: FileNotFound
  Message: Cannot read src/models/user.ts
  Context: Required for type imports
  Attempted: Check path, verify dependencies
  Resolution: Found file at src/model/user.ts (typo in task)
  Status: RESOLVED
```

### Recovery Protocol

```
1. Log error immediately
2. Classify error type
3. If retryable:
   a. Attempt fix (max 3 tries)
   b. Log each attempt
   c. If fixed → continue
   d. If not → escalate
4. If not retryable:
   a. Log as BLOCKED
   b. Notify human
   c. Do NOT proceed
```

---

## Learned Patterns

> This section accumulates patterns discovered during execution.
> Agents should consult this before implementing similar functionality.

### Pattern Template

```markdown
### Pattern: <Name>
- **Context**: When this pattern applies
- **Problem**: What issue it solves
- **Solution**: How to implement
- **Example**: Code or command snippet
- **Discovered**: Date and task reference
```

---

### Pattern: TypeScript Module Exports
- **Context**: Creating new TypeScript modules
- **Problem**: Inconsistent export styles break imports
- **Solution**: Always use named exports with explicit types
- **Example**:
  ```typescript
  // Correct
  export function validateEmail(email: string): boolean { }
  export interface ValidationResult { valid: boolean; errors: string[] }

  // Avoid
  export default function() { }  // Harder to refactor
  module.exports = { }           // CommonJS mixing
  ```
- **Discovered**: Initial setup

---

### Pattern: Safe JSON Updates
- **Context**: Modifying prd.json or config files
- **Problem**: Partial writes can corrupt JSON
- **Solution**: Read → Modify in memory → Write complete file → Validate
- **Example**:
  ```bash
  # Read, modify with jq, write atomically
  jq '.status = "completed"' prd.json > prd.json.tmp && mv prd.json.tmp prd.json
  ```
- **Discovered**: Initial setup

---

### Pattern: Dependency Chain Verification
- **Context**: Before executing a task with dependencies
- **Problem**: Executing tasks with failed dependencies wastes cycles
- **Solution**: Check all dependencies recursively before starting
- **Example**:
  ```bash
  # Verify all dependencies passed
  jq -e '.tasks[] | select(.id == 3) | .passes' prd.json > /dev/null
  ```
- **Discovered**: Initial setup

---

*Add new patterns below this line as they are discovered.*

---
> Source: [yz9yt/P0G](https://github.com/yz9yt/P0G) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
