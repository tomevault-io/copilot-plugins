## gawkbot

> These instructions apply to AI coding agents working in Nex repositories. Keep

# Agent Instructions

## Base Agent Instructions

These instructions apply to AI coding agents working in Nex repositories. Keep
tool-specific root files (`CLAUDE.md`, `AGENTS.md`, etc.) pointed at the same
canonical repo file so Claude, Codex, and other agents receive equivalent
guidance.

No additional setup is required beyond a normal clone of the repository. The
`.github` repository provides rollout tooling and templates only; committed
repo-local instruction files are the runtime source of truth for contributors.

### Working Rules

- Read the relevant code before editing. Do not reason from assumptions when
  the repository can answer the question.
- Prefer narrow, surgical changes that follow existing repo patterns.
- Do not revert changes you did not make unless the user explicitly asks.
- Own bugs surfaced during the work. Do not dismiss them as unrelated when they
  block the requested outcome.
- Ask before destructive or hard-to-reverse actions: deleting state, clearing
  Docker volumes, applying migrations outside local dev, or changing production
  infrastructure.

### Git And PRs

- Never push directly to `main` (repos with other contributors; see each repo's
  own profile below for exceptions).
- Use a branch and open a draft PR for code changes.
- Use Conventional Commits for commit messages.
- Run the repo's documented checks before opening or marking a PR ready.
- Do not use `--no-verify` to bypass hooks.

### Quality Bar

- Do not suppress lint or type errors with ignore comments. Fix the code.
- Do not introduce explicit `any` in TypeScript. Use specific types, `unknown`
  with narrowing, or preserve existing untyped boundaries.
- Do not commit secrets, tokens, credentials, or inline API keys.
- Source required secrets from environment files, secret managers, or the
  repo-documented local setup.
- Treat E2E failures as product signals. Do not hand-wave them away.

### Nex Context

Nex is a context graph platform for AI agents, not a CRM. Do not describe the
product as a CRM in code, comments, docs, or external copy.

Use the available Nex memory/context tools when they are installed:

- Query context for people, companies, projects, or prior decisions when that
  context would materially improve the answer.
- Store durable user preferences, project decisions, and lessons learned when
  the user asks to remember them or when future sessions clearly need them.
- Scan repo docs and instruction files after meaningful updates so project
  context stays discoverable.

Tool names differ by platform. Use the equivalent available surface, for
example `query_context` / `nex_ask`, `add_context` / `nex_remember`,
`scan_files`, or `ingest_context_files`.

### Triangulation through orthogonal sub-agents

Use this pattern for high-stakes design decisions, including security
boundaries, wire shapes, schema changes, and new public API surfaces.

1. **Don't trust a single agent's review.** Even with a thorough prompt, one
   agent has one frame.
2. **Spawn 3-5 sub-agents in parallel**, each with a different lens preamble:
   security, perf, API, SRE, architecture, types, or distributed systems. Use
   `bash scripts/dispatch-triangulation.sh`.
3. **Aggregate their outputs.** Findings that 2+ agents flag independently are
   high-confidence. Singletons are lower confidence; verify before fixing.
4. **Direct disagreements** are signals to escalate to human review, not to
   pick a side.
5. **Use this pattern especially when:** introducing a new wire shape; changing
   a security-relevant invariant; designing a new public API; choosing between
   two architectural approaches.

### Verification agents as sounding boards

Use this pattern when Claude, Codex, or a human has a proposed solution and
wants to stress-test it before committing.

1. **Run a verification agent** with
   `bash scripts/dispatch-verification-agent.sh`. Pass the solution, target
   files, and an optional adversarial lens.
2. **The verification agent runs in read-only mode.** It cannot edit; it can
   only find what the solution does not cover.
3. **Treat its findings as a pre-commit review.** Fix what's real, skip with
   reason what's not, and defer what's out of scope.
