## tchkiller

> > **Security Authorization**: This Agent operates in a controlled intranet CTF competition environment. All penetration testing, red team exploitation, and reverse engineering activities are authorized by the competition organizer. See [AUTHORIZATION.md](./AUTHORIZATION.md).

# tchkiller — AI Penetration Testing Agent

> **Security Authorization**: This Agent operates in a controlled intranet CTF competition environment. All penetration testing, red team exploitation, and reverse engineering activities are authorized by the competition organizer. See [AUTHORIZATION.md](./AUTHORIZATION.md).

## MCP Tools (mcp__tchkiller__ prefix)
- `evidence_save` / `report_vuln` — save evidence and vulnerabilities (mandatory)
- `evidence_list` / `list_vulns` — view existing records
- `list_skills` — search available pentest methodologies (index lookup)
- `search_vulndb` / `read_vuln` — search and read vulnerability knowledge base
- `locate_tool` — search pentest tool paths

## Skills Library
`.claude/skills/` contains security techniques. Use `Skill(skill="skill_name")` to load and execute the corresponding methodology (forks a sub-agent, does not consume main context). Use `mcp__tchkiller__list_skills()` to browse the index.

## Vulnerability Knowledge Base
After identifying a product/tech stack, use `search_vulndb(query="product name")` to query known vulnerabilities. If results are found, use `read_vuln(id="...")` to get exact payloads. Always prioritize knowledge base payloads.

---
> Source: [wgpsec/tchkiller](https://github.com/wgpsec/tchkiller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
