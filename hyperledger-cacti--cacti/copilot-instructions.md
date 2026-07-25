## cacti

> Machine-readable guidance for AI coding agents.

# AGENTS.md — Hyperledger Cacti

Machine-readable guidance for AI coding agents.

**Hyperledger Cacti** is a blockchain interoperability framework (Apache-2.0).
TypeScript monorepo — Yarn 4 + Lerna, Node.js v20.20.0, packages in
`packages/`, `examples/`, `extensions/`, `weaver/`.

## Restrictions

| Action | Rule |
|--------|------|
| `git commit` / `git push` | Forbidden — report changes, let the human commit |
| `git rebase` / `git merge` / `git reset --hard` | Forbidden, unless rebase to get latest code from upstream |
| `git tag` / `git clean -f` | Forbidden |
| `Signed-off-by` / `Assisted-by` tags | Forbidden — human submitter only |
| Create / merge PRs or issues | Requires explicit human approval |
| `--no-verify` / `HUSKY=0` | Forbidden |
| Delete files or branches | Requires explicit human confirmation |
| Edit files under `generated/` | Forbidden — run `yarn codegen` instead |

Safe read-only: `git status`, `git diff`, `git log`, `git show`,
`git blame`, `git fetch`, `git branch` (list).

## Build and Test

```bash
nvm use 20.20.0          # required Node version
yarn configure           # install deps + build
yarn tsc                 # TypeScript compilation
yarn lint                # ESLint + Prettier + spellcheck
yarn codegen             # regenerate OpenAPI clients (never hand-edit generated/)
```

## Before Proposing Changes

1. `yarn tsc` passes with no errors.
2. `yarn lint` passes and produces no unstaged file changes.
3. Every behaviour change has a test that fails without the change.
4. No edits outside the scope stated in the task.
5. No commits, pushes, or DCO tags added — report the diff to the human.

## Do not modify without explicit approval 
`generated/` · `weaver/` · `whitepaper/` · `GOVERNANCE.md` · `SECURITY.md` ·
`CODEOWNERS` · `.github/workflows/` · root `tsconfig.json` references ·
root `package.json` workspaces · `lerna.json` · `tsconfig.base.json`

## Agent Pipelines

| Goal | Pipeline |
|------|----------|
| New feature | `feature-implementer → tdd-implementer → review → security-review → code-reviewer` |
| Bug fix | `debugger → ci-debugger → code-reviewer` |
| Code review | `review → security-review → code-reviewer` |
| CI triage | `ci-debugger → code-reviewer` |
| Security audit | `security-review → debugger → code-reviewer` |

Agents: [`.github/agents/`](.github/agents/) ·
Skills: [`.github/skills/`](.github/skills/) (ci-triage, codeql-scanning, doublecheck, package-removal, refactor-plan)

## Authoritative References

| Document | Covers |
|----------|--------|
| [CONVENTIONS.md](./CONVENTIONS.md) | Naming, structure, TypeScript, testing, OpenAPI |
| [CONTRIBUTING.md](./CONTRIBUTING.md) | Contribution workflow, DCO |
| [AI_GUIDELINES.md](./AI_GUIDELINES.md) | AI-assisted contribution policies |
| [PULL.md](./PULL.md) | PR scope, size, commit format |
| [BUILD.md](./BUILD.md) | Dev environment setup |
| [SECURITY.md](./SECURITY.md) | Security policy and reporting |
| [.github/copilot-instructions.md](.github/copilot-instructions.md) | Copilot workspace rules + code review checklist |
| [.github/instructions/](.github/instructions/) | Path-scoped rules (TS, tests, plugins, connectors, SATP, OpenAPI, docs, workflows) |

---
> Source: [hyperledger-cacti/cacti](https://github.com/hyperledger-cacti/cacti) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
