## sudoku-of-jesus

> Se não existir um repo criado, crie um novo, perguntando para o user qual será o nome do repo;

# **Git & GitHub Best Practices**

Se não existir um repo criado, crie um novo, perguntando para o user qual será o nome do repo;
Se não existir um README.md crie um novo;


## **0. Princípio de Rastreabilidade (NOVO)**
🔗 **Toda alteração deve ter um commit explicativo**  
- Nada de "quick fix" sem registro
- Mudanças no CI/CD? Commit
- Atualização de dependência? Commit
- Correção de typo? Commit

**Exemplo Ruim:**  
`git commit -m "fix"`

**Exemplo Bom:**  
- fix(typo): corrige erro de português na homepage [WEB-42]



## **1. Core Principles**
✅ **Atomic Commits** - One logical change per commit  
✅ **Clear Messages** - Explain "why" not just "what"  
✅ **Branch Hygiene** - Keep main/master always deployable  
✅ **Security First** - Never commit secrets  
✅ **Code Review Culture** - PRs are mandatory

## **2. Branch Strategy**
### **Branch Naming Convention**
feature/[ticket-id]-short-description (feature/ABC-123-add-login)
fix/[issue]-description (fix/456-email-validation)
hotfix/urgent-issue (hotfix/login-crash)

text

### **Branch Lifecycle**
1. Create from latest `main`  
2. Rebase (don't merge) to update  
3. Delete after PR merge  
4. Sync local branches weekly

## **3. Commit Standards**
### **Commit Message Format**
type(scope): subject [ticket]

body (optional)

footer (optional)

text

**Types:**  
- feat (new feature)  
- fix (bug fix)  
- docs (documentation)  
- refactor (code improvement)  
- test (tests)  
- chore (maintenance)

**Example:**
feat(auth): add password reset [ABC-123]

Implemented reset flow

Added email templates

Updated tests

BREAKING CHANGE: New env vars required

text

## **4. Pull Request Guidelines**
### **PR Requirements**
✔️ Linked to ticket/issue  
✔️ Small scope (< 300 lines ideally)  
✔️ Passing CI checks  
✔️ Updated documentation  
✔️ Screenshots for UI changes  

### **Review Standards**
- **Required approvals:** 2 for features, 1 for fixes  
- **Timebox:** 24h for normal PRs, 2h for hotfixes  
- **Comments must be:**  
  - Actionable ("Move this to helper")  
  - Specific ("Line 42 could use...")  
  - Polite ("Consider...")  

## **5. Security Rules**
### **Never Commit**
- API keys  
- Database credentials  
- .env files  
- Private certificates  
- AWS/GCP credentials  

### **If Accidentally Committed**
1. Rotate ALL exposed secrets immediately  
2. Use `git filter-branch` or BFG to purge history  
3. Add to `.gitignore`  
4. Verify with `git log -p | grep "secret"`

## **6. .gitignore Essentials**
Standard
node_modules/
.env
*.log
.DS_Store
dist/

IDE
.idea/
.vscode/
*.swp

System
Thumbs.db
ehthumbs.db

text

## **7. Automation & Hooks**
### **Recommended Hooks**
- Pre-commit: Run linter  
- Pre-push: Run tests  
- Commit-msg: Enforce message format  

**Example Husky Config:**
```json
{
  "hooks": {
    "pre-commit": "npm run lint",
    "commit-msg": "commitlint -E HUSKY_GIT_PARAMS",
    "pre-push": "npm test"
  }
}

## **8. GitHub Specifics**
Repository Settings
Require signed commits

Enforce branch protections:

Require PRs

Require status checks

Require linear history

Enable vulnerability alerts

Issue Management
Use templates for bugs/features

Apply labels consistently

Reference commits in issues (#123)

## **9. Code Review Checklist**
✅ Does what it claims in description
✅ Follows existing patterns
✅ No commented-out code
✅ Proper error handling
✅ No sensitive data
✅ Documentation updated
✅ Tests included

## **10. Emergency Protocols**
Hotfix Process
Branch from production tag

Fix and test locally

PR with "HOTFIX" prefix

Approve + merge

Tag new release

Rollback Steps
Revert merge commit:
git revert -m 1 <commit-hash>

Force push if necessary

Communicate widely

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Alex-luna) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
