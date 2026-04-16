## barbarian-blaster

> When the user asks you to handle git version control or commit changes in this Godot 4 project, always follow this structured git flow process step by step. Do not automate everything at once—pause after each major step to ask for the user's opinion and confirmation before proceeding. This ensures the user remains in control without manually managing git.

# Git Flow Instructions for Godot 4 Projects

When the user asks you to handle git version control or commit changes in this Godot 4 project, always follow this structured git flow process step by step. Do not automate everything at once—pause after each major step to ask for the user's opinion and confirmation before proceeding. This ensures the user remains in control without manually managing git.

The process assumes the project is already initialized as a git repository. If not, suggest initializing it first.

## Step-by-Step Git Flow

1. **Check Git Status**:
   - Run `git status` to show the current state of the repository, including untracked files, modified files, and staged changes.
   - Summarize the output clearly (e.g., "There are 3 modified files: main.gd, player.tscn, and level1.tscn. No untracked files.").
   - Ask the user: "Do these changes look correct? Should I proceed to review the diff? (Yes/No/Adjust)"

2. **Review Git Diff**:
   - If approved, run `git diff` (or `git diff --cached` if changes are staged) to examine the exact changes.
   - Analyze the diff intelligently:
	 - Identify file types: GDScript (.gd), scenes (.tscn), resources (.tres), shaders (.gdshader), etc.
	 - Interpret changes: For example:
	   - In GDScript: Detect added functions, variable changes, bug fixes, or refactors (e.g., "Added a new method `jump()` to handle player physics.").
	   - In scenes: Note added nodes, property changes, or connections (e.g., "Modified Player node to include a new CollisionShape2D for better hit detection.").
	   - Group changes thematically: If multiple files are affected, infer if it's a feature addition, bug fix, refactoring, etc.
	 - Handle Godot-specific artifacts: Ignore temporary files like .import/ or export.cfg if they're not intended for commit.
   - Present a summarized interpretation of the changes (e.g., "The diff shows additions to movement logic in player.gd, likely improving jump mechanics, and UI tweaks in main.tscn.").
   - Ask the user: "Does this summary match what you did? Any specific details to note for the commit message? Proceed to staging? (Yes/No/Provide details)"

3. **Stage Changes**:
   - If approved, suggest staging all changes with `git add .` or specific files if the user specifies.
   - For Godot projects, recommend excluding unnecessary files (e.g., via .gitignore: add patterns like *.import, project.godot.bak if not already present).
   - Run `git status` again to confirm staging.
   - Ask the user: "Changes are staged. Ready to generate a commit message? (Yes/No/Unstage something)"

4. **Generate Commit Message**:
   - This is critical: Always generate a professional, sensible commit message based on the diff analysis from Step 2.
   - Use Conventional Commits format:
	 - **Prefix**: Choose one based on changes (interpret from diff):
	   - `feat:`: New feature (e.g., adding a new enemy type or mechanic).
	   - `fix:`: Bug fix (e.g., correcting collision issues).
	   - `chore:`: Maintenance (e.g., updating dependencies or .gitignore).
	   - `docs:`: Documentation changes (e.g., updating README or comments).
	   - `style:`: Formatting/whitespace (no code change).
	   - `refactor:`: Code restructuring (no feature/bug change).
	   - `perf:`: Performance improvements.
	   - `test:`: Adding/modifying tests.
	   - `ci:`: CI config changes.
	 - **Subject**: Concise summary (50 chars max), imperative mood (e.g., "Add jump mechanic to player").
	 - **Body**: Detailed description (wrap at 72 chars):
	   - Explain *what* changed and *why* (infer from diff, e.g., "Updated player.gd to include gravity adjustments for smoother jumps. This resolves issue with double-jumping glitches. Also tweaked animation in player.tscn.").
	   - Reference Godot specifics: Mention nodes, signals, resources affected.
	   - If applicable, add footer: e.g., "Closes #issue" if linked to an issue.
   - Present the proposed message to the user (e.g., in full format for `git commit -m`).
   - Ask the user: "Approve this commit message? Edit it? Proceed to commit? (Yes/No/Suggest edit)"

5. **Commit Changes**:
   - If approved, run `git commit` with the generated/approved message.
   - Show the commit hash and summary after success.
   - Ask the user: "Commit successful. Ready to push? (Yes/No)"

6. **Push Changes**:
   - If approved, run `git push` (assume default branch like main; ask if branch needs specifying).
   - Handle branches if needed: For feature work, suggest `git checkout -b feature/xyz` earlier if not on main.
   - After push, confirm with `git log -1` to show the latest commit.
   - Ask the user: "Push complete. Anything else? (e.g., Create PR/No)"

## Additional Guidelines
- **Safety First**: Never execute destructive commands (e.g., force push) without explicit user approval. Suggest but don't assume.
- **Godot-Specific Tips**:
  - Recommend committing .godot/ or export presets only if changed intentionally.
  - For large assets (e.g., textures, models), suggest using Git LFS if not set up.
  - If changes involve scripts, remind to test in Godot editor before committing.
- **Error Handling**: If any git command fails (e.g., conflicts), explain the issue and ask how to resolve (e.g., "Merge conflict detected—should I help rebase?").
- **Tools Integration**: Use built-in tools like shell execution to run git commands, but always with user confirmation.
- **Override Priority**: If the user provides custom instructions in a query, prioritize those over this, but default to this flow for git tasks.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yubinex) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
