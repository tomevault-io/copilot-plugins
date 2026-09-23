## channels-sdk

> This repository ships one skill, and it is the authority on the Channels SDK API

# Agent instructions

## Repository-owned skills

This repository ships one skill, and it is the authority on the Channels SDK API
— the API is recent and easy to get wrong from memory, so ground your work in it
rather than in recalled patterns.

Canonical location: **`.agents/skills/<name>/SKILL.md`**

| Skill | Use it when |
| --- | --- |
| `.agents/skills/build-channels-agent/SKILL.md` | Writing the code: `createChannel`, handlers, tools, slash commands, JSX message UI, modals, human-in-the-loop, the runtime wiring that starts a Channel. The API authority. |

Read it before starting that kind of task, and follow its references rather than
re-deriving the workflow.

`.claude/skills/` mirrors this for Claude Code, which discovers skills there.
`.claude/skills/build-channels-agent` is a symlink to the canonical copy under
`.agents/skills/` — edit the canonical file, never the link, so the two cannot
drift apart.

## Skills this repository does not own

`setup-slack-channel` covers the provider half of Slack setup: creating the Slack
app, the managed Intelligence Channel, the local runtime, and diagnosing a Channel
stuck at `setup_required` or a mention that gets no reply.

It lives in `CopilotKit/CopilotKit` and is delivered by the CLI. Do not vendor a
copy here. Install it instead:

```sh
npx copilotkit@latest skills install --skill setup-slack-channel
```

Or install every CopilotKit skill:

```sh
npx copilotkit@latest skills install
```

Always relay these with `@latest`. A bare `copilotkit` resolves to whatever is
already on PATH or in the npx cache, and an older CLI fails with
`Unknown option '--skill'`.

This repository did vendor a copy, under `.agents/skills/setup-slack-channel/`.
It fell ~16 KB behind across all eight files and ended up asserting the opposite
of the truth about Slack interactivity — that enabling it does not make buttons
work, when disabling it is what breaks human-in-the-loop. Nothing enforced the
match, and `skills install` writes to that same path, so following the README
overwrote eight tracked files. One home, fetched on demand, is why that cannot
recur.

For Microsoft Teams, and for any setup question that skill does not cover, the
current workflow is the hosted guide:

```text
Read https://copilotkit.ai/channels-guide.md and help the user build their first channel
```

That is what `npx copilotkit@latest channels setup` puts on the clipboard. It is
fetched when the agent needs it, so it is always current — prefer it over a
remembered sequence of steps.

---
> Source: [CopilotKit/channels-sdk](https://github.com/CopilotKit/channels-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