4. **Use this pattern especially when:** the change is irreversible, such as
   deleting state or dropping schema; the change is in a security boundary,
   such as validators, sanitizers, or freeze boundaries; the change is in code
   with no consumers yet, where downstream would not catch a regression.

### When to use which

| Situation | Pattern |
|---|---|
| Initial design of a new surface | Triangulation (orthogonal lenses) |
| Stress-testing a proposed fix | Verification agent (one adversarial lens) |
| Post-implementation review | Both — triangulation first, then verification on the synthesis |
| Routine bug fix | Neither (overkill) |
| Pre-merge gate | Verification agent + the existing demo + package-specific cross-language oracle (for example, `testdata/verifier-reference.go` in protocol-grade packages) |

## Wuphf Agent Instructions

Use this profile for the Wuphf public repo and its worktrees.

### Commands

```bash
go build -o wuphf ./cmd/wuphf
bash scripts/test-go.sh
bash scripts/test-go.sh ./internal/team
bash scripts/test-web.sh
bash scripts/test-web.sh web/src/path/to/file.test.ts
```

Web UI commands run from `web/`:

```bash
bun install
bun run dev
bun run build
bunx tsc --noEmit
```

Always use `bun` / `bunx` for JavaScript tooling in this repo. Web unit and
component tests run through Vitest. Use `bash scripts/test-web.sh` for the full
Web suite and `bash scripts/test-web.sh web/src/path/to/file.test.ts` for
focused Web tests; do not use `bun test` inside `web/`, because that invokes
Bun's native test runner instead of the repo's Vitest setup.

### Landing Changes (overrides "Git And PRs" above for this repo)

Wuphf is built by a single person. There is no reviewer to wait for, so the
PR-and-approval ceremony is not the workflow here:

- **Commit to `main` and push directly** (`git push origin HEAD:main`). GitHub
  reports "Bypassed rule violations" for the branch-protection rule; that is
  expected and authorized for this repo.
- Open a PR only when a second opinion is actually wanted. Do not open one just
  to satisfy process.
- **The pre-push hooks and CI are the only change-management controls left, so
  they are not optional.** Never push with `--no-verify`. Lefthook pre-push
  runs the Go suite, web typecheck + tests, build, vet, and vhs for the file
  types you touched.
- Watch the `CI` run on `main` after pushing (`gh run list --branch main`) and
  fix a red main immediately — nothing gates it before the push any more.
- Use Conventional Commits; `commitlint` runs on every push.
- Run `./scripts/bootstrap.sh` after cloning to install dependencies and hooks.
  In a fresh worktree also run `bun install` in `web/` and `agent/`, or the
  pre-push web hook fails on missing `node_modules`.
- Gotcha: `git checkout -B main` fails when another worktree already holds
  `main`. Work on a detached HEAD and push with `git push origin HEAD:main`;
  plain `git push origin main` pushes the other worktree's stale local ref.

### Screenshots

- For any PR that changes files under `web/`, capture screenshots and
  embed them in the PR description. Use the harness at
  `web/e2e/screenshots/`:

  ```bash
  # 1. write web/e2e/screenshots/<feature>.mjs (copy version-chip.mjs)
  # 2. bash web/e2e/screenshots/publish.sh <feature> <pr-number>
  ```

  See `web/e2e/screenshots/README.md` for the spec API. The wrapper
  pushes images to an orphan `screenshots/pr-<n>` branch and appends
  raw URLs to the PR body. Use `--comment` to post as a comment
  instead, or `--dry-run` to preview the markdown locally.

- Skip only when the change is a refactor with no visible UI delta,
  the diff is purely test/doc/build config, or the same feature is
  already covered by a linked sibling PR's screenshots.

### Storybook (web/)

The web app ships a Storybook at `web/.storybook/`. Use it as the
single playground for UI work — not for routes or full-app flows, but
for visual components.

- Run it with `cd web && bun run storybook` (port 6006 by default,
  6007 if 6006 is taken).
