## ballerina-integrator-workspace

> This workspace contains TWO independent git repositories cloned as siblings:

# Ballerina Integrator (BI) — Cursor Rules

## Workspace structure

This workspace contains TWO independent git repositories cloned as siblings:

- `vscode-extensions/` — TypeScript monorepo, sparse checkout (BI paths only)
- `ballerina-language-server/` — Java/Maven, full clone

Both repos are .gitignored from this workspace root. Commits and PRs go to their
own upstream repos, not here.

## Critical: sparse checkout scope

`vscode-extensions/` uses git sparse checkout. ONLY these paths exist on disk:

  common/                        → Rush config and autoinstallers (do not edit)
  workspaces/ballerina/          → core BI extension (TypeScript)
  workspaces/bi/bi-extension/    → BI extension entry point (TypeScript)
  workspaces/wso2-platform/      → shared WSO2 platform utilities (TypeScript)
  workspaces/common-libs/        → shared common libraries (TypeScript)

Never suggest, reference, or attempt to access any other path inside vscode-extensions/.
Directories like workspaces/choreo/, workspaces/mi/, etc. do not exist on disk.
Only edit within the four workspaces/ paths above — do not modify common/.

## Technology

- vscode-extensions workspaces: TypeScript, Rush monorepo build tool, webpack/esbuild
- ballerina-language-server: Java 17+, Maven 3.8+, LSP over stdio
- The extension and language server communicate via Language Server Protocol

## Build

The VS Code extensions use **Rush** — always run Rush commands from the `vscode-extensions/` root.

```bash
# Install all dependencies (once or after package.json changes)
cd vscode-extensions && rush install

# Build all BI extension projects (incremental)
cd vscode-extensions && rush build

# Clean rebuild
cd vscode-extensions && rush rebuild

# Build a single project with rushx
cd vscode-extensions/workspaces/ballerina/ballerina-extension && rushx build

# Language server
cd ballerina-language-server && mvn clean install -DskipTests
```

## Debug mode

Five launch configurations are defined in `ballerina-integrator.code-workspace`. Use VS Code's
Run and Debug panel (Ctrl+Shift+D) to select and launch them. Each runs `rushx watch` as a
preLaunchTask and opens an Extension Development Host window.

| Config name | What loads |
|-------------|-----------|
| Ballerina & BI Extensions | ballerina-extension + bi-extension |
| Ballerina, BI & Platform Extensions | all three + wso2-platform-extension |
| BI Extension | bi-extension only (fastest) |
| Ballerina Extension | ballerina-extension only |
| Ballerina Extension Tests | ballerina-extension + test suite |

Before the first session, copy `.env.example` → `.env` in each extension directory.
The configs set `WEB_VIEW_WATCH_MODE=true` and `WEB_VIEW_DEV_HOST=http://localhost:9000`
for hot-reload of webview UI — start the dev server with `rushx watch:webview` if needed.

## Git rules

1. Each repo has its own git history. Run git commands with the correct working directory.
2. Use ./scripts/bi-branch.sh for branch operations — it keeps both repos in sync.
3. Use ./scripts/sync.sh to pull latest from both repos.
4. Branch names must match across both repos for a given feature.
5. Never commit to the workspace root repo unless changing workspace-level files
   (scripts, CLAUDE.md, AGENTS.md, .cursorrules, .code-workspace).

## PR rules

- Features touching both repos → two PRs, one per repo, linked to each other.
- Extension-only change → one PR to wso2/vscode-extensions.
- Language server only → one PR to ballerina-platform/ballerina-language-server.
- Merge language server PR before extension PR when a new LSP capability is added.
- Commit format: Conventional Commits — feat(bi):, fix(bi):, chore(bi):

## Preferred helper commands

./scripts/sync.sh                              # pull latest from both repos
./scripts/bi-branch.sh create feature/<name>  # branch in both repos
./scripts/bi-branch.sh status                 # current branch per repo
./scripts/bi-branch.sh switch <branch>        # switch both repos

## Code style

- TypeScript: ESLint + Prettier (configs in each workspace directory)
- Java: existing Checkstyle config in ballerina-language-server/
- No trailing whitespace, files end with a newline

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/madushajg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
