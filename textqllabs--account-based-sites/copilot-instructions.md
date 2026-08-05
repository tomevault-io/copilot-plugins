## account-based-sites

> - **Repository:** https://github.com/TextQLLabs/account-based-sites

# Claude Code Workflow for account-based-sites

## Project Context
- **Repository:** https://github.com/TextQLLabs/account-based-sites
- **Local path:** `/Users/ethanding/projects/account-based-sites`
- **Deployment:** Netlify via `./deploy.sh`
- **Sites:** athenahealth, UnitedHealthcare, Capital One, Mastercard
- **Tech stack:** Chart.js visualizations, Express server, vanilla HTML/CSS/JS

## Required Workflow - START EVERY SESSION WITH THIS:

### 1. Always Pull First
```bash
cd /Users/ethanding/projects/account-based-sites
git pull origin main
```

### 2. Check for Merge Conflicts
- Resolve any conflicts before making changes
- Test that local server still works: `npm run dev`

### 3. Standard Development
- Make requested changes
- Test locally at http://localhost:3000

### 4. Always Commit & Push When Done
```bash
git add .
git commit -m "Descriptive commit message"
git push origin main
```

## Collaboration Notes
- Small team with direct write access to main branch
- No PR reviews required
- Multiple people may be working on this simultaneously
- **CRITICAL:** Always pull before starting work to avoid conflicts

## Site URLs (Local)
- Main: http://localhost:3000
- athenahealth: http://localhost:3000/athenahealth  
- UnitedHealthcare: http://localhost:3000/unitedhealthcaregroup
- Capital One: http://localhost:3000/capitalone
- Mastercard: http://localhost:3000/mastercard

## Site URLs (Live)
- Main: https://workshops.textql.com
- athenahealth: https://workshops.textql.com/athenahealth
- UnitedHealthcare: https://workshops.textql.com/unitedhealthcaregroup

---
> Source: [TextQLLabs/account-based-sites](https://github.com/TextQLLabs/account-based-sites) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-03 -->