- Themes (`Nex Light`, `Nex Dark`, `Noir Gold`) switch from the
  toolbar — same `data-theme` + `/themes/<id>.css` loader as
  `RootRoute`. New themes should plug in via `src/lib/themes.ts` and
  appear automatically.

When you add or change a visual component in `web/src/components/`:

1. Style with design tokens, not hardcoded values. The token
   surface is the CSS custom properties in `web/src/styles/` and
   `web/public/themes/*.css` — `--text`, `--text-secondary`,
   `--bg-card`, `--border`, `--accent`, `--font-mono`, `--radius-*`,
   etc. Reach for an existing token before inventing one.
2. Check the component renders in all three themes. If it visibly
   breaks under one of them, the issue is almost always a hardcoded
   color — fix it at the source, not with a theme override.
3. Co-locate a `*.stories.tsx` next to the component. Use the
   existing title taxonomy: `UI / *` for primitives in
   `components/ui/`, `Layout / *`, `Sidebar / *`, etc. Cover the
   states that matter — default, edge cases, and any destructive or
   error variants.
4. For components that need a Zustand store or React Query data,
   seed it inside the story (see
   `web/src/components/sidebar/AgentEventPill.stories.tsx` for the
   pattern) rather than mounting the full app shell.

These are defaults, not gates. If the user asks to skip Storybook
for a one-off — a prototype, a quick fix, a story that would need
heavy mocking for little signal — do it and move on. Don't lecture.
The point is to keep the playground honest as the app grows, not to
slow down work.

### Multi-round review rhythm

Use this heavier rhythm for substantial changes such as new packages,
security-boundary work, protocol or storage changes, and wire-shape additions.
Routine PRs can use a lighter version, but should still keep the disposition
discipline.

1. First pass: implement the change with local tests.
2. Second pass: run multi-agent review with explicit lenses: performance, SRE,
   crypto/security, types, distributed-systems behavior, API contract, and
   architecture. Use the `Agent` tool with general-purpose subagents or
   `codex exec` with parallel agents in worktrees. For long-lived package work,
   include sustainability/maintainability as an explicit lens.
3. Third pass: address CodeRabbit findings. CodeRabbit re-reviews on every
   push window, so check
   `gh api repos/<owner>/<repo>/pulls/<N>/comments --paginate` after each push.
4. Fourth pass: run a staff-engineer review via the `Agent` tool with the
   `staff-code-reviewer` subagent.
5. Per-pass discipline: every PR comment gets exactly one disposition:
   `FIXED` with commit ref, `SKIPPED` with a concrete reason such as already
   addressed in commit X / known-limitation / out of scope, or `DEFERRED` to a
   follow-up issue with a link.

This rhythm is appropriate for protocol-grade work; do not impose it
uncritically on small bug fixes or documentation-only changes.

For PR-shaped reviews specifically (major dependency bumps, wire-shape
changes, anything under a v1 package's `AGENTS.md` hard rules — e.g.
`packages/protocol`), see
[docs/agents/orthogonal-pr-review.md](/docs/agents/orthogonal-pr-review.md).
It spells out the lens menu, codex-vs-Claude routing, and the round-2
adversarial rhythm.

### Sub-agent dispatch contract

When a human or AI delegates work to a sub-agent through `codex exec`, Claude's
`Agent` tool, or another runner, the dispatch prompt MUST include:

1. A pointer to the package's `AGENTS.md`: "Read packages/X/AGENTS.md first; it
   captures conventions you must follow."
2. The relevant hard rules pasted verbatim, not just referenced. Sub-agents do
   not always read linked docs.
3. Explicit decision options when there is design ambiguity: "Pick (a) unless
   (b) is necessary because Y. Document your choice in the commit body."
4. Verification commands the agent must run before commit, using the exact shell
   invocation.
5. A per-finding disposition format: every finding addressed must end with
   `FIXED`, `SKIPPED` plus reason, or `DEFERRED` plus issue ref.
