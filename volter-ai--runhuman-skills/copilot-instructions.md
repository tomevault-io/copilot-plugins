## runhuman-skills

> Create and manage human-powered QA tests using Runhuman CLI. Use this skill when you need to test web applications with real human testers, get UX feedback, validate user flows, check mobile responsiveness, or find bugs that automated tests miss.


# Runhuman CLI

Runhuman connects AI coding tools to on-demand professional human testers. You describe what to test in natural language, a real human performs the test, and you get structured feedback back.

## When to Use Runhuman

**Good for:** User flows (signup, checkout), visual/layout testing, mobile responsiveness, UX feedback, exploratory testing, cross-browser checks.

**Not for:** Unit tests, performance benchmarks, security audits, or high-frequency automated test suites.

## Quick Start

```bash
# 1. Install (or use npx runhuman for any command without installing)
npm install -g runhuman

# 2. Authenticate (opens browser for GitHub OAuth)
runhuman login

# 3. Set up a project
#    List existing projects:
runhuman projects list
#    Or create a new one:
runhuman projects create "My App" --organization <orgId> --default-url https://staging.myapp.com --set-default
#    Switch to an existing project:
runhuman projects switch <projectId>

# 4. Create a test
runhuman job create https://staging.myapp.com \
  -d "Test the signup flow: click Sign Up, fill the form, verify confirmation page"
```

**Deciding between new vs existing project:** Ask the user whether they want to create a new project or use an existing one. Use `runhuman projects list` to show what exists, then either `projects create` with `--set-default` or `projects switch` to set the active project. `projects switch` sets the default globally (in `~/.config/runhuman/config.json`) by default; pass `--global false` to set it locally for the current directory only.

**Setting defaults:** Use `runhuman config set <key> <value>` to set defaults like `project`, `color`, or `apiUrl`. Use `runhuman config list` to see the full configuration hierarchy.

**Local repo config:** Run `runhuman init` to create a `.runhumanrc` in the current directory with project name, default duration/device class, and one-or-more linked GitHub repos. Pass `--yes` to skip prompts, or `--name`, `--github-repo <owner/repo>`, or `--github-repos <a/b,c/d>` to fill values non-interactively.

## Job Command Namespace

All job commands live under `runhuman job`:

```bash
runhuman job create   # create a new test
runhuman job status   # check a job's status
runhuman job wait     # block until a job completes
runhuman job results  # view detailed results
runhuman job list     # list jobs (with filters)
runhuman job watch    # tail a live job
runhuman job artifact # download a raw session artifact
runhuman job share    # toggle public sharing on
runhuman job unshare  # toggle public sharing off
runhuman job create-issue  # file an extracted finding as a GitHub issue
```

> **Deprecated flat shims.** The older flat forms (`runhuman create`, `runhuman status`, `runhuman wait`, `runhuman results`, `runhuman list`, `runhuman watch`) still resolve but emit a one-line deprecation warning and are scheduled for removal. **Always emit `runhuman job <subcommand>` in new scripts** — don't introduce flat-shim invocations.

### Truncated IDs

All commands accept truncated ID prefixes, similar to git short hashes. The CLI resolves the best match automatically:

```bash
runhuman job status 712e          # resolves to full job ID
runhuman projects switch proj     # resolves to matching project
```

If multiple IDs match, the CLI asks for more characters. Destructive operations (`projects delete`, `keys delete`, `projects transfer --to-org`) require full IDs.

### Wait for Results

```bash
# Block until the test completes (useful in CI/CD)
runhuman job create https://staging.myapp.com \
  -d "Test checkout flow end-to-end" \
  --sync

# Check status of an existing job
runhuman job status <jobId>

# Wait for a previously-created job to finish
runhuman job wait <jobId>

# View detailed results
runhuman job results <jobId>
```

## Creating Tests

The `job create` command is the primary way to submit tests. At minimum, provide a URL and either a description or a template.

### With a Description

```bash
runhuman job create https://staging.myapp.com \
  -d "Test the signup flow: click Sign Up, fill the form, verify confirmation page"
```

### With an Output Schema

Use `--schema` or `--schema-inline` to get structured JSON results back:

```bash
# Schema from a file
runhuman job create https://staging.myapp.com \
  -d "Test the search feature" \
  --schema ./search-schema.json \
  --sync --json

# Inline schema
runhuman job create https://staging.myapp.com \
  -d "Test signup and login" \
  --schema-inline '{"signupWorks":{"type":"boolean"},"loginWorks":{"type":"boolean"},"issues":{"type":"array","items":{"type":"string"}}}' \
  --sync --json
```

