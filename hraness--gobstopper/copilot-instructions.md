## gobstopper

> - `crates/gobstopper-core/` holds the normalized transcript model, the `Edit` IR, the `Strategy` trait, all built-in strategies, and the `compaction-events-v1` telemetry schema. No I/O beyond event-log append.

# Contents

- `crates/gobstopper-core/` holds the normalized transcript model, the `Edit` IR, the `Strategy` trait, all built-in strategies, and the `compaction-events-v1` telemetry schema. No I/O beyond event-log append.
- `crates/gobstopper-adapters/` holds session discovery, Codex and Claude Code JSONL parsing and pure byte transforms, separate-copy publication, structural verification, and the `vault` content-addressed snapshot store. Direct provider-file replacement APIs refuse mutation.
- `crates/gobstopper-adapters/src/request/` holds the request-time compaction engine (a port of CliffCompaction's rule; MIT notice in `THIRD_PARTY_NOTICES.md`): Anthropic Messages, OpenAI Responses, and OpenAI Chat Completions dialects, the prefix store, and offline replay. It is pure JSON in, JSON out.
- `crates/gobstopper-cli/` holds the `gobstopper` binary, layered config/preset resolution, the read-only `mcp` stdio server (`mcp.rs`) that exposes vault/recall/plan/verify as agent tools (it must never surface a mutating operation), and the loopback `proxy` (`proxy.rs`).
- `docs/design.md` is the architecture and research record; `docs/roadmap.md` is the phased plan; `docs/integration-contract.md` is the historical integration contract for the runtime that preceded xcb (retired 2026-09-19).
- `site/content/blog/` holds the blog post bodies in Markdown; `site/app/blog/articles.ts` holds each post's title, sources, and review record, and `bun run sync:blog` renders the bodies into `site/app/blog/posts.generated.ts`.
- `STYLE.md` and `WRITING.md` are synced from hraness/.github. Their “Repository additions” list the Gobstopper facts that public copy most often gets wrong.

# Guidelines

- Strategies are pure: transcript in, plan out. Execution lives in adapters.
- Payload transforms preserve original record order and linkage (`parentUuid`, `ordinal`); publication creates a separate copy and preserves the source.
- Never emit transcript content (prompts, tool output, paths beyond what the provider record carries) into stdout, logs, or digests unless the user asked for that field.
- `detect`/`policy-check`/`plan --json`/`verify --json`/`vault --json` are the stable machine surfaces; keep their fields additive-only.
- Every mutating path (`apply`, `watch`) snapshots into the vault before writing and emits a `compaction-events-v1` record after; telemetry failures are non-fatal, snapshot failures abort the edit.
- The `auto` strategy prefers provider delegation for live sessions. Released native dispatch is disabled until provider ownership and correlation are verified. Separate copies require source binding and structural verification.
- Keep dependency count small; prefer `std` + `serde_json` over new crates.
- The proxy binds loopback only, passes request headers to `curl` through its environment (never argv), logs no request or response content, and forwards the client's original bytes on any parse, engine, or non-length provider failure.

# Local development and install

For fast iteration, build and install the release binary once instead of running `cargo run` each time:

```bash
cargo build --release
cargo install --path crates/gobstopper-cli --locked
```

`~/.cargo/bin/gobstopper` is then on `$PATH` after a shell restart and `gobstopper --version` reflects the current checkout.

<!-- hraness-public-copy:start -->
- Public copy (websites, READMEs, docs, package and GitHub descriptions, CLI help, `llms.txt`, generated pages) follows `STYLE.md`, synced from hraness/.github. Text a model writes for publication also follows `GENERATION_STYLE.md`.
- The delivery vocabulary in this file (admission, qualification, custody, receipt, bounded, lane, gate, surface, projection) is internal. Translate it into what the reader gets.
- Take one-line product and sibling descriptions from the portfolio registry and versions from the release record. Tests pin facts, not prose.
- Run `bun run check:copy` before handoff when the repository has it.
<!-- hraness-public-copy:end -->

<!-- hraness-releases:start -->
- GitHub Release pages follow `RELEASES.md` in hraness/.github: the title is the registry product name and the tag, and the body is a summary, `## Changes`, `## Install`, `## Verify`, then the repository's identity record as a trailing HTML comment.
- The summary and changes come from the version's section of `CHANGELOG.md` in the tagged commit. Write that section in the version bump pull request. The release workflow copies it, generates Install and Verify from the release record, fails when the section is missing or empty, and never uses GitHub's generated notes.
<!-- hraness-releases:end -->

<!-- hraness-articles:start -->
- Essays and blog posts follow the essay addendum in `GENERATION_STYLE.md` and `ARTICLE_COPY.md` in `@hraness/design-kit`. The byline is “Hraness”, every post shows the provenance note naming its recorded reviewer, and no AI-drafted post is credited to a person unless that person rewrites and adopts it.
- Take product names, one-line descriptions, addresses, status labels, and relations from the portfolio facts in `@hraness/design-kit`. Render versions from the release record (`package.json`, a published-release file), never typed by hand.
- Write a “How X uses Y” post only for a registered relation that has a description. Change the relation and its post in the same change. Link between products only along registered relations, and between a technique post and product posts about the same technique.
- Every post has a review record: reader job, non-obvious answer, sources with the date checked, owner, reviewer identity, reviewer type (`ai` or `human`), a score out of 12, and a `reassessOn` date 28 to 56 days after review. The reviewer is independent of the run or person that drafted the post. An AI reviewer is recorded and shown as AI; `humanReview` stays null unless a person reviewed the post.
- A new post starts out of search indexes, sitemaps, and feeds. It becomes indexable only when its review record is complete, scores at least 9 of 12 with no zero score, and the page shows the provenance note.
- When a product is renamed or a relation changes, update the post bodies that mention it in the same change.
<!-- hraness-articles:end -->

<!-- hraness-delivery:start -->
- Treat the user's request to change this repository as standing authorization for routine task-owned commits, pushes, pull requests, merges, releases, deployments, and production verification after the gates applicable to that action pass. Do not ask for duplicate confirmation. Build confidence through relevant automated checks, bounded diagnostics, and independent review, not another human approval. Passing checks does not expand task scope or authority.
- Prefer agentic service provisioning for new infrastructure. Check Vercel Marketplace for a native product that can provision the required resource first; use Stripe Projects as a supported alternative when it better covers the service or the Marketplace route only connects an existing account. Verify the current catalog, account, region, plan, recurring cost and resource capabilities before selecting a route. Prefer supported provider CLIs or APIs over browser-only setup when neither catalog fits, and explain the concrete exception. Reuse existing owner-controlled resources where appropriate; this preference alone does not authorize migrations, duplicate accounts, paid upgrades or wider access. Continue setup already authorized by the task and budget without duplicate confirmation. Keep provider credentials and generated environment files private, complete required interactive authentication, and verify deployment, persistence and recovery separately from successful provisioning.
- Separate artifact admission from live qualification and operational activation. Use applicable automated source, security, package/install, and provenance evidence for artifact admission; live provider qualification is not a universal publication prerequisite. Preserve explicit live acceptance criteria and require relevant live evidence for claims that depend on it. If publication or an artifact's install, upgrade, or default-use path activates risky unqualified behavior, keep that behavior guarded or disabled, or obtain bounded relevant evidence before shipping or activation.
- Use the repository's documented delivery workflow and preserve the identity, target, capacity, migration, and recovery guards applicable to operational activation. Replace an obsolete gate through a reviewed source and policy change with corresponding tests, never an ad hoc skip. Preserve every runtime-enforced approval, access control, branch protection, environment rule, safety policy, and required final gate. Ask for user input only when delivery needs a material product decision, missing credentials or authority, unavoidable interactive authentication, an irreversibly destructive action outside task scope, or resolution of a failure that cannot be handled safely and autonomously.
- Preserve production and user data throughout delivery. Inspect the exact account, environment, deployment, and data target before writes. For data changes, inspect a dry run or equivalent migration plan and validate the recovery path before any effect that could lose or corrupt data. Prefer additive, backward-compatible migrations and bounded batches. Record mutation intent, use idempotency or conditional writes, and reconcile uncertain results before retrying. Verify deployed identity, health, and relevant data invariants after delivery. Routine delivery never authorizes resetting, truncating, dropping, or overwriting user data; stop the unsafe operation if preservation or recovery cannot be established.
- Prefer short-lived repository workload identities such as OIDC trusted publishing, GitHub Apps, and narrowly scoped machine identities. Use unattended stable publication and production promotion when supported by the provider and repository. Establish supported machine authority once and verify it with a non-publishing preflight where available; routine releases should not require recurring interactive authentication or conversational approval. Releases and deployments run without a human in the loop: do not add required reviewers, manual approval environments, or wait timers to release or deployment paths, and remove any you find through a reviewed change. Keep account two-factor authentication, and do not add long-lived personal tokens.
- Keep delivery gates proportional to the failure they prevent. Prefer required checks on the current integration candidate, independent agent review, and atomic or conditional integration. Add a merge queue or another approval stage only for a demonstrated coordination or safety need. Replace redundant queues, serial waits, and duplicate checks through reviewed policy changes while retaining evidence for the integrated result.
- Preserve useful reasoning fan-out, but avoid unnecessary checkout fan-out. Prefer subagents in the current task for bounded research, review, diagnosis, and focused checks when they can safely share one working tree; create a separate task or worktree only for independently deliverable divergent edits, an isolated verification tree, or a different execution environment.
- Give each expensive focused validation command and external wait one owner. The integration owner reviews that evidence and runs the repository-required aggregate or final gate once after convergence. Reuse evidence only for the exact Git tree, command, lockfiles, toolchain, relevant environment, and validity period, and never to skip a required final integration, merge, release, deployment, or production-verification gate.
- On Hraness development machines, use the installed host scheduler for heavyweight top-level commands when available. Keep ordinary work in the compute lane; give authenticated browser/dev-server/Chromium work one `browser-auth` owner and Mac-only validation one `mac-native` owner.
- When a CI or policy gate scans complete Git history, check out the exact governed SHA and fetch only the fully qualified governed refs before scanning. Preserve the complete-history gate and reject unexpected refs instead of importing unrelated concurrent heads.
- At closeout, record applicable branch, PR, check, merge, release, deployment, and production evidence. Archive only conclusively finished tasks, never from silence alone, and reclaim only freshly revalidated clean merged worktrees through the guarded exact-path flow.
<!-- hraness-delivery:end -->

<!-- algal-skills:start -->
## ALGAL skills

- Prefer the globally installed `algal-*` skills for bounded git state, diffs, repository survey/search, test and release gates, CI polling, and web fetches instead of repeated raw tool output.
- At session start, if the `algal` skill is unavailable, install the current skill pack for Claude Code, Codex, and Devin for Terminal with `bunx skills add 0thernet/algal-skills --global --agent claude-code codex devin --skill '*' --copy --yes`.
- Confirm installation with `bunx skills list --global`. If Bun or network access is unavailable, continue with repository-native tools instead of blocking delivery.
- Treat ALGAL receipts as execution evidence, not provider attestation, and preserve the repository's normal verification and release gates.
<!-- algal-skills:end -->

---
> Source: [hraness/gobstopper](https://github.com/hraness/gobstopper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-27 -->
