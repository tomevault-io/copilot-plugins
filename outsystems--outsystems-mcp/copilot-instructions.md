## outsystems-mcp

> Guidance for Claude Code (and other coding agents) when working in this repository.

# CLAUDE.md

Guidance for Claude Code (and other coding agents) when working in this repository.

## Skill docs must stay in lockstep across hosts

This repo ships **three parallel skill documents**:

- `skills/outsystems/SKILL.md` is the Claude Code marketplace skill, consumed when a user runs `claude plugin install outsystems@outsystems`.
- `kiro/outsystems/steering/skill.md` is the Kiro Power steering doc, consumed by Kiro.
- `SKILL.md` at the repo root is the top-level fallback skill doc, consumed by hosts that look at the repo root or by anyone reading the repo on GitHub.

All three carry an identical `## Rules` section, and broadly the same `## Tools at a glance`, `## Caveats`, and `## Workflows` sections. **Any behavioral change to one MUST be applied to all three.** Updating only one creates a host-specific protection gap. For example, a confirm-before-destructive rule added to `skills/outsystems/SKILL.md` alone protects Claude Code users but leaves Kiro Power users (which is how OutSystems uses the skill internally) with no protection.

The common failure mode is to edit only `skills/outsystems/SKILL.md` because the marketplace install path points there. Don't.

### Check before opening a PR

After any skill-doc change, grep for a distinctive phrase from the change across all three files and confirm the count matches:

```bash
PHRASE="<a distinctive substring from your change>"
for f in skills/outsystems/SKILL.md kiro/outsystems/steering/skill.md SKILL.md; do
  printf '%s  %s\n' "$(grep -c "$PHRASE" "$f")" "$f"
done
```

All three counts must be equal. If they aren't, the change is incomplete and the PR will create a host-specific drift.

### Exception: setup / installation flows

Setup steps legitimately diverge between the three (Claude Code uses `claude mcp add`, Kiro Power patches `~/.kiro/settings/mcp.json`, the root SKILL.md describes the wire-level tool names without prescribing a host). The lockstep rule applies to `## Rules` and to behavioral guidance, not to host-specific install recipes.

---
> Source: [OutSystems/outsystems-mcp](https://github.com/OutSystems/outsystems-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-28 -->
