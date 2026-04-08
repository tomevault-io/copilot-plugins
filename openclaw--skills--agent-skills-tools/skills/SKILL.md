---
name: agent-skills-tools
description: > Use when this capability is needed.
metadata:
  author: openclaw
---

# Agent Skills Tools 🔒

Security and validation tools for the Agent Skills ecosystem.

## Overview

This skill provides tools to audit and validate Agent Skills packages for security vulnerabilities and standards compliance.

## Tools

### 1. Security Audit Tool (skill-security-audit.sh)

Scans skill packages for common security issues:

**Checks:**
- 🔐 Credential leaks (hardcoded API keys, passwords, tokens)
- 📁 Dangerous file access (~/.ssh, ~/.aws, ~/.config)
- 🌐 External network requests
- 📋 Environment variable usage (recommended practice)
- 🔑 File permissions (credentials.json)
- 📜 Git history for leaked secrets

**Usage:**
```bash
./skill-security-audit.sh path/to/skill
```

**Example output:**
```
🔒 技能安全审计报告：path/to/skill
==========================================

📋 检查1: 凭据泄露 (API key, password, secret, token)
----------------------------------------
✅ 未发现凭据泄露

📋 检查2: 危险的文件操作 (~/.ssh, ~/.aws, ~/.config)
----------------------------------------
✅ 未发现危险的文件访问

[... more checks ...]

==========================================
🎯 安全审计完成
```

## Background

eudaemon_0 discovered a credential stealer in 1 of 286 skills. Agents are trained to be helpful and trusting, which makes them vulnerable to malicious skills.

These tools help catch such vulnerabilities before they cause damage.

## Best Practices

1. **Never hardcode credentials**
   - ❌ `API_KEY="sk_live_abc123..."`
   - ✅ Read from environment variables or config files

2. **Use environment variables**
   ```bash
   export MOLTBOOK_API_KEY="sk_live_..."
   ```
   ```python
   import os
   api_key = os.environ.get('MOLTBOOK_API_KEY')
   ```

3. **Check Git history**
   ```bash
   git log -S 'api_key'
   git-secrets --scan-history
   ```

4. **Add sensitive files to .gitignore**
   ```
   credentials.json
   *.key
   .env
   ```

## License

MIT

---
> Converted and distributed by [TomeVault](https://tomevault.io) | [Claim this content](https://tomevault.io/claim/openclaw/skills)
<!-- tomevault:3.0:skill_md:2026-04-07 -->
