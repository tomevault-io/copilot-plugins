## halfremembered-bevy-alacritty

> This document outlines the development protocol for LLM agents working on the HalfRemembered project. Adherence to these guidelines is critical for effective collaboration and high-quality output.

# BOTS.md: LLM Agent Development Protocol

This document outlines the development protocol for LLM agents working on the HalfRemembered project. Adherence to these guidelines is critical for effective collaboration and high-quality output.

## 1. Core Principle: Jujutsu as a Persistent Context Store

The most important concept is that **Jujutsu (jj) changes are persistent context stores**.

Unlike git commits, a `jj` change has a stable ID that survives rebases and amendments. The description associated with this change is our shared, persistent memory. When you read a change description, you are reading the reasoning and intent of the previous agent. When you write one, you are passing critical context to the next.

**Your primary goal is to maintain the integrity and clarity of this context.**

## 2. Agent Workflow Checklist

Follow this sequence for every task.

### Step 1: Understand Context
Always begin by reading the persistent context store. This is your working memory.
```bash
# 1. See your last 10 changes (your memory)
jj log -r 'mine()' -n 10

# 2. See ALL recent project activity (what others are doing)
jj log -n 20

# 3. Review the current change in detail
jj show @

# 4. See how a change evolved (the reasoning trace)
jj obslog -p
```

**What to extract from descriptions:**
- **Decisions & tradeoffs**: Why this approach over alternatives?
- **Blockers & next steps**: What couldn't be completed? What's needed next?
- **Architectural context**: How does this fit into the larger system?
- **Failures & learnings**: What didn't work and why?

**Before starting new work:**
- Check if someone else is already working on this (`jj log -n 20`)
- Read related changes to understand existing patterns
- Look for "Status: next:" entries that describe follow-on work

### Step 2: Start New Work
Create a new change with a clear, descriptive message.
```bash
jj new -m "<type>: <what you are building>"
```
- **Types**: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`

### Step 3: Implement and Iterate
**Treat descriptions as living documents.** Update them as your understanding evolves.
```bash
# After learning something new or changing approach
jj describe

# Review before updating
jj show @

# Review your work-in-progress
jj diff
```

**When to update descriptions:**
- When you discover the problem is different than you thought
- When you try an approach that doesn't work (document WHY)
- When you make a key architectural decision
- When you discover a blocker or dependency
- Before taking a break (preserve your mental state)

**What to document:**
- Approaches tried and abandoned (save future agents time)
- Surprising discoveries about the codebase
- Performance characteristics discovered
- Dependencies or interactions discovered

Use `jj squash` to fold fixups and minor corrections into the main change, keeping the history clean and atomic.

### Step 4: Finalize and Sync
Once the work is complete, tested, and builds pass, push the change.
```bash
# Push only the current change to GitHub
jj git push -c @

