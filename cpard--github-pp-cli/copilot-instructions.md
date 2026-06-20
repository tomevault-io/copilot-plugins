## github-pp-cli

> Every gh command, plus a local store, cross-org rollups, and FTS no other GitHub tool ships. Trigger phrases: `what's open across my org on github`, `cross-repo pr review load`, `github flake rank`, `release diff between tags`, `bottlenecks in github`, `use github-pp`, `run github-pp`.


# GitHub — Printing Press CLI

## Prerequisites: Install the CLI

This skill drives the `github-pp-cli` binary. **You must verify the CLI is installed before invoking any command from this skill.** If it is missing, install it first:

1. Install via the Printing Press installer:
   ```bash
   npx -y @mvanhorn/printing-press install github --cli-only
   ```
2. Verify: `github-pp-cli --version`
3. Ensure `$GOPATH/bin` (or `$HOME/go/bin`) is on `$PATH`.

If the `npx` install fails (no Node, offline, etc.), fall back to a direct Go install (requires Go 1.26.3 or newer):

```bash
go install github.com/mvanhorn/printing-press-library/library/developer-tools/github/cmd/github-pp-cli@latest
```

If `--version` reports "command not found" after install, the install step did not put the binary on `$PATH`. Do not proceed with skill commands until verification succeeds.

github-pp-cli matches `gh` surface for surface and adds a sync engine + SQLite store so you can answer cross-repo questions in one query — `recent`, `bottlenecks`, `flake-rank`, `release-diff`, `reviewer-load`, `velocity`, `who-broke-main` — plus FTS that bypasses GitHub's 1000-result cap and a rate-limit-aware paginate that doesn't cliff-fail. Every command is agent-native by default: `--json`, `--select`, typed exit codes, `--dry-run` on every mutation.

## When to Use This CLI

Reach for github-pp-cli when the question is cross-repo or cross-org, when you need a structured + filtered answer (every command takes `--json --select`), when you want to grep beyond GitHub's 1000-result search cap, or when you want typed exit codes for orchestration. Use `gh` for one-shot interactive flows like `gh pr create` and `gh repo clone` — github-pp-cli wraps both surfaces but `gh`'s human ergonomics for those are world-class.

## Unique Capabilities

These capabilities aren't available in any other tool for this API.

### Cross-org rollups
- **`reviewer-load`** — See review load (given, received, pending) per teammate over a window — for 1:1 prep, perf cycles, and cross-squad rollups.

  _Reach for this when you need to know who's overloaded with reviews — `gh` cannot answer this without N round trips, and the 1000-result search cap breaks it on big orgs._

  ```bash
  github-pp-cli reviewer-load --org acme --since 30d --json --select user,reviews_given,reviews_pending,median_ttr_hours
  ```
- **`recent`** — One command, all repos: PRs merged, issues closed, releases cut, runs failed since a watermark.

  _Reach for this for daily / weekly cross-repo standup rollups instead of opening N tabs of `gh` invocations._

  ```bash
  github-pp-cli recent --since 24h --org acme --json --select kind,repo,number,title,actor
  ```
- **`velocity`** — Per-repo throughput: PRs/day merged, median PR lifetime, p50 time-to-first-review, median CI duration.

  _Reach for this for engineering health checks and rollups instead of pasting `gh pr list --json` into a spreadsheet._

  ```bash
  github-pp-cli velocity --org acme --window 30d --json --select repo,merged_per_day,median_lifetime_h,p50_ttfr_h
  ```

### Local store leverage
- **`fts`** — Full-text search across every cached issue, PR, commit, and release across every synced repo — bypasses GitHub's 1000-result cap.

  _Use this when GitHub's search would silently truncate at 1000 hits or when you need to grep comments + bodies + commit messages in one shot._

  ```bash
  github-pp-cli fts "flaky test" --type prs,issues --since 90d --json --select repo,number,title,updated_at
  ```
- **`bottlenecks`** — Surfaces PRs open >N days awaiting review, PRs red >24h, issues without a triage label after 48h.

  _Use this for Friday triage or sprint hygiene — one command instead of six tabs._

  ```bash
  github-pp-cli bottlenecks --org acme --review-stale 7d --check-stale 24h --json --select kind,repo,number,reason,age_hours
  ```