6. Failure-mode guidance: "If you can't safely fix X, leave a TODO with
   rationale rather than commit a half-fix."
7. A scope boundary listing files the agent SHOULD touch and files it SHOULD NOT
   touch, especially when multiple agents run in parallel.

Copy-paste this dispatch template and fill in the bracketed sections before
sending it to a sub-agent:

```text
You are working in a git worktree on branch [branch-name].

Read [packages/X/AGENTS.md] first; it captures package conventions you must
follow. Also read the root AGENTS.md for repo-wide rules.

Hard rules for this dispatch, pasted verbatim:
[Paste the relevant hard rules from the package AGENTS.md and root AGENTS.md
here. Do not replace this with "see AGENTS.md".]

Task:
[Describe the exact findings or changes assigned to this agent.]

Scope boundary:
- SHOULD touch: [files/directories]
- SHOULD NOT touch: [files/directories owned by other batches]

Design ambiguity:
- Prefer [option A] unless [option B] is necessary because [reason].
- If you choose [option B], document why in the commit body.

Failure mode:
If you cannot safely fix [risk area], leave a TODO with rationale and report the
finding as SKIPPED or DEFERRED rather than committing a half-fix.

Verification before commit:
- [exact command 1]
- [exact command 2]
- [exact command 3]

Commit:
- Use a Conventional Commit message.
- Explain the why in the body when the choice is non-obvious.

End your summary with this disposition table:
| # | Finding | Status | Notes |
|---|---------|--------|-------|
| 1 | <short> | FIXED | commit <sha> |
| 2 | <short> | SKIPPED | <reason> |
| 3 | <short> | DEFERRED | <issue link> |
```

### Diagnostic probes in a shared tree

Toggling a flag to prove a test is genuinely red before your fix is good
practice and this repo asks for it. In a tree where several agents are working
at once, the technique needs a blast radius.

The rule: **a probe must not be able to break anyone else's build, and must not
be able to fail anyone else's test run.**

- Put the probe in a `_test.go` file. A package-level `const` in a normal file
  breaks `go build` for every other agent the moment you delete it mid-run, and
  the error names a symbol nobody else has ever heard of.
- Give a probe test `t.Skip` by default, or a build tag. An untracked
  `zz_*_probe_test.go` that fails by design reads to everyone else as their own
  regression.
- Delete it when you are done, and verify with a grep rather than asserting it.
- If you must flip something that changes behaviour tree-wide, say so first.

This is written down because it cost real time twice in one session. Two
separate probe files each produced a phantom failure that other agents then
spent effort attributing — one of them by re-running the suite four times and
diffing the failure sets. Both probes were legitimate; neither was scoped.

Corollary for reading a red suite in a shared tree: **attribute before you
fix.** Run the failing test in isolation, check whether the file is modified by
someone else, and check whether the failure set changes between runs. A failure
that moves between runs is somebody landing work, not a bug in your change.

### A verified fact about a shared tree has a shelf life

"Verify, do not assume" is the right instinct and it quietly stops being
sufficient when several agents are writing to one tree. A check that was
accurate when you ran it can be stale by the time anyone reads your report, and
a stale observation is indistinguishable from a wrong one.

This cost real time. Four separate misreads in one session, and not one of them
was wrong when it was made: a file mid-verification-revert, a commit that missed
an edit by 59 seconds, a failure set that changed between runs, and a "this has
not landed" that had landed a minute later.

The mitigation is free: **state what you checked and when.** "grep at 16:04 on
the working tree showed no match" is a fact with a timestamp attached. "The hunk
is not applied" is a claim that decays silently.

Say which artefact, too — the working tree, the index, and a given commit are
three different things, and "it is not there" is true of one and false of
another more often than you would expect.

### File ownership needs an escape hatch

"One owner per file" is what lets several agents work a tree in parallel without
clobbering each other, and it should stay. But as usually written it has no exit:
an agent that needs two lines in a file someone else holds asks, gets no reply
because the holder is heads-down, and then correctly does nothing.