Example schema file (`search-schema.json`):
```json
{
  "searchWorks": {
    "type": "boolean",
    "description": "Does the search return results?"
  },
  "resultCount": {
    "type": "number",
    "description": "Number of results shown"
  },
  "issues": {
    "type": "array",
    "description": "List of any bugs or issues found"
  }
}
```

When a job completes with a schema, `runhuman job results <jobId> --json` returns the extracted data inside the standard envelope at `data.result.data`:

```json
{
  "success": true,
  "data": {
    "jobId": "job_abc123",
    "status": "completed",
    "result": {
      "passed": true,
      "explanation": "All tests passed successfully",
      "data": {
        "searchWorks": true,
        "resultCount": 10,
        "issues": []
      }
    },
    "costUsd": 0.18,
    "testDurationSeconds": 120
  }
}
```

Use `--schema-only` to get just the extracted schema data: `runhuman job results <jobId> --schema-only`.

### With a Template

Templates are reusable test configurations. Use `--template` to reference one by name or `--template-file` to point to a local `.md` file:

```bash
# By name (resolved from repo .runhuman/templates/ → project templates → built-ins)
runhuman job create https://staging.myapp.com --template "Find Bugs"

# By local file path
runhuman job create --template-file .runhuman/templates/smoke-test.md
```

When using a template, URL and description are optional — they can come from the template itself.

### Device Class

Specify `--device-class desktop` or `--device-class mobile` to control what device the tester uses:

```bash
runhuman job create https://staging.myapp.com -d "Test mobile layout" --device-class mobile
```

### Tester Pool Requirements

Filter which testers are eligible for the job. **Max / Enterprise / Enterprise Pro plans only** — on lower plans, the server rejects job creation with an upgrade link. Do not add these flags unless the user has said they're on a qualifying plan or explicitly asked for pool filtering.

| Flag | Values | Match |
|------|--------|-------|
| `--required-devices` | comma-separated: `ios`, `android`, `pc`, `mac` | any — tester needs at least one |
| `--required-languages` | comma-separated: `english`, `spanish` | all — tester must speak every one |
| `--require-social-videos` | boolean | — |
| `--require-apk-install` | boolean | — |

So `--required-languages english,spanish` requires a bilingual tester, not either/or.

```bash
runhuman job create https://staging.myapp.com \
  -d "Test the checkout flow on Android" \
  --required-devices android
```

### Multi-Repo Context

A job can be linked to one or more GitHub repos so AI-extracted findings can be filed back as issues. Provide repos in any of three ways:

```bash
# Single repo (back-compat flag)
runhuman job create https://staging.myapp.com -d "Test checkout" \
  --github-repo owner/repo

# Multiple repos, comma-separated
runhuman job create https://staging.myapp.com -d "Test checkout" \
  --github-repos "owner/frontend,owner/backend"

# Or inherit from the project / template / .runhumanrc default
runhuman job create https://staging.myapp.com -d "Test checkout"
```

Add `--auto-create-github-issues` to have the server file each AI-extracted finding as a GitHub issue automatically once the job completes. Requires at least one linked repo (via the flags above, the template, or the project default).

### Async Workflow (Create, Wait, Get Results)

If you don't use `--sync`, create the job first, then wait for it separately:

```bash
# 1. Create the job (returns immediately)
JOB_ID=$(runhuman job create https://staging.myapp.com -d "Test search" --json | jq -r '.data.jobId')

# 2. Wait for the tester to complete (blocks with live status updates)
runhuman job wait "$JOB_ID"

# 3. Get the results
runhuman job results "$JOB_ID" --json
```

`job wait` accepts `--timeout <seconds>` (default: 600). Use `--json` on any command when you need structured output for scripting or automation. JSON responses use a stable envelope — list payloads sit at `data.items`, single-entity payloads sit at `data` directly.

## Templates

Templates define what to test, how long to test, and what results to collect. There are three sources, checked in this order:

| Source | Location | How to create |
|--------|----------|---------------|
| **Repo templates** | `.runhuman/templates/*.md` in the user's repo | Commit markdown files |
| **Project templates** | Stored in Runhuman database | `templates create` or dashboard |
| **Built-in templates** | Bundled with Runhuman | Available automatically |

### Built-in Templates

Three built-in templates cover common QA scenarios:

