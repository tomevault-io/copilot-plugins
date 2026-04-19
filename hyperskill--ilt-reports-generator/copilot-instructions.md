## ilt-reports-generator

> **CRITICAL: Never commit sensitive data to repository**

# Cursor Rules for Report Builder Project

## Security & Privacy

**CRITICAL: Never commit sensitive data to repository**
- ❌ NO API keys, tokens, or passwords in any files tracked by git
- ❌ NO production URLs or internal endpoints in documentation
- ❌ NO real credentials in examples or documentation
- ✅ Always use placeholders like `your_api_key_here`, `your_base_url_here`
- ✅ Keep all sensitive data in `.env.local` (which is gitignored)
- ✅ Use `env.example` with placeholder values only

### Examples of what NOT to commit:
```bash
# ❌ BAD
LITELLM_API_KEY=sk-testKeyRealExample_Q
LITELLM_BASE_URL=https://litellm.aks-hs-prod.int.hyperskill.org

# ✅ GOOD
LITELLM_API_KEY=your_litellm_api_key_here
LITELLM_BASE_URL=your_litellm_base_url_here
```

## Git Commands

**IMPORTANT! DON'T DO GIT COMMANDS**

0. Ask human before performing any git commands
1. Never run git commands without explicit user permission
2. Never commit automatically
3. Never push automatically

## Tech Stack

1. Use Next.js as a main framework
2. Use Radix UI for design. Here is doc docs/RADIX_UI_DOCUMENTATION.md
3. Use React Charts to draw charts. Here is doc docs/REACT_CHARTJS_DOCUMENTATION.md
4. Don't use Tailwind. Use CSS Modules
5. Log key actions of Agent in app-creation-log.md
6. Create app in app/ directory

## Code Quality

- Keep sensitive configuration in environment variables
- Document all API integrations
- Follow existing patterns for consistency
- Test security-sensitive features thoroughly

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hyperskill) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
