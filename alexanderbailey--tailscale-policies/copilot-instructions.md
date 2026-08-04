## tailscale-policies

> This is a GitOps repository that serves as the **single source of truth** for Tailscale ACL (Access Control List) policies. Changes to `policy.hujson` are automatically validated on pull requests and deployed to the Tailscale network on merge to `main`.

# CLAUDE.md — Tailscale GitOps Policy Repository

## Repository Purpose

This is a GitOps repository that serves as the **single source of truth** for Tailscale ACL (Access Control List) policies. Changes to `policy.hujson` are automatically validated on pull requests and deployed to the Tailscale network on merge to `main`.

> **Critical rule:** Never edit policies directly in the Tailscale admin console. All changes must go through this repository.

---

## Repository Structure

```
tailscale_policies/
├── .github/
│   └── workflows/
│       └── tailscale.yml   # CI/CD: validates PRs, deploys on merge to main
├── .vscode/
│   └── settings.json       # Associates *.hujson with jsonc (JSON with comments)
├── policy.hujson            # The ACL policy file — the only file you should edit
├── README.md               # Human-readable setup guide
└── CLAUDE.md               # This file
```

---

## The Policy File: `policy.hujson`

### Format

The policy file uses **HuJSON** — a superset of JSON that allows:
- `//` line comments and `/* */` block comments
- Trailing commas in arrays and objects

Standard JSON parsers will reject HuJSON. Use a HuJSON-aware tool or the Tailscale action for validation.

### Current Policy Structure

```hujson
{
  "tagOwners": { ... },   // Who can assign which tags
  "acls": [ ... ],        // Traffic allow/deny rules
  "ssh": [ ... ],         // SSH access rules
}
```

### Current Policy Summary

| Section | Rule | Effect |
|---|---|---|
| `tagOwners` | `tag:k8s-operator: []` | The operator tag has no owners (self-managed) |
| `tagOwners` | `tag:k8s: ["tag:k8s-operator"]` | Only the operator can assign the `k8s` tag to proxies |
| `acls` | `src: ["*"]` → `dst: ["tag:k8s:*"]` | All users can reach Kubernetes-exposed services |
| `ssh` | `src: ["autogroup:member"]` → `dst: ["autogroup:self"]` | Members can SSH to their own devices as root or non-root |

---

## Development Workflow

### Making Policy Changes

1. Create a feature branch off `main`
2. Edit `policy.hujson`
3. Open a pull request targeting `main`
4. CI automatically runs `tailscale/gitops-acl-action@v1` with `action: test` to validate syntax and semantics
5. Once the PR is approved and merged, CI runs `action: apply` to deploy the new policy

### CI/CD Pipeline (`.github/workflows/tailscale.yml`)

| Trigger | Action | Effect |
|---|---|---|
| Pull Request to `main` | `action: test` | Validates ACL without deploying |
| Push to `main` (merge) | `action: apply` | Deploys policy to the Tailscale network |

**Required GitHub Secrets:**

| Secret | Description |
|---|---|
| `TS_OAUTH_CLIENT_ID` | OIDC Client ID from Tailscale trust credentials |
| `TS_AUDIENCE` | OIDC Audience from Tailscale trust credentials |
| `TS_TAILNET` | Tailnet name from Tailscale admin settings |

Authentication uses **OpenID Connect (OIDC)** with GitHub as the issuer — no long-lived tokens are stored.

---

## Conventions

### Commit Messages

This repository uses emoji-prefixed commit messages:

| Emoji | Meaning |
|---|---|
| `🎉` | Initial setup or major additions |
| `📝` | Documentation changes |
| `👷` | CI/CD or build system changes |
| `💚` | Cleanup or fixes |
| `🧑‍💻` | Developer experience improvements |

### HuJSON Editing Guidelines

- Keep all comments that explain the intent of each rule
- Trailing commas are allowed and encouraged for clean diffs
- Group related rules with `// comment` headers
- The `tagOwners`, `acls`, and `ssh` top-level keys are the standard Tailscale policy sections

### What Not to Do

- Do not add files beyond what is listed in the structure above unless there is a clear reason
- Do not introduce package managers, build tools, or dependencies — this is a pure configuration repository
- Do not apply policies directly in the Tailscale admin console (they will be overwritten on next deploy)
- Do not merge to `main` without a passing CI check on the PR

---

## Key Concepts

### Tailscale Tags

Tags classify devices and are used in ACL rules instead of user identities. This repository defines two tags:

- **`tag:k8s-operator`** — Assigned to the Tailscale Kubernetes operator pod itself
- **`tag:k8s`** — Assigned to services that the operator exposes into the tailnet; only the operator can assign this tag

### ACL Rule Syntax

```hujson
{
  "action": "accept",   // or "deny"
  "src": ["*"],         // Source: users, groups, tags, or autogroups
  "dst": ["tag:k8s:*"], // Destination: host:port (* = any port)
}
```

### SSH Rule Syntax

```hujson
{
  "action": "check",                       // "check" = require Tailscale SSH auth
  "src": ["autogroup:member"],             // Who can initiate SSH
  "dst": ["autogroup:self"],               // Which devices they can target
  "users": ["autogroup:nonroot", "root"],  // Which OS users are allowed
}
```

### Autogroups

| Autogroup | Meaning |
|---|---|
| `autogroup:member` | All authenticated tailnet members |
| `autogroup:self` | The device the user owns themselves |
| `autogroup:nonroot` | Any non-root OS user |
| `*` | Everyone (including tagged devices) |

---

## Local Development

No local build toolchain is required. To edit policies:

1. Install VS Code — the `.vscode/settings.json` configures HuJSON syntax highlighting automatically
2. Edit `policy.hujson` directly
3. Push and open a PR; CI will validate the syntax

There is no local validation command. Validation only runs via CI through the `tailscale/gitops-acl-action@v1` GitHub Action.

---
> Source: [alexanderbailey/tailscale_policies](https://github.com/alexanderbailey/tailscale_policies) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
