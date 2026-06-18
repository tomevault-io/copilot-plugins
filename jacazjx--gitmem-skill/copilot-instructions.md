## gitmem-skill

> Use when editing files in a git project and you need safe, traceable, reversible changes. Creates a separate .gitmem repository to track every agent edit independently from the main git history. Triggers for: undo/rollback agent edits, compare file versions, restore previous states, create checkpoints before risky changes, or inspect recent agent modifications. Distinguishes from normal git operations - use this for agent edit safety, not for standard git workflows like branching, merging, or pushing.


# Gitmem Safe Editing

Treat GitMem as an **agent editing safety layer**, not as user-facing version control and not as general memory.

Core outcome:

1. record every agent file edit immediately
2. make each edit easy to diff and roll back
3. give the user a safe way to undo or time-travel when the agent starts degrading the codebase

## Operating model

Use two repositories side by side:

- the user's main repository in `.git/`
- an independent agent edit log in `.gitmem/.git`

GitMem must remain isolated from the main repository.
Do not reuse `.git/` for GitMem history.
Do not introduce databases, mcp servers, or external storage.

Expected layout:

```text
project/
  src/
  docs/
  .git/
  .gitmem/
    .git/
```

## Auto-initialization on skill activation

When this skill is triggered, **you MUST immediately execute these commands** using the Bash tool:

### Step 1: Check initialization status

Run this command to check if GitMem is initialized:

```bash
test -d .gitmem/.git && echo "initialized" || echo "not initialized"
```

**You MUST run this command and check the output.** If it outputs "not initialized", proceed to Step 2.

### Step 2: Auto-initialize if needed

If GitMem is not initialized, **MUST run these commands**:

```bash
mkdir -p .gitmem
git init .gitmem
git --git-dir=.gitmem/.git --work-tree=. config user.email "agent@gitmem.local"
git --git-dir=.gitmem/.git --work-tree=. config user.name "GitMem Agent"
```

Then inform the user: "GitMem initialized at `.gitmem/`"

### Step 3: Start auto-watch mode

**MUST run these commands** to start the file watcher:

```bash
# Check if watch is already running, if not start it in background
if ! pgrep -f "gitmem-watch" > /dev/null 2>&1; then
  # Find the watch script
  WATCH_SCRIPT="$(find ~/.claude/skills -name 'gitmem-watch' 2>/dev/null | head -1)"
  if [ -x "$WATCH_SCRIPT" ]; then
    nohup "$WATCH_SCRIPT" > /tmp/gitmem-watch.log 2>&1 &
    disown
  fi
fi
```

Then inform the user: "GitMem auto-watch started - all file changes will be automatically tracked"

**Note:** If `inotify-tools` (Linux) or `fswatch` (macOS) is not installed, inform the user: "Auto-watch requires `inotify-tools` (Linux) or `fswatch` (macOS). Please install for automatic tracking, or manually commit changes using `gitmem commit <file>`."

## Required edit workflow

For every edited file, follow this sequence:

```text
read file
edit file
commit that file to gitmem immediately
```

Never batch multiple edits and commit only at the end.
Never edit repeatedly without checkpointing history in `.gitmem`.

After changing one file, create a GitMem commit for that file before editing the next file.

## Command setup

Use GitMem commands through the separate git dir and the project worktree:

```bash
git --git-dir=.gitmem/.git --work-tree=. <command>
```

Run commands from the project root unless the user explicitly specifies another path.

## Core operations

### 1. commit an edited file

Use this after each file change.

```bash
git --git-dir=.gitmem/.git --work-tree=. add -- <file>
git --git-dir=.gitmem/.git --work-tree=. commit -m "agent(edit): <file>\n\nreason: <why the change was made>"
```

Rules:

- add only the current file
- never use `git add .`
- explain the reason in the commit message
- keep each commit scoped to the current file change

Commit message format:

```text
agent(edit): <file>

reason: <why the change was made>
```

Example:

```text
agent(edit): src/api.ts

reason: add retry logic
```

### 2. inspect file history

Use when the user asks what changed before, which version was better, or when the agent is unsure how the file evolved.

```bash
git --git-dir=.gitmem/.git --work-tree=. log -- <file>
```

Summarize at least:

- commit id
- commit message
- time

### 3. diff two states

Use when deciding whether to keep, revert, or compare changes.

```bash
git --git-dir=.gitmem/.git --work-tree=. diff <commit_a> <commit_b> -- <file>
```

### 4. roll back one file

Use when the user says the earlier file version was better or asks to restore one file.

```bash
git --git-dir=.gitmem/.git --work-tree=. checkout <commit> -- <file>
```

After rollback, commit the restored file to GitMem with a clear reason so the recovery itself is traceable.

### 5. create a safe checkpoint

Use when the user is satisfied, a feature reaches a stable state, or before risky edits.

```bash
git --git-dir=.gitmem/.git --work-tree=. tag gitmem-checkpoint-<name>
```

Checkpoint names should be short and descriptive, such as:

