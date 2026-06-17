## skills-guardian

> Detect suspicious OpenClaw skills and reduce supply-chain risk. Use when installing skills from ClawHub/GitHub, reviewing a SKILL.md, auditing a skill folder for malicious patterns (curl/wget/powershell/rm/exfil), or when the user wants an install workflow that automatically scans every newly installed skill.


# Skills Guardian (suspicious-skill detector)

## What this does

- Scan a skill folder (usually `<workspace>/skills/<skill-name>/`) and produce a **risk report**.
- Provide a **safe install workflow**: install via ClawHub, then immediately scan what was installed.

This is a heuristic scanner (static checks). It cannot guarantee safety.

## Quick usage

### A) Audit an existing skill

Run (Windows):

```powershell
$py="C:\\Users\\lam00\\AppData\\Local\\Programs\\Python\\Python311\\python.exe"
& $py C:\\Users\\lam00\\.openclaw\\workspace\\skills\\skills-guardian\\scripts\\audit_skill.py "C:\\Users\\lam00\\.openclaw\\workspace\\skills\\<skill-name>"
```

### B) Install + auto-scan (recommended)

Use the provided wrapper:

```powershell
C:\Users\lam00\.openclaw\workspace\skills\skills-guardian\scripts\safe_install.cmd <clawhub-skill-slug>
```

It will:
1) install into `C:\Users\lam00\.openclaw\workspace\skills\`
2) scan the newly installed folder(s)
3) write `skills-guardian-report.json` + `skills-guardian-report.txt`

## What to do if something is flagged

- **High risk**: do not enable/use the skill; open the flagged files and review manually.
- **Medium risk**: verify URLs/domains, look for secrets handling and data exfil patterns.
- **Low risk**: still review if the skill will run shell/network tools.

## Notes / limitations

- Skills are mostly instructions + scripts; a SKILL.md can still trick humans. Treat unknown skills as untrusted.
- This scanner looks for common red flags; it will produce false positives.

---
> Source: [ok406lhq/skills-guardian](https://github.com/ok406lhq/skills-guardian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
