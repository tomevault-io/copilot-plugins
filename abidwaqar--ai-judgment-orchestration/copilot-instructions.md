## ai-judgment-orchestration

> > **This file is a skeleton.** Replace every `<...>` with your project's reality and delete the guidance blockquotes. It loads into context at the start of every session (and into every subagent), so it's the highest-leverage document in this setup — it's what stops each agent from re-reading your whole codebase to orient itself.

# <Project name> — orientation for Claude Code

> **This file is a skeleton.** Replace every `<...>` with your project's reality and delete the guidance blockquotes. It loads into context at the start of every session (and into every subagent), so it's the highest-leverage document in this setup — it's what stops each agent from re-reading your whole codebase to orient itself.
>
> Keep it **thin**. This root file is a map plus the cross-cutting facts that apply everywhere. Area-specific detail belongs in an area-level `CLAUDE.md` (see "Two-tier pattern" below) so it loads only when an agent works in that area.

## What this project is

<One paragraph: what the product does, who it's for, what stage it's at. The agents make better calls when they know whether this is a pre-launch prototype or a system with paying users.>

## Repo map

> The single most useful section. An agent that knows where things live doesn't grep blindly.

| Directory | What it is | Read first |
|---|---|---|
| `<dir>/` | <what lives here> | `<dir>/CLAUDE.md` |
| `<dir>/` | <what lives here> | `<README or key file>` |

## Stack at a glance

<Languages, frameworks, key libraries, datastore, hosting. State the non-obvious choices explicitly — "state via X, not Y"; "plain types, no codegen" — so agents match your patterns instead of importing defaults from training data.>

## Conventions (MUST)

> The rules an agent must follow to write code that looks like yours. Be specific and name the canonical example.

- <e.g., "Use the design tokens in `<path>`, never hardcoded colors/spacing.">
- <e.g., "All I/O goes through the service layer; UI never calls the datastore directly.">
- <e.g., "Run `<lint command>` and `<test command>` before considering a change done.">

## Invariants (load-bearing — break these and things silently corrupt)

> The things that look safe to change but aren't. Server-authoritative fields the client must never write; documents that are immutable from one side; ordering/dedupe guarantees; write-once fields. This is the section that prevents the most expensive mistakes.

- <invariant + why it matters>

## Anti-patterns (MUST NOT)

- <e.g., "Don't add defensive checks for 'can't happen' cases — trust framework guarantees; validate only at boundaries.">
- <e.g., "Don't introduce a new state container when a derived value would do.">

## Vocabulary

> If your domain has specific terms (and a "wrong" synonym you never use), define them here so copy, code, and comments stay consistent.

- **<term>** — <definition>. Never call it <wrong synonym>.

---

## Two-tier pattern (why this file stays short)

Put **cross-cutting** facts here. Put **area-specific** facts in an area-level `CLAUDE.md` next to the code:

```
CLAUDE.md                 # this file — always loaded; thin map + global rules
<area>/CLAUDE.md          # loaded only when an agent reads a file in <area>/
<other-area>/CLAUDE.md    # area-specific conventions, quirks, gotchas
```

An area-level `CLAUDE.md` is where the dense, load-bearing detail goes: the quirks of that subsystem, its file-by-file layout, the invariants local to it. Because it loads only when an agent touches that area, you can be exhaustive there without taxing every other task's context budget.

## Skill / agent routing (optional but recommended)

> Tell the orchestrator which specialist owns which kind of decision, so it routes instead of guessing.

- Implementation, debugging, refactors → `senior-software-engineer`
- Structural / data-model / cost decisions → `solutions-architect`
- UI, flows, accessibility, copy → `ui-ux-designer` (+ your design-system skill if you have one)
- "Is this safe?" — auth, secrets, rules → `security-engineer`
- Privacy / app-store / subscription compliance → `legal-compliance`
- What to build, in what order, and why → `product-lead`
- Pricing, unit economics, strategy → `business-mentor`

---
> Source: [abidwaqar/AI-Judgment-Orchestration](https://github.com/abidwaqar/AI-Judgment-Orchestration) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
