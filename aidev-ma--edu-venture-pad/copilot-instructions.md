## edu-venture-pad

> Standards Git et versioning pour IFES


# Git & Versioning Standards - Enterprise Grade

## Branches Strategy

IFES utilise **Git Flow** avec branches features numérotées.

### Structure des Branches

```
main (production)
  ├── develop (intégration)
  │   ├── 001-jwt-auth-impl (feature JWT auth)
  │   ├── 002-payment-integration (feature paiements)
  │   ├── 003-course-management (feature gestion cours)
  │   └── hotfix/fix-login-bug
```

### Branches Principales

**main:**
- Code production stable
- Déployé automatiquement
- Protected: Require PR + review
- Tag pour chaque release (v1.0.0, v1.1.0)

**develop:**
- Code intégration (pre-production)
- Toujours stable et testable
- Merge features via PR
- Protected: Require PR

### Speckit et specs depuis `develop` : `SPECIFY_FEATURE`

Pour travailler sur **`specs/NNN-feature/`** tout en restant sur **`develop`** (sans `git checkout NNN-feature` pour chaque spec) :

- Définir dans PowerShell : **`$env:SPECIFY_FEATURE = 'NNN-feature'`** (nom du dossier sous `specs/`) **avant** les scripts `.specify/scripts/powershell/*.ps1` qui résolvent le répertoire de la feature.
- Les **agents** ne doivent pas supposer que le nom de branche Git = la spec active ; en cas de doute, utiliser ou demander **`SPECIFY_FEATURE`**.
- Détail et obligations agents : [.cursor/rules/spec-driven-development.mdc](mdc:.cursor/rules/spec-driven-development.mdc) (section « Mode intégration : branche develop + SPECIFY_FEATURE »).

### Feature Branches

**Naming Convention:**
```
<number>-<feature-name>

Exemples:
001-jwt-auth-impl
002-payment-integration
003-course-management
004-multi-agent-system
```

**Règles:**
- Créer depuis `develop`
- Nom descriptif et numéroté
- Une feature = une branche
- Delete après merge

**Workflow:**
```bash
# Créer feature branch
git checkout develop
git pull origin develop
git checkout -b 003-course-management

# Développer
git add .
git commit -m "feat: add course creation endpoint"

# Pousser régulièrement
git push origin 003-course-management

# Merge request vers develop
# Après review et merge, supprimer branch
git checkout develop
git pull origin develop
git branch -d 003-course-management
```

### Hotfix Branches

**Naming Convention:**
```
hotfix/<bug-name>

Exemples:
hotfix/fix-login-redirect
hotfix/cors-error
hotfix/payment-validation
```

**Workflow:**
```bash
# Créer depuis main
git checkout main
git pull origin main
git checkout -b hotfix/fix-login-redirect

# Fix
git add .
git commit -m "fix: correct login redirect logic"

# Merge vers main ET develop
git checkout main
git merge hotfix/fix-login-redirect
git push origin main

git checkout develop
git merge hotfix/fix-login-redirect
git push origin develop

# Tag release
git tag v1.0.1
git push origin v1.0.1

# Delete hotfix
git branch -d hotfix/fix-login-redirect
```

## Commit Messages

### Conventional Commits

IFES utilise **Conventional Commits 1.0.0**.

**Format:**
```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

```
feat:     Nouvelle fonctionnalité
fix:      Correction de bug
docs:     Documentation uniquement
style:    Formatage (pas de changement code)
refactor: Refactoring (pas de nouvelle feature ni fix)
perf:     Amélioration performance
test:     Ajout/modification tests
chore:    Maintenance (build, dépendances, etc.)
ci:       Configuration CI/CD
revert:   Revert d'un commit précédent
```

### Scopes

```
frontend:  Code frontend (React)
backend:   Code backend (Express)
auth:      Authentification
courses:   Gestion cours
payments:  Paiements
database:  Migrations Supabase
docs:      Documentation
config:    Configuration
```

### Exemples

✅ **CORRECT:**

```bash
# Feature simple
git commit -m "feat(auth): implement JWT authentication"

# Fix avec description
git commit -m "fix(backend): correct CORS configuration

CORS was blocking requests from Lovable subdomains.
Updated origin validation to support wildcard subdomains.

