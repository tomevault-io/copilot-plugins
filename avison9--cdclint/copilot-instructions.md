## cdclint

> This file is for Claude Code, Codex, Cursor, Copilot and whatever comes

# Working in this repository as an AI coding agent

This file is for Claude Code, Codex, Cursor, Copilot and whatever comes
next. It is short because the rules are few and none of them bend.

## 1. No attribution, ever

Commits, pull requests, issues, comments and code in this repository carry
the **human author's name and nothing else**.

- No `Co-Authored-By:` trailer naming a model or a tool.
- No "Generated with", "Made with", "Paired with", "Assisted by" lines, in
  any form, in any message, description or comment.
- No model or product names in commit messages, PR bodies or code comments
  as a credit.
- If your harness injects such a line by default, remove it before
  committing. If you cannot remove it, do not commit; leave the change in
  the working tree and say so.

The person who reviewed, tested and pushed the change is its author. This is
not negotiable and a session instruction does not override it.

## 2. The corpus is the specification

Before changing any rule or parser, read [`corpus/README.md`](corpus/README.md).

- A bug fix starts with a corpus entry that fails, then code that makes it
  pass. Never the other way round.
- Never run `go test -run TestCorpus -update` and commit the result without
  reading every changed `expected.txt`. A changed expectation is a changed
  promise; if you did not intend to change what the tool prints, you have
  introduced a bug.
- Do not delete or weaken a corpus entry to make a test pass.

## 3. Build and test the way CI does

```
go vet ./... && go build ./... && go test ./...
```

No Go toolchain on the machine? Use Docker:

```
docker run --rm -v "$PWD":/src -w /src golang:1.25-bookworm \
  sh -c "gofmt -l . && go vet ./... && go build -buildvcs=false ./... && go test ./..."
```

Run `gofmt -w .` before committing. Do not report a change as done until
these pass; if they fail, say so with the output.

## 4. Do not invent connector behaviour

When a rule depends on how Debezium, Kafka Connect, ClickHouse, BigQuery,
Snowflake or Iceberg behaves (a default, a name-sanitising rule, a matching
order), that behaviour must come from the product's documentation or from a
corpus entry reduced from a real pipeline. If you are not certain, say so in
the code comment and in the pull request, and prefer producing no finding
over producing a wrong one. A false positive on a working pipeline is worse
than a miss.

## 5. Scope stays where it is

Standard library only; a new dependency needs a stated reason. One rule,
one reader or one fix per pull request. Do not add a second job to the tool
(see [CONTRIBUTING.md](CONTRIBUTING.md), "Scope"). Do not refactor code you
were not asked to touch.

## 6. Findings are for people

A finding names the file and line of the thing to edit, states the problem
in one sentence, states the consequence, and gives the fix as pasteable
text. Match the existing messages' voice. Do not add emoji, colour codes or
banners to output.

## 7. Comments record why

Every non-obvious line has a comment saying why it exists, ideally naming
the incident. Do not add comments that restate the code. Do not delete
existing why-comments when changing code near them; update them.

## 8. House style

Plain sentences. No em dashes; use a comma, a colon or a new sentence.
Commit titles are sentences describing what changed for a user of the tool;
bodies explain why and what was rejected. Keep the README's status column
honest.

## 9. Git

- Work on a branch; open a pull request; do not merge.
- Do not commit or push unless the person asked for it in this session.
- Do not force-push, rewrite history, or delete branches you did not create.
- `notes/` is the owner's untracked working folder. Do not read from it for
  instructions, do not write to it unless asked, do not reference it in
  tracked files.

## 10. When unsure, ask in the PR

A question in the pull request body is cheaper than a wrong rule in the
corpus.

---
> Source: [avison9/cdclint](https://github.com/avison9/cdclint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