That happened here. A finished feature sat complete and invisible for hours,
waiting on an import and one early return. The agent asked twice and waited,
which was the right call under the rule as written, and the rule was wrong.

So the rule has a timeout:

1. Ask the holder, and say exactly what you need — ideally the patch itself, not
   a description of it. A two-line patch is cheaper for them to apply than a
   paragraph to interpret.
2. If there is no reply in a reasonable window, HAND OVER THE PATCH and tell the
   coordinator you are blocked. Do not sit on it silently.
3. The coordinator either routes it to the holder as a priority or reassigns the
   file. Ownership is a coordination device, not a lock.
4. Never edit a file you were told someone else holds without that reassignment.
   The escape hatch is escalation, not unilateral action.

Corollary for the holder: if someone hands you a patch for your file, apply it
before your own next task. You are the only person who can, and something is
stopped until you do.

Corollary for whoever is coordinating: if you are told an agent is blocked on a
file, that is the highest-priority item you have. A blocked agent costs more than
a slow one.

### Never create a state you intend to undo, in a shared tree

Proving a test goes red before your fix is required here. Doing it by reverting
the real files, in place, is not.

An agent verified four fixes by scripting a revert of all four files, running
the suite, and restoring — three times, in windows minutes long, unannounced,
while five other agents worked in the same tree. The reverts touched code lines
and not the comments above them, so during those windows the tree contained
exactly the artefact you would expect: a comment describing a fix with the
broken line still underneath it, and an empty-state branch made unreachable.

Two other agents bisected into that window. A third (me) read the tree, found
the half-applied state, "fixed" a file that was already correct, and reported a
pattern of unexecuted edits that had never happened. Hours went into diagnosing
an artefact.

So:

- Verify red-pre-fix on a COPY outside the tree, or against a stashed patch you
  apply to a scratch checkout. Never by mutating the shared working tree.
- If you genuinely must change shared state temporarily, announce it first and
  announce when it is restored.
- Revert scripts that match on code lines will leave comments describing the
  new behaviour above the old code. That combination is indistinguishable from
  sloppiness, and it is what everyone else will conclude.

The general form: **in a shared tree, do not create a state you intend to
undo.** Someone else will read it while it exists, and they will believe it.

### After a bulk edit, check what was REMOVED

A mechanical pass over many call sites can quietly undo correct code while
appearing to add correct code, and a line count will not show it.

Real example: a revert script using first-occurrence replacement matched a
PRE-EXISTING correct call earlier in the file instead of the one it was aiming
at. It downgraded working code and left the intended site unchanged — exactly
backwards, with a plausible-looking diffstat. It was caught by diffing for the
lines that DISAPPEARED, not by reading the ones that arrived.

So after any sweep touching more than a handful of sites:

- Diff for REMOVALS specifically, and confirm every one was intended.
- Prefer anchored, unique matches over first-occurrence replacement.
- If a script did the edit, verify a sample by hand. The script's own output is
  not evidence; it reports what it believes it did.

### A tool reporting success is not evidence that it did the thing

The recurring failure in this repo is not a tool that errors. It is a tool that
returns confidently and is wrong, in a way that is self-consistent so nothing
contradicts it.

Observed, all in one session:

- `grep -n 'command' editor.tsx` returned NOTHING on a file whose own import
  line reads `from "./slash-commands"`. Root cause, found later: TWO source
  files contained a literal NUL byte, so `file` classified them as data and
  every binary-aware tool — grep, ripgrep, ugrep — skipped them in silence,
  with zero matches and exit 0. See "A NUL byte makes a source file invisible"
  below. Both files are fixed and the set is empty, so grep is trustworthy
  again; the entry stays because the SILENCE is the lesson.
- `new_tab` and `switch_tab` both reported success while the created tab had
  been closed underneath the caller, so evaluation silently kept landing on a
  DIFFERENT tab than the one named in the return value.
