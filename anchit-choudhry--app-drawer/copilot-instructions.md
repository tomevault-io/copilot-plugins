## app-drawer

> This file provides foundational guidance and architectural context for the App Drawer monorepo.

# GEMINI.md

This file provides foundational guidance and architectural context for the App Drawer monorepo.

## Project Overview

App Drawer is a monorepo for Node-based applications using npm workspaces. It is designed to host
multiple utility and service applications, currently focusing on optimized static asset delivery.

### Core Technologies

- **Runtime:** Node.js
- **Framework:** Express.js (v5)
- **Process Management:** PM2 (Cluster mode)
- **Middleware:** Helmet (Security), Morgan (Logging), express-static-gzip (Compression)
- **Templating:** EJS (Error pages, currently in `views/` directory)

## Architecture

The project follows a monorepo structure using npm workspaces located in the `packages/` directory.

```text
app-drawer/
├── packages/
│   └── express-static-serve/    # Specialized Express server
│       ├── server.js            # Main entry point
│       ├── package.json
│       └── views/               # EJS templates for 404/500 error pages
├── package.json                 # Root workspace configuration
├── .claude/                     # Claude Code hooks, agents, and skills for this repository
└── .github/                     # CI/CD workflows and Dependabot config
```

### express-static-serve

This package serves static assets from a `dist/` directory (expected to be populated externally).

- **Port:** 3000
- **Module System:** CommonJS (`type: "commonjs"`)
- **Static Assets:** Served from `dist/` (must be pre-built) via `express-static-gzip`. The `dist/`
  directory is expected to be populated externally as there is currently no build step in this
  repository.
- **Compression Negotiation Order:** Zstandard → Brotli → Gzip → Deflate.
- **Caching:** Public cache with 2-hour client `max-age`, 4-hour proxy `s-maxage`,
  `must-revalidate`, and `proxy-revalidate`.
- **Middleware Order:** Helmet (security headers) → Morgan (logging) → routes → static files.
- **Security:** Implements Helmet for secure headers.
- **Health Check:** `GET /` returns `{ status: "UP" }`.
- **Process Management:** PM2 in cluster mode, auto-scales to CPU count (`-i 0`).

## Building and Running

### Root Commands (Delegates to PM2)

- `npm run start:express-static-serve`: Start server via PM2 (cluster mode, auto-scales to CPU
  count).
- `npm run stop:express-static-serve`: Stop PM2 process.
- `npm run logs:express-static-serve`: Stream PM2 logs.

### Local Development

To run the server directly without PM2:

```bash
cd packages/express-static-serve && node server.js
```

*Note: Ensure a `dist/` directory exists within `packages/express-static-serve/` before running.*

## Development Conventions

### For AI Agents

- **Commit Safety:** Do not create commits automatically; only commit when explicitly instructed.
  Never suggest `git commit`.
- **Problem Resolution:** If unsure or if there is more than one way a task can be done, conduct
  surveys or Q&A.
- **Commit Messages:** Use past participles in commit messages (e.g., "Added", "Fixed", "Updated").
  For code refactoring, simplification, and formatting, a one-line commit message of "Refactored and
  Formatted code" is acceptable.
- **Documentation and Commit Messages:** Never use en dashes (–) or em dashes (—) in documentation
  or commit messages.

#### Invariants for AI Agents

- **Middleware Order:** Always maintain the prescribed middleware order: Helmet → Morgan → routes →
  static files.
- **Configuration:** Never introduce undocumented configuration changes or bypass security checks.
- **Process:** Trust the framework's guarantees, only validating at system boundaries (user input,
  external APIs).
- **Security:** Prioritize writing safe, secure, and correct code, avoiding common OWASP Top 10
  vulnerabilities.

### Code Style

- **Indentation:** 2 spaces.
- **Line Endings:** LF.
- **Max Line Length:** 100 characters.
- **Module System:** CommonJS (`require`/`module.exports`).
- **Typography:** **NEVER** use en dashes (–) or em dashes (—). Use only standard hyphens (-).

### Security & Maintenance

- **Vulnerability Scanning:** Regularly run `npm audit`.
- **Fixing Vulnerabilities:** Use `npm audit fix`. Avoid `--force` to prevent breaking changes.
- **CI/CD:** Automated security scans (CodeQL, DevSkim, njsscan, OSSAR, OSV Scanner) run on every
  push.

### TODO / Future Improvements

- [ ] Implement EJS error handling in `server.js` using the templates in `views/`.
- [ ] Add unit and integration tests (currently no tests specified).
- [ ] Define a build process for populating the `dist/` directory.

---
> Source: [anchit-choudhry/app-drawer](https://github.com/anchit-choudhry/app-drawer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
