## rl-cc-sam

> **DO NOT MODIFY ANY FILES IN A GIT REPOSITORY USED AS A SUBMODULE**

# Git Submodule Handling Rules

## Core Rule
**DO NOT MODIFY ANY FILES IN A GIT REPOSITORY USED AS A SUBMODULE**

## Why This Rule Exists
1. Submodules are external dependencies that should be treated as read-only
2. Modifications to submodules can cause conflicts with upstream changes
3. Changes in submodules are not tracked in the main repository
4. Submodules should be updated only through proper Git submodule commands

## Best Practices
1. Keep submodules at their original state
2. Every time we need to introduce an open-source library as dependency, add it as a submodule, rather than tracking its sources directly by top level git repo
3. If you need to modify submodule code:
   - Fork the original repository
   - Make changes in your fork
   - Update the submodule to point to your fork
   - Document the changes and reasons

## Proper Submodule Management
```bash
# Update submodule to latest commit
git submodule update --remote

# Initialize submodules after cloning
git submodule update --init --recursive

# Check submodule status
git submodule status
```

## Common Mistakes to Avoid
1. Directly editing files in the submodule directory
2. Committing changes to submodule files
3. Modifying submodule configuration
4. Ignoring submodule updates

## When You Need to Make Changes
If you absolutely need to modify submodule code:
1. Document the required changes
2. Create a new branch in your main repository
3. Fork the original repository
4. Make changes in your fork
5. Update the submodule reference to your fork
6. Document the changes in your main repository's README

## Related Files
- [.gitmodules](mdc:.gitmodules): Contains submodule configuration
- [segment-anything/](mdc:segment-anything): SAM submodule directory

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chenqiu1024)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chenqiu1024)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
