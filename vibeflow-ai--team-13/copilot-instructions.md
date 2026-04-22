## team-13

> JJ Version Control System rules for safe Git-to-JJ migration within Cursor IDE and using JJ in workflow

# JJ Version Control System Rules for Cursor AI

## CRITICAL SAFETY PRINCIPLES
- NEVER use dangerous commands without explicit user request.
- ALWAYS provide safe alternatives to Git workflows.
- MAINTAIN Git compatibility when working in colocated repositories.
- PRESERVE existing Git history and branches during transition.
- EXPLAIN the differences between JJ and Git concepts clearly.

## JJ FUNDAMENTALS

### Core Concepts
- JJ uses "changes" instead of commits as the primary unit.
- Every change has a unique **Change ID** that persists across rewrites.
- The working copy is always a change (`@`), automatically tracked.
- No staging area – all modifications are automatically snapshotted.
- Conflicts are first-class objects that don't block workflow.
- **Bookmarks** (equivalent to Git branches) are pointers to changes.

### Safe Transition Strategy
1. Start with colocated repositories (`jj git init --colocate`).
2. Use JJ commands while maintaining Git compatibility.
3. Gradually adopt JJ-native workflows.
4. Keep Git remotes and tooling functional.

---

## ESSENTIAL JJ COMMANDS AND USAGE

### Repository Setup (Safe Git Integration)
```bash
# Initialize JJ in an existing Git repository (RECOMMENDED)
jj git init --colocate

# Clone with colocated setup
jj git clone --colocate <url>

# Create bookmark to track remote
jj bookmark track main@origin

# Safe configuration
jj config set --user user.name "Your Name"
jj config set --user user.email "your.email@example.com"
```

### Daily Workflow Commands
```bash
# Check repository status
jj status                   # Show working copy status
jj log                      # Show change history
jj log -r @                 # Show current change only

# Creating and managing changes
jj new                      # Create new empty change
jj new main                 # Create change from main bookmark
jj describe -m "message"    # Add description to current change
jj commit -m "message"      # Describe and create new change on top

# Working with bookmarks (Git branch equivalents)
jj bookmark list            # List all bookmarks
jj bookmark create name     # Create bookmark at current change
jj bookmark create name -r @ # Create bookmark at working copy
jj bookmark move name -r @  # Move bookmark to current change
jj bookmark delete name     # Delete bookmark

# Safe change navigation
jj edit <rev>               # Switch to different change
jj new @-                   # Create change on parent
jj prev                     # Move to parent change
jj next                     # Move to child change

# Change operations
jj show                     # Show current change details
jj show <rev>               # Show specific change
jj diff                     # Show changes in working copy
jj diff -r <rev>            # Show changes in specific change

# Safe change manipulation
jj squash                   # Squash working copy into parent
jj squash -r <a>::<b> -i    # Interactive squash between changes
jj duplicate <rev>          # Create copy of change
jj abandon <rev>            # Remove change (children preserved)

# File operations
jj restore <path>           # Restore file to parent version
jj restore --from <rev> <path>  # Restore from specific change
jj file track <path>        # Start tracking file
jj file untrack <path>      # Stop tracking file

# Conflict resolution
jj resolve                  # Resolve conflicts interactively
jj resolve --tool meld      # Use specific merge tool

# Safety and undo
jj undo                     # Undo last operation
jj op log                   # Show operation history
jj op restore <op_id>       # Restore to specific operation
```

### Git Integration Commands
```bash
# Syncing with Git remotes
jj git fetch                       # Fetch from all remotes
jj git fetch --remote origin       # Fetch from specific remote
jj git push                        # Push current bookmark
jj git push --bookmark <name>      # Push specific bookmark
jj git push --all-bookmarks        # Push all bookmarks
jj git push -c @-                  # Push parent change with auto-generated bookmark

# Git compatibility operations
jj git import                      # Import Git changes
jj git export                      # Export to Git
```

---

## WORKFLOW PATTERNS

### Basic Feature Development
```bash
# 1. Create feature branch
jj new main
jj bookmark create feature/awesome-feature

# 2. Make changes (files automatically tracked)
# Edit files...
jj describe -m "Add awesome feature"

# 3. Continue development
jj new            # Create new change for next iteration
# Edit more files...
jj describe -m "Improve awesome feature"

# 4. Push to remote
jj git push --bookmark feature/awesome-feature
```