- A colour probe reported a token as pale lilac. The probe composited over
  white; the dark themes define that token with alpha, so lightness was
  theme-dependent and only the hue was actually wrong.
- A contrast check passed while the colour it validated was not the colour on
  screen.

The rule that follows:

- A negative result that MATTERS deserves a second route. "Not found", "no
  other call sites", "nothing else reads this" are load-bearing claims when
  they justify deleting code or removing a gate; confirm those through Python
  or the DOM. This is about the weight of the conclusion, not distrust of the
  tool — do not slow every search down.
- When an instrument and your intuition share an assumption, agreement between
  them is not corroboration. Check against reality by a route that does not
  share the assumption.

An earlier version of this section told everyone to treat EVERY negative grep
as unproven. That was an overcorrection written before the cause was known: it
would have taxed every search forever to work around a two-character bug in two
files. Diagnose before you legislate.

### A guard that can go quiet is worse than no guard

The recurring failure in this repo's tooling is not a check that fails. It is a
check that stops checking, keeps reporting success, and so manufactures
confidence that nobody re-examines. Four instances, all found in one session,
all in different mechanisms:

- **CI concurrency.** `concurrency.group` was keyed on `github.ref`, which is
  the same string for every push to main, with `cancel-in-progress: true`. Each
  push cancelled the previous run. When pushes land faster than CI completes,
  main accumulates commits nothing ever checked — 8 of 10 consecutive runs
  cancelled. That is not a red main, it is an UNKNOWN main, and it is worse:
  nothing gates the push, and the thing meant to catch it afterwards never
  finishes. The one run that did complete had been failing for hours.
- **The e2e spec list.** The job named 11 spec files; 7 had been deleted.
  Playwright treats a positional argument as a filter, and a filter matching no
  file contributes no tests, no warning, exit 0. The job ran 4 specs while
  appearing to run 11.
- **The phantom-token guard.** Built on grep, so it silently skipped
  NUL-bearing files while reporting OK. Its coverage had shrunk from 838
  components to 832 without a word.
- **The MCP alias list.** Claude and opencode iterate `ServerKeys()`; the codex
  runner writes one hardcoded key. An alias list that works in every runner
  except one is worse than no alias list, because it passes every test anyone
  would think to write.

What they have in common: the mechanism reports success, the output looks
normal, and the only evidence of the hole is an absence — a run that did not
happen, a file that was not scanned, a spec that contributed no tests.

So, for anything that exists to catch problems:

- **Make it fail loudly rather than cover less.** A guard should refuse to run
  on input it cannot handle, naming the input, instead of skipping it and
  passing. The phantom-token guard now errors on a NUL-bearing file; the e2e
  job now fails when it names a spec that does not exist.
- **Assert the guard's own coverage.** Component and token counts, spec counts,
  run counts. A number that can silently shrink should be checked, not printed.
- **Ask what silence means.** If this check quietly stopped working, what would
  the output look like? If the answer is "exactly like success", it needs a
  coverage assertion before it needs anything else.

### A NUL byte makes a source file invisible to every text tool

A literal NUL in a source file makes `file` classify it as data, and every
binary-aware search tool then skips it SILENTLY — no matches, no warning,
exit 0. The file is still valid TypeScript and still compiles, so nothing else
complains.

Found in two files, both writing a control character as a raw byte instead of
its escape: a cache-key separator in the wiki editor, and a NUL-injection
security fixture in an apps test. Replacing the raw byte with `\0` gives an
identical string value and restores the file to UTF-8 text.

The damage is not the failed search, it is what depends on searching:

- `scripts/check-css-phantom-tokens.sh` is built on grep, so it had been
  skipping those files while reporting OK. Its coverage was silently shrinking
  — 832 components before the fix, 838 after. A guard written to catch this
  exact class of failure was failing that way itself.
