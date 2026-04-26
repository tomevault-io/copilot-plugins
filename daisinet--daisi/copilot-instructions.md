## daisi

> This project spans multiple git repos, all under `C:\repos\daisinet`. The full list is whatever `daisi-multi.ps1 status` reports — repos are added over time.

# Daisinet Project

## Multi-Repo Structure

This project spans multiple git repos, all under `C:\repos\daisinet`. The full list is whatever `daisi-multi.ps1 status` reports — repos are added over time.

All repos use `main` as the default branch and `dev` as the integration branch.

## Multi-Repo Management Script

Use `daisi\daisi-multi.ps1` for any operation that should apply across multiple repos:

```
.\daisi\daisi-multi.ps1 status              # Check branch, dirty, ahead/behind for all repos
.\daisi\daisi-multi.ps1 branch <name>       # Create feature branch from dev in all repos
.\daisi\daisi-multi.ps1 checkout <name>     # Switch all repos to a branch
.\daisi\daisi-multi.ps1 pull                # Pull current branch in all repos
.\daisi\daisi-multi.ps1 push                # Push unpushed commits in all repos
.\daisi\daisi-multi.ps1 pr-create           # Create PRs for all repos with changes vs base
.\daisi\daisi-multi.ps1 pr-merge            # Merge open PRs for current branch in all repos
.\daisi\daisi-multi.ps1 pr-dev-to-main      # Create PRs from dev to main where diverged
.\daisi\daisi-multi.ps1 worktree-add <name>  # Create worktree for parallel branch work
.\daisi\daisi-multi.ps1 worktree-remove <name> # Remove a worktree and clean up
```

### When to use this script

- **Starting a new feature**: always use `worktree-add <branch>` to create a parallel working copy at `C:\repos\daisinet-<branch>`. Never switch the main directory off `dev`.
- **Pushing and PRs**: use `push`, `pr-create`, and `pr-merge` to batch operations.
- **Releasing**: use `pr-dev-to-main` to promote dev to main across all repos.
- **Cleaning up**: use `worktree-remove <branch>` after a feature is merged.

### User Shortcuts

- **"ship to dev"** — For each repo with uncommitted changes: stage and commit with a descriptive message, then `push`, then `pr-create` (which auto-merges).
- **"closeout this feature"** — Do "ship to dev", then run `worktree-remove <branch>` from the main `C:\repos\daisinet` directory and close the console. Use: `powershell.exe -Command "Start-Process powershell -ArgumentList '-Command', 'Set-Location C:\repos\daisinet; powershell -ExecutionPolicy Bypass -File C:\repos\daisinet\daisi\daisi-multi.ps1 worktree-remove <branch>'"` then exit.
- **"ship to main"** — Only from main worktree on `dev`. Pull all repos, then `pr-dev-to-main`. Never delete `dev` or `main` branches. Never use `--delete-branch`.
- **"start the dev env"** — Start ORC (HTTPS, port 5001), Host (HTTPS), and optionally a UI project if relevant to current work.
- **"release to production|beta|group1|group2"** — Ship dev to main first, verify the build compiles, generate a timestamp version, then dispatch `orchestrate-release.yml` via `gh workflow run`.

### Running Tests

When asked to "run all tests":
1. **Unit tests**: All `*Tests.csproj` projects in the solution.
2. **Integration tests**: Start dev env (ORC + Host), then run `daisi-sdk-dotnet/Daisi.SDK.Tests`.
3. **Tools tests**: Only if a GGUF model exists in `C:\GGUFS`.

### Important flags

- **`-DryRun`**: Use for `pr-dev-to-main` to preview before executing.
- **`-Repos`**: Filter to specific repos.
- **`-Base`**: Override base branch for PR operations (default: `dev`).
- **`-MergeStrategy`**: `merge`, `squash`, or `rebase` (default: `merge`).

## Documentation Requirements

After every coding task, update relevant documentation. This is not optional.

- **README files**: Update when you change a repo's API, add a feature, or change behavior.
- **Learn section** (`daisi-web-public/Components/Pages/Learn/`): Update when user-facing functionality changes.
- **SDK reference**: Update when proto definitions, client APIs, or service contracts change.
- **Inline docs**: Add XML doc comments to new public classes and methods.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/daisinet) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
