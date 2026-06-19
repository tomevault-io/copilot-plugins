## claude-code-pr-review-respond

> Respond to all code review comments on a pull request, implement fixes, run tests, commit, reply on GitHub (in Chinese), and resolve conversations. Supports multi-round review loops with AI review bots. Use when the user says \"处理审查意见\", \"回复 review\", \"resolve PR comments\", \"修复审查意见\", \"处理PR评论\", \"review feedback\", \"address comments\", \"reply to review\", or similar phrases.


# /pr-review-respond

Handle all code review comments on a GitHub PR: categorize into fixed / needs-fix / wontfix, implement fixes, test, commit, push, reply in Chinese, and resolve all threads. Supports multi-round review loops to handle AI review bot feedback on pushed changes.

## When to use

- A PR has pending review comments that need to be addressed
- The user says "处理审查意见", "回复 review", "resolve PR comments", etc.
- Review feedback from any review tool (codereviewbot, human reviewers, etc.)

## When NOT to use

- The PR has no review comments yet — nothing to respond to
- You don't have write access to the repository
- The review comments require discussion with the original author before acting
- The skill is being invoked on a fork without upstream write permissions
- The PR is merged, closed, or in draft state
- The current branch is the repository's default branch
- The `gh` CLI is not installed or not authenticated

## Usage

```
/pr-review-respond <owner/repo> <PR number>
```

If no arguments given, detect from `git remote get-url origin` and the current branch's associated PR:

```bash
# Auto-detect PR number from current branch
gh pr view --json number --jq '.number'
```

## Workflow

### Pre-Step — Validate environment and initialize session state

Before starting, run these validation checks:

1. **gh CLI check**: Run `gh --version`. If it fails, abort with "错误：未安装 GitHub CLI（gh），请先安装。"
2. **Auth check**: Run `gh auth status`. If it fails, abort with "错误：GitHub CLI 未登录，请先运行 gh auth login。"
3. **PR state check**: Run:
   ```bash
   gh pr view <PR> --json state,isDraft,merged,headRefName,baseRefName,headRefOid
   ```
   - If `state == "CLOSED"` or `merged == true`: abort with "PR #{PR} 已关闭/合并，无需处理审查意见。"
   - If `isDraft == true`: abort with "PR #{PR} 为草稿状态，请先将其标记为 Ready for Review。"
   - Save `headRefName`, `baseRefName`, `headRefOid` as `pr_metadata`.
4. **Default branch check**: Run:
   ```bash
   gh repo view <owner/repo> --json defaultBranch --jq .defaultBranch
   ```
   If current branch equals default branch: abort with "错误：当前分支为仓库的默认分支（{default_branch}），禁止直接在该分支上处理审查意见。"
5. **Fetch latest base**: `git fetch origin <baseRefName>`

Then initialize session state:
- `round = 1`
- `max_rounds = 3` (configurable via environment variable `PR_REVIEW_MAX_ROUNDS`)
- `processed_thread_ids = []` — set of thread IDs already replied to and resolved in prior rounds
- `round_summary = []` — structured log of what each round accomplished

### Loop body: Steps 1–4

The four steps below form the body of the multi-round loop. After completing Step 4, a decision is made whether to continue to the next round.

### Step 1 — Fetch all review comments

Use `gh api` with retry logic for all calls below. For each API call:
1. Execute the command
2. If it fails with `rate limit` / `403` / `429`: wait 60 seconds and retry
3. For other errors: retry up to `PR_REVIEW_RETRY_COUNT` times (default 3) with 1-second backoff
4. If all retries fail: report the error and abort

Fetch all pull request review comments via REST:

```bash
comments_json=$(gh api repos/<owner>/<repo>/pulls/<PR>/comments --paginate \
  --jq '.[] | {id, body: .body[0:100], path, commit_id, user: .user.login, created_at, in_reply_to_id, node_id}')
```

Validate output: if `comments_json` is empty or `null`, report "PR #{PR} 没有待处理的审查评论。" and exit.

Also fetch review threads via GraphQL with cursor-based pagination:

```bash
# Initialize
all_threads="[]"
cursor="null"
has_next_page="true"

while [ "$has_next_page" = "true" ]; do
  page=$(gh api graphql -f query='
  query($owner:String!,$repo:String!,$pr:Int!,$after:String) {
    repository(owner:$owner,name:$repo) {
      pullRequest(number:$pr) {
        reviewThreads(first:100, after:$after) {
          pageInfo { hasNextPage endCursor }
          nodes { id isResolved comments(first:1) { nodes { id } } }
        }
      }
    }
  }' -F owner=<owner> -F repo=<repo> -F pr=<PR> ${cursor:+-F after="$cursor"})

  page_nodes=$(echo "$page" | jq '.data.repository.pullRequest.reviewThreads.nodes')
  all_threads=$(echo "$all_threads" | jq --argjson nodes "$page_nodes" '. + $nodes')
  has_next_page=$(echo "$page" | jq -r '.data.repository.pullRequest.reviewThreads.pageInfo.hasNextPage')
  cursor=$(echo "$page" | jq -r '.data.repository.pullRequest.reviewThreads.pageInfo.endCursor')
done
```

