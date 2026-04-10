## skin-cancer-detector

> > **SYNC REQUIREMENT**: This file must be kept in sync with `.cursor/rules/project-standards.mdc`.

# Skin Cancer Detector Project Rules

> **SYNC REQUIREMENT**: This file must be kept in sync with `.cursor/rules/project-standards.mdc`.
> When updating rules, update BOTH files to maintain consistency across Cursor and Claude Code.

## About This Project

Skin Cancer Detector is an AI-powered application (web, Android, and iOS) that analyzes images of skin lesions to detect potential skin cancer. Users upload a photo through a Flutter frontend, the image is processed by a PyTorch deep learning model via a Flask backend, and results (confidence score + verdict) are returned to the user. Anonymous accounts are used to protect privacy.

### Architecture

```
Flutter -> Firebase Storage -> Flask -> OpenCV/PyTorch -> Firestore -> Flutter
```

### Tech Stack

- **Frontend**: Flutter (Dart) -- web, Android, and iOS
- **Backend**: Flask (Python) -- REST API
- **AI/ML**: PyTorch (CNN classification), OpenCV (image preprocessing)
- **Cloud**: Google Cloud Platform, Firebase (Auth, Storage, Firestore)
- **Auth**: Firebase Anonymous Authentication

## Code Standards

### Dart (Frontend)

