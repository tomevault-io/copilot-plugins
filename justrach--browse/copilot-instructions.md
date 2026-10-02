## browse

> browse is a WebKit browser for the Mac with Codegraff built in.

# Repository instructions

browse is a WebKit browser for the Mac with Codegraff built in.
The browser is this repository; the agent is `graff`, from
[justrach/codegraff](https://github.com/justrach/codegraff). Read
[CONTRIBUTING.md](CONTRIBUTING.md) for what gets merged, and
`skill/browse-bench/SKILL.md` before driving the app.

## Search runs on graff: where each half lives

Search does not ship an agent. It runs the `graff` on the Mac and talks to it
both ways:

- **Search → graff, over ACP** (JSON-RPC on graff's stdin and stdout):
  `Agent.swift` starts `graff acp --yolo`, opens and loads sessions, lists
  models (`graff/models`), sends prompts, and answers graff's questions
  (`session/answer`) and permission requests.
- **graff → Search, over MCP** (HTTP on 127.0.0.1, with a bearer token):
  `AgentTools.swift` serves the browser's tools — `read_pages`, `search`,
  `form_fields`, `fill` and the rest — and writes where they are to
  `.mcp.json` in graff's folder and to `Agent/mcp.json`. Search passes that
  file to graff as its only MCP config unless Settings › Agent asks for all.
- `AgentColumn.swift` and `AgentHome.swift` are what people see of it.

graff's source is `~/codegraff` on this Mac (Zig: `zig build`,
`zig build test -Dtest-filter=…`). That checkout is often on someone else's
branch: work in a `git worktree` off `origin/main`, and read its AGENTS.md
before touching anything there.

## Whose bug is it

The same failure with `graff` in a terminal, or with another ACP client, is
graff's. Search sending the wrong request, or misreading a right answer, is
Search's. When unsure, run the ACP exchange by hand against `graff acp`, or
read the code on both sides — the cause in code terms is what an issue needs.

A bug that is Search's is fixed here. A bug that is graff's goes through the
line below, even when Search can work around it.

## The line to codegraff

1. **Tell a live codegraff session first.** If one is running on this Mac
   (ListAgents in Claude Code; `peer_message action=list` in graff), message it
   directly: what breaks, where in graff's code, and the evidence — traces,
   timings, the model involved. That channel is private; it is where evidence
   goes. It may already be on it.
2. **File it with `./graff-issue`.** `./graff-issue file "Title" < body.md`
   checks the body against codegraff's public-tracker rules, stops at likely
   duplicates, files it, and adds a line to
   [docs/graff-issues.md](docs/graff-issues.md). The body is the bare
   minimum: the symptom, graff's own error text, the cause in code terms, the
   fix. No local paths, traces, transcripts, session ids, timings or sizes,
   model or provider names, or attribution — anything posted in breach is
   deleted with `gh api graphql` `deleteIssue`, not edited, and the user is
   told what was public and for how long.
3. **Work around it here, if people would feel it**, with a comment that
   names the issue (`justrach/codegraff#N`) — `GRAFF_CODEX_WS=off` in
   `Agent.environment` is the pattern.
4. **Fix it upstream, if asked**: a branch in a worktree of `~/codegraff`, a
   pull request there, its pre-push checks passing. Never merge into its
   `main` or a `release/v…` branch without being asked.
5. **Close the loop.** `./graff-issue list` shows each filed issue's state.
   Once a graff release carries the fix, take the workaround out and the line
   off docs/graff-issues.md in the same commit.

## Commits

No assistant or model-vendor attribution: no `Co-Authored-By` for Claude or
any other vendor's model, no "Generated with" lines, no session links, in
commits or pull requests. Commits carry the owner's git identity. When graff
makes the commit, `Co-Authored-By: Codegraff <blackfloofie@codegraff.com>` is
allowed (and is the only allowed trailer). Omit it if the user says not to.

## Building, testing, releasing

- `./build.sh` builds `build/browse.app`; `swift build` must stay
  free of warnings you introduced.
- `./bench --world NAME …` drives a test copy with a profile of its own. Never
  drive or quit the installed app unless asked.
- A release: `./release.sh X.Y.Z "What's new."` bumps the version, writes
  the notes, tags and pushes; `.github/workflows/release.yml` builds, signs,
  notarises and publishes the GitHub release with the DMG, the ZIP and
  `appcast.json`. Every installed copy reads
  `releases/latest/download/appcast.json` and swaps the new build in for its
  next launch (`Updater.swift`); one not signed by the same Developer ID
  team is never swapped in. docs/releasing.md has the secrets it needs.
- Branches: work reaches `main` by pull request. Each X.Y has a
  `release/vX.Y` branch, cut from `main` by `./release.sh X.Y.0`; its tags are
  made only there, and a fix reaches it by `git cherry-pick -x` from `main`.
  Never push to a `release/v…` branch or tag a release without being asked.
- The Keychain: never dump it or walk every item.

---
> Source: [justrach/browse](https://github.com/justrach/browse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
