## my-vibe-scaffolding

> **CRITICAL: Read system information from `config.toml` before executing commands.**


## System Environment

**CRITICAL: Read system information from `config.toml` before executing commands.**

The `[system]` section contains auto-detected operating system information:

```toml
[system]
os_type = "macOS"  # or "Linux", "Windows"
os_version = "26.3"
shell = "/bin/zsh"

[system.commands]
timeout_command = "none"  # macOS doesn't have timeout by default
sed_inplace = "sed -i ''"  # macOS requires empty string argument
has_brew = true
has_apt = false
# ... other command availability flags
```

### Command Selection Examples

**Before using commands with OS-specific differences:**

1. **timeout command** (Linux has it, macOS doesn't):
   ```bash
   # ❌ WRONG: Assume timeout exists
   timeout 5 command
   
   # ✅ CORRECT: Check config.toml first
   # If timeout_command = "none" → use alternative (sleep + kill)
   # If timeout_command = "gtimeout" → use gtimeout
   # If timeout_command = "timeout" → use timeout
   ```

2. **sed in-place editing** (macOS vs Linux syntax difference):
   ```bash
   # ❌ WRONG: Use Linux syntax on macOS
   sed -i 's/foo/bar/' file.txt  # Fails on macOS
   
   # ✅ CORRECT: Use config.toml value
   # Read sed_inplace from config.toml:
   #   macOS: sed -i ''
   #   Linux: sed -i
   ```

3. **Package managers**:
   ```bash
   # Check has_brew, has_apt, has_yum, has_choco
   # Install based on available package manager
   ```

### Updating System Information

System information is automatically detected and updated by:

```bash
./.scaffolding/scripts/detect-os.sh
```

This script is run during project initialization. Re-run it if:
- Operating system changes (e.g., WSL → native Linux)
- New development tools are installed
- AI agent encounters "command not found" errors


# AGENTS.md

This document serves as the primary instruction set for AI agents (like OpenCode) working on this project.

## Project Overview

<!-- TODO: Fill in project description, goals, and context -->

### 📋 Product Requirements Document (PRD)

**For projects with detailed requirements, maintain a PRD file:**

- **Location**: `docs/PRD.md` (recommended) or `docs/specs/PRD.md`
- **Purpose**: Define features, technical requirements, user flows for AI-assisted development
- **Template**: See [.scaffolding/docs/templates/PRD_TEMPLATE.md](./.scaffolding/docs/templates/PRD_TEMPLATE.md)
- **Guide**: See [.scaffolding/docs/PRD_GUIDE.md](./.scaffolding/docs/PRD_GUIDE.md)

**Example PRD reference**:
## Project Overview

YourProject is a [brief project description].

**📋 Product Requirements**: See [docs/PRD.md](./docs/PRD.md) for complete specification.

**Key Features** (from PRD):
- [Feature 1]
- [Feature 2]
- [Feature 3]
- [Feature 4]

**Current Phase**: Phase 1 - MVP (see PRD Section 6)

**Benefits of PRD for AI coding**:
- ✅ AI has complete context at session start
- ✅ Consistent implementation across features
- ✅ No lost context between sessions
- ✅ Explicit edge cases and error handling


<!-- TODO: Fill in project description, goals, and context -->

## Project Setup for AI Agents

**IMPORTANT: This section is for AI agents (OpenCode, Cursor, Claude). Human users see simplified instructions in README.**

### Installation & Update (Project Mode)

**Single command handles both first-time setup and updates:**

```bash
./.scaffolding/scripts/init-project.sh
```

**The script auto-detects mode:**

- **First-time mode** (no `.template-version` file exists):
  - Creates project-specific files (VERSION, README, etc.)
  - Sets up Git hooks
  - Initializes OpenCode configuration
  - Creates `.template-version` for tracking

- **Update mode** (`.template-version` file exists):
  - Consolidates agent configs (`.claude`, `.roo` → `.agents`)
  - Updates template version tracking
  - Reinstalls Git hooks (may have new features)
  - Preserves all user customizations

**When to run this command:**

- User mentions "setup", "initialize", "configure" the project
- User mentions "update template", "upgrade", "sync template"
- Missing required files detected (VERSION, git hooks, etc.)
- User wants latest template features
- Need to consolidate scattered agent configurations

### Template Development (Scaffolding Mode)

When developing the template itself, modifications go to `.scaffolding/` directory:

```bash
# Template mode doesn't need init script
# You're directly modifying template files
```

**Key differences:**
- Edit `.scaffolding/docs/`, `.scaffolding/scripts/`, etc.
- Changes apply to the template, not a specific project
- Commit to template repository

---

## Working Mode

**This scaffolding has two working modes configured in `config.toml`:**

### Scaffolding Mode (`mode = "scaffolding"`)

You're developing this scaffolding itself. File organization:

- **Scaffolding ADRs**: `.scaffolding/docs/adr/`
- **Scaffolding scripts**: `.scaffolding/scripts/`
- **Scaffolding assets**: `.scaffolding/assets/`
- **Root directories**: Keep `docs/`, `scripts/`, `assets/` empty or minimal

**AI Agent behavior:**
- Create new ADRs in `.scaffolding/docs/adr/`
- Reference scaffolding scripts from `.scaffolding/scripts/`
- Reference scaffolding assets from `.scaffolding/assets/`
- **Generate bilingual README** following [README_BILINGUAL_FORMAT.md](./.scaffolding/docs/README_BILINGUAL_FORMAT.md)
- **CHANGELOG**: Update `.scaffolding/CHANGELOG.md` (template changes)
- **README sync**: If `sync_readme = true`, **all** `README*.md` files (all locales) auto-sync: root → `.scaffolding/`

### Project Mode (`mode = "project"`)

You're using this scaffolding for your project. File organization:

- **Project ADRs**: `docs/adr/`
- **Project scripts**: `scripts/`
- **Project assets**: `assets/`
- **.scaffolding/ directory**: Contains reference examples only

**AI Agent behavior:**
- Create new ADRs in `docs/adr/`
- Place project-specific scripts in `scripts/`
- Place project-specific assets in `assets/`
- Reference `.scaffolding/` examples but don't modify them
- **CHANGELOG**: Update root `CHANGELOG.md` (your project changes)
- **README**: Edit root `README.md` for your project (independent from template)

**To change mode:** Edit `config.toml` and set `[project] mode = "scaffolding"` or `"project"`

## OpenCode 配置 (2026-03-03 更新)

**推薦配置：專案獨立資料庫**

### 問題：多專案共用資料庫導致衝突

預設情況下，所有 VSCode OpenCode 實例共用 `~/.local/share/opencode/opencode.db`，導致：
- 🔥 多專案同時開啟 → 崩潰、資料損壞
- 💔 Session 歷史丟失
- 🐌 資料庫膨脹 (> 50MB)

### 解決方案：每個專案使用獨立資料庫

**1. 自動化設定（推薦）**

```bash
# 在專案根目錄執行
./.scaffolding/scripts/init-opencode.sh
```

**2. 手動設定**

建立 `.vscode/settings.json`：
```json
{
  "opencode.dataDir": "${workspaceFolder}/.opencode-data",
  "opencode.logLevel": "info"
}
```

更新 `.gitignore`：
```bash
echo ".opencode-data/" >> .gitignore
```

**3. 重啟 VSCode**（必須）

### 效果

- ✅ 可安全同時開啟多個專案
- ✅ Session 與專案綁定
- ✅ 資料庫大小可控 (< 10MB/專案)
- ✅ 崩潰率接近零

### 詳細文件

- [ADR 0005 - 技術調查](./.scaffolding/docs/adr/0005-single-instance-opencode-workflow.md)
- [設定指南](./.scaffolding/docs/OPENCODE_SETUP_GUIDE.md)
- [批次部署腳本](./.scaffolding/scripts/init-opencode.sh)

---


---

## Specialized Agents

**版本**: 1.0.0  
**來源**: [everything-claude-code](https://github.com/affaan-m/everything-claude-code) (Anthropic Hackathon Winner)  
**文件**: [`.scaffolding/agents/agents/README.md`](./.scaffolding/agents/agents/README.md)

### 為什麼需要專業化 Agents？

單一 agent 承擔所有任務會導致品質不穩定、上下文過載、決策疲勞。專業化 agents 透過分工提供一致的高品質輸出。

### Available Agents

| Agent | 用途 | 何時使用 | 調用方式 |
|-------|------|---------|---------|
| **planner** | 任務規劃分解 | 複雜功能、重構、多檔案變更 | `task(subagent_type="planner", ...)` |
| **architect** | 架構設計決策 | 技術選型、系統設計、模式選擇 | `task(subagent_type="architect", ...)` |
| **tdd-guide** | TDD 工作流程 | 新功能開發、測試驅動開發 | `task(subagent_type="tdd-guide", ...)` |
| **code-reviewer** | 程式碼審查 | PR 前審查、品質檢查 | `task(subagent_type="code-reviewer", ...)` |
| **security-reviewer** | 安全審查 | 敏感功能、API、認證授權 | `task(subagent_type="security-reviewer", ...)` |

### 使用範例

#### 規劃複雜功能

```typescript
User: "新增使用者認證系統"

Main Agent: "這是複雜任務，讓我調用 planner 進行規劃"

task(
  subagent_type="planner",
  load_skills=[],
  prompt=`規劃使用者認證系統實作
  
  要求：
  - JWT token 認證
  - 密碼雜湊（bcrypt）
  - Session 管理
  - 登入/登出/註冊 API
  `, 
  run_in_background=false
)

// Planner 會返回詳細的分階段計畫、檔案路徑、依賴關係圖
```

#### 架構設計決策

```typescript
User: "REST 還是 GraphQL？"

task(
  subagent_type="architect",
  load_skills=[],
  prompt=`評估 REST vs GraphQL
  
  專案：電商平台 MVP
  團隊：3 名全端工程師
  `, 
  run_in_background=false
)

// Architect 會提供多維度技術評估、權衡分析、明確推薦
```

#### TDD 工作流程

```typescript
User: "實作購物車結帳功能"

task(
  subagent_type="tdd-guide",
  load_skills=["test-driven-development"],
  prompt="指導購物車結帳的 TDD 開發流程",
  run_in_background=false
)

// TDD Guide 會提供測試案例列表、Red-Green-Refactor 步驟
```

#### 程式碼審查

```typescript
User: "審查登入 API"

task(
  subagent_type="code-reviewer",
  load_skills=["requesting-code-review"],
  prompt=`審查 src/api/auth/login.ts
  
  重點：錯誤處理、輸入驗證、安全性、程式碼品質
  `,
  run_in_background=false
)

// Code Reviewer 會返回問題清單、改進建議、程式碼範例
```

#### 安全審查

```typescript
User: "檢查密碼重設功能的安全性"

task(
  subagent_type="security-reviewer",
  load_skills=[],
  prompt=`審查密碼重設功能
  
  檢查：Token 安全性、時序攻擊、Rate limiting、OWASP Top 10
  `,
  run_in_background=false
)

// Security Reviewer 會返回安全漏洞列表、攻擊場景、修復建議
```

### Agent 協作模式

**Sequential（順序執行）** - 複雜功能的完整流程：

```
planner (規劃) → architect (設計) → tdd-guide (開發) → 
code-reviewer (審查) → security-reviewer (安全)
```

**Conditional（條件觸發）** - 根據複雜度決定：

```
簡單變更：直接實作
複雜變更：planner → 實作
安全敏感：security-reviewer → 修正
```

**Parallel（平行執行）** - 獨立任務：

```typescript
task(subagent_type="code-reviewer", ..., run_in_background=true)
task(subagent_type="security-reviewer", ..., run_in_background=true)
// 平行執行兩個審查
```

### 決策樹

```
收到任務
├─ 新功能/重構？
│  ├─ 複雜 → planner
│  └─ 簡單 → 直接實作
├─ 架構決策？ → architect
├─ 開發功能？
│  ├─ 用 TDD → tdd-guide
│  └─ 否 → 直接實作
├─ 審查程式碼？ → code-reviewer
└─ 安全相關？ → security-reviewer
```

### 最佳實踐

**DO ✅**
- 提供明確的任務描述和專案上下文
- 循序漸進：規劃 → 設計 → 實作 → 審查
- 載入相關 skills 加強 agent 能力
- 安全功能必經 security-reviewer

**DON'T ❌**
- 簡單任務過度委派（改變數名不需要 agent）
- 缺乏上下文的提示（"選個資料庫"）
- 跳過必要審查步驟

**完整指南**: [`.scaffolding/agents/agents/README.md`](./.scaffolding/agents/agents/README.md)

---

## Core Skills System

**Version**: 1.0.0  
**Source**: [everything-claude-code](https://github.com/affaan-m/everything-claude-code) (Anthropic Hackathon Winner)  
**Documentation**: [`.scaffolding/agents/skills/README.md`](./.scaffolding/agents/skills/README.md)

### Why Core Skills?

Skills provide specialized implementation patterns and best practices for specific domains. Unlike agents (which delegate tasks), skills guide **how** to execute tasks correctly.

### Available Skills (15 total)

Skills are organized by domain expertise:

| Category | Count | Skills |
|----------|-------|--------|
| **Universal** | 5 | api-design, security-review, tdd-workflow, coding-standards, verification-loop |
| **Backend** | 3 | backend-patterns, database-optimization, error-handling |
| **Frontend** | 3 | frontend-patterns, react-hooks, component-design |
| **Testing** | 2 | e2e-testing, unit-testing |
| **Other** | 2 | content-engine, market-research |

### Quick Reference

#### Universal Skills (All Projects)

- **api-design**: REST API design patterns, status codes, pagination, versioning
- **security-review**: Authentication, input validation, SQL injection prevention, XSS/CSRF
- **tdd-workflow**: Test-driven development, 80%+ coverage, Red-Green-Refactor
- **coding-standards**: Code style, naming conventions, TypeScript patterns
- **verification-loop**: Systematic pre-completion checks

#### Backend Skills

- **backend-patterns**: Node.js async patterns, middleware, microservices
- **database-optimization**: Query optimization, N+1 prevention, indexing, caching
- **error-handling**: Error middleware, logging, circuit breakers, graceful degradation

#### Frontend Skills

- **frontend-patterns**: React/Next.js architecture, state management, performance
- **react-hooks**: useState, useEffect, useMemo, useCallback, custom hooks
- **component-design**: Composition patterns, render props, compound components

#### Testing Skills

- **e2e-testing**: Playwright patterns, page objects, CI/CD integration
- **unit-testing**: Jest/Vitest patterns, mocking, AAA structure

### Usage Patterns

**Automatic Loading** (Recommended):
Skills auto-load when AGENTS.md detects relevant keywords:

```typescript
User: "Design a REST API for user management"
// Auto-loads: api-design, security-review, backend-patterns
```

**Manual Loading**:

```typescript
// Load single skill
@use api-design
User: "Design API endpoints"

// Load multiple skills
@use tdd-workflow
@use e2e-testing
User: "Implement login with tests"
```

### Skill Combinations (Best Practices)

**Backend API Development**:
```typescript
@use api-design
@use security-review
@use backend-patterns
@use tdd-workflow
User: "Build user authentication API"
```

**Frontend Component Development**:
```typescript
@use frontend-patterns
@use react-hooks
@use component-design
@use unit-testing
User: "Create reusable modal component"
```

**Full-stack Feature**:
```typescript
@use api-design
@use frontend-patterns
@use tdd-workflow
@use e2e-testing
User: "Implement checkout flow"
```

### Integration with Superpowers

These skills complement your existing superpowers skills:

| ECC Skill | Superpowers Equivalent | Recommendation |
|-----------|------------------------|----------------|
| `tdd-workflow` | `superpowers/test-driven-development` | Use both together |
| `verification-loop` | `superpowers/verification-before-completion` | Use both together |
| `security-review` | — | New capability |
| `api-design` | — | New capability |

**Strategy**: ECC skills provide implementation patterns, superpowers provide workflow guidance.

### When to Use Which Skill

| Task Type | Skills to Load |
|-----------|----------------|
| New API endpoint | `api-design`, `security-review`, `tdd-workflow` |
| Database query slow | `database-optimization`, `backend-patterns` |
| React component | `frontend-patterns`, `react-hooks`, `component-design` |
| Authentication | `security-review`, `api-design`, `backend-patterns` |
| Pre-deployment | `verification-loop`, `security-review`, `e2e-testing` |
| Bug fix | `tdd-workflow`, `unit-testing`, `systematic-debugging` |

### Skill Auto-Detection Keywords

AGENTS.md automatically loads skills based on these keywords:

- **API/Backend**: "API", "endpoint", "backend", "Node.js", "Express", "microservices"
- **Security**: "authentication", "security", "auth", "sensitive", "password"
- **Frontend**: "React", "Next.js", "component", "UI", "frontend", "hooks"
- **Database**: "database", "query", "slow", "N+1", "index", "cache"
- **Testing**: "test", "TDD", "E2E", "Playwright", "Jest", "Vitest"
- **Deployment**: "deploy", "production", "verify", "complete"

### Complete Documentation

**Full guides available**:
- [`.scaffolding/agents/skills/README.md`](./.scaffolding/agents/skills/README.md) - Complete skill documentation
- [`docs/PRD-claude-code-inspired-upgrades.md`](./docs/PRD-claude-code-inspired-upgrades.md) - Integration PRD
- [`docs/adr/0009-reference-claude-code-architecture.md`](./docs/adr/0009-reference-claude-code-architecture.md) - Architecture decisions

---

## Tech Stack

<!-- TODO: List technologies, frameworks, and tools used in this project -->

## MCP (Model Context Protocol) Usage

**Priority: Always check for MCP tools first, then fallback to CLI.**

### GitHub Operations Priority

When performing GitHub operations (issues, PRs, releases):

1. **First**: Check if GitHub MCP server tools are available
   - Tool names: `github_*` (e.g., `github_create_issue`, `github_create_pull_request`)
   - Advantages: Faster, structured responses, cross-tool compatible

2. **Fallback**: Use `gh` CLI if MCP not available
   - Via `bash` tool: `gh issue create`, `gh pr create`, etc.
   - Reliable but slower (subprocess overhead)

### How to Check MCP Availability

```typescript
// At session start, list available tools
// If you see tools starting with 'github_', 'git_', 'filesystem_' → MCP is active
// If not → Use CLI fallbacks (gh, git commands via bash)
```

### MCP Servers Configuration

- **filesystem**: File operations (read/write/search)
- **git**: Git operations (status, diff, commit, push)
- **memory**: Persistent memory across sessions
- **github**: GitHub API (issues, PRs, releases, workflows)

Configuration: `opencode.json`  
Setup guide: [.scaffolding/docs/MCP_SETUP_GUIDE.md](./.scaffolding/docs/MCP_SETUP_GUIDE.md)

## Commands

**Reference**: AGENTS.md 2026 Standard - Commands are one of 6 core blocks required for AI agent coordination.

### AI Development Commands

**Source**: [everything-claude-code](https://github.com/affaan-m/everything-claude-code)  
**Documentation**: [`.scaffolding/agents/commands/README.md`](./.scaffolding/agents/commands/README.md)

These commands provide task-specific workflows combining agents and skills:

| Command | Description | Agent | Use When |
|---------|-------------|-------|----------|
| **plan** | Create implementation plan with risk assessment | planner | Starting complex features |
| **code-review** | Review code for quality, security, maintainability | code-reviewer | Before commits, PR review |
| **build-fix** | Diagnose and fix build errors | architect | Build failures |
| **e2e** | Create end-to-end tests | tdd-guide | Testing workflows |
| **checkpoint** | Save state before major changes | — | Before refactoring |
| **test-all** | Run all tests with coverage | tdd-guide | Pre-commit, CI/CD |
| **security-scan** | Security vulnerability audit | security-reviewer | Pre-deployment |
| **analyze** | Codebase quality analysis | architect | Code health check |
| **refactor** | Systematic code refactoring | architect | Improving maintainability |
| **document** | Generate/update documentation | architect | API docs, README |

**Usage**: Commands are invoked by referencing the command file (e.g., "run plan command for user auth").

**Command Chains** (Common Workflows):
- **Feature Development**: plan → checkpoint → [implement] → test-all → code-review → security-scan → document
- **Bug Fix**: checkpoint → analyze → [fix] → test-all → code-review
- **Refactoring**: analyze → checkpoint → test-all → refactor → test-all → code-review
- **Pre-Deployment**: test-all → code-review → security-scan → document


### Development

- **Install**: (Project-specific - depends on tech stack)
  - Node.js: `npm install` or `pnpm install`
  - Python: `pip install -r requirements.txt`
  - Go: `go mod download`
  - Note: This scaffolding template itself has no development commands (it's a template, not an application)

- **Dev**: (Project-specific - depends on tech stack)
  - Node.js: `npm run dev`
  - Python: `python manage.py runserver` (Django) or `flask run`
  - Go: `go run main.go`

- **Build**: (Project-specific - depends on tech stack)
  - Node.js: `npm run build`
  - Python: (typically no build step)
  - Go: `go build`

- **Test**: (Project-specific - depends on tech stack)
  - Node.js: `npm test`
  - Python: `pytest` or `python -m unittest`
  - Go: `go test ./...`

### Template Management

These commands manage the scaffolding template itself:

- **Init project**: `./.scaffolding/scripts/init-project.sh`
  - First-time project setup or template updates
  - Auto-detects: first-time mode (no `.template-version`) vs update mode (existing `.template-version`)
  - Creates project files, sets up git hooks, initializes OpenCode config

- **Bump version**: `./.scaffolding/scripts/bump-version.sh [patch|minor|major]`
  - Update `.scaffolding/VERSION` and `VERSION` files
  - Create git commit and tag
  - Update `CHANGELOG.md` and `README.md` badges
  - Usage:
    - `./template/scripts/bump-version.sh patch` - Bug fixes (1.0.0 → 1.0.1)
    - `./.scaffolding/scripts/bump-version.sh minor` - New features (1.0.0 → 1.1.0)
    - `./.scaffolding/scripts/bump-version.sh major` - Breaking changes (1.0.0 → 2.0.0)

- **Generate README**: `./.scaffolding/scripts/generate-readme.sh`
  - Generate `README.md` and `README.{lang}.md` from `i18n/locales/{lang}/readme.toml`
  - Sync to `.scaffolding/README.md` and `.scaffolding/README.{lang}.md`
  - Add language switcher links automatically
  - **CRITICAL**: Always use this script to update README, never edit README.md directly

- **Sync README**: `./.scaffolding/scripts/sync-readme.sh`
  - Sync root README files to `.scaffolding/` directory (if `sync_readme = true` in scaffolding mode)
  - Used in scaffolding mode when developing the template itself

- **Sync template**: `./.scaffolding/scripts/sync-template.sh`
  - Sync template changes from `.scaffolding/` to project root
  - Version comparison (.template-version vs .scaffolding/VERSION)
  - Change summary display (git-style diff)
  - Selective file sync with exclude patterns (--exclude)
  - Conflict detection and resolution guidance
  - Usage:
    - `sync-template.sh` - Interactive sync
    - `sync-template.sh -n` - Dry run (preview)
    - `sync-template.sh -e "docs/*"` - Exclude specific files
    - `sync-template.sh -f` - Force sync regardless of version

### Git Hooks

- **Install hooks**: `./.scaffolding/scripts/install-hooks.sh`
  - Install pre-commit and pre-push git hooks
  - Pre-push hook: Enforce version bump before pushing to main
  - Location: `.git/hooks/`

### OpenCode Specific

These commands help manage OpenCode stability and workflow:

- **Health check**: `./.scaffolding/scripts/health-check.sh`
  - Check OpenCode database size, session count, startup time
  - Recommend cleanup if thresholds exceeded
  - Run manually or via cron job

- **Clean sessions**: `./.scaffolding/scripts/smart-cleanup.sh`
  - Smart session cleanup (keeps recent/active sessions)
  - Triggered by health check or run manually
  - Archives old sessions to `.opencode-data/archive/`

- **Monitor stability**: `./.scaffolding/scripts/monitor-stability.sh`
  - Monitor OpenCode crashes and performance
  - Generate daily stability reports
  - Track memory usage and session duration

- **Init OpenCode**: `./.scaffolding/scripts/init-opencode.sh`
  - Set up project-specific OpenCode database
  - Configure `.vscode/settings.json` for isolated data directory
  - Prevents multi-project database conflicts

- **Detect OS**: `./.scaffolding/scripts/detect-os.sh`
  - Auto-detect operating system and available commands
  - Update `config.toml` [system] section
  - Run during project initialization or when environment changes

### Utility Scripts

- **Verify setup**: `./.scaffolding/scripts/verify-setup.sh`
  - Verify project configuration integrity
  - Check required files exist
  - Validate configuration format

- **Check version sync**: `./.scaffolding/scripts/check-version-sync.sh`
  - Ensure `.scaffolding/VERSION` and `VERSION` are in sync
  - Used by pre-push git hook

### Usage Tips

**For template maintainers** (scaffolding mode):
- Use `bump-version.sh` before every commit to main
- Run `generate-readme.sh` after updating i18n translation files
- Use `sync-readme.sh` to keep `.scaffolding/` README in sync

**For template users** (project mode):
- Run `init-project.sh` once after creating project from template
- Use development commands specific to your tech stack
- Run `health-check.sh` if OpenCode becomes unstable

**For all users**:
- `install-hooks.sh` - Run once to set up automatic version enforcement
- `verify-setup.sh` - Run to check configuration integrity


## Service Detection Protocol

**CRITICAL: AI agents MUST check service availability BEFORE calling external services.**

**Reference**: [`.scaffolding/agents/service-detection.md`](./.scaffolding/agents/service-detection.md) | **ADR**: [0008](./
.scaffolding/docs/adr/0008-opencode-config-claude-code-reference.md)

### Quick Protocol

**Before calling ANY external service:**

1. ✅ **Check `config.toml`** `[services.unsupported]` list
2. ✅ **If service is unsupported** → Look up alternatives in `[services.capabilities]`
3. ✅ **Use alternative tool** and inform user of substitution
4. ✅ **Provide clear reasoning** for why alternative was chosen

### Example: Handling Unsupported google-search

**Bad (DON'T DO THIS)**:
```
User: "Search the web for React best practices"
Agent: [Attempts google-search]
Result: 403 Forbidden - Gemini for Google Cloud API has not been used...
Agent: "Sorry, I encountered an error."
```

**Good (DO THIS)**:
```
User: "Search the web for React best practices"
Agent: [Checks config.toml → google-search in unsupported list]
Agent: [Looks up alternatives → websearch_web_search_exa available]
Agent: "Using websearch_web_search_exa as alternative to google-search (provides LLM-optimized results)"
Agent: [Executes search successfully]
```

### Service Configuration Location

**File**: `config.toml` (copy from `config.toml.example`)

**Structure**:
```toml
[services]
unsupported = ["google-search", "google_search"]

[services.capabilities]
web_search = ["websearch_web_search_exa", "webfetch"]
code_search = ["grep_app_searchGitHub"]
documentation = ["context7_query-docs", "context7_resolve-library-id"]

[services.fallback]
mode = "suggest"  # "suggest" | "auto" | "fail"
log_attempts = true
show_reason = true
```

### Error Message Template

When service is unavailable:

```markdown
❌ Service '{service_name}' is not available in this configuration.

Reason: {specific_reason}

✅ Available alternatives:
  1. {alternative_1} - {description}
  2. {alternative_2} - {description}

Recommended: {best_alternative}
Using: {chosen_alternative}

[Continues execution with alternative]
```

### Service Capability Matrix

| Functionality | Unsupported Services | Available Alternatives | Recommended |
|---------------|----------------------|------------------------|-------------|
| **Web Search** | `google-search`, `google_search` | `websearch_web_search_exa`, `webfetch` | `websearch_web_search_exa` (LLM-optimized) |
| **Code Search** | — | `grep_app_searchGitHub` | `grep_app_searchGitHub` |
| **Documentation** | — | `context7_query-docs`, `context7_resolve-library-id` | `context7_query-docs` |
| **Web Fetch** | — | `webfetch` | `webfetch` (direct URL) |

**Full details**: See [`.scaffolding/agents/service-detection.md`](./.scaffolding/agents/service-detection.md)


## Module Loading Protocol

**Version**: 2.0.0  
**Purpose**: Conditional loading of documentation modules based on project type and task context

### Overview

This scaffolding uses a **config-driven module system**. Instead of loading all documentation at once:

1. AI agent reads `config.toml` to understand project type
2. AI agent detects task keywords to determine needed modules
3. AI agent loads ONLY relevant modules for the current task
4. Token usage reduced by 70%+ compared to full-inline approach

### Module Categories

**31 total modules** organized in 7 categories:

| Category | Count | Modules |
|----------|-------|---------|
| **Core** | 5 | STYLE_GUIDE, TERMINOLOGY, GIT_WORKFLOW, TESTING_STRATEGY, SECURITY_CHECKLIST |
| **Software Dev** | 6 | FRONTEND_PATTERNS, BACKEND_PATTERNS, API_DESIGN, DATABASE_CONVENTIONS, CLI_DESIGN, LIBRARY_DESIGN |
| **Academic** | 6 | ACADEMIC_WRITING, CITATION_MANAGEMENT, TRANSLATION_GUIDE, LITERATURE_REVIEW, RESEARCH_ORGANIZATION, DOCUMENT_STRUCTURE |
| **Feature** | 4 | I18N_GUIDE, AUTH_IMPLEMENTATION, REALTIME_PATTERNS, FILE_HANDLING |
| **Quality** | 4 | PERFORMANCE_OPTIMIZATION, TROUBLESHOOTING, PRODUCTION_READINESS, ACCESSIBILITY |
| **Collaboration** | 4 | README_STRUCTURE, ADR_TEMPLATE, RELEASE_PROCESS, ONBOARDING_GUIDE |
| **Scaffolding** | 2 | SCAFFOLDING_DEV_GUIDE, MODE_GUIDE |

### Configuration-Driven Loading

**config.toml structure**:

```toml
[project]
type = "fullstack"  # frontend | backend | fullstack | cli | library | academic | documentation
features = ["api", "database", "auth", "i18n"]
quality = ["performance", "accessibility"]

[academic]
citation_style = "APA"  # APA | MLA | Chicago | IEEE
field = "computer_science"

[modules]
always_enabled = ["STYLE_GUIDE", "TERMINOLOGY", "GIT_WORKFLOW"]
manual_enabled = []
manual_disabled = []
```

### Module Loading Table

**When to load each module** (AI agents follow this table):

| Module | Load When | Trigger Keywords | Location |
|--------|-----------|------------------|----------|
| **STYLE_GUIDE** | Always | (all tasks) | `.scaffolding/docs/STYLE_GUIDE.md` |
| **TERMINOLOGY** | Always | (all tasks) | `.scaffolding/docs/terminology/` |
| **GIT_WORKFLOW** | Always | git, commit, branch | `.scaffolding/docs/GIT_WORKFLOW.md` |
| **FRONTEND_PATTERNS** | `type = "frontend"` or `"fullstack"` | React, component, UI, frontend | `.scaffolding/docs/FRONTEND_PATTERNS.md` |
| **BACKEND_PATTERNS** | `type = "backend"` or `"fullstack"` | API, server, backend, Node.js | `.scaffolding/docs/BACKEND_PATTERNS.md` |
| **API_DESIGN** | `features` contains `"api"` | API, endpoint, REST, GraphQL | `.scaffolding/docs/API_DESIGN.md` |
| **DATABASE_CONVENTIONS** | `features` contains `"database"` | database, SQL, query, schema | `.scaffolding/docs/DATABASE_CONVENTIONS.md` |
| **AUTH_IMPLEMENTATION** | `features` contains `"auth"` | authentication, login, JWT, OAuth | `.scaffolding/docs/AUTH_IMPLEMENTATION.md` |
| **I18N_GUIDE** | `features` contains `"i18n"` | i18n, translation, locale, multilingual | `.scaffolding/docs/I18N_GUIDE.md` |
| **ACADEMIC_WRITING** | `type = "academic"` | paper, thesis, research, citation | `.scaffolding/docs/ACADEMIC_WRITING.md` |
| **CITATION_MANAGEMENT** | `type = "academic"` | reference, bibliography, APA, MLA | `.scaffolding/docs/CITATION_MANAGEMENT.md` |
| **PERFORMANCE_OPTIMIZATION** | `quality` contains `"performance"` | slow, optimize, performance, speed | `.scaffolding/docs/PERFORMANCE_OPTIMIZATION.md` |
| **ACCESSIBILITY** | `quality` contains `"accessibility"` | a11y, WCAG, screen reader, keyboard | `.scaffolding/docs/ACCESSIBILITY.md` |

**Full module list**: See [ADR 0012](./docs/adr/0012-module-system-and-conditional-loading.md)

### TERMINOLOGY Loading Logic

Terminology files are loaded hierarchically based on project configuration:

**1. Always Load**:
- `.scaffolding/docs/terminology/terminology.md` (universal terms)

**2. Domain-Specific** (based on `[project].type`):

```
type = "frontend" or "fullstack":
  → software/common.md
  → software/frontend.md

type = "backend" or "fullstack":
  → software/common.md
  → software/backend.md

features contains "database":
  → software/database.md

type = "academic":
  → academic/common.md
  → academic/{field}.md  (e.g., computer-science.md)
```

**3. Custom Terminology** (highest priority):
- `.agents/terminology/custom.md` (user overrides)

**Priority Order**: Custom > Domain-Specific > Common > Universal

### Conditional Loading Examples

**Example 1: Fullstack Project with Auth**

```toml
[project]
type = "fullstack"
features = ["api", "database", "auth"]
```

**AI Agent loads**:
- Always: STYLE_GUIDE, TERMINOLOGY, GIT_WORKFLOW
- Type-based: FRONTEND_PATTERNS, BACKEND_PATTERNS
- Feature-based: API_DESIGN, DATABASE_CONVENTIONS, AUTH_IMPLEMENTATION
- Terminology: terminology.md, software/common.md, software/frontend.md, software/backend.md, software/database.md

**Example 2: Academic Research Project**

```toml
[project]
type = "academic"

[academic]
citation_style = "APA"
field = "computer_science"
```

**AI Agent loads**:
- Always: STYLE_GUIDE, TERMINOLOGY, GIT_WORKFLOW
- Type-based: ACADEMIC_WRITING, CITATION_MANAGEMENT
- Terminology: terminology.md, academic/common.md, academic/computer-science.md

### Manual Module Control

Override automatic loading in `config.toml`:

```toml
[modules]
# These modules ALWAYS load regardless of project type
always_enabled = ["STYLE_GUIDE", "TERMINOLOGY", "GIT_WORKFLOW", "SECURITY_CHECKLIST"]

# Force-load additional modules
manual_enabled = ["PERFORMANCE_OPTIMIZATION", "ACCESSIBILITY"]

# Disable modules even if project type suggests them
manual_disabled = ["FRONTEND_PATTERNS"]
```

**Priority**: `manual_disabled` > `manual_enabled` > auto-detection > `always_enabled`

### AI Agent Protocol

**On session start**:

1. Read `config.toml` → identify project type, features, quality requirements
2. Check for manual module overrides (`manual_enabled`, `manual_disabled`)
3. Load `always_enabled` modules
4. Load type-based modules (e.g., FRONTEND_PATTERNS if type=frontend)
5. Load feature-based modules (e.g., API_DESIGN if features contains "api")
6. Load quality-based modules (e.g., ACCESSIBILITY if quality contains "accessibility")
7. Load terminology files hierarchically

**During task execution**:

1. Detect task keywords (e.g., "API", "authentication", "performance")
2. If module not yet loaded → load on-demand
3. Reference loaded modules when providing guidance

**When in doubt**: Load the module. Token cost is minimal compared to incorrect guidance.

### Module File Locations

**Current Status** (v2.0.0):

| Status | Modules | Note |
|--------|---------|------|
| ✅ **Exists** | TERMINOLOGY (7 files) | Fully implemented terminology system |
| ⏸️ **Planned** | 30 other modules | Will be created in future versions |

**Terminology System** (Complete):
```
.scaffolding/docs/terminology/
├── README.md
├── terminology.md                    # Universal terms (Git, naming, etc.)
├── software/
│   ├── common.md                     # SDLC, patterns, testing
│   ├── frontend.md                   # React, CSS, performance
│   ├── backend.md                    # Node.js, API, security
│   └── database.md                   # SQL, ORM, optimization
├── academic/
│   ├── common.md                     # Research, writing, ethics
│   └── computer-science.md           # AI/ML, algorithms, HCI
└── project/
    └── custom.md.example             # User overrides
```

**Module files will be created at** (Future):
```
.scaffolding/docs/
├── STYLE_GUIDE.md
├── GIT_WORKFLOW.md
├── FRONTEND_PATTERNS.md
├── BACKEND_PATTERNS.md
├── API_DESIGN.md
├── DATABASE_CONVENTIONS.md
├── AUTH_IMPLEMENTATION.md
├── I18N_GUIDE.md
├── ACADEMIC_WRITING.md
├── CITATION_MANAGEMENT.md
├── PERFORMANCE_OPTIMIZATION.md
├── ACCESSIBILITY.md
└── ... (18 more modules)
```

### Version & Maintenance

- **Module system version**: 2.0.0
- **Terminology system version**: 2.0.0 (Complete)
- **Future module files**: Will align with scaffolding versions
- **Updates**: Documented in `.scaffolding/CHANGELOG.md`

### Related Documentation

- **[ADR 0012 - Module System & Conditional Loading](./docs/adr/0012-module-system-and-conditional-loading.md)** - Complete design decisions
- **[config.toml.example](./config.toml.example)** - Configuration reference
- **[Terminology README](./.scaffolding/docs/terminology/README.md)** - Terminology system guide


## Coding Conventions

**Core Principles** (For full details, see `.scaffolding/docs/STYLE_GUIDE.md` when available):

- **永遠先寫測試** (Test-First): 所有新功能和 bug 修復都必須先寫測試
  - Use TDD workflow (Red-Green-Refactor)
  - 80%+ code coverage target
  - See `tdd-workflow` skill for detailed guidance

- **型別安全** (Type Safety): 所有函數要有型別標注
  - TypeScript: Explicit types for function parameters and return values
  - Python: Use type hints (`def func(param: str) -> int:`)
  - 避免 `any` / `@ts-ignore` unless absolutely necessary

- **簡單優先** (Simplicity First): 避免過度工程化
  - YAGNI (You Aren't Gonna Need It)
  - 只實作當前需要的功能
  - Defer optimization until proven bottleneck

- **程式碼可讀性** (Readability):
  - 使用清晰的命名（變數、函數、類別）
  - 適當的註解（why, not what）
  - 一致的程式碼風格（遵循專案 linter）

- **不可違反** (Never Do):
  - ❌ 刪除或跳過失敗的測試
  - ❌ 使用 `as any` / `@ts-ignore` 來繞過型別錯誤
  - ❌ Commit 包含 secrets 的檔案（.env, API keys）
  - ❌ 在沒被要求的情況下重構既有程式碼

**詳細規範**: 當 STYLE_GUIDE.md 模組可用時，將包含：
- 命名規範（Naming conventions）
- 函數長度與複雜度限制
- 錯誤處理模式
- 註解與文件規範
- 專案特定風格指南

## Commit Message

Write commit messages in English, format:

```
type: brief description
```

**Allowed types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation update
- `refactor`: Refactoring (code improvement without changing functionality)
- `test`: Testing related
- `chore`: Other maintenance work

**Examples:**
```
feat: add user login functionality
fix: resolve database connection error
docs: update API documentation
```

## Pull Request

### PR Title Format (Angular Style)

**使用英文，格式為：**

```
type(scope): brief description
```

**允許的 type：**
- `feat`: 新功能 (New feature)
- `fix`: 修復 bug (Bug fix)
- `docs`: 文件更新 (Documentation)
- `style`: 程式碼風格 (Code style)
- `refactor`: 重構 (Refactoring)
- `test`: 測試 (Tests)
- `chore`: 維護性工作 (Maintenance)

**範例：**
```
feat(auth): add JWT authentication
fix(api): resolve memory leak in user service
docs(readme): update installation guide
refactor(core): simplify error handling logic
```

### PR 內容原則

- **簡潔為主**：只寫重點，避免冗長說明
- **中英並列**：重要資訊使用中英文對照
- **條列式**：使用 bullet points，每點簡短明確
- **必要資訊**：What（做了什麼）、Why（為什麼）、Testing（如何測試）

## File Structure

<!-- TODO: Document the project's directory structure and organization -->

## What NOT to do

- ❌ **不要自作主張改架構**：任何架構變更都必須先討論
- ❌ **不要在沒被要求的情況下重構既有程式碼**：專注在當前任務
- ❌ **不要安裝沒討論過的 dependency**：新增套件前必須先討論必要性和替代方案

## Skill System

This scaffolding supports the **SKILL.md format** for reusable AI behavior patterns. Skills are portable across OpenCode, Cursor, Windsurf, and Claude.

### What are Skills?

**Skills** are encapsulated AI workflows - think of them as functions for AI agents:

- 📦 **Portable**: Work across different AI tools
- 🔄 **Reusable**: Use in multiple projects
- 🎯 **Focused**: One skill, one job (TDD, API design, security audit)
- 📝 **Self-documenting**: Frontmatter describes purpose and usage

### Skill Discovery Paths

Skills are loaded from these directories (in priority order):

```
1. .agents/skills/                      # Project-specific skills (highest priority)
2. .scaffolding/agents/skills/          # Template-provided skills
3. ~/.config/opencode/skills/           # User-installed skills (superpowers)
```

**Priority rule**: Project skills override template skills, template skills override user skills.

### Available Skills

**Project Skills** (`.agents/skills/`):
- Empty by default
- Place project-specific skills here
- These override template and user skills with same name

**Template Skills** (`.scaffolding/agents/skills/`):
- 10 domain-specific skills with Iron Laws (v2.0)
- Backend: backend-patterns, database-optimization, error-handling
- Frontend: frontend-patterns, react-hooks, component-design
- Universal: api-design, security-review
- Testing: e2e-testing, unit-testing

**User Skills** (`~/.config/opencode/skills/superpowers/`):
- `brainstorming` - Feature ideation and planning
- `test-driven-development` - TDD workflow
- `systematic-debugging` - Systematic bug diagnosis
- `using-git-worktrees` - Isolated feature development
- `executing-plans` - Execute implementation plans
- `finishing-a-development-branch` - Complete development work
- ... (13 total skills)

### Bundles & Workflows

**Bundles** (`.agents/bundles.yaml`) - Role-based skill collections:

- `backend-dev` - API design, database, testing, security
- `frontend-dev` - Component design, state management, a11y
- `devops-specialist` - Infrastructure, deployment, monitoring
- `security-engineer` - Security auditing and hardening
- `project-start` - Brainstorming, architecture, setup
- `feature-development` - Building new features
- `production-ready` - Production deployment preparation
- `debugging-master` - Troubleshooting and fixing issues

**Workflows** (`.agents/workflows.yaml`) - Step-by-step playbooks:

- `feature-development` - Complete feature from idea to deployment
- `bug-fix` - Systematic bug diagnosis and fix
- `refactoring` - Safe code refactoring
- `security-audit` - Comprehensive security audit
- `ci-cd-setup` - CI/CD pipeline setup

### Using Skills

**Load single skill:**
```
@use brainstorming
User: "Design a user authentication system"
```

**Load bundle:**
```
@use bundle:backend-dev
User: "Create a REST API for blog posts"
```

**Execute workflow:**
```
@workflow feature-development
User: "Add user profile editing"
```

### Creating New Skills

1. **Copy template:**
   ```bash
   cp -r .scaffolding/docs/examples/skills/template-skill .scaffolding/agents/skills/my-skill
   ```

2. **Edit SKILL.md:**
   - Update frontmatter (name, version, description, tags)
   - Define "When to Use" triggers
   - Write step-by-step "Instructions"
   - Add "Examples" and "Anti-Patterns"

3. **Test:**
   ```
   @use my-skill
   User: "Test my skill"
   ```

### Documentation

- **[SKILL_FORMAT_GUIDE.md](./.scaffolding/docs/SKILL_FORMAT_GUIDE.md)** - Complete skill format guide
- **[AGENTS_MD_GUIDE.md](./.scaffolding/docs/AGENTS_MD_GUIDE.md)** - AGENTS.md standard
- **[ADR 0007](./.scaffolding/docs/adr/0007-agent-skills-ecosystem-integration.md)** - Integration decisions

### Architecture

```
my-vibe-scaffolding/
├── .agents/                    # Project skills & config
│   └── skills/                 # Custom skills
├── .scaffolding/
│   ├── docs/
│   │   ├── SKILL_FORMAT_GUIDE.md   # Skill creation guide
│   │   └── examples/
│   │       └── skills/
│   │           └── template-skill/     # Skill template
├── data/
│   ├── bundles.yaml            # Role-based skill collections
│   └── workflows.yaml          # Step-by-step playbooks
└── AGENTS.md                   # This file
```


### Default Skills for This Project

**Auto-load these skills based on task context:**

| Task Type | Skills | Trigger Keywords |
|-----------|--------|------------------|
| Feature Development | `brainstorming`<br>`test-driven-development` | "新增功能", "實作", "開發" |
| Bug Fixing | `systematic-debugging` | "bug", "錯誤", "修正", "不work" |
| Code Review | `requesting-code-review` | "review", "檢查程式碼" |
| Planning | `brainstorming`<br>`writing-plans` | "規劃", "設計", "架構" |
| Git Workflow | `using-git-worktrees` | "feature branch", "worktree" |

**Usage:**
```
# AI will auto-detect based on your request
User: "新增使用者登入功能"  → auto-loads brainstorming + test-driven-development
User: "修正登入 bug"        → auto-loads systematic-debugging

# Or manually load
@use brainstorming
User: "設計認證系統"
```


## Internationalization (i18n)

This template supports multiple natural languages for documentation and templates using **BCP 47 (RFC 5646)** language tags.

### Quick Start

1. Copy the config example:
   ```bash
   cp config.toml.example config.toml
   ```

2. Edit `config.toml` to set your preferred language:
   ```toml
   [locale]
   primary = "zh-TW"  # or "en-US", "ja-JP", etc.
   fallback = "en-US"
   ```

3. The following content will adapt to your language:
   - This file (AGENTS.md) - Coding conventions, commit format
   - README.md - Project description, usage instructions
   - GitHub templates - Issue/PR templates
   - ADR templates - Architecture decision records

**Note:** Code (variable names, function names, comments) stays in English for universal comprehension.

### Available Languages

- `en-US` - English (United States) - Base language
- `zh-TW` - Traditional Chinese (Taiwan)
- `zh-HK` - Traditional Chinese (Hong Kong) - Planned
- `zh-CN` - Simplified Chinese (China) - Planned
- `ja-JP` - Japanese (Japan) - Planned

### Why BCP 47?

BCP 47 (RFC 5646) is the IETF standard for language identification, used by:
- W3C WCAG (Web accessibility)
- HTML `lang` attribute
- EPUB (e-books)
- Unicode CLDR

This allows precise distinction between:
- `zh-TW` (台灣繁體) vs `zh-HK` (香港繁體) vs `zh-CN` (簡體)
- `en-US` (American English) vs `en-GB` (British English)

### Git Strategy

**Committed to Git:**
- `config.toml.example` - Configuration template
- `i18n/locales/en-US/` - English (base)
- `i18n/locales/zh-TW/` - Traditional Chinese (Taiwan)

**Not committed (`.gitignored`):**
- `config.toml` - Your personal language preference

This allows each team member to use their preferred language locally while maintaining a language-agnostic codebase.

For detailed information, see [`i18n/README.md`](./i18n/README.md).

**Reference:** [RFC 5646 - Language Tags](https://www.rfc-editor.org/rfc/rfc5646.html)

## AI Agent Communication Protocol

**CRITICAL: AI agents MUST follow this protocol at the start of EVERY session.**

### 1. Read User's Language Preference

On session start, ALWAYS read the user's language configuration:

```bash
# Read config.toml
[i18n]
primary_locale = "zh-TW"  # User's preferred language
fallback_locale = "en-US"
```

**If config.toml doesn't exist:** Use `en-US` as default.

### 2. Load Translation Files

Load translations from `.scaffolding/i18n/locales/{primary_locale}/`:

- `agents.toml` - Coding conventions, commit format, PR guidelines
- `readme.toml` - Project documentation phrases
- `templates.toml` - Issue/PR template text
- `adr.toml` - ADR template phrases

**Example (zh-TW):**
```bash
# Load coding conventions
.scaffolding/i18n/locales/zh-TW/agents.toml

[coding_conventions]
title = "編碼規範"
test_first = "**永遠先寫測試**：所有新功能和 bug 修復都必須先寫測試"
```

### 3. Communication Language Rules

**Use the user's configured language for ALL responses and communication:**

| Configuration | Communication Language | Example |
|---------------|------------------------|---------|
| `primary_locale = "zh-TW"` | 繁體中文（台灣） | "我已經完成了這個功能..." |
| `primary_locale = "en-US"` | English (US) | "I've completed this feature..." |
| `primary_locale = "ja-JP"` | 日本語 | "この機能を完了しました..." |

**Code and Technical Terms:**
- Variable names, function names → Always English
- Code comments → Use primary_locale language
- Commit messages → Follow locale-specific format in `agents.toml`
- Technical documentation → Use primary_locale language

**Documentation Language Guidelines:**

| File/Directory | Language | Reason |
|----------------|----------|--------|
| Root `README.md` | 🌐 Multi-language (i18n) | User-facing, needs language support |
| `.scaffolding/*` files | 🇬🇧 English only | AI-facing, English is most direct |
| `AGENTS.md` | 🌐 Multi-language | AI reads, but users also reference |
| `docs/adr/*.md` | 🇬🇧 English only | Technical decisions, for AI and developers |
| `scripts/*` | 🇬🇧 English only | Tool documentation |

**Why English for `.scaffolding/` files:**
1. AI models are primarily trained on English
2. International accessibility for technical content
3. Reduces translation maintenance cost

**Exceptions:**
- Project root `CHANGELOG.md` → Use project's primary language
- Project `docs/` files → Project decides

### 4. Fallback Strategy

If a translation key is missing:

1. Check `fallback_locale` in config.toml (usually `en-US`)
2. Load the key from fallback locale
3. Continue without error
4. Optionally note the missing translation

### 5. Session Start Checklist

**Before responding to ANY user message:**

- [ ] Read `config.toml` and identify `primary_locale`
- [ ] Load translation files from `.scaffolding/i18n/locales/{primary_locale}/`
- [ ] Set communication language to match `primary_locale`
- [ ] Verify fallback locale is available

### 6. README Generation Protocol

**CRITICAL: README files are auto-generated from i18n translations. DO NOT edit README.md directly.**

### Workflow

1. **Edit translation files:**
   ```bash
   .scaffolding/i18n/locales/en-US/readme.toml
   .scaffolding/i18n/locales/zh-TW/readme.toml
   ```

2. **Generate README files:**
   ```bash
   ./.scaffolding/scripts/generate-readme.sh
   ```

   This script:
   - Reads content from `i18n/locales/{lang}/readme.toml`
   - Generates `README.md` (English) and `README.zh-TW.md` (中文)
   - Syncs to `.scaffolding/README.md` and `.scaffolding/README.zh-TW.md`
   - Adds language switcher links automatically
   - **NO markdown code fences** around content (README is already markdown)

3. **MANDATORY Verification (ALWAYS do this before committing):**
   ```bash
   # Verify all 4 files exist and are in sync
   diff README.md .scaffolding/README.md
   diff README.zh-TW.md .scaffolding/README.zh-TW.md
   ```
   
   **Success criteria:**
   - ✅ Both diffs show no differences (files are identical)
   - ✅ Both language versions updated (en-US and zh-TW)
   - ✅ Language switcher links present in both files
   - ✅ No ```markdown code fences wrapping content
   
   **If verification fails:**
   - Re-run generate-readme.sh
   - Do NOT commit until all checks pass
   - Reads content from `i18n/locales/{lang}/readme.toml`
   - Generates `README.md` (English) and `README.zh-TW.md` (中文)
   - Syncs to `.scaffolding/README.md` and `.scaffolding/README.zh-TW.md`
   - Adds language switcher links automatically
   - **NO markdown code fences** around content (README is already markdown)

   ### Current Configuration

This template uses **`separate` strategy**:
- `README.md` - English (auto-generated from `en-US/readme.toml`)
- `README.zh-TW.md` - 繁體中文 (auto-generated from `zh-TW/readme.toml`)

### Important Rules

1. **NEVER edit README.md or README.zh-TW.md directly** - Changes will be overwritten
2. **Edit i18n TOML files** - All content comes from `i18n/locales/{lang}/readme.toml`
3. **Run generate-readme.sh** - After editing TOML files, regenerate READMEs
4. **NO ```markdown``` code fences** - README is markdown, content doesn't need wrapping

3. **Verify language switcher (for separate strategy):**
   - Top of README.md: Links to all language versions
   - Format: `English | [繁體中文](./README.zh-TW.md) | [日本語](./README.ja-JP.md)`
   - Current language shows as plain text (no link to itself)

4. **Validation:**
   - Separate: Each README file exists and is single-language
   - Bilingual: Single README.md follows bilingual formatting rules
   - Primary only: Only README.md exists

**Reference:** [`.scaffolding/docs/README_BILINGUAL_FORMAT.md`](./.scaffolding/docs/README_BILINGUAL_FORMAT.md)

**This is MANDATORY. No exceptions.**


## Documentation Standards

**CRITICAL: All AI agents MUST follow these rules when creating or modifying documentation.**

### Core Principles

1. **Read First**: Before creating ANY new document, check [`.scaffolding/docs/DOCUMENTATION_GUIDELINES.md`](./.scaffolding/docs/DOCUMENTATION_GUIDELINES.md)
2. **Root Level Simplicity**: Keep root directory minimal (only core files)
3. **No Intermediate Files**: No `GET_STARTED.md`, `TASK_*.md`, etc.
4. **Template vs Project**: Distinguish framework docs from project-specific docs

### Required Reading

- **[`.scaffolding/docs/DOCUMENTATION_GUIDELINES.md`](./.scaffolding/docs/DOCUMENTATION_GUIDELINES.md)** - File organization standards (MUST READ)
- **[`.scaffolding/docs/README_GUIDE.md`](./.scaffolding/docs/README_GUIDE.md)** - How to write project README when using this template
- **[`.scaffolding/docs/TEMPLATE_SYNC.md`](./.scaffolding/docs/TEMPLATE_SYNC.md)** - How to sync template updates

### When Creating Documents

**Ask yourself:**
1. Is this core functionality? → `docs/` or `docs/adr/`
2. Is this temporary? → Use `.worklog/` (gitignored) or don't create
3. Is this tool-specific? → Belongs in tool's own docs, not project root
4. Is this already documented? → Update existing file instead

### File Placement Rules

| File Type | Location | Example |
|-----------|----------|---------|
| Architecture decisions | `docs/adr/NNNN-*.md` | `0005-single-instance-opencode.md` |
| Core documentation | `docs/*.md` | `DOCUMENTATION_GUIDELINES.md` |
| Work logs (personal) | `.worklog/YYYY-MM-DD.md` | Gitignored, daily files |
| Tool usage guides | Tool's own README | `scripts/wl` → usage in `--help` |

**❌ NEVER create:**
- Root-level intermediate files (GET_STARTED, TASK_COMPLETION, etc.)
- Problem-specific guides (OPENCODE_STABILITY, etc.)
- Tool tutorials as separate docs

### Version Management

**CRITICAL: Every push to main MUST have a version bump.**

**Why:** Direct main branch workflow (no dev branch) means every commit is potentially "released". Without version bumps, v1.2.0 before your change and v1.2.0 after your change are different, causing confusion.

**Enforcement:**
- Pre-push git hook automatically checks if VERSION has been updated
- Push will be blocked if version hasn't changed since last tag
- Install hooks: `./.scaffolding/scripts/install-hooks.sh`

**Workflow:**
1. Make your changes
2. **Before committing**, bump version:
   ```bash
   ./.scaffolding/scripts/bump-version.sh patch  # Bug fixes
   ./.scaffolding/scripts/bump-version.sh minor  # New features
   ./.scaffolding/scripts/bump-version.sh major  # Breaking changes
   ```
3. The script will:
   - Update `.scaffolding/VERSION` and `VERSION`
   - Update `.scaffolding/CHANGELOG.md` and `README.md` badges
   - Create commit with version bump
   - Create git tag
4. Push (hook will verify version changed):
   ```bash
   git push && git push --tags
   ```

**For template maintainers:**
- Version file: `.scaffolding/VERSION`
- Always update `.scaffolding/CHANGELOG.md` when bumping version
- Create meaningful release notes

**Semantic Versioning Rules (MAJOR.MINOR.PATCH):**

Given a version number MAJOR.MINOR.PATCH (e.g., 1.3.0):

**PATCH (1.3.0 → 1.3.1)** - Bug fixes, documentation updates:
- ✅ Fix typos in documentation
- ✅ Fix broken links
- ✅ Correct code comments
- ✅ Fix linter warnings
- ✅ Security patches that don't change API
- ✅ Performance improvements (no API change)

**MINOR (1.3.0 → 1.4.0)** - New features (backward compatible):
- ✅ Add new optional parameters to functions
- ✅ Add new files/scripts
- ✅ Add new configuration options (with defaults)
- ✅ Deprecate features (but still functional)
- ✅ Add new documentation sections
- ✅ Enhance existing features without breaking old usage

**MAJOR (1.3.0 → 2.0.0)** - Breaking changes (STRICT CRITERIA):

**Only these scenarios qualify as breaking:**

1. **Remove or rename files that users depend on**
   - ❌ DELETE: `scripts/my-tool.sh` (if users run it)
   - ❌ RENAME: `AGENTS.md` → `AI_AGENTS.md`
   - ✅ OK: Add new files (not breaking)
   - ✅ OK: Rename internal `.scaffolding/` files (users don't directly use)

2. **Change required configuration format**
   - ❌ CHANGE: `config.toml` structure requires migration
   - ❌ REMOVE: Required config key without default
   - ✅ OK: Add optional config with sensible default
   - ✅ OK: Deprecate config (still works with warning)

3. **Change command signatures that break existing usage**
   - ❌ CHANGE: `bump-version.sh <type>` → `bump-version.sh --type <type>`
   - ❌ REMOVE: Required parameter
   - ✅ OK: Add optional parameter
   - ✅ OK: Add new command

4. **Change output format that tools depend on**
   - ❌ CHANGE: Script output from JSON to YAML
   - ❌ REMOVE: Expected output field
   - ✅ OK: Add new output fields
   - ✅ OK: Improve error messages

**Common misconceptions (NOT breaking):**
- ❌ Moving files within `.scaffolding/` (internal structure)
- ❌ Refactoring code (if API unchanged)
- ❌ Improving documentation
- ❌ Adding new features alongside old ones
- ❌ Changing internal implementation
- ❌ Reorganizing directory structure (if paths still work)

**When in doubt: Choose MINOR over MAJOR.**

Breaking changes require users to modify their code/config. If they don't need to change anything, it's NOT breaking.


**For template users:**
- After "Use this template": run `./.scaffolding/scripts/init-project.sh`
- This creates `.template-version` to track which template version you're using
- See [`.scaffolding/docs/README_GUIDE.md`](./.scaffolding/docs/README_GUIDE.md) for project README guidance

## Spec-Driven Development (SDD) Workflow

**Version**: 1.0.0 (v3.0.0+)  
**Integration**: OpenSpec + Superpowers TDD

### Overview

SDD provides a structured workflow from ideation to implementation with automatic change tracking.

### Workflow Phases

```
brainstorming → sdd-propose → sdd-apply → sdd-archive
```

**Phase 1: Explore** (`brainstorming`)
- Free-form ideation
- Requirements gathering
- Approach discussion
- **Output**: Informal notes

**Phase 2: Formalize** (`sdd-propose`)
- Convert brainstorming to OpenSpec format
- Create proposal.md, design.md, tasks.md
- **Output**: `.openspec/specs/{feature}/`

**Phase 3: Implement** (`sdd-apply`)
- Execute spec with **mandatory TDD**
- Track all changes
- **Output**: `.openspec/changes/{change-id}/` + production code

**Phase 4: Archive** (`sdd-archive`)
- Move to archive after PR merge
- Document completion
- **Output**: `.openspec/changes/archive/{change-id}/`

### SDD Skills

**sdd-propose** (`.scaffolding/agents/skills/sdd/sdd-propose.md`):
- Reads brainstorming output
- Creates OpenSpec structure (proposal, design, tasks)
- Location: `.openspec/specs/{feature}/`

**sdd-apply** (`.scaffolding/agents/skills/sdd/sdd-apply.md`):
- **Mandatory TDD**: Always calls `test-driven-development`
- Tracks changes in `.openspec/changes/{change-id}/`
- Documents implementation decisions

**sdd-archive** (`.scaffolding/agents/skills/sdd/sdd-archive.md`):
- Archives after PR merge
- Creates completion summary
- Updates spec status

### Configuration

Enable SDD in `config.toml`:

```toml
[sdd]
enabled = true  # Default: false (opt-in)
auto_tracking = true  # Auto-create change tracking
```

### Usage Example

```
# Phase 1: Brainstorm
@use brainstorming
User: "I want to add user authentication"

# Phase 2: Formalize
@use sdd-propose
User: "Create the auth spec"
# Creates .openspec/specs/user-auth/

# Phase 3: Implement
@use sdd-apply
User: "Implement the auth spec"
# Automatically calls TDD, creates .openspec/changes/20260327-1400-user-auth/

# Phase 4: Archive (after PR merge)
@use sdd-archive
User: "PR #123 merged, archive it"
# Moves to .openspec/changes/archive/
```

### Key Features

**Change Tracking**: Every implementation tracked in `.openspec/changes/`
**Mandatory TDD**: `sdd-apply` enforces test-driven development
**Traceability**: Complete history from idea to deployment
**Metrics**: Actual vs estimated effort tracking

### Directory Structure

```
.openspec/
├── project.md          # Project metadata
├── specs/              # Formal specifications
│   └── {feature}/
│       ├── proposal.md
│       ├── design.md
│       ├── tasks.md
│       └── STATUS.md
└── changes/            # Implementation tracking
    ├── {change-id}/    # Active changes
    └── archive/        # Completed changes
        └── {change-id}/
            ├── implementation.md
            ├── tests.md
            ├── decisions.md
            └── COMPLETION.md
```

### References

- **OpenSpec**: https://kaochenlong.com/openspec
- **Blog Post**: https://kaochenlong.com/ai-superpowers-skills
- **Skills Location**: `.scaffolding/agents/skills/sdd/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/matheme-justyn)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/matheme-justyn)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