```text
gitmem-checkpoint-feature-working
gitmem-checkpoint-tests-passing
```

### 6. undo the last agent change

Use when the user says the most recent agent edit broke something and wants the last GitMem step removed quickly.

```bash
git --git-dir=.gitmem/.git --work-tree=. reset --hard HEAD~1
```

Only use this for the latest GitMem change.
If the user wants a specific older state, inspect history and use rollback or time travel instead.

### 7. time travel the whole project

Use when the user wants to return the worktree to an older GitMem state across the project.

```bash
git --git-dir=.gitmem/.git --work-tree=. checkout <commit>
```

Before doing full-project time travel, inspect recent history and explain what point will be restored.
After restoration, tell the user which commit is now active.

### 8. inspect recent changes

Use when diagnosing agent loops or reviewing the latest edit sequence.

```bash
git --git-dir=.gitmem/.git --work-tree=. log -n <limit>
```

## Loop guard

GitMem exists largely to catch the classic agent loop:

```text
edit
break
fix
break
fix
break
```

When a file has been modified repeatedly in a short recent window and there is no stable checkpoint, treat that as a warning sign.

Default heuristic:

- if the same file appears in 5 recent GitMem commits
- and there is no relevant checkpoint marking a stable state
- warn the user before continuing risky edits

Use language like:

```text
This file has been modified multiple times recently without a stable checkpoint.
Consider restoring a previous stable version or creating a checkpoint before more edits.
```

Do not silently continue infinite edit-fix cycles when GitMem history clearly shows churn.
Inspect recent history, compare versions, and recommend rollback or checkpointing.

## Mandatory rules

1. after any file modification, commit it to GitMem immediately
2. each GitMem commit should cover only the current file change
3. never use `git add .` for GitMem commits
4. every GitMem commit message must include a real reason
5. for requests like `undo`, `revert`, `rollback`, `restore`, or `go back`, prefer GitMem operations
6. when uncertain about prior states, inspect file history before editing further
7. after restoring a file version, commit the restored result so recovery remains traceable
8. keep `.gitmem` separate from `.git` at all times

## Decision guide

Use this quick routing logic:

- **skill triggered / user says `use gitmem`** → check if `.gitmem/.git` exists, auto-initialize if needed, then start auto-watch mode
- **new code edit request** → edit file (auto-watch will commit changes automatically)
- **user says `the previous version was better`** → inspect file history, diff likely candidates, roll back the file (auto-watch will commit the restoration)
- **user says `go back a few steps`** → inspect recent changes, then time-travel or undo depending on scope
- **agent is unsure whether a fix helped or harmed** → diff recent commits before editing again
- **user confirms a version works** → create a checkpoint
- **file has many recent edits and instability** → activate loop guard and recommend rollback or checkpointing

## Response behavior

When using this skill, be explicit about safety state.
Briefly tell the user:

- whether GitMem was initialized or already present
- whether auto-watch mode was started or already running
- which file or commit you are operating on
- whether you created a commit, checkpoint, rollback, undo, or time-travel action
- when loop guard warnings apply

Keep the wording short, operational, and traceable.
Do not describe GitMem as replacing the user's main git workflow.

## Example request patterns

Trigger this skill for prompts such as:

- `edit this file safely and let me undo later`
- `you just broke the code, revert your last change`
- `show me what changed between your last two edits`
- `go back to the working version from earlier`
- `checkpoint this state before you refactor`
- `this file keeps getting worse, compare recent agent edits`

## Using the scripts

GitMem comes with helper scripts in `scripts/` for common operations.

### Installation

```bash
cd /path/to/gitmem-safe-editing/scripts
./install.sh
```

This creates symlinks in `~/.local/bin/`. Add it to PATH if needed:
```bash
export PATH="$HOME/.local/bin:$PATH"
```

### Available commands

| Command | Description |
|---------|-------------|
| `gitmem init` | Initialize .gitmem repository |
| `gitmem commit <file> [reason]` | Commit a single file |
| `gitmem history [file]` | Show edit history |
| `gitmem diff <file>` | Show diff between last two versions |
| `gitmem rollback <file> [commit]` | Roll back a file |
| `gitmem undo` | Undo last gitmem change |
| `gitmem checkpoint <name>` | Create a checkpoint tag |
| `gitmem status` | Show gitmem status |
| `gitmem watch` | Start file watcher for auto-commit |
| `gitmem check-loop <file>` | Check for edit loop warning |

### Auto-watch mode

To automatically commit every file change to GitMem:

```bash
gitmem watch
```

This monitors the project directory and auto-commits changes to `.gitmem`.
Requirements: `inotify-tools` (Linux) or `fswatch` (macOS).

Options:
- `--debounce SECONDS` - Wait before committing (default: 2)
- `--exclude PATTERN` - Additional exclude pattern
- `--dry-run` - Show what would be committed

### Using scripts in skill context

When using this skill, prefer the direct git commands documented in Core operations for maximum clarity and control. Use the helper scripts for:
- Quick initialization: `gitmem init`
- Auto-monitoring: `gitmem watch`
- Loop guard checks: `gitmem check-loop <file>`