After each successful API call, check remaining rate limit:
```bash
remaining=$(gh api rate_limit --jq '.rate.remaining' 2>/dev/null)
if [ "$remaining" -lt 100 ]; then
  echo "警告：GitHub API 速率限制剩余 ${remaining} 次。"
fi
```

**Thread-to-comment ID mapping**: The REST API returns comments with a numeric `id` (e.g., `12345`) and a globally unique `node_id` (e.g., `MDI0OlB1bGxSZXF1ZXN0UmV2aWV3Q29tbWVudDEyMzQ1`). The GraphQL API returns threads with an `id` field that corresponds to the REST comment's `node_id`. Mapping: `rest_comment.node_id === graphql_thread.id`.

- Use the numeric `id` in commit messages (`Addresses review comments: #12345`)
- Use the GraphQL thread `id` for reply and resolve mutations
- Cross-reference by matching `node_id` from REST with `id` from GraphQL

If this is round 2 or later, filter the fetched threads:

1. Remove any thread whose `id` is already in `processed_thread_ids`
2. Remove any thread whose `isResolved` is true (resolved externally)
3. The remaining threads are "new unresolved threads" for this round

**Outdated diff detection**: For each remaining comment, compare its `commit_id` against `pr_metadata.headRefOid`. If they differ, the comment targets an older version of the file. For such comments:
- Read the current file content at the relevant lines
- If the issue still exists in the current code: treat as "needs fix" but note the file may have shifted
- If the issue no longer applies (resolved by subsequent changes): classify as "already fixed"

Map each remaining comment's `node_id` to its GitHub comment `id` (numeric) for cross-referencing in commit messages.

### Step 2 — Categorize each comment

For each top-level comment (not a reply):

| Category | Criteria | Round 1 Action | Round 2+ Action |
|----------|----------|----------------|-----------------|
| **Already fixed** | The fix exists in a prior commit (pre-existing or from a prior round) | Reply explaining which commit, then resolve | Reply with "已在第N轮修复：..." referencing the prior round's commit SHA |
| **Needs fix** | Valid issue, code change required | Fix → test → commit → push → reply → resolve | Same as round 1 |
| **Wontfix** | Invalid, out of scope, or design choice | Reply with rationale, then resolve | Same as round 1 |

