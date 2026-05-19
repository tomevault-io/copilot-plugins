## github-cli-example

> Instructions for using the GitHub CLI to create/view issues, commits, and pull reqests.

---
# Specify the following for Cursor rules
description: Instructions for using the GitHub CLI to create issues, commits, and pull requests.
---

## GitHub CLI Workflow Guide

This guide provides a walkthrough for a common development workflow using the GitHub CLI, from viewing issues to creating pull requests.

### 1. Managing Issues

#### Viewing Issues

List all open issues in the repository:

```bash
gh issue list | cat
```

To filter by a specific label:

```bash
gh issue list --label "bug" | cat
```

#### Creating an Issue

1.  **Create a temporary issue body file.**
    Create a file named `_docs/temp-issue-body.md` and add the issue description there. This allows for proper Markdown formatting. Issue descriptions should be detailed, specific, and context-rich.

2.  **Run the `gh issue create` command.**
    Use the following command to create an issue, providing a title and labels. See the "Available Labels" section for guidance.

    ```bash
    gh issue create --title "Your Issue Title" --body-file "_docs/temp-issue-body.md" --label "bug,high-priority"
    ```

3.  **Delete the temporary file.**
    After creating the issue, delete `_docs/temp-issue-body.md`.

### 2. Making Commits & Resolving Issues

1. **Check the working state and determine which files should be added (likely all)**
    TODO: add this section; should including checking the current branch (and creating a new one if on `main` or `master`), staging the necessary files, etc

2.  **Create a temporary commit message file.**
    Create a file named `_docs/temp-commit-message.md` and add your commit message there. To link the commit to an issue and automatically close it upon merge, include a keyword like `closes #42` in this file.

3.  **Run the `git commit` command.**
    Use the `-F` flag to use the file content as the commit message.

    ```bash
    git commit -F _docs/temp-commit-message.md
    ```

4.  **Delete the temporary file.**
    After creating the commit, delete `_docs/temp-commit-message.md`.

### 3. Creating Pull Requests

1.  **Create a temporary pull request body file.**
    Create a file named `_docs/temp-pr-body.md` and add the description there. **IMPORTANT: If your PR resolves issues, include closing keywords in this file or the PR title.**

2.  **Run the `gh pr create` command.**
    Use the `--body-file` flag to use the file content as the PR body.

    ```bash
    gh pr create --title "Pull Request Title" --body-file "_docs/temp-pr-body.md"
    ```

3.  **Delete the temporary file.**
    After creating the pull request, delete `_docs/temp-pr-body.md`.

#### Important: Closing Issues with Squash Merges

If a repostory uses **squash and merge**, which affects how issues are auto-closed:

- **Individual commit messages with `closes #XX` are LOST during squash merge**
- **Only keywords in PR title or PR description will auto-close issues**

**Best Practices for Issue Closing:**

1. **Option A: Include closing keywords in PR title**
   ```bash
   gh pr create --title "feat: dashboard refactoring (closes #47, #48, #49)"
   ```

2. **Option B: Include closing keywords in PR description**
   Add this to your `_docs/temp-pr-body.md`:
   ```markdown
   ## Issues Resolved
   - closes #47
   - closes #48  
   - closes #49
   ```

3. **For single-issue PRs:** Either approach works fine
4. **For multi-issue PRs:** Use Option B (PR description) for better organization

**Valid Closing Keywords:**
- `closes #XX`, `fixes #XX`, `resolves #XX`
- `close #XX`, `fix #XX`, `resolve #XX`
- Multiple issues: `closes #47, closes #48, closes #49`

### Available Labels

Here is a list of available labels for issues. **All issues must have exactly one priority label and at least one standard label.**

#### Priority Labels

- `high-priority`: High priority - needs to be addressed immediately.
- `medium-priority`: Medium priority - should be addressed in the near future.
- `low-priority`: Low priority - can be addressed when time permits.

#### Standard Labels

- `bug`: Something isn't working
- `documentation`: Improvements or additions to documentation
- `enhancement`: New feature or request
- `help wanted`: Extra attention is needed
- `question`: Further information is requested
- `refactor`: Code refactoring

---
> Source: [palmerwenzel/new-project-video-demo](https://github.com/palmerwenzel/new-project-video-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
