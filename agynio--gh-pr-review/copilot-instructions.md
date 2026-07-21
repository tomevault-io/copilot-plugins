## gh-pr-review

> You have a pre-installed `gh pr-review` extension that simplifies working with Pull Requests. Below are the commands for retrieving PR reviews/comments and replying to review comments in a thread, and open and sumbit a review.

## Documentation `gh pr-review`
You have a pre-installed `gh pr-review` extension that simplifies working with Pull Requests. Below are the commands for retrieving PR reviews/comments and replying to review comments in a thread, and open and sumbit a review.

## Quick PR overview with gh
View details: `gh pr-review review view <PR number> -R <owner>/<repo>`

Default scope:
- Includes every reviewer and review state (APPROVED, CHANGES_REQUESTED,
  COMMENTED, DISMISSED).
- Threads are grouped by parent inline comment; replies are sorted by
  `created_at` ascending.
- Optional fields are omitted rather than rendered as `null`.

Useful filters:
- **`--reviewer <login>`** — Limit to a specific reviewer login (case-insensitive).
- **`--states <list>`** — Comma-separated list of review states.
- **`--unresolved`** — Only include unresolved threads.
- **`--not_outdated`** — Drop threads marked as outdated.
- **`--tail <n>`** — Keep the last `n` replies per thread (0 keeps all).

Example capturing the latest actionable work:

```bash
gh pr-review review report 51 -R agyn/repo \
  --reviewer emerson \
  --states CHANGES_REQUESTED,COMMENTED \
  --unresolved \
  --not_outdated \
  --tail 2
```

## Reply to an inline comment
Use the **thread_id** values with `gh pr-review comments reply <PR number>` to continue discussions. 
Example:

```sh
gh pr-review comments reply 51 -R owner/repo \
 --thread-id PRRT_kwDOAAABbcdEFG12 \
 --body "Follow-up addressed in commit abc123" 
```

Note: If you want to leave a high-level comment on a PR that isn’t tied to any specific review thread, you can use gh pr comment. This allows you to add general feedback directly to the pull request.

## Submit Review

1. **Start a pending review.** Use `gh pr-review review --start <PR number> -R <owner>/<repo>`

   ```sh
   gh pr-review review --start 42 -R owner/repo

   {
     "id": "PRR_kwDOAAABbcdEFG12",
     "state": "PENDING"
   }
   ```

2. **Add inline comments with the pending review ID.** The `review --add-comment`requiers `review-id` identifier `PRR_…`. Use `gh pr-review review --add-comment <PR number> -R <owner>/<repo> ...`. Example:

   ```sh
   gh pr-review review --add-comment 42 -R owner/repo \
     --review-id PRR_kwDOAAABbcdEFG12 \
     --path internal/service.go \
     --line 42 \
     --body "nit: use helper"
   {
     "id": "PRRT_kwDOAAABbcdEFG12",
     "path": "internal/service.go",
     "is_outdated": false,
     "line": 42
   }
   ```

3. **Submit the review.** Reuse the pending review `PRR_…`
   identifier when finalizing. Successful submissions emit a status-only
   payload. Errors are returned as structured JSON for
   troubleshooting. Use `gh pr-review review --submit <PR number> -R <owner>/<repo> ...`. Example:

   ```sh
   gh pr-review review --submit 42 -R owner/repo \
     --review-id PRR_kwDOAAABbcdEFG12 \
     --event REQUEST_CHANGES \
     --body "Please add tests"

   {
     "status": "Review submitted successfully"
   }
   ```

Optimization Tips:
- The `gh pr-review review --add-comment` can be executed in batch instead of one by one to optimize performance. Grouping calls where possible is recommended.

## Resolving review comments

To see a list of your unresolved threads, use `gh pr-review review report` with filters `--unresolved` and `--reviewer <your login>`


Mark an inline thread as resolved:
```
gh pr-review threads resolve 42 -R owner/repo --thread-id PRRT_kwDOAAABbcdEFG12

{
  "thread_node_id": "PRRT_kwDOAAABbcdEFG12",
  "is_resolved": true
}
```

---
> Source: [agynio/gh-pr-review](https://github.com/agynio/gh-pr-review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