| Template | What it does |
|----------|-------------|
| **Find Bugs** | Test the app and report bugs with severity, steps to reproduce, expected behavior |
| **Assess UX** | Evaluate usability, visual design, mobile-friendliness, and rate 1-10 |
| **Give Product Feedback** | Provide honest feedback: first impressions, strengths, weaknesses, suggestions |

### Repo Templates

Repo templates are `.md` files in `.runhuman/templates/` with YAML frontmatter:

```markdown
---
name: Smoke Test
duration: 5
device_class: desktop
url: https://staging.example.com
---

Verify the app loads and core features work:

1. Load the homepage and confirm it renders without errors
2. Navigate to the login page and verify the form appears
3. Check that the main navigation links work

## Results

Page loads without errors: [ ]
Login form is accessible: [ ]
Navigation works: [ ]
Issues found: ___
```

Frontmatter fields (all optional): `name`, `duration` (minutes, 1-60), `device_class` (`desktop`/`mobile`), `url`, `max_extension_minutes`, `extension_count`, `github_repos` (YAML list of `owner/repo` strings).

### Creating Project Templates via CLI

```bash
runhuman templates create "Search Test" \
  --project proj_abc123 \
  -d "Search for 'recursion' and confirm the joke appears" \
  --schema ./search-test-schema.json
```

Options: `--duration <minutes>` (1-60), `--device-class <class>`, `--schema <path>`.

List templates: `runhuman templates list --project <projectId>`

## Job Output: Results, Artifacts, Sharing, Issues

`job results` is the high-level view. For raw access or follow-on actions, four other sub-commands are available:

```bash
# High-level results view (with optional rich-data flags)
runhuman job results <jobId>
runhuman job results <jobId> --transcript --console-logs
runhuman job results <jobId> --all          # every rich section
runhuman job results <jobId> --schema-only  # just extracted schema data
runhuman job results <jobId> --raw          # raw tester response

# Download a single raw artifact (good for piping)
runhuman job artifact <jobId> transcript
runhuman job artifact <jobId> console-logs --json | jq '.data.consoleMessages[]'
# Other types: structured-output, network-requests, conversation, events, key-moments

# Toggle public sharing — `share` prints the public URL
runhuman job share <jobId>
runhuman job unshare <jobId>

# File one extracted finding as a GitHub issue
runhuman job create-issue <jobId> --index 0
runhuman job create-issue <jobId> --index 0 --repo owner/repo  # override default
```

Rich-data sections (transcript, logs, network, events, key-moments, conversation) and the corresponding `job artifact` types are paid-tier gated server-side. Ungated requests return a clean "subscription required" error.

`job create-issue` mirrors the dashboard's per-finding "Create Issue" button — it submits the finding unedited. Use the dashboard or `gh issue edit` if you want to tweak title/body afterwards.

## Projects

```bash
runhuman projects list
runhuman projects create "My App" --organization <orgId> [--default-url <url>] [--github-repo <owner/repo>] [--github-repos <a/b,c/d>] [--set-default]
runhuman projects show <projectId>
runhuman projects update <projectId> [--name <text>] [--default-url <url>] [--github-repo <owner/repo>] [--github-repos <a/b,c/d>]
runhuman projects switch <projectId>            # `--global false` for current dir only
runhuman projects transfer <projectId> --to-org <orgId>   # full IDs required
runhuman projects delete <projectId> --force    # full ID required
```

`--github-repo` accepts a single `owner/repo`; `--github-repos` accepts a comma-separated list and replaces (rather than appends to) the project's repo set.

## Organizations

```bash
runhuman orgs list
runhuman orgs show <orgId>
runhuman orgs balance <orgId>
runhuman orgs projects <orgId>
runhuman orgs switch <orgId>                # `--global` (default) or local

# Membership
runhuman orgs members <orgId>
runhuman orgs invite <orgId> <email> --role <admin|contributor|viewer>
runhuman orgs remove <orgId> <userId>

# Pending invitations
runhuman orgs invitations list --organization <orgId>
runhuman orgs invitations revoke <invitationId> --organization <orgId> --yes

# Usage analytics
runhuman orgs usage                          # current org, last 30 days
runhuman orgs usage --period 7d              # 7d | 30d | 90d | all
runhuman orgs usage --year 2026 --month 4    # specific month (year+month)
runhuman orgs usage --json                   # full response, incl. overTime / byHour / bySource
```

`orgs invite` defaults to `--role contributor`. Default-org context is used by `invitations list/revoke` and `usage` when `--organization` is omitted.

## Billing

User-scoped reads. Purchase flows and plan changes stay in the web UI — use `billing portal` to get there.