- Follow [Effective Dart](https://dart.dev/effective-dart) style guidelines
- Use `lowerCamelCase` for variables, functions, and parameters
- Use `UpperCamelCase` for classes, enums, typedefs, and type parameters
- Use `snake_case` for file and directory names
- Prefer `const` constructors where possible
- Use `final` for variables that are assigned once
- Include type annotations for function signatures and public APIs
- Use null safety (`?`, `!`, `late`) appropriately -- avoid unnecessary `!` assertions
- Handle all async operations with proper error handling (`try/catch` or `.catchError`)

### Python (Backend / ML)

- Follow [PEP 8](https://peps.python.org/pep-0008/) style guidelines
- Use `snake_case` for functions, variables, and file names
- Use `PascalCase` for class names
- Use `UPPER_SNAKE_CASE` for constants
- Type hints for all function signatures (`def func(param: str) -> dict:`)
- Docstrings for all public functions and classes (Google style)
- Maximum line length: 88 characters (Black formatter)
- Use `pathlib.Path` over `os.path` for file operations
- Use `logging` module, not `print()`, for backend output
- Virtual environment required -- never install packages globally

### General

- Write clear, maintainable code with proper documentation
- Include error handling for all asynchronous operations
- No hardcoded secrets, API keys, or credentials in source code
- Use environment variables for all configuration

## Development Workflow

- Test changes before committing
- Write descriptive commit messages following conventional commits format
- Keep dependencies up to date
- Follow the existing project structure
- Run linters before pushing (`dart analyze` for Flutter, `flake8`/`black` for Python)

## File Organization

- Keep related files together in logical directories
- Frontend code stays in `frontend/`, backend in `backend/`
- Maintain clear separation of concerns:
  - `frontend/lib/screens/` -- UI screens
  - `frontend/lib/services/` -- Firebase/API service classes
  - `frontend/lib/widgets/` -- Reusable UI components
  - `frontend/lib/models/` -- Dart data models
  - `backend/cloud/main.py` -- Cloud Function entry point
  - `backend/cloud/Dockerfile` -- Container image for backend deployment
  - `backend/requirements.txt` -- Python dependencies

## Documentation

- Document complex logic with inline comments
- Update README when adding new features or changing setup steps
- Keep API documentation current (endpoint signatures, request/response formats)
- Docstrings on all Python public functions
- Dart doc comments (`///`) on all public classes and methods

## Firebase & Cloud Operations

### Firestore

- Use the Firebase Admin SDK in the backend for Firestore reads/writes
- Structure: `results/{analysisId}`
- Always include `createdAt` timestamp in documents
- Keep Firestore security rules restrictive -- anonymous users should only read their own results

### Firebase Storage

- Images are uploaded to `uploads/{anonymousUserId}/{timestamp}_{filename}`
- Set lifecycle rules to auto-delete images after processing (privacy)
- Keep storage security rules restrictive

### Firebase Auth

- Anonymous authentication only -- no email/password, no OAuth
- Session persistence handled by Flutter Firebase SDK

## AI Model

- Inference endpoint must return: `{ verdict: string, confidence: float, recommendation: string }`
- Confidence score is a percentage (0-100) representing the model's certainty

## Rules Synchronization

**CRITICAL**: When modifying project rules:

1. **For Claude Code users**: Edit this `AGENTS.md` file, then copy changes to `.cursor/rules/project-standards.mdc` (preserving frontmatter)
2. **For Cursor users**: Edit `.cursor/rules/project-standards.mdc`, then copy changes to this `AGENTS.md` (removing frontmatter)
3. **Content must match**: The main content body should be identical in both files
4. **Version control**: Commit both files together when making rule changes

### File Locations

- **Claude Code**: `/AGENTS.md` (this file)
- **Cursor**: `/.cursor/rules/project-standards.mdc`

### Differences to Preserve

- Cursor files may have frontmatter metadata (`---` blocks at top)
- Claude Code AGENTS.md should NOT have frontmatter
- Otherwise, content should be identical

## External File References

When referencing project files in rules, use the `@filename` syntax:

```
@pubspec.yaml
@requirements.txt
```

This works in both Cursor and Claude Code to include file content in context.

## Code Review with CodeRabbit

This project uses [CodeRabbit](https://coderabbit.ai) for automated code review.

- The CodeRabbit **agent skill** is installed at `.agents/skills/code-review/SKILL.md` and is self-documenting
- CodeRabbit **automatically reviews PRs** on GitHub when they are opened or updated
- For local reviews, ask your AI agent to "review my code" or use the `cr` CLI directly

## PR Workflow & Development Strategy

### Small, Focused PRs

Follow these principles for all pull requests:

**Size Guidelines:**
- **Ideal**: 100-300 lines of changes
- **Maximum**: 500 lines (if larger, split into multiple PRs)
- **Goal**: Each PR should be reviewable in 15-30 minutes

**Breaking Down Features:**

Use **vertical splits** (full-stack per feature) or **horizontal splits** (layer by layer):

```
Vertical (Recommended for small features):
  PR 1: Add image upload (Storage + Flutter UI)
  PR 2: Add AI inference endpoint (Flask + PyTorch)
  PR 3: Add results display (Firestore + Flutter UI)

Horizontal (Better for large features):
  PR 1: Backend model loading + preprocessing
  PR 2: Flask API endpoints + tests
  PR 3: Flutter UI screens + Firebase integration
```

**Stacking PRs:**
- Don't wait for PR #1 to merge before starting PR #2
- Create new branch from previous PR's branch
- Keep momentum while reviews happen
- Update base branch after PR #1 merges

**Example workflow:**
```bash
# PR 1
git checkout -b feature/image-upload
# ... implement & commit ...
git push -u origin feature/image-upload

# PR 2 (don't wait for PR 1!)
git checkout -b feature/ai-inference
# ... implement & commit ...
git push -u origin feature/ai-inference
```

### Learning Through Code Review

**For every PR you create:**

1. **Self-review first** (5-10 min)
   - Read your own diff as if reviewing someone else's code
   - Check for obvious issues
   - Ensure tests cover the changes

2. **Use CodeRabbit** (automated)
   - Run `cr review --plain --base main` or ask your AI agent to "review my code"
   - Fix Critical and Warning issues
   - Consider Info suggestions

3. **Manual review** (10-30 min)
   - **Design**: Does this approach make sense?
   - **Edge cases**: What could break?
   - **Security**: Any vulnerabilities? (especially around image uploads and anonymous auth)
   - **Tests**: Are they comprehensive?
   - **Learning**: What patterns did AI use? Would you do it differently?

4. **Document learnings**
   - Keep notes on patterns you discover
   - Update project rules if you find recurring issues

**Review checklist:**
```
- [ ] Overall design makes sense
- [ ] Edge cases handled (null, empty, boundaries)
- [ ] Error handling is robust
- [ ] Security considerations addressed (file upload validation, auth, Firestore rules)
- [ ] Tests cover happy path + edge cases
- [ ] Performance is acceptable (image size limits, model inference time)
- [ ] Code is readable and maintainable
- [ ] I understand what the code does and why
```

### When to Create a PR

**Ready to merge when:**
- Feature/fix is complete and working
- CodeRabbit review passes (no Critical/Warning issues)
- Manual self-review completed
- Tests pass (if applicable)
- You understand all the changes
- Documentation updated (if needed)

**Not ready if:**
- "Just trying something out"
- Incomplete feature (unless explicitly marked WIP)
- You don't understand parts of the code
- Known bugs or issues not documented

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Josh7511)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Josh7511)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