### CI / release forensics
- **`flake-rank`** — Per-job success rate over the last N runs of a workflow, ranked by flakiness with mean duration and last failure.

  _Use this to triage 'is this job flaking for the team or just on my PR' — `gh run list` cannot rank._

  ```bash
  github-pp-cli flake-rank --workflow ci.yml --repo acme/web --since 50runs --json --select job,success_rate,mean_duration_s,last_failure
  ```
- **`release-diff`** — Show PRs merged, issues closed, contributors, and files changed between two tags in one repo.

  _Reach for this to draft release notes or audit what shipped between two tags without walking `git log` and pasting SHAs into `gh pr view`._

  ```bash
  github-pp-cli release-diff v2.14.0 v2.15.0 --repo acme/web --json --select prs,authors,files_changed
  ```
- **`who-broke-main`** — For each red main-branch run, the merge commit, PR, author, and elapsed-red duration.

  _Use this for incident retros and deploy postmortems instead of scrolling `gh run list` and clicking through._

  ```bash
  github-pp-cli who-broke-main acme/web --since 7d --json --select run_id,pr,author,started_at,red_duration_s
  ```

### Agent-native plumbing
- **`pr ship-check`** — One-shot rollup of required reviewers, branch-protection rules, all check runs, merge queue position, and conflicts. Typed exit (0=mergeable, 2=blocked, 3=conflicted).

  _Agents reach for this to decide whether to merge; humans reach for it to debug 'why won't this PR merge'._

  ```bash
  github-pp-cli pr ship-check acme/web#1234 --json --select mergeable,blockers,required_reviewers,checks
  ```

## Command Reference

**advisories** — Manage advisories

- `github-pp-cli advisories security-get-global-advisory` — Gets a global security advisory using its GitHub Security Advisory (GHSA) identifier.
- `github-pp-cli advisories security-list-global` — Lists all global security advisories that match the specified parameters. If no other parameters are defined, the...

**agents** — Endpoints for Agents secrets and variables.

- `github-pp-cli agents tasks-create-task-in-repo` — > [!NOTE] > This endpoint is in public preview and is subject to change. Starts a new Copilot cloud agent task for a...
- `github-pp-cli agents tasks-get-task-by-id` — > [!NOTE] > This endpoint is in public preview and is subject to change. Returns a task by ID with its associated...
- `github-pp-cli agents tasks-get-task-by-repo-and-id` — > [!NOTE] > This endpoint is in public preview and is subject to change. Returns a task by ID scoped to an...
- `github-pp-cli agents tasks-list-tasks` — > [!NOTE] > This endpoint is in public preview and is subject to change. Returns a list of tasks for the...
- `github-pp-cli agents tasks-list-tasks-for-repo` — > [!NOTE] > This endpoint is in public preview and is subject to change. Returns a list of tasks for a specific...

**app** — Information for integrations and installations.