### Updating from Remote
```bash
# Safe update pattern
jj git fetch
jj rebase -d main@origin  # Rebase current change stack
```

### Stack-based Development
```bash
# Create change stack
jj new main
jj describe -m "Foundation change"

jj new
jj describe -m "Build on foundation"

jj new
jj describe -m "Final touches"

# Push entire stack
jj bookmark create feature/stack -r @--  # Bookmark the bottom change
jj git push --bookmark feature/stack
```

### Conflict Resolution Workflow
```bash
# When conflicts occur
jj status                   # Check conflict status
jj resolve                  # Start resolution process
# Conflicts don't block other work – continue on other changes
jj new @- -m "Different work while conflicts exist"
```

---

## CONFIGURATION EXAMPLES

### Safe Configuration Template
```toml
# ~/.config/jj/config.toml

[user]
name = "Your Name"
email = "your.email@example.com"

[ui]
default-command = "log"
diff-editor = ":builtin"
merge-editor = "meld"  # or your preferred tool

[aliases]
# Safe aliases for Git users
st = ["status"]
br = ["bookmark", "list"]
co = ["edit"]
unstage = ["restore"]

# Workflow aliases
sync = ["git", "fetch"]
update = ["rebase", "-d", "main@origin"]

[git]
fetch = "origin"
push = "origin"

[revsets]
log = "@:: | @- | main@origin"
```

### Advanced Safety Configuration
```toml
[ui]
# Prevent accidental large operations
always-allow-large-revsets = false

# Safe conflict markers
conflict-marker-style = "diff"

# Show warnings for potentially destructive operations
paginate = "auto"

[snapshot]
# Auto-track common file types safely
auto-track = "glob:'**/*.{rs,py,js,ts,go,java,c,cpp,h}'"
max-new-file-size = "1MiB"

[revset-aliases]
# Safe immutable commits
"immutable_heads()" = "builtin_immutable_heads() | main@origin"
```

---

## MIGRATION STRATEGIES

### Gradual Migration Approach
1. **Phase 1** – Use JJ in colocated mode with Git workflows.
2. **Phase 2** – Adopt JJ change-based thinking while keeping Git remotes.
3. **Phase 3** – Use native JJ workflows (stacking, conflict handling).
4. **Phase 4** – Full JJ adoption with advanced features.

### Team Integration Guidelines
- Use colocated repositories to avoid disrupting team workflows.
- JJ changes appear as normal Git commits to teammates.
- Git tooling (IDEs, CI/CD) continues working unchanged.
- Enables gradual team adoption without coordination overhead.

---

## TROUBLESHOOTING AND SAFETY

### Common Issues and Solutions
```bash
# If working copy becomes detached
jj new @-            # Create new working copy

# If bookmark conflicts occur
jj bookmark list <name>     # See conflicted state
jj bookmark move <name> -r <rev>  # Resolve

# If an operation went wrong
jj undo             # Safe undo of last operation
jj op log           # Review operation history
jj op restore <id>  # Restore to specific state

# Emergency Git fallback
git status          # Check Git state in colocated repo
git checkout <ref>  # Use Git commands if needed
jj git import       # Re-sync with Git
```

### Safety Checklist
- Run `jj status` before major operations.
- Review `jj log` to understand current state.
- Remember `jj undo` is always available.
- Keep `jj op log` handy for operation history.

---

## CODE GENERATION GUIDELINES FOR CURSOR AI

### When Suggesting JJ Commands
1. Always explain the Git equivalent for context.
2. Provide safe alternatives to potentially destructive operations.
3. Show the expected outcome of commands.
4. Include safety checks and verification steps.
5. Reinforce JJ concepts (changes, bookmarks) when introducing them.

#### Example Pattern
```bash
# INSTEAD OF blindly suggesting
jj abandon <rev>

# SUGGEST
jj abandon <rev>  # Safely removes change, preserves children
# Git equivalent: git reset --hard HEAD~1 (but safer)

# Always follow up with
jj undo           # to revert if the result isn't as expected
```

### Error-Prevention Rules
- Never suggest commands that could lose work without warnings.
- Always mention `jj undo` availability.
- Explain immutable commit concepts where relevant.
- Show how to verify results (`jj log`, `jj status`).
- Provide rollback procedures (`jj undo`, `jj op restore`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/VibeFlow-AI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
