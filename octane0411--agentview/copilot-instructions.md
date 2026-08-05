## agentview

> This repository is for `agentview`: a local Agent View-style controller for Codex sessions.

# AGENTS.md

## Project

This repository is for `agentview`: a local Agent View-style controller for Codex sessions.

The long-term product goal is to match the practical workflow of Claude Code Agent View:

- dispatch background coding sessions
- monitor session status from one list
- peek latest output or blocking questions
- reply without leaving the list
- enter a full session and continue the conversation
- detach back to the list without stopping the session
- isolate parallel file edits with git worktrees

The current canonical product spec is:

- `docs/codex-agent-view-spec.md`

## Current Scope

Start with a Codex-only MVP.

Do not implement Claude orchestration until the Codex session model is working.

Prefer this sequence:

1. Local CLI/TUI skeleton.
2. Local job store.
3. Codex session dispatch.
4. Job list and status updates.
5. Peek latest output.
6. Attach/resume a Codex session.
7. Worktree isolation.
8. Stop/delete/archive flows.

## Design Constraints

- Keep the UI close to Claude Code Agent View behavior where practical.
- Keep provider-specific behavior behind adapters.
- Treat Codex conversation state as Codex-owned.
- Treat Agent View job metadata as this app's responsibility.
- Do not depend on undocumented Claude Agent View internals.
- If using experimental Codex app-server APIs, isolate the protocol code and document the Codex CLI version tested.

## Implementation Notes

- Implement app code in Rust. Do not add TypeScript or JavaScript runtime code.
- Use Serde schemas for persisted state and external event boundaries.
- Use structured state rather than parsing terminal output when an API or JSON event stream is available.
- Keep worktree cleanup conservative. Never delete uncommitted work silently.
- Prefer small vertical slices over broad scaffolding.
- Avoid adding cloud, multi-user, or web UI assumptions in the MVP.

## Verification

Before finishing implementation work:

- Run the relevant formatter and tests.
- Manually verify dispatch, list, peek, attach/resume, and stop behavior for at least one Codex job.
- Check `git status --short` and explain any remaining changes.

---
> Source: [Octane0411/agentview](https://github.com/Octane0411/agentview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