Closes #42"

# Breaking change
git commit -m "feat(courses)!: change course pricing structure

BREAKING CHANGE: Course price is now stored in cents instead of MAD.
Migration required for existing data."

# Multiple changes
git commit -m "refactor(frontend): reorganize service layer

- Move authService to services/authService.ts
- Update imports in components
- Add JSDoc comments"
```

❌ **INCORRECT:**

```bash
# Trop vague
git commit -m "fix bug"
git commit -m "update code"
git commit -m "changes"

# Pas de type
git commit -m "authentication implementation"

# Plusieurs types
git commit -m "feat: add login + fix bug + update docs"

# Capitalized subject
git commit -m "feat: Add Login Feature"
```

### Règles Commits

1. **Type obligatoire:** Toujours commencer par type
2. **Subject lowercase:** Pas de majuscule après `:` (sauf noms propres)
3. **No period:** Pas de point final dans subject
4. **Imperative mood:** "add", "fix", pas "added", "fixed"
5. **Max 72 chars:** Subject ≤ 72 caractères
6. **Body si nécessaire:** Expliquer WHY et WHAT, pas HOW
7. **Breaking changes:** Ajouter `!` après type et `BREAKING CHANGE:` dans footer
8. **Reference issues:** `Closes #42`, `Fixes #123`, `Refs #456`

## Pull Requests

### PR Title

**Format:** Même que commit (Conventional Commits)

```
feat(auth): implement JWT authentication
fix(cors): support Lovable subdomains
docs: update API documentation
```

### PR Description Template

```markdown
## Description

Brief description of changes.

## Type of Change

- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Documentation update
- [ ] Performance improvement
- [ ] Code refactoring

## Related Issues

Closes #42
Refs #123

## Changes Made

- Implemented JWT authentication
- Added authMiddleware for protected routes
- Updated frontend to store tokens in localStorage
- Added token refresh logic

## Testing Done

- [ ] Manual testing on localhost
- [ ] Tested login flow
- [ ] Tested protected routes
- [ ] Tested token expiration
- [ ] Tested CORS with Lovable

## Screenshots (if applicable)

[Add screenshots]

## Checklist

- [ ] Code follows project style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex logic
- [ ] Documentation updated
- [ ] No console.log in production code
- [ ] TypeScript types added
- [ ] Error handling implemented
- [ ] Security considerations addressed
- [ ] No secrets committed

## Additional Notes

[Any additional information]
```

### PR Review Checklist

**Reviewer doit vérifier:**

- [ ] Code suit architecture hybride (Frontend → Backend → Supabase)
- [ ] Types TypeScript complets et corrects
- [ ] Gestion erreurs complète (try/catch)
- [ ] Validation inputs (Zod)
- [ ] Pas de secrets exposés
- [ ] CORS correctement configuré
- [ ] Logging approprié (structured logs)
- [ ] Pas de console.log en production
- [ ] Tests manuels effectués
- [ ] Documentation mise à jour
- [ ] Commits suivent Conventional Commits
- [ ] Pas de conflits avec develop

## Git Hooks

### Pre-commit Hook

```bash
#!/bin/sh
# .git/hooks/pre-commit

# Lint TypeScript
npm run lint
if [ $? -ne 0 ]; then
  echo "❌ ESLint failed. Fix errors before committing."
  exit 1
fi

# Type check
npm run type-check
if [ $? -ne 0 ]; then
  echo "❌ TypeScript type check failed."
  exit 1
fi

# Check for console.log (warning only)
if git diff --cached | grep -E "console\.(log|debug|info)" > /dev/null; then
  echo "⚠️  Warning: console.log found in staged files."
  echo "Consider removing before commit."
fi

# Check for secrets
if git diff --cached | grep -iE "(password|secret|api_key|token).*=.*['\"][^'\"]{20,}" > /dev/null; then
  echo "❌ Possible secret detected in commit."
  echo "Review and remove before committing."
  exit 1
fi

echo "✅ Pre-commit checks passed."
exit 0
```

### Commit-msg Hook