- `github-pp-cli app create-installation-access-token` — Creates an installation access token that enables a GitHub App to make authenticated API requests for the app's...
- `github-pp-cli app delete-installation` — Uninstalls a GitHub App on a user, organization, or enterprise account. If you prefer to temporarily suspend an...
- `github-pp-cli app get-authenticated` — Returns the GitHub App associated with the authentication credentials used. To see how many app installations are...
- `github-pp-cli app get-installation` — Enables an authenticated GitHub App to find an installation's information using the installation id. You must use a...
- `github-pp-cli app get-webhook-config-for` — Returns the webhook configuration for a GitHub App. For more information about configuring a webhook for your app,...
- `github-pp-cli app get-webhook-delivery` — Returns a delivery for the webhook configured for a GitHub App. You must use a...
- `github-pp-cli app list-installation-requests-for-authenticated` — Lists all the pending installation requests for the authenticated GitHub App.
- `github-pp-cli app list-installations` — The permissions the installation has are included under the `permissions` key. You must use a...
- `github-pp-cli app list-webhook-deliveries` — Returns a list of webhook deliveries for the webhook configured for a GitHub App. You must use a...
- `github-pp-cli app suspend-installation` — Suspends a GitHub App on a user, organization, or enterprise account, which blocks the app from accessing the...
- `github-pp-cli app unsuspend-installation` — Removes a GitHub App installation suspension. You must use a [JWT](https://docs.github.com/apps/building-github-apps/...
- `github-pp-cli app update-webhook-config-for` — Updates the webhook configuration for a GitHub App. For more information about configuring a webhook for your app,...

**app-manifests** — Manage app manifests


**applications** — Manage applications


**apps** — Information for integrations and installations.

- `github-pp-cli apps <app_slug>` — > [!NOTE] > The `:app_slug` is just the URL-friendly name of your GitHub App. You can find this on the settings page...

**assignments** — Manage assignments

- `github-pp-cli assignments <assignment_id>` — Gets a GitHub Classroom assignment. Assignment will only be returned if the current user is an administrator of the...

**classrooms** — Interact with GitHub Classroom.

- `github-pp-cli classrooms get-a` — Gets a GitHub Classroom classroom for the current user. Classroom will only be returned if the current user is an...
- `github-pp-cli classrooms list` — Lists GitHub Classroom classrooms for the current user. Classrooms will only be returned if the current user is an...

**codes-of-conduct** — Insight into codes of conduct for your communities.

- `github-pp-cli codes-of-conduct get-all` — Returns array of all GitHub's codes of conduct.
- `github-pp-cli codes-of-conduct get-conduct-code` — Returns information about the specified GitHub code of conduct.

**credentials** — Revoke compromised or leaked GitHub credentials.

- `github-pp-cli credentials` — Submit a list of credentials to be revoked. This endpoint is intended to revoke credentials the caller does not own...

**emojis** — List emojis available to use on GitHub.

- `github-pp-cli emojis` — Lists all the emojis available to use on GitHub.

**enterprises** — Manage enterprises


**events** — Manage events

- `github-pp-cli events` — > [!NOTE] > This API is not built to serve real-time use cases. Depending on the time of day, event latency can be...

**feeds** — Manage feeds

- `github-pp-cli feeds` — Lists the feeds available to the authenticated user. The response provides a URL for each feed. You can then get a...

**gists** — View, modify your gists.

- `github-pp-cli gists create` — Allows you to add a new gist with one or more files. > [!NOTE] > Don't name your files 'gistfile' with a numerical...
- `github-pp-cli gists delete` — Delete a gist
- `github-pp-cli gists get` — Gets a specified gist. This endpoint supports the following custom media types. For more information, see '[Media...
- `github-pp-cli gists get-revision` — Gets a specified gist revision. This endpoint supports the following custom media types. For more information, see...
- `github-pp-cli gists list` — Lists the authenticated user's gists or if called anonymously, this endpoint returns all public gists:
- `github-pp-cli gists list-public` — List public gists sorted by most recently updated to least recently updated. Note: With...
- `github-pp-cli gists list-starred` — List the authenticated user's starred gists:
- `github-pp-cli gists update` — Allows you to update a gist's description and to update, delete, or rename gist files. Files from the previous...

**github-search** — Manage github search

- `github-pp-cli github-search code` — Searches for query terms inside of a file. This method returns up to 100 results [per...
- `github-pp-cli github-search commits` — Find commits via various criteria on the default branch (usually `main`). This method returns up to 100 results [per...
- `github-pp-cli github-search issues-and-pull-requests` — Find issues by state and keyword. This method returns up to 100 results [per...
- `github-pp-cli github-search labels` — Find labels in a repository with names or descriptions that match search keywords. Returns up to 100 results [per...
- `github-pp-cli github-search repos` — Find repositories via various criteria. This method returns up to 100 results [per...
- `github-pp-cli github-search topics` — Find topics via various criteria. Results are sorted by best match. This method returns up to 100 results [per...
- `github-pp-cli github-search users` — Find users via various criteria. This method returns up to 100 results [per...

**gitignore** — View gitignore templates

- `github-pp-cli gitignore get-all-templates` — List all templates available to pass as an option when [creating a...
- `github-pp-cli gitignore get-template` — Get the content of a gitignore template. This endpoint supports the following custom media types. For more...

**installation** — Manage installation

- `github-pp-cli installation apps-list-repos-accessible-to` — List repositories that an app installation can access.
- `github-pp-cli installation apps-revoke-access-token` — Revokes the installation token you're using to authenticate as an installation and access this endpoint. Once an...

**issues** — Interact with GitHub Issues.

- `github-pp-cli issues` — List issues assigned to the authenticated user across all visible repositories including owned repositories, member...

**licenses** — View various OSS licenses.

- `github-pp-cli licenses get` — Gets information about a specific license. For more information, see '[Licensing a repository...
- `github-pp-cli licenses get-all-commonly-used` — Lists the most commonly used licenses on GitHub. For more information, see '[Licensing a repository...

**markdown** — Render GitHub flavored Markdown

- `github-pp-cli markdown render` — Depending on what is rendered in the Markdown, you may need to provide additional token scopes for labels, such as...
- `github-pp-cli markdown render-raw` — You must send Markdown as plain text (using a `Content-Type` header of `text/plain` or `text/x-markdown`) to this...

**marketplace-listing** — Manage marketplace listing

- `github-pp-cli marketplace-listing apps-get-subscription-plan-for-account` — Shows whether the user or organization account actively subscribes to a plan listed by the authenticated GitHub App....
- `github-pp-cli marketplace-listing apps-get-subscription-plan-for-account-stubbed` — Shows whether the user or organization account actively subscribes to a plan listed by the authenticated GitHub App....
- `github-pp-cli marketplace-listing apps-list-accounts-for-plan` — Returns user and organization accounts associated with the specified plan, including free plans. For per-seat...
- `github-pp-cli marketplace-listing apps-list-accounts-for-plan-stubbed` — Returns repository and organization accounts associated with the specified plan, including free plans. For per-seat...
- `github-pp-cli marketplace-listing apps-list-plans` — Lists all plans that are part of your GitHub Marketplace listing. GitHub Apps must use a...
- `github-pp-cli marketplace-listing apps-list-plans-stubbed` — Lists all plans that are part of your GitHub Marketplace listing. GitHub Apps must use a...

**meta** — Endpoints that give information about the API.

- `github-pp-cli meta` — Returns meta information about GitHub, including a list of GitHub's IP addresses. For more information, see '[About...

**networks** — Manage networks


**notifications** — Manage notifications

- `github-pp-cli notifications activity-delete-thread-subscription` — Mutes all future notifications for a conversation until you comment on the thread or get an **@mention**. If you are...
- `github-pp-cli notifications activity-get-thread` — Gets information about a notification thread.
- `github-pp-cli notifications activity-get-thread-subscription-for-authenticated-user` — This checks to see if the current user is subscribed to a thread. You can also [get a repository...
- `github-pp-cli notifications activity-list-for-authenticated-user` — List all notifications for the current user, sorted by most recently updated.
- `github-pp-cli notifications activity-mark-as-read` — Marks all notifications as 'read' for the current user. If the number of notifications is too large to complete in...
- `github-pp-cli notifications activity-mark-thread-as-done` — Marks a thread as 'done.' Marking a thread as 'done' is equivalent to marking a notification in your notification...
- `github-pp-cli notifications activity-mark-thread-as-read` — Marks a thread as 'read.' Marking a thread as 'read' is equivalent to clicking a notification in your notification...
- `github-pp-cli notifications activity-set-thread-subscription` — If you are watching a repository, you receive notifications for all threads by default. Use this endpoint to ignore...

**octocat** — Manage octocat

- `github-pp-cli octocat` — Get the octocat as ASCII art

**organizations** — Manage organizations

- `github-pp-cli organizations` — Lists all organizations, in the order that they were created. > [!NOTE] > Pagination is powered exclusively by the...

**orgs** — Interact with organizations.

- `github-pp-cli orgs delete` — Deletes an organization and all its repositories. The organization login will be unavailable for 90 days after...
- `github-pp-cli orgs enable-or-disable-security-product-on-all-repos` — > [!WARNING] > **Closing down notice:** The ability to enable or disable a security feature for all eligible...
- `github-pp-cli orgs get` — Gets information about an organization. When the value of `two_factor_requirement_enabled` is `true`, the...
- `github-pp-cli orgs update` — > [!WARNING] > **Closing down notice:** GitHub will replace and discontinue...

**rate-limit** — Check your current rate limit status.

- `github-pp-cli rate-limit` — > [!NOTE] > Accessing this endpoint does not count against your REST API rate limit. Some categories of endpoints...

**repos** — Interact with GitHub Repos.

- `github-pp-cli repos delete` — Deleting a repository requires admin access. If an organization owner has configured the organization to prevent...
- `github-pp-cli repos get` — The `parent` and `source` objects are present when the repository is a fork. `parent` is the repository this...
- `github-pp-cli repos update` — **Note**: To edit a repository's topics, use the [Replace all repository...

**repositories** — Manage repositories

- `github-pp-cli repositories` — Lists all public repositories in the order that they were created. Note: - For GitHub Enterprise Server, this...

**teams** — Interact with GitHub Teams.

- `github-pp-cli teams delete-legacy` — > [!WARNING] > **Endpoint closing down notice:** This endpoint route is closing down and will be removed from the...
- `github-pp-cli teams get-legacy` — > [!WARNING] > **Endpoint closing down notice:** This endpoint route is closing down and will be removed from the...
- `github-pp-cli teams update-legacy` — > [!WARNING] > **Endpoint closing down notice:** This endpoint route is closing down and will be removed from the...

**user** — Interact with and view information about users and also current user.

- `github-pp-cli user add-email-for-authenticated` — OAuth app tokens and personal access tokens (classic) need the `user` scope to use this endpoint.
- `github-pp-cli user codespaces-create-for-authenticated` — Creates a new codespace, owned by the authenticated user. This endpoint requires either a `repository_id` OR a...
- `github-pp-cli user codespaces-list-for-authenticated` — Lists the authenticated user's codespaces. OAuth app tokens and personal access tokens (classic) need the...
- `github-pp-cli user create-gpg-key-for-authenticated` — Adds a GPG key to the authenticated user's GitHub account. OAuth app tokens and personal access tokens (classic)...
- `github-pp-cli user delete-email-for-authenticated` — OAuth app tokens and personal access tokens (classic) need the `user` scope to use this endpoint.
- `github-pp-cli user get-authenticated` — OAuth app tokens and personal access tokens (classic) need the `user` scope in order for the response to include...
- `github-pp-cli user list-blocked-by-authenticated` — List the users you've blocked on your personal account.
- `github-pp-cli user list-emails-for-authenticated` — Lists all of your email addresses, and specifies which one is visible to the public. OAuth app tokens and personal...
- `github-pp-cli user list-followed-by-authenticated` — Lists the people who the authenticated user follows.
- `github-pp-cli user list-followers-for-authenticated` — Lists the people following the authenticated user.
- `github-pp-cli user list-gpg-keys-for-authenticated` — Lists the current user's GPG keys. OAuth app tokens and personal access tokens (classic) need the `read:gpg_key`...
- `github-pp-cli user update-authenticated` — **Note:** If your email is set to private and you send an `email` parameter as part of this request to update your...

**users** — Interact with and view information about users and also current user.

- `github-pp-cli users get-by-username` — Provides publicly available information about someone with a GitHub account. If you are requesting information about...
- `github-pp-cli users list` — Lists all users, in the order that they signed up on GitHub. This list includes personal user accounts and...

**versions** — Manage versions

- `github-pp-cli versions` — Get all supported GitHub API versions.

**zen** — Manage zen

- `github-pp-cli zen` — Get a random sentence from the Zen of GitHub


### Finding the right command

When you know what you want to do but not which command does it, ask the CLI directly:

```bash
github-pp-cli which "<capability in your own words>"
```

`which` resolves a natural-language capability query to the best matching command from this CLI's curated feature index. Exit code `0` means at least one match; exit code `2` means no confident match — fall back to `--help` or use a narrower query.

## Recipes


### Friday triage: what's stuck

```bash
github-pp-cli bottlenecks --org acme --review-stale 7d --check-stale 24h --json --select kind,repo,number,reason,age_hours
```

One query for every PR or issue stuck across every synced repo, structured for piping into Slack.

### Draft release notes from tag-to-tag

```bash
github-pp-cli release-diff v2.14.0 v2.15.0 --repo acme/web --json --select prs.number,prs.title,prs.author
```

Joins synced commits × prs × users so release notes don't require pasting SHAs into `gh pr view`.

### Pick the right reviewer

```bash
github-pp-cli reviewer-load --org acme --since 30d --json --select user,reviews_pending,median_ttr_hours
```

Sorted reviewer load — assign the next PR to whoever is freshest, not whoever is loudest.

### Big payload? select fields

```bash
github-pp-cli pr view acme/web#1234 --json --select number,title,reviews.user.login,reviews.state,statusCheckRollup.contexts.name,statusCheckRollup.contexts.conclusion
```

PR responses are huge; `--select` with dotted paths cuts the payload to the four facts you actually need before piping to jq.

### Rank flaky CI jobs

```bash
github-pp-cli flake-rank --workflow ci.yml --repo acme/web --since 50runs --json --select job,success_rate,mean_duration_s
```

Per-job pass rate over the last 50 runs — surfaces the one job that's been red for two weeks before it eats a release.

## Auth Setup

No authentication required.

Run `github-pp-cli doctor` to verify setup.

## Agent Mode

Add `--agent` to any command. Expands to: `--json --compact --no-input --no-color --yes`.

- **Pipeable** — JSON on stdout, errors on stderr
- **Filterable** — `--select` keeps a subset of fields. Dotted paths descend into nested structures; arrays traverse element-wise. Critical for keeping context small on verbose APIs:

  ```bash
  github-pp-cli classrooms list --agent --select id,name,status
  ```
- **Previewable** — `--dry-run` shows the request without sending
- **Offline-friendly** — sync/search commands can use the local SQLite store when available
- **Non-interactive** — never prompts, every input is a flag
- **Explicit retries** — use `--idempotent` only when an already-existing create should count as success, and `--ignore-missing` only when a missing delete target should count as success

### Response envelope

Commands that read from the local store or the API wrap output in a provenance envelope:

```json
{
  "meta": {"source": "live" | "local", "synced_at": "...", "reason": "..."},
  "results": <data>
}
```

Parse `.results` for data and `.meta.source` to know whether it's live or local. A human-readable `N results (live)` summary is printed to stderr only when stdout is a terminal — piped/agent consumers get pure JSON on stdout.

## Agent Feedback

When you (or the agent) notice something off about this CLI, record it:

```
github-pp-cli feedback "the --since flag is inclusive but docs say exclusive"
github-pp-cli feedback --stdin < notes.txt
github-pp-cli feedback list --json --limit 10
```

Entries are stored locally at `~/.github-pp-cli/feedback.jsonl`. They are never POSTed unless `GITHUB_FEEDBACK_ENDPOINT` is set AND either `--send` is passed or `GITHUB_FEEDBACK_AUTO_SEND=true`. Default behavior is local-only.

Write what *surprised* you, not a bug report. Short, specific, one line: that is the part that compounds.

## Output Delivery

Every command accepts `--deliver <sink>`. The output goes to the named sink in addition to (or instead of) stdout, so agents can route command results without hand-piping. Three sinks are supported:

| Sink | Effect |
|------|--------|
| `stdout` | Default; write to stdout only |
| `file:<path>` | Atomically write output to `<path>` (tmp + rename) |
| `webhook:<url>` | POST the output body to the URL (`application/json` or `application/x-ndjson` when `--compact`) |

Unknown schemes are refused with a structured error naming the supported set. Webhook failures return non-zero and log the URL + HTTP status on stderr.

## Named Profiles

A profile is a saved set of flag values, reused across invocations. Use it when a scheduled agent calls the same command every run with the same configuration - HeyGen's "Beacon" pattern.

```
github-pp-cli profile save briefing --json
github-pp-cli --profile briefing classrooms list
github-pp-cli profile list --json
github-pp-cli profile show briefing
github-pp-cli profile delete briefing --yes
```

Explicit flags always win over profile values; profile values win over defaults. `agent-context` lists all available profiles under `available_profiles` so introspecting agents discover them at runtime.

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 2 | Usage error (wrong arguments) |
| 3 | Resource not found |
| 5 | API error (upstream issue) |
| 7 | Rate limited (wait and retry) |
| 10 | Config error |

## Argument Parsing

Parse `$ARGUMENTS`:

1. **Empty, `help`, or `--help`** → show `github-pp-cli --help` output
2. **Starts with `install`** → ends with `mcp` → MCP installation; otherwise → see Prerequisites above
3. **Anything else** → Direct Use (execute as CLI command with `--agent`)

## MCP Server Installation

1. Install the MCP server:
   ```bash
   go install github.com/mvanhorn/printing-press-library/library/developer-tools/github/cmd/github-pp-mcp@latest
   ```
2. Register with Claude Code:
   ```bash
   claude mcp add github-pp-mcp -- github-pp-mcp
   ```
3. Verify: `claude mcp list`

## Direct Use

1. Check if installed: `which github-pp-cli`
   If not found, offer to install (see Prerequisites at the top of this skill).
2. Match the user query to the best command from the Unique Capabilities and Command Reference above.
3. Execute with the `--agent` flag:
   ```bash
   github-pp-cli <command> [subcommand] [args] --agent
   ```
4. If ambiguous, drill into subcommand help: `github-pp-cli <command> --help`.

---
> Source: [cpard/github-pp-cli](https://github.com/cpard/github-pp-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
