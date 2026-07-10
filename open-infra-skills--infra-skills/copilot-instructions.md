## infra-skills

> Maintain a portable, evidence-driven library of AI infrastructure agent skills. Favor workflows that generalize across organizations while preserving the technical details needed to reproduce a diagnosis or optimization.

# Repository Instructions

## Purpose

Maintain a portable, evidence-driven library of AI infrastructure agent skills. Favor workflows that generalize across organizations while preserving the technical details needed to reproduce a diagnosis or optimization.

## Structure

- Put installable skills at `skills/<category>/<skill-name>/`.
- Keep exactly one category layer between `skills/` and the skill directory for broad CLI compatibility.
- Make the skill directory name match the frontmatter `name`.
- Do not add `README.md` inside a skill directory.
- Use `references/`, `scripts/`, and `agents/` only when they materially improve the skill.
- Treat `catalog.json` as generated output; update it with `npm run catalog`.

## Skill Authoring

- Keep `SKILL.md` focused, imperative, and under 500 lines.
- Put trigger terms and task boundaries in the frontmatter description.
- Use progressive disclosure instead of loading every detail in the main file.
- Prefer primary documentation and measured evidence for hardware or software claims.
- Mark examples as templates when paths, ports, versions, or device IDs must be adapted.
- Make correctness and reproducibility checks explicit.
- Record useful negative results when they prevent repeated work.

## Portability And Security

- Do not assume Windows, PowerShell, VS Code, SSH, a jump host, Slurm, Kubernetes, or any one cloud unless the skill is explicitly about that transport.
- Keep credentials, tokens, private IP addresses, internal hostnames, proprietary paths, and private registry names out of committed files.
- Never weaken TLS, host-key verification, authentication, or container isolation as a convenience shortcut.
- Avoid destructive commands. Give stop conditions before high-impact profiling or distributed experiments.

## Validation

Run before committing:

```bash
npm test
```

Also execute representative paths for any changed helper scripts. Do not hand-edit generated catalog entries.

---
> Source: [open-infra-skills/infra-skills](https://github.com/open-infra-skills/infra-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
