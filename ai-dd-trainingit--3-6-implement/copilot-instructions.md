## project-git

> Rules for using git repository commits and release tags


# Git rules

## Initialize repo if not yet done

- Add a `.gitignore` file for the current tech stack

## Git commits

- Review the Changes:
  - Carefully review the pending changes in the codebase.
  - Identify the type of changes (e.g., new features, bug fixes, documentation updates, etc.).
- Craft a Conventional Commit Message:
 - Use the conventional commit format: <type>(<scope>): <subject>.
 - Common types include `feat`, `fix`, `docs`, `style`, `refactor`, `test`, and, `chore`.
 - The scope is optional and can be used to specify the area of the codebase affected.
 - The subject should be a concise description of the change
- Commit the Changes:
 - Ensure the message accurately reflects the changes and follows the conventional commit guidelines 

## Git release

- Create or update a `CHANGELOG.md` file with messages from commits since last release
- Update version in `package.json` using semantic versioning
- Document the changes following [docs.mdc](mdc:.cursor/rules/docs.mdc) guidelines
- Create a _tag_ for the release using semantic versioning
- Do not push any change, keep your work at local repository

---
> Source: [AI-DD-TrainingIT/3_6-implement](https://github.com/AI-DD-TrainingIT/3_6-implement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
