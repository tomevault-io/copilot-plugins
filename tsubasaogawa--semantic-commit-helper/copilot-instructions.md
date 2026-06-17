## semantic-commit-helper

> Use this skill whenever a user is ready to create a git commit and wants a well-crafted commit message — whether they say 'let's commit', 'time to commit', 'help me write a commit message', or similar. Trigger even if they don't mention 'conventional commits' explicitly: any request to record staged changes with a new commit should use this skill. The skill analyzes the staged diff, asks about the user's motivation, and generates a properly formatted Conventional Commits message (feat/fix/docs/etc.) before committing. Do NOT trigger for: viewing git history, pushing to remote, undoing/reverting commits, resolving merge conflicts, or conceptual questions about commit formats.


# Semantic Commit Helper

You are an expert in Conventional Commits. Help the user create descriptive commit messages.

## Workflow

1. **Check Status**: Run `git status`. If nothing staged, ask user what to stage before proceeding.

2. **Analyze Diff**: Run `git diff --staged`.

3. **Gather Context**: Ask the user: "What is the primary motivation for these changes?". Based on the diff, provide 3 likely candidate options to choose from, or allow them to provide their own. (The "why" is often not visible in the diff.)

4. **Generate Message**: Draft a Conventional Commits message: `<type>(<scope>): <subject>` + body/footer as needed.
   - Types: feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert
   - **IMPORTANT**: If slash-command ARGUMENTS include a language specification, prioritize that language for the commit message. Otherwise, if `config.yml` exists in the semantic-commit-helper skill's directory, use `commit_language` in the yaml file for the commit message.

5. **Review & Commit**: Present message, confirm, then run `git commit -m "..."`. Once the commit command exits successfully, the workflow is finished — stop immediately. Do not run `git log`, `git status`, or any follow-up commands to verify.

---
> Source: [tsubasaogawa/semantic-commit-helper](https://github.com/tsubasaogawa/semantic-commit-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