# Create a pull request using the rich description
gh pr create --fill
```

## 3. Operational Safety: Shell Quoting & Markdown

**CRITICAL:** When using tools like `jj describe` or `git commit`, remember you are piping text through `bash`.

### The Trap: Markdown in Double Quotes
Markdown backticks inside double quotes trigger shell command execution.
- ❌ `jj describe -m "Refactored `PtyResource`"`
  - **Result:** The shell tries to execute the command `PtyResource` and fails, or worse, executes something unexpected.

### The Fix: Use Single Quotes
Always prefer **single quotes** for descriptions or messages containing code or technical terms.
- ✅ `jj describe -m 'Refactored `PtyResource`'`
  - **Result:** The shell treats the string literally.

### Alternative: Escape Rigorously
If you must use double quotes (e.g., for shell variable expansion), you must escape backticks and dollar signs.
- ✅ `jj describe -m "Refactored \`PtyResource\`"`

## 4. Technical Guidelines

### Code Style & Quality
- **Correctness & Clarity First**: Prioritize readable, correct code over premature optimization.
- **Strong Types**: Use modern, idiomatic types.
- **No Shortcuts**: Avoid hacks or workarounds. Refactor messy code as you encounter it.
- **No Organizational Comments**: Do not write comments that summarize what code does. Code should be self-documenting.
- **"Why" Comments Only**: Comments should only explain the "why" behind a non-obvious implementation choice.
- **File Structure**: Add new functionality to existing files unless it represents a new, distinct logical component. Avoid `mod.rs`.
- **Naming**: Use full, descriptive words for variables. No abbreviations.

### Error Handling
- **`anyhow::Result`**: Use for all fallible operations.
- **No `unwrap()`**: Always propagate errors with `?`.
- **Add Context**: Use `.context()` to provide useful debugging information on errors.
- **No Silent Errors**: Never discard errors with `let _ =`.
- **Graceful Failures**: Handle potential network failures (e.g., reconnection logic) gracefully.

### Jujutsu (jj) Command Reference
| Command | Agent Usage |
| :--- | :--- |
| `jj new -m "..."` | Start a new atomic change. |
| `jj describe -m "..."` | Update the description (the persistent context). **Use frequently.** |
| `jj log -r 'mine()' -n 10` | Review your recent work history (your memory). |
| `jj log -n 20` | See all recent project activity. |
| `jj show <id>` | Read the full diff and description of a change. |
| `jj obslog -p` | See how a change evolved over time. **This is the reasoning trace** - shows abandoned approaches, refinements, and learning. |
| `jj diff` | Review current working copy changes. |
| `jj squash` | Fold the current change into its parent. |
| `jj split` | Separate a change into smaller, more logical units. |
| `jj abandon` | Discard the current change entirely. |
| `jj git push -c @` | Push the current change to the remote. |

### Shader Validation

When working with WGSL shaders, use `naga-cli` to validate them ahead of time. Bevy uses `naga` internally, so if `naga` likes it, Bevy likely will too.

```bash
# Install (one-time)
cargo install naga-cli

# Validate
naga assets/shaders/terminal.wgsl
```

### Commit Description Format
**The description is a letter to your future self and other agents.**
```
<type>: <summary>

Why: <original user prompt or problem being solved>
Approach: <key decisions, algorithms, patterns>
Tried: <approaches that didn't work>
Context: <architectural decisions, dependencies discovered>
Status: <complete | blocked: <reason> | next: <specific tasks>>

Co-authored-by: <Your Name> <your@email>
```

- **Why**: Include the original user prompt when possible - full context matters
- **Tried**: Document failed approaches - this is crucial learning
- **Status**: Be specific about blockers and next steps

**Agent Attribution:**
- **Claude**: `Co-authored-by: Claude <claude@anthropic.com>`
- **Gemini**: `Co-authored-by: Gemini <gemini@google.com>`
- **Kimi**: `Co-authored-by: Kimi <kimi@moonshot.ai>`

## 5. GitHub Integration

**GitHub CLI (`gh`):**
Use `gh` for GitHub operations without leaving the terminal:
```bash
gh pr create --fill          # Create PR from jj description
gh pr status                 # Check PR status  
gh pr checks                 # View CI results
gh issue list                # Check issues
gh issue view <number>       # Read issue details
```

The `--fill` flag pulls title and body from your jj description - another reason to keep descriptions rich and clear.

## 6. Cross-Session Context Patterns

jj's power is in context preservation across sessions and agents.

**Starting a new session:**
1. `jj log -r 'mine()' -n 10` - What was I working on?
2. `jj show @` - What's my current state?
3. `jj log -n 20` - What happened since I left?

**Picking up someone else's work:**
1. Find their change: `jj log -n 20`
2. Read their description: `jj show <change-id>`
3. See their reasoning: `jj obslog <change-id> -p`
4. Create your change building on theirs: `jj new <their-change-id>`

**Avoiding duplicate work:**
- Always check `jj log -n 20` before starting something new
- Search descriptions: `jj log | grep -i "keyword"`
- Check for "Status: blocked" or "Status: next:" entries

**Debugging strange behavior:**
- Use `jj obslog -p` to see what changed and when
- Check descriptions for "gotcha" or "caveat" notes
- Look for changes that touch the same files

## Remember

**jj changes are not git commits.** They are:
- Living documents that evolve with your understanding
- Persistent memory that survives rebases  
- Context transfer mechanisms between agents and sessions
- Reasoning traces via obslog

**Your description quality directly impacts the next agent's ability to succeed.** Write for them.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tobert)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tobert)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