```bash
runhuman billing balance                # credit balance for primary org (or --organization)
runhuman billing subscription           # current tier, period, add-ons
runhuman billing portal                 # one-time URL to the Polar customer portal
runhuman billing portal --open          # open it in the default browser
```

## Tester Notes (paid feature)

Notes are a knowledge-base entries shown to testers as additional context. **Requires an active organization subscription** — calls fail cleanly with a subscription-required error otherwise.

```bash
runhuman notes list                          # everything visible to you
runhuman notes list --search "checkout" --tags "context,gotcha"
runhuman notes show <noteId>
runhuman notes create "Auth context" -b "Use the demo account..." --tags "context"
runhuman notes create "Auth context" --body-file ./note.md --project <projectId>
runhuman notes update <noteId> [--title <t>] [--body <text> | --body-file <path>] [--tags <list>] [--public|--internal]
runhuman notes delete <noteId>               # full ID required
```

Notes default to internal-only; pass `--public` on `create`/`update` to make them customer-visible. Scope is org by default, or pass `--project` to scope to a single project.

## GitHub Integration

```bash
runhuman github link <owner/repo> --project <projectId>
runhuman github repos --organization <orgId> [--search <query>]
runhuman github issues <owner/repo> [--state open|closed|all] [--labels "bug,needs-qa"]
runhuman github test <issueNumber> -r <owner/repo> -u <url> [--template <id>] [--sync]
runhuman github bulk-test -r <owner/repo> -u <url> [--labels <list>] [--state <state>] [--limit <n>]

# Force-refresh a GitHub App installation's repo/permissions list — use after granting
# the app access to new repos if they don't show up in `github repos` yet.
runhuman github installation refresh --installation <installationId> --organization <orgId>
```

## Schedules and Transfers

```bash
# Recurring scheduled jobs (cron-style)
runhuman schedules list
runhuman schedules create / show / update / delete   # see --help for each

# Project transfers (accept/reject/cancel a transfer to/from another org)
runhuman transfers list
runhuman transfers accept <transferId>
runhuman transfers reject <transferId>
runhuman transfers cancel <transferId>
```

Run `runhuman schedules --help` and `runhuman transfers --help` for full surfaces.

## Auth, Init, Config

```bash
# Auth
runhuman login                    # browser OAuth
runhuman login --token <apiKey>   # non-interactive
runhuman logout
runhuman whoami

# Initialize a local .runhumanrc
runhuman init
runhuman init --name "My App" --github-repos "owner/frontend,owner/backend" --yes

# Config
runhuman config get <key>
runhuman config set <key> <value> [--global]
runhuman config list [--show-secrets]
runhuman config reset --project|--global|--all --force
```

Config priority: CLI flags > env vars (`RUNHUMAN_API_KEY`, `RUNHUMAN_API_URL`, `RUNHUMAN_PROJECT_ID`, `RUNHUMAN_NO_COLOR`) > `.runhumanrc` (project) > `~/.config/runhuman/config.json` (global) > defaults.

## Command Groups

Run `runhuman <command> --help` for full usage details on any command.

| Group | Commands | Purpose |
|-------|----------|---------|
| **Jobs** | `job create`, `status`, `wait`, `results`, `list`, `watch`, `artifact`, `share`, `unshare`, `create-issue` | Create and manage QA test jobs (canonical namespace) |
| **Auth** | `login`, `logout`, `whoami` | Authentication |
| **Init** | `init` | Bootstrap a `.runhumanrc` in the current directory |
| **Projects** | `projects list`, `create`, `show`, `update`, `switch`, `transfer`, `delete` | Manage projects |
| **Organizations** | `orgs list`, `show`, `balance`, `projects`, `switch`, `members`, `invite`, `remove`, `invitations`, `usage` | Manage organizations and membership |
| **Billing** | `billing balance`, `subscription`, `portal` | User-scoped billing reads |
| **Templates** | `templates list`, `create`, `show`, `update`, `delete` | Reusable test configurations |
| **Schedules** | `schedules list`, `create`, `show`, `update`, `delete` | Recurring scheduled jobs |
| **Transfers** | `transfers list`, `accept`, `reject`, `cancel` | Project transfers between orgs |
| **API Keys** | `keys list`, `create`, `show`, `delete` | Manage API keys |
| **GitHub** | `github link`, `repos`, `issues`, `test`, `bulk-test`, `installation refresh` | GitHub integration |
| **Notes** (paid) | `notes list`, `show`, `create`, `update`, `delete` | Tester knowledge-base notes |
| **Config** | `config get`, `set`, `list`, `reset` | CLI configuration |

