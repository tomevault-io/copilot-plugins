## merit-agent-access

> Use Merit Protocol APIs to upload Agent Skills packages, discover Skills Token markets, request access, and download reviewed skill packages.


# Merit Agent Access

Use this skill when an agent needs to work with Merit Protocol directly:

- discover listed Skills Token markets;
- upload an agentskills.io-compatible package for review;
- request paid access to a listed skill;
- download the reviewed package after access settlement.

Merit Protocol does not execute downloaded skills for you. In V1, usage means a paid access or package delivery event. Run the downloaded skill in your own agent environment.

## Configuration

The bundled CLI is `scripts/merit-agent.js`. It requires Node.js 20+.

Set these environment variables when possible:

```bash
export MERIT_API_URL=http://localhost:4000
export MERIT_SESSION_TOKEN=<wallet-session-token>
```

For local development only, obtain a demo session token from a non-production Merit API:

```bash
node scripts/merit-agent.js login --wallet <wallet-address>
```

Production APIs require a real wallet signature. In that case, provide `MERIT_SESSION_TOKEN` from a signed wallet login flow.

## Discover Skills

List visible skills:

```bash
node scripts/merit-agent.js list --limit 10
```

Search or inspect one skill:

```bash
node scripts/merit-agent.js list --query repo
node scripts/merit-agent.js detail repo-review-copilot
```

## Upload A Skill Package

Submit another agentskills.io package for Merit review:

```bash
node scripts/merit-agent.js submit-package ./my-skill.zip
```

This command uploads the archive to `POST /v1/skills/package-upload`, then creates a reviewing skill with server-confirmed package metadata. The package must contain exactly one `SKILL.md` at the archive root or inside one top-level directory whose name matches the `SKILL.md` frontmatter `name`.

## Request Access And Download

Request package access:

```bash
node scripts/merit-agent.js use repo-review-copilot --merit 0.1
```

Download the package after the access event is settled:

```bash
node scripts/merit-agent.js download repo-review-copilot --output ./downloads
```

The wallet in `MERIT_SESSION_TOKEN` must match the wallet used for access unless the session belongs to the creator or an admin.

## Output

Commands print JSON by default so other agents can parse results. Add `--pretty` for formatted JSON.

---
> Source: [Synchestria/merit-agent-access](https://github.com/Synchestria/merit-agent-access) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
