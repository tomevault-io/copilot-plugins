## expo-app-store-guideline-check

> This file contains guidelines for AI assistants working on this repository.

# CLAUDE.md - Development Guidelines

This file contains guidelines for AI assistants working on this repository.

## Commit Conventions

This repository follows **Conventional Commits** specification. All commits must follow this format:

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### Commit Types

| Type | Description | Example |
|------|-------------|---------|
| `feat` | New feature | `feat(templates): add English privacy policy template` |
| `fix` | Bug fix | `fix(scripts): correct path resolution in scan-project.sh` |
| `docs` | Documentation changes | `docs(readme): update installation instructions` |
| `refactor` | Code refactoring without behavior change | `refactor(checks): simplify permission validation logic` |
| `style` | Formatting, whitespace changes | `style(all): apply consistent markdown formatting` |
| `test` | Adding or modifying tests | `test(evals): add test cases for permission checking` |
| `chore` | Maintenance tasks | `chore(deps): update dependency versions` |
| `i18n` | Internationalization/translation | `i18n(references): translate Apple App Store guide to English` |

### Scopes

Use the directory or component being modified as the scope:

- `skill` - Main SKILL.md file
- `readme` - README.md
- `references` - Reference documentation files
- `checklists` - Checklist files
- `templates` - Template files
- `scripts` - Bash scripts
- `evals` - Test cases

### Atomic Commits

Each commit should represent **one logical change**:

✅ **Good:**
- `i18n(skill): translate SKILL.md to English`
- `i18n(references): translate all reference files to English`
- `feat(templates): add English versions of legal templates`

❌ **Bad:**
- `update files` (too vague, no type)
- `fix: translated everything` (multiple unrelated changes)
- `feat: new stuff` (no scope, unclear)

### Commit Message Guidelines

1. **Use imperative mood**: "translate" not "translated"
2. **Lowercase description**: `i18n(skill): translate to English` not `i18n(skill): Translate to English`
3. **No period at end**: `translate SKILL.md` not `translate SKILL.md.`
4. **Keep it concise**: Max 72 characters for the first line
5. **Reference issues**: Include issue numbers when applicable

### Examples

```bash
# Translation commits
git commit -m "i18n(skill): translate SKILL.md to English"
git commit -m "i18n(references): translate Apple App Store guide to English"
git commit -m "i18n(checklists): translate all checklists to English"

# Feature commits
git commit -m "feat(templates): add English privacy policy template"
git commit -m "feat(scripts): add permission validation script"

# Documentation commits
git commit -m "docs(readme): update installation instructions"
git commit -m "docs(skill): clarify execution flow section"

# Fix commits
git commit -m "fix(scripts): resolve path resolution in scan-project.sh"
```

## File Organization

```
expo-app-store-guideline-check/
├── SKILL.md              # Main skill definition
├── README.md             # Project documentation
├── CLAUDE.md             # This file - AI assistant guidelines
├── references/           # Reference documentation
│   ├── apple-app-store.md
│   ├── google-play-store.md
│   ├── lgpd-privacy.md
│   ├── gdpr-privacy.md
│   ├── lgpd-vs-gdpr.md
│   ├── react-native-expo.md
│   └── common-rejections.md
├── checklists/           # Pre-submission checklists
│   ├── pre-submission-ios.md
│   ├── pre-submission-android.md
│   ├── privacy-compliance.md
│   └── security-checklist.md
├── templates/            # Legal and form templates
│   ├── privacy-policy-pt-br.md
│   ├── privacy-policy-en.md
│   ├── privacy-policy-gdpr-en.md
│   ├── terms-of-use-pt-br.md
│   ├── terms-of-use-en.md
│   ├── app-privacy-labels.md
│   └── data-safety-form.md
├── scripts/              # Automation scripts
│   └── *.sh
└── evals/                # Test cases
    └── evals.json
```

## Language Policy

- **Main documentation**: English (default)
- **Legal templates**: Both Portuguese (PT-BR) and English (EN) versions
- **Code comments**: English
- **Commit messages**: English

## Translation Guidelines

When translating files:

1. **Preserve structure**: Keep the same markdown structure and formatting
2. **Keep technical terms**: Don't translate terms like "React Native", "Expo", "iOS", "LGPD", "GDPR"
3. **Preserve code blocks**: Don't translate code examples
4. **Keep URLs unchanged**: Don't modify links or file paths
5. **Maintain legal accuracy**: For legal templates, preserve legal terminology

## Pull Request Guidelines

1. Create a feature branch from `main`
2. Make atomic commits following the conventions above
3. Ensure all checks pass before requesting review
4. Reference related issues in the PR description

## Code Review Checklist

Before merging:

- [ ] All commits follow conventional commit format
- [ ] Commit messages are clear and descriptive
- [ ] Changes are atomic (one logical change per commit)
- [ ] Documentation is updated if needed
- [ ] No sensitive information in commits

---
> Source: [nandofalcao/expo-app-store-guideline-check](https://github.com/nandofalcao/expo-app-store-guideline-check) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