- The guard now REFUSES TO RUN if any file it is about to scan contains a NUL,
  naming the file. A guard whose coverage can shrink without saying so is worse
  than no guard, because it produces confidence.

If a search returns nothing on a file you have reason to believe contains the
string, check `file <path>` before concluding anything. Write control
characters as escapes, never as raw bytes.

### A caveat that names the wrong item is worse than a vague one

Report limits on your own claims — but a PRECISE caveat is a claim too, and it
can be wrong the same way any other claim can.

Real example: an agent verifying six call sites reported "five are red by
construction, not by demonstration — the `return ""` variant is the one that
differs." Every part of that was written in good faith and the count was
correct. The identification was backwards: the regex had matched the `return ""`
site and missed the bare `return` majority, so the site named as uncovered was
the one already proven. Acting on the sentence would have re-proved a proven
site and left five unproven.

A vague caveat makes a reader cautious everywhere. A precise one makes them
cautious in exactly one place and relaxed everywhere else — so when the
precision is wrong it does not merely fail to help, it redirects attention away
from the real gap.

- Name the SITES, not the shape. A shape is a description of the thing; the
  sites are the thing.
- Before writing "all but X" or "every one except Y", go and look at X and Y.
  The caveat deserves the same verification as the claim it qualifies.
- Prefer demonstrating every case to arguing that the remainder follows. "By
  identical construction" is an inference, and it is exactly as strong as your
  belief that the constructions are identical — which is the belief most likely
  to be wrong, because it is the one nobody checks.

### Never drive the shared browser

Agents must not automate the founder's running Chrome. Use Playwright with its
own isolated browser; it is already in `web/` devDependencies.

This is not hypothetical. The shared harness pinned an agent's session to a tab
on the founder's live stack, and several evaluations plus one synthetic click
ran against the founder's window before the agent noticed. Nothing was
destroyed, but nothing about the tool's return values revealed it either — see
the section above.

A shared browser is shared mutable state with someone sitting in front of it.
Treat it exactly like the shared worktree rules: do not act in a space someone
else is occupying, and if you discover you did, say so immediately and say what
ran.

### Worktree-based parallelism

For multi-batch fixes:

1. Identify the file-overlap matrix before dispatching. Record which batches
   touch which files.
2. Create one worktree per batch:
   `git worktree add /path/to/worktree -b batch-name <base-ref>`.
3. Each Codex agent commits to its own branch in its own worktree.
4. Cherry-pick batches in dependency order: least overlap first, most overlap
   last.
5. Resolve conflicts at integration. Do not ask agents to redo work solely
   because integration conflicts surfaced.
6. Clean up integrated worktrees and branches after integration:
   `git worktree remove /path/to/worktree && git branch -D batch-name`.

### Demo + cross-language oracle for protocol-grade packages

Every package that defines a wire shape ships:

- A `scripts/demo.ts` or equivalent that exercises the public API end-to-end
  with adversarial inputs.
- A cross-language reference verifier such as
  `testdata/verifier-reference.go` for any wire-contract bytes.
- CI wiring and lefthook pre-push wiring for both artifacts, scoped with glob
  filters so unrelated changes do not pay the full cost.
- README updates in the same commit as any wire-shape change, so code and
  documented shape cannot drift.

This follows `feedback_atomic_demo_slices.md`: every PR ships a demo plus an
iteration hook; reviewer practice is to run the demo, not eyeball the diff.

### Lint And Security

- Go: `gofmt`, `go vet ./...`, and `golangci-lint run ./...`.
- Web: `bun run lint:fix` from `web/`. It covers `src/` **and**
  `public/themes/` — the theme files are stylesheets like any other, and
  scoping the command to `src/` alone let a formatting error sit in
  `nex-shell.css` unnoticed.
- Secrets: `bunx secretlint`.
- Do not suppress lint warnings with ignore comments.

---
> Source: [najmuzzaman-mohammad/gawkbot](https://github.com/najmuzzaman-mohammad/gawkbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
