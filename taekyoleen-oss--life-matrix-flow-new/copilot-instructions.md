## life-matrix-flow-new

> - When making significant changes to the codebase, automatically document them in `HISTORY.md` or `CHANGELOG.md`

# Cursor Rules for Life Matrix Flow Project

## History File Auto-Recording Rules

### 1. Change History Documentation

- When making significant changes to the codebase, automatically document them in `HISTORY.md` or `CHANGELOG.md`
- Include the following information for each change:
  - Date and time of change
  - Description of what was changed
  - Files affected
  - Reason for the change (if applicable)
  - Author information (if available)

### 2. Commit Message Format

- Use conventional commit format: `<type>(<scope>): <description>`
- Types: feat, fix, refactor, style, docs, test, chore
- Examples:
  - `feat(modules): Add Python execution support for LoadData module`
  - `fix(ui): Fix module rendering issue in ComponentRenderer`
  - `refactor(layout): Improve Auto Layout algorithm for special modules`

### 3. Code Change Tracking

- Track significant changes in code comments when:
  - Adding new features
  - Fixing critical bugs
  - Refactoring major components
  - Changing API interfaces

### 4. File Modification Rules

- When modifying existing files, add a comment at the top indicating:
  - Last modified date
  - Brief description of changes
  - Related issue or feature (if applicable)

### 5. History File Structure

- Maintain a structured history file with:
  - Version numbers (if applicable)
  - Chronological order (newest first)
  - Categorized changes (Features, Bug Fixes, Improvements, etc.)
  - Breaking changes section (if any)

### 6. Automatic History Updates

- When creating or modifying files, consider if the change warrants a history entry
- For major features or breaking changes, always add to history
- For minor fixes or style changes, batch them in periodic updates

### 7. History.md File Management

- All significant changes must be documented in `HISTORY.md`
- Each entry must include:
  - Date and time (ISO 8601 format: YYYY-MM-DD HH:MM:SS)
  - Description of what was changed
  - List of affected files
  - Reason for the change
  - Git commit hash (for recovery purposes)
  - **Simple recovery command** - Include both stash backup method and direct reset method
- Recovery commands should be simple and clear, prioritizing stash backup method
- Update HISTORY.md immediately after making changes, before or during commit
- Use the standard format provided in HISTORY.md for consistency

### 8. Git Commit and History Integration

- When committing changes, update `HISTORY.md` simultaneously
- Ensure commit messages match History entries
- Record commit hash in History for traceability
- Use `git rev-parse HEAD` to get the current commit hash before committing

### 9. Recovery Procedures

- Check commit hash in HISTORY.md file when recovery is needed
- **Simple recovery method (recommended)**: Use `git stash` to backup current changes, then `git reset --hard <commit-hash>`
- Recovery commands should include both stash backup and direct reset options
- Always document recovery commands in HISTORY.md with simple, clear instructions
- Example recovery format:

  ```bash
  # Backup and recover
  git stash push -u -m "백업"
  git reset --hard <커밋해시>

  # Or direct recovery
  git reset --hard <커밋해시>
  ```

### 10. AI Assistant History Recording

- When AI Assistant makes changes, automatically update HISTORY.md
- Before completing a task, check if HISTORY.md needs updating
- Include commit hash after changes are committed
- Format entries according to the template in HISTORY.md
- For multiple related changes, group them in a single entry
- Always record significant changes immediately, don't defer to later

### 11. Commit Workflow

- Before committing: Update HISTORY.md with the change details
- Get commit hash: `git rev-parse HEAD` (after commit)
- Update HISTORY.md with the actual commit hash
- Ensure commit message matches HISTORY.md entry description
- Use conventional commit format in both commit message and HISTORY.md

## Project-Specific Rules

### Code Style

- Use TypeScript with strict type checking
- Follow React best practices
- Use functional components with hooks
- Maintain consistent naming conventions

### Module Development

- When adding new modules, update:
  - `types.ts` with new ModuleType enum value
  - `constants.ts` with module definition
  - `codeSnippets.ts` with code generation logic
  - Documentation in module description

### UI/UX Guidelines

- Maintain consistent styling with Tailwind CSS
- Ensure responsive design for different screen sizes
- Follow accessibility best practices
- Keep user interactions intuitive

### Testing and Quality

- Test modules individually before integration
- Verify pipeline execution flow
- Check for edge cases and error handling
- Ensure proper error messages for users

## AI Assistant Guidelines

When making changes:

1. Always check existing code patterns before implementing new features
2. Maintain consistency with existing codebase style
3. Update related files when adding new functionality
4. Consider backward compatibility
5. Document significant changes in code comments
6. Follow the project's file structure and naming conventions
7. **Update HISTORY.md** when making significant changes (see section 10)
8. Record commit hash in HISTORY.md after committing changes
9. Follow the commit workflow (section 11) for proper change tracking

---
> Source: [taekyoleen-oss/life-matrix-flow-new](https://github.com/taekyoleen-oss/life-matrix-flow-new) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