For round 2+, when checking "already fixed": read the relevant file at the relevant lines. If the code already reflects the fix (i.e., the fix was applied in a prior round's commit), classify as "already fixed" and reference the prior round's commit SHA.

If after categorization there are zero "needs fix" and zero "already fixed" comments, skip Step 3 (no code changes needed) and proceed to Step 4 to reply to and resolve wontfix threads.

### Step 3 — Implement fixes

For each "needs fix" comment:

1. Read the relevant file(s)
2. Make the minimal code change
3. Run the project's test suite. Auto-detect the test runner using this
   priority-ordered detection table. Scan in order and use the first match:

   | Priority | Detection Signal | Command |
   |----------|------------------|---------|
   | 1 | `package.json` has `"scripts": { "test": "..." }` | the script command |
   | 2 | `pnpm-lock.yaml` exists | `pnpm test` |
   | 3 | `yarn.lock` exists | `yarn test` |
   | 4 | `pyproject.toml` with `[tool.pytest.ini_options]` | `pytest` |
   | 5 | `uv.lock` exists | `uv run pytest` |
   | 6 | `poetry.lock` exists | `poetry run pytest` |
   | 7 | `tox.ini` or `pyproject.toml` with `[tool.tox]` | `tox` |
   | 8 | `requirements*.txt` containing `pytest` | `pytest` |
   | 9 | `*.csproj` or `*.sln` exists | `dotnet test` |
   | 10 | `pom.xml` exists | `mvn test` |
   | 11 | `build.gradle` or `build.gradle.kts` exists | `gradle test` |
   | 12 | `Cargo.toml` exists | `cargo test` |
   | 13 | `go.mod` exists | `go test ./...` |
   | 14 | `Gemfile` exists | `bundle exec rspec` |
   | 15 | `mix.exs` exists | `mix test` |
   | 16 | `Makefile` with a `test` target | `make test` |
   | 17 | Nothing matched | Ask the user: "无法自动检测测试命令，请提供测试命令。" |
4. Commit with a descriptive message following the format in "Commit message style".
   The "Addresses review comments:" line **must** include the comment ID(s) from each
   review comment addressed by this commit. Example:
   ```
   fix(logger): prevent stale file writer accumulation

   Addresses review comments: #12345, #12346

   Clean up writers keyed by dates other than today to avoid
   resource leaks in long-running processes.

   Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
   ```
   If one commit addresses multiple review comments, list all their IDs comma-separated.
5. **Rebase safety check**: Before pushing, check if the PR branch has fallen behind its base:
   ```bash
   behind=$(git rev-list --count HEAD..origin/<baseRefName> 2>/dev/null)
   if [ "$behind" -gt 0 ]; then
     echo "警告：当前分支落后目标分支 ${behind} 个提交，建议先变基。"
   fi
   ```
   If the branch is behind, ask the user whether to rebase before pushing.
6. **User confirmation**: Before the first push in round 1, present a summary:
   "即将修复 {N} 条评论，创建 {M} 个提交并推送到 {headRefName}。是否继续？(Y/n)"
   Skip this confirmation if `PR_REVIEW_SKIP_CONFIRM=true`.
7. **Push**:
   ```bash
   git push origin <headRefName>
   ```
8. **Push failure diagnosis**: If push fails, inspect the error message:
   - Contains `non-fast-forward`: suggest `git pull --rebase origin <baseRefName>` and retry
   - Contains `protected branch` or `branch protection`: report the rule violation and stop
   - Contains `no upstream`: suggest `git push -u origin <headRefName>` and retry
   - Other errors: report the raw error and stop

Group related fixes into sensible commits. Each commit should fix a cohesive set of issues and list all comment IDs it addresses.

### Step 4 — Reply to each comment on GitHub and resolve

For each review thread, reply individually using GraphQL. Use a heredoc with
placeholder replacement to avoid shell escaping issues with Chinese text:

```bash
# Write mutation to temp file (quoted 'GRAPHQL' prevents shell expansion)
cat > /tmp/pr_reply_$$.gql << 'GRAPHQL'
mutation {
  addPullRequestReviewThreadReply(input: {
    pullRequestReviewThreadId: "THREAD_ID_PLACEHOLDER",
    body: "BODY_PLACEHOLDER"
  }) {
    comment { id body }
  }
}
GRAPHQL

# Replace placeholders safely using Python (avoids sed delimiter conflicts)
python3 -c "
import sys
with open('/tmp/pr_reply_$$.gql', 'r') as f:
    q = f.read()
q = q.replace('THREAD_ID_PLACEHOLDER', sys.argv[1])
q = q.replace('BODY_PLACEHOLDER', sys.argv[2])
with open('/tmp/pr_reply_$$.gql', 'w') as f:
    f.write(q)
" "$thread_id" "$reply_body"

# Execute and clean up
gh api graphql --input /tmp/pr_reply_$$.gql
rm -f /tmp/pr_reply_$$.gql
```

**Reply format rules:**
- **Already fixed (first round)**: "已修复：<what was done>。Commit: <short SHA>（文件：<path>，对应评论 #id）"
- **Already fixed (subsequent rounds)**: "已在第N轮修复：<what was done>。Commit: <short SHA from prior round>（文件：<path>，对应评论 #id）"
- **Wontfix**: "不采纳：<reason>"
- Always mention the commit SHA (short form, 7 chars) if a fix was made
- Always append file path and comment ID reference: "（文件：<path>，对应评论 #id）"
- Always reply in Chinese (中文)
- Keep replies concise — one to two sentences
- **DO NOT add a general summary comment at the PR level** — reply only under each specific comment thread

After replying to a thread, resolve it immediately via GraphQL (same temp-file pattern):

```bash
cat > /tmp/pr_resolve_$$.gql << 'GRAPHQL'
mutation {
  resolveReviewThread(input: { threadId: "THREAD_ID_PLACEHOLDER" }) {
    thread { id }
  }
}
GRAPHQL

python3 -c "
import sys
with open('/tmp/pr_resolve_$$.gql', 'r') as f:
    q = f.read()
q = q.replace('THREAD_ID_PLACEHOLDER', sys.argv[1])
with open('/tmp/pr_resolve_$$.gql', 'w') as f:
    f.write(q)
" "$thread_id"

gh api graphql --input /tmp/pr_resolve_$$.gql
rm -f /tmp/pr_resolve_$$.gql
```

Process one thread at a time: reply → resolve → next thread. Do NOT batch resolve all threads at the end.

If `resolveReviewThread` returns an error (e.g., the thread was already resolved externally), note it and continue to the next thread — no action needed.

### After processing this round's threads

1. For every thread replied to in this round, add its `thread.id` to `processed_thread_ids`
2. Record in `round_summary`:
   - Round number
   - All commit SHAs created in this round
   - Mapping of comment_id → commit_sha for each fix
   - List of wontfix comment IDs

### Loop exit / continue decision

After completing the loop body (Steps 1–4), decide whether to continue to the next round:

1. If `round >= max_rounds`: exit. Report "已达到最大轮次（{max_rounds}轮）。"
2. If zero commits were pushed this round (only wontfix replies): exit. No new code to review.
3. Otherwise, wait for AI review bot:
   a. Report: "等待AI审查机器人重新审查...（等待{interval}秒）"
   b. Sleep for `PR_REVIEW_POLL_INTERVAL` seconds (default 45, configurable via env var)
   c. Fetch fresh thread list via the same GraphQL query from Step 1
   d. Identify unresolved threads whose IDs are NOT in `processed_thread_ids`
   e. If none found: "未发现新的审查评论，退出循环。"
   f. If found: "发现 {N} 条新的审查评论，进入第 {round+1} 轮处理。" Increment `round` and go to Step 1.

### Step 5 — Summary report

After the loop exits, compile and display a summary:

- **总完成轮次**: N
- **总提交数**: N
- **已修复的评论**: list of #comment_id → commit_sha mappings
- **不采纳的评论**: list of #comment_id
- **未处理的评论**: if any threads remain unresolved due to max_rounds, list them with a note suggesting the user re-run the skill or handle manually

### Step 6 — Update project documentation

If the project maintains a `CLAUDE.md` or similar convention file, follow its documentation update procedures. Common patterns include:

- `docs/change-log.md` — append fix summary to today's entry
- `docs/progress.md` — update recent progress and today's entry
- `CHANGELOG.md` — add entry under Unreleased

Read the relevant files first, then edit only the sections that need updating. If no documentation conventions exist, skip this step.

## Commit message style

Use this format:

```
<type>(<scope>): <short description in imperative mood>

Addresses review comments: #comment_id1, #comment_id2

<optional body explaining WHY>

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
```

The "Addresses review comments:" line is **required** for any commit that fixes review feedback. It lists the GitHub comment IDs addressed by this commit, separated by commas.

Types: `fix` (default), `test`, `refactor`, `chore`

## Key rules

1. **Minimal changes** — fix only what the review comment asks for, no extra refactoring
2. **Test before commit** — always run relevant tests, confirm pass
3. **Chinese replies** — all GitHub comment replies must be in Chinese
4. **Group related fixes, annotate comment IDs** — combine related fixes into sensible commits (don't create a commit per comment). Each commit's message MUST include an "Addresses review comments:" line listing the comment ID(s) it addresses.
5. **Reply under each specific comment** — DO NOT add a general summary comment at the PR level. Reply individually under each review thread using `addPullRequestReviewThreadReply`.
6. **Reply then resolve immediately** — process one thread at a time: post reply, then immediately resolve that thread. Do NOT batch resolve all threads at the end.
7. **Multi-round loop** — after resolving all threads in a round, wait for the AI review bot to post new comments (45s default, configurable via `PR_REVIEW_POLL_INTERVAL`). Process up to `max_rounds` rounds (default 3). If no new unresolved threads appear, exit early. Log a summary of all rounds at the end.
8. **Bidirectional referencing** — every fix commit message must reference the comment IDs it addresses (via "Addresses review comments:" line). Every reply about a fix must reference the commit SHA, file path, and comment ID. This creates a complete traceability chain: comment ↔ commit ↔ reply.
9. **Don't push without testing** — if tests fail, fix the issue and re-test
10. **Resolve ALL threads** — even wontfix threads must be resolved after replying
11. **Pre-flight validation** — Always verify `gh` CLI, auth, PR state (not merged/closed/draft), and branch safety before starting
12. **No main branch pushes** — Never push directly to the repository's default branch
13. **Outdated diff awareness** — Flag comments on old `commit_id` values; verify the issue still applies to the current file state
14. **Confirm before destructive operations** — Ask the user before pushing commits or making significant changes (override with `PR_REVIEW_SKIP_CONFIRM=true`)

## Environment variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PR_REVIEW_POLL_INTERVAL` | `45` | Seconds to wait for AI review bot to post new comments between rounds |
| `PR_REVIEW_MAX_ROUNDS` | `3` | Maximum number of review rounds before stopping |
| `PR_REVIEW_SKIP_CONFIRM` | `false` | Set to `true` to skip user confirmation prompts for automated environments |
| `PR_REVIEW_RETRY_COUNT` | `3` | Number of retry attempts for transient `gh api` failures |

## References

- GitHub API endpoints: see [`references/github-api.md`](references/github-api.md)
- Commit conventions: see [`references/commit-conventions.md`](references/commit-conventions.md)
- Project-specific fix patterns: see [`examples/`](examples/)

---
> Source: [kingside33/claude-code-pr-review-respond](https://github.com/kingside33/claude-code-pr-review-respond) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
