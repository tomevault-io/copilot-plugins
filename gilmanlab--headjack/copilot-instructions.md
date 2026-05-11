## headjack

> Headjack is a CLI tool for spawning isolated CLI-based LLM coding agents in predefined container environments. Each agent runs in its own VM-isolated container with a dedicated git worktree, enabling safe parallel development across multiple branches.

# Headjack

Headjack is a CLI tool for spawning isolated CLI-based LLM coding agents in predefined container environments. Each agent runs in its own VM-isolated container with a dedicated git worktree, enabling safe parallel development across multiple branches.

## Repository Structure

```
├── main.go                 # Entry point
├── internal/               # Application code
│   ├── cmd/                # CLI commands (cobra)
│   ├── catalog/            # Instance catalog/persistence
│   ├── config/             # Configuration management
│   ├── container/          # Container runtime abstraction
│   ├── exec/               # Command execution utilities
│   ├── git/                # Git operations
│   ├── instance/           # Instance lifecycle management
│   ├── logging/            # Log file handling
│   ├── multiplexer/        # Terminal multiplexer (zellij)
│   ├── names/              # Random name generation
│   └── version/            # Version information
├── images/                 # Container images
│   └── base/               # Base container image Dockerfile
├── docs/                   # Documentation
│   ├── designs/            # Design documents
│   └── adrs/               # Architecture Decision Records
├── ref/                    # Reference documentation
│   └── go/                 # Go style guides
├── .github/workflows/      # CI/CD workflows
├── .golangci.yml           # Linter configuration
└── justfile                # Development task runner
```

## Development Commands

Use `just` to run common development tasks:

```bash
just check    # Run all checks (format, lint, test)
just fmt      # Format code
just lint     # Run linter
just test     # Run tests
```

## Code Quality Requirements

### Go Code

All changes to Go code MUST:

1. **Pass all checks**: Run `just check` before considering any change complete
2. **Follow style guidelines**: Conform to `ref/go/style.md`
3. **Follow testing guidelines**: Conform to `ref/go/testing.md`

### Dockerfiles

All changes to Dockerfiles MUST pass hadolint:

```bash
hadolint images/base/Dockerfile
```

### Pull Requests

All PRs MUST pass CI before being considered done. Monitor PR status with:

```bash
gh pr checks <pr-number> --watch
```

Wait for all checks to pass. If checks fail, fix the issues and push again.

### Commit Messages

All commits MUST follow the Conventional Commits standard with scoped subjects and structured bodies:

```
<type>(<scope>): <subject>

Current behavior:
<what was happening before, or "N/A - new feature" for new additions>

New behavior:
<what happens now>

Breaking Changes:
<list breaking changes, or omit section if none>

Closes: #<issue number, or omit if none>
```

**Types:** `feat`, `fix`, `refactor`, `chore`, `docs`, `ci`, `test`, `style`, `perf`

**Scopes:** Infer from the primary module changed: `cli`, `auth`, `container`, `git`, `catalog`, `config`, `instance`, `logging`, `multiplexer`, `names`, `version`, `images`, `ci`

**Rules:**
- Subject: imperative mood, no period, max 50 chars
- Body sections should be concise but informative
- Do NOT explain "how" in the body - that's what the diff is for
- Omit sections that don't apply (e.g., no Breaking Changes section if none)

**Example:**

```
feat(auth): add JWT refresh token endpoint

Current behavior:
Tokens expire after 15 minutes and users are forced to log in again,
causing frustration during long sessions.

New behavior:
Implemented a silent refresh flow using a secure HTTP-only cookie.
Users now stay logged in as long as they are active.

Breaking Changes:
The `POST /login` response schema has changed. It now returns
`{ token, refreshToken }` instead of just `{ token }`.

Closes: #892
```

---
> Source: [GilmanLab/headjack](https://github.com/GilmanLab/headjack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