## Error handling

### GitMem not initialized

**Problem:** GitMem commands fail with "not a git repository" or similar errors.

**Diagnosis:**
```bash
ls -la .gitmem/.git
```

**Solutions:**
- If `.gitmem` directory doesn't exist → Initialize: `mkdir -p .gitmem && git init .gitmem`
- If `.gitmem` exists but `.gitmem/.git` is missing → Re-initialize: `git init .gitmem`
- If `.gitmem/.git` exists but is corrupted → Reinitialize with backup:
  ```bash
  mv .gitmem .gitmem.backup
  mkdir -p .gitmem
  git init .gitmem
  ```

### Corrupted GitMem repository

**Symptoms:**
- `git --git-dir=.gitmem/.git` commands fail unexpectedly
- "fatal: bad object" errors
- Inability to read history

**Recovery:**
1. **Backup first:** `cp -r .gitmem .gitmem.corrupted`
2. **Try fsck:** `git --git-dir=.gitmem/.git fsck --full`
3. **If unrecoverable:** Reinitialize fresh and inform user that history is lost
4. **Document the loss:** Create a note explaining what happened

### File conflicts between .git and .gitmem

**Scenario:** The user has uncommitted changes in main `.git` and GitMem rollback overwrites them.

**Solution:**
1. **Before any rollback, check main git status:**
   ```bash
   git status --short
   ```
2. **If there are uncommitted changes:**
   - Warn the user: "You have uncommitted changes in main git that will be affected."
   - Offer options: stash changes, commit to main git, or proceed with rollback
3. **After rollback, verify the file state matches expectations**

### No history found for file

**Scenario:** User asks to rollback a file that has no GitMem history.

**Diagnosis:**
```bash
git --git-dir=.gitmem/.git --work-tree=. log -- <file>
```

**Response:**
- Inform user: "This file has no GitMem edit history yet."
- Explain that GitMem only tracks edits made after initialization
- Offer to create a checkpoint now before future edits

### Merge conflict in worktree

**Scenario:** After time-travel or rollback, the worktree shows conflicts.

**Solution:**
1. **List conflicted files:** `git --git-dir=.gitmem/.git status`
2. **For each conflict:**
   - If user wants GitMem version: `git --git-dir=.gitmem/.git checkout --theirs -- <file>`
   - If user wants to keep current: `git --git-dir=.gitmem/.git checkout --ours -- <file>`
3. **Commit resolution:** `git --git-dir=.gitmem/.git commit -m "agent(edit): resolved conflicts"`

### GitMem accidentally committed to main .git

**Problem:** `.gitmem/` directory was committed to the main repository.

**Solution:**
1. **Add to .gitignore:**
   ```bash
   echo ".gitmem/" >> .gitignore
   ```
2. **Remove from main git tracking (keep local):**
   ```bash
   git rm -r --cached .gitmem
   git commit -m "chore: stop tracking .gitmem in main repo"
   ```

### Accidental `git add .` in GitMem

**Problem:** Too many files were staged in GitMem at once.

**Solution:**
```bash
# Reset the staging area
git --git-dir=.gitmem/.git --work-tree=. reset HEAD

# Add only the intended file
git --git-dir=.gitmem/.git --work-tree=. add -- <specific-file>
git --git-dir=.gitmem/.git --work-tree=. commit -m "agent(edit): <file>

reason: <reason>"
```

### Disk space concerns

**Problem:** `.gitmem` grows large over time.

**Mitigation:**
1. **Check size:** `du -sh .gitmem`
2. **Prune old history (with user consent):**
   ```bash
   # Keep only last 100 commits
   git --git-dir=.gitmem/.git --work-tree=. checkout --orphan temp-branch
   git --git-dir=.gitmem/.git --work-tree=. commit -m "agent(edit): history pruned

reason: disk space cleanup"
   git --git-dir=.gitmem/.git --work-tree=. branch -D main 2>/dev/null || true
   git --git-dir=.gitmem/.git --work-tree=. branch -m main
   ```
3. **Garbage collect:**
   ```bash
   git --git-dir=.gitmem/.git gc --aggressive --prune=now
   ```

### Recovery checklist

When GitMem state is problematic, follow this sequence:

1. **Assess the damage:**
   - Can you run `git --git-dir=.gitmem/.git status`?
   - Are there recent commits in `git --git-dir=.gitmem/.git log`?
   - Are checkpoint tags still present?

2. **Preserve what you can:**
   - Backup `.gitmem` before any recovery attempts
   - Note any important checkpoint names

3. **Recover or reinitialize:**
   - Minor corruption → try `git fsck` and repair
   - Major corruption → reinitialize and inform user

4. **Document and communicate:**
   - Tell user what was lost (if anything)
   - Suggest creating an early checkpoint to rebuild safety net

---
> Source: [jacazjx/GitMem-Skill](https://github.com/jacazjx/GitMem-Skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