> **Reminder:** the flat job shims (`runhuman create`, `status`, `wait`, `results`, `list`, `watch`) are deprecated. Always use the `runhuman job <sub>` form in new scripts.

## Writing Effective Test Descriptions

The `-d` / `--description` flag is the most important part of a test. Be specific.

**Good:**
```
Test the checkout process:
1. Add a product to cart
2. Click "Checkout"
3. Fill shipping address
4. Complete purchase
Report: bugs, UX confusions, mobile layout issues
```

**Bad:**
```
Test the site
```

Tips:
- Include exact steps or flows to follow
- Specify what pages and buttons to interact with
- State what to look for (bugs, UX issues, visual problems)
- For mobile testing, mention that explicitly in the description

When the project has a repo connected, Runhuman has learned the codebase and test results come back richer and more detailed — you can act on findings directly. See the [Code Context docs](https://runhuman.com/docs/code-context).

## CI/CD Integration

> **When users ask about CI/CD, GitHub Actions, or automating QA in their pipeline:** Fetch the comprehensive agent guide at `https://runhuman.com/for_agents_github_actions.md` and follow its instructions to walk the user through setup. The guide covers PR-triggered testing, scheduled exploratory QA, on-demand testing, label management, and platform-specific examples (Vercel, Netlify, Fly.io, Railway).

> **Determining the latest action version:** Before writing any workflow file, check the latest release of the GitHub Action by running `gh release list --repo volter-ai/runhuman-action --limit 1` or checking `https://github.com/volter-ai/runhuman-action/releases`. Use the latest version tag rather than hardcoding an outdated version.

**Using the GitHub Action (recommended for CI/CD):**

```yaml
# .github/workflows/qa.yml
- name: Run QA tests
  uses: volter-ai/runhuman-action@v1  # Check latest version first
  with:
    url: ${{ env.PREVIEW_URL }}
    pr-numbers: '[${{ github.event.pull_request.number }}]'
    api-key: ${{ secrets.RUNHUMAN_API_KEY }}
    on-success-add-labels: '["qa:passed"]'
    on-failure-add-labels: '["qa:failed"]'
    fail-on-failure: true
```

**Code Context for PR-triggered tests:** When the project has a repo connected, pass the four `code-context` inputs together so test results reflect the code at the head of the PR rather than the default branch.

```yaml
- uses: volter-ai/runhuman-action@v1
  with:
    url: ${{ env.PREVIEW_URL }}
    pr-numbers: '[${{ github.event.pull_request.number }}]'
    api-key: ${{ secrets.RUNHUMAN_API_KEY }}
    enable-code-context: true
    commit-sha: ${{ github.event.pull_request.head.sha }}
    commit-base-sha: ${{ github.event.pull_request.base.sha }}
    wait-for-code-context: true
```

See the [Code Context docs](https://runhuman.com/docs/code-context) for what the inputs do and what to expect in extracted issues.

**Using the CLI in a workflow:**

```bash
# Environment variable auth (for CI)
export RUNHUMAN_API_KEY=rh_live_...

# Synchronous test that blocks until complete
runhuman job create https://staging.myapp.com \
  -d "Test signup flow" \
  --sync \
  --wait 300
```

All commands support `--json` for machine-readable output. JSON responses use a stable envelope: `{ success, data, pagination?, warnings?, timestamp }` on success; `{ success: false, error: { message, code?, details? }, timestamp }` on failure. List commands put items at `data.items`; single-entity commands put the entity at `data`.

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | General error |
| `2` | Authentication error |
| `3` | Not found |
| `4` | Validation error |
| `5` | Timeout |
| `6` | Insufficient balance (add funds at the link in the error output) |

## Learning More

```bash
# Top-level help
runhuman --help

# Help for any command
runhuman job create --help
runhuman projects --help
runhuman orgs invite --help
runhuman github installation refresh --help
```

## Resources

- **Website:** https://runhuman.com
- **Dashboard:** https://runhuman.com/dashboard
- **GitHub Action:** https://github.com/marketplace/actions/runhuman-qa-test
- **GitHub Actions Setup Guide (for agents):** https://runhuman.com/for_agents_github_actions.md - Fetch this when users ask about GitHub Actions, CI/CD pipelines, or automating QA.

---
> Source: [volter-ai/runhuman-skills](https://github.com/volter-ai/runhuman-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