```bash
#!/bin/sh
# .git/hooks/commit-msg

commit_msg=$(cat "$1")

# Check Conventional Commits format
if ! echo "$commit_msg" | grep -qE "^(feat|fix|docs|style|refactor|perf|test|chore|ci|revert)(\(.+\))?(!)?: .+"; then
  echo "❌ Commit message does not follow Conventional Commits format."
  echo ""
  echo "Format: <type>(<scope>): <subject>"
  echo ""
  echo "Examples:"
  echo "  feat(auth): add JWT authentication"
  echo "  fix(cors): support Lovable subdomains"
  echo "  docs: update README"
  exit 1
fi

echo "✅ Commit message format valid."
exit 0
```

## Versioning (SemVer)

IFES utilise **Semantic Versioning 2.0.0**.

**Format:** `MAJOR.MINOR.PATCH`

```
v1.0.0 → v1.0.1 → v1.1.0 → v2.0.0
```

### Quand incrémenter

**MAJOR (1.0.0 → 2.0.0):**
- Breaking changes
- Changements incompatibles API
- Architecture majeure changée

Exemples:
- Changement format données (course price cents → MAD)
- Suppression endpoints API
- Changement authentification (Supabase → Custom)

**MINOR (1.0.0 → 1.1.0):**
- Nouvelles features
- Ajout endpoints API (backward compatible)
- Améliorations fonctionnelles

Exemples:
- Ajout feature paiements
- Nouveaux endpoints /api/analytics
- Support multi-devises

**PATCH (1.0.0 → 1.0.1):**
- Bug fixes
- Security patches
- Performance improvements

Exemples:
- Fix login redirect
- Correct CORS configuration
- Optimiser query Supabase

### Tagging Releases

```bash
# Créer tag
git tag -a v1.0.0 -m "Release v1.0.0: MVP Launch"

# Pousser tag
git push origin v1.0.0

# Lister tags
git tag -l

# Supprimer tag (si erreur)
git tag -d v1.0.0
git push origin --delete v1.0.0
```

## .gitignore

```gitignore
# Dependencies
node_modules/
.pnp
.pnp.js

# Build outputs
dist/
build/
out/
.next/

# Environment variables (IMPORTANT)
.env
.env.local
.env.production
.env.*.local

# Logs
logs/
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db

# Testing
coverage/
.nyc_output/

# Temporary
tmp/
temp/
*.tmp

# Secrets (CRITICAL)
*.pem
*.key
*.cert
secrets/

# Database
*.db
*.sqlite
*.sqlite3

# Cloudflare Workers
.wrangler/
wrangler.toml.bak

# Supabase
.branches/
```

## Git Best Practices

### Do's ✅

1. **Commit souvent:** Petits commits atomiques
2. **Pull avant push:** Toujours `git pull` avant `git push`
3. **Review avant commit:** `git diff --staged`
4. **Messages descriptifs:** Expliquer WHY, pas HOW
5. **Branch par feature:** Une feature = une branch
6. **Squash si nécessaire:** `git rebase -i` pour nettoyer historique
7. **Tag releases:** Toujours tag versions stables
8. **Protect main/develop:** Branch protection rules

### Don'ts ❌

1. **❌ Commit secrets:** JAMAIS commit .env, tokens, passwords
2. **❌ Commit node_modules:** Toujours dans .gitignore
3. **❌ Force push main:** JAMAIS `git push --force` sur main
4. **❌ Commit WIP:** Finir feature avant commit
5. **❌ Large commits:** Éviter commits > 500 lignes
6. **❌ Mixed changes:** Un commit = un changement logique
7. **❌ Skip review:** Toujours PR review avant merge

## Git Workflow Summary

```bash
# 1. Créer feature branch
git checkout develop
git pull origin develop
git checkout -b 003-course-management

# 2. Développer avec commits atomiques
git add src/routes/courses.ts
git commit -m "feat(courses): add GET /api/courses endpoint"

git add src/routes/courses.ts
git commit -m "feat(courses): add GET /api/courses/:id endpoint"

# 3. Pousser régulièrement
git push origin 003-course-management

# 4. Créer Pull Request
# Via GitHub/GitLab interface

# 5. Review et merge
# Reviewer approuve → Merge

# 6. Update local
git checkout develop
git pull origin develop

# 7. Delete feature branch
git branch -d 003-course-management
git push origin --delete 003-course-management
```

---

Ces standards garantissent un historique Git propre, traçable et professionnel.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aidev-ma) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
