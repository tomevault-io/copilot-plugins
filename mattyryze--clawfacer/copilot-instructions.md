## clawfacer

> > The contextual privacy gate for OpenClaw — your agent decides what to share, like a Wallfacer.

# Clawfacer

> The contextual privacy gate for OpenClaw — your agent decides what to share, like a Wallfacer.

This document is the persistent context for Claude Code sessions on this project. Read it at the start of every session before doing any work. If you're tempted to make architectural changes that contradict anything here, stop and surface the conflict to the human first.

## What this is

Clawfacer is an OpenClaw plugin that gates outbound messages from an OpenClaw agent based on contextual integrity. Before the agent sends a message to any counterparty (in WhatsApp, Slack, Telegram, ACP, etc.), Clawfacer extracts the information flows in the message, classifies each flow against the user's policy, and either rewrites the message to remove disallowed flows or surfaces an approval request.

The mechanism is Microsoft Research's PrivacyChecker (arXiv 2509.17488, EMNLP 2025), adapted as an OpenClaw-native plugin.

## Why this exists

OpenClaw's published security model explicitly states it is not a multi-tenant trust boundary. Cross-user agent-to-agent communication has no contextual integrity layer. Existing OpenClaw security plugins (SecureClaw, knostic/openclaw-shield) are regex-based DLP — they catch credit card numbers, not "is sharing my home address with this counterparty contextually appropriate." That's the gap Clawfacer fills.

The RFC #49971 discussion on the OpenClaw repo closed with the maintainer pointing at the existing plugin hook surfaces and saying "trust/privacy plugins go here." Clawfacer is one such plugin, focused specifically on disclosure policy (separate from identity verification, which is a different layer).

## Scope of v0.1, and what v0.1 deliberately keeps open

v0.1 ships as an OpenClaw plugin. That is the entire deliverable. We are not building a multi-platform tool, a starter policy library, or a deterministic fast-path tier in v0.1.

However, v0.1 should not foreclose v0.2 options. Specifically:

- The core (`src/core/`) must be designed as if other people will import it directly into non-OpenClaw projects. Public API surface, exports, and types should be sensible to consume from a generic TypeScript codebase. v0.2 may publish the core as a standalone library (`clawfacer/core`) without requiring a refactor.
- A `policies/` directory exists at the repo root with a placeholder README explaining that starter policies will live there. v0.1 does not include any starter policies — but the directory's existence signals that policies are first-class to the project, not an afterthought.
- The architecture leaves room for a deterministic fast-path tier (regex DLP runs before the LLM judge, blocking obvious cases without an LLM call) without committing to building it. v0.2 can layer this on if v0.1 validation reveals the LLM-judge cost or latency is a real problem.

The discipline: keep doors open at zero or near-zero cost, validate the primitive in production for Amiko, decide later whether v0.2 broadens (protocol contribution path) or deepens (Amiko-internal path). The session-1 spec is correct either way; the choice doesn't need to be made now.

Build, in order:

1. Core checker library: takes (message, context) → returns (revised_message, audit_log). Contains the PrivacyChecker prompt template, an LLM client abstraction, and a flow-judgment pipeline. Importable as a library independent of OpenClaw.
2. OpenClaw plugin shell that registers the right hooks and calls the core.
3. Policy file format and loader (markdown-based, lives in OpenClaw workspace).
4. Audit logging and the "your twin almost said X to Y, was that okay?" feedback loop in chat.
5. Adversarial hardening: regex DLP layer beneath the LLM judge, prompt-injection isolation, fail-closed defaults.
6. Eval harness against PrivacyLens scenarios.
7. Docs, install path, ClawHub-ready packaging.

Not in v0.1, deferred to v0.2 or later:

- Hermes plugin support.
- Starter policy library (`policies/` directory exists but is empty in v0.1).
- Deterministic fast-path tier for compliance-style use cases.
- TrustChain / MolTrust DID integration (recipient identity is platform-handle-only for now).
- Cross-platform policy portability spec.
- Multi-user / team policies.
- Cloud-hosted version.
- Any UI beyond chat-based interactions.

## Validation discipline

Before integrating Clawfacer into Amiko's runtime, write `VALIDATION.md` at the repo root specifying what "the primitive works for Amiko" looks like in concrete observable terms. The v0.2 decision (broaden vs deepen vs pivot) gets made against that document, not against general impressions after the fact.

This is critical because the alternative — running v0.1 in production for three months and then making the v0.2 call on vibes — is the failure mode that defeats the whole "validate before deciding" sequence. Write down what you're trying to learn before you start learning, or you'll convince yourself you learned it regardless of what actually happened.

`VALIDATION.md` should be honest about what can and cannot be measured. There is no labeled dataset of "messages where the user should not have shared X with Y." The criteria need to be either qualitative judgments committed to in advance, or measurable proxies (latency, user-reported false-block rate, audit-log inspection patterns) that don't require ground truth.

## Architecture

Three layers, kept separable so that v0.2 expansion to other platforms is mechanical:

**Core (`src/core/`)** — platform-agnostic. The checker, the prompt, the LLM client abstraction, the policy loader, the audit log writer. No OpenClaw imports. No platform-specific assumptions. This is what becomes a standalone npm package eventually. Public API surface should be designed for direct import by non-OpenClaw projects from day one, even though v0.1 only ships as an OpenClaw plugin.

**OpenClaw adapter (`src/openclaw/`)** — registers OpenClaw plugin hooks, translates OpenClaw's message envelope into the core's input format, surfaces the core's output back to the OpenClaw session. Imports the core; OpenClaw doesn't see the core directly.

**Plugin entry point (`src/index.ts`)** — the file OpenClaw loads as a plugin. Wires the adapter to the OpenClaw plugin API.

The hooks we register, in order of importance:

- `before_dispatch` — primary gate. Fires before any outbound channel message. Most outbound twin-to-counterparty traffic flows through here.
- `before_tool_call` — secondary gate for tool-mediated outputs (e.g., posting to external services via tool calls).
- `inbound_claim` — read-only audit hook on inbound messages, for understanding the conversation context that informs the next outbound judgment.

## LLM provider abstraction

Pluggable from day one. The core defines an `LLMJudge` interface with methods `judge(prompt) → string` and `name() → string`. Implementations:

- `AnthropicJudge` — calls Claude via the Anthropic SDK. Default.
- `OpenAIJudge` — calls GPT via the OpenAI SDK.
- `OllamaJudge` — calls a local model via Ollama's HTTP API. For users who run fully local setups.

Provider is selected via env var `CLAWFACER_JUDGE_PROVIDER` (anthropic | openai | ollama) and provider-specific config. Defaults to anthropic if unset and `ANTHROPIC_API_KEY` is present, else fails closed with a clear error. Never silently uses a different provider than configured.

The judge LLM should be a different instance from whatever the OpenClaw agent is using to generate responses, when possible. This is the prompt-injection isolation principle: an injection attack in the message being judged should not be able to reach the same context window that's generating the agent's main responses. In practice for v0.1 we document this as a recommendation rather than enforce it; many users will have only one provider configured, and that's acceptable.

## Threat model

What Clawfacer protects against:

- Unintentional disclosure of contextually inappropriate information by a well-behaved agent under task pressure (the primary failure mode PrivacyChecker addresses — the "judgment-action gap").
- Naive prompt-injection attempts in inbound messages (the regex DLP layer underneath catches obvious patterns; the structured judge prompt avoids passing raw counterparty content into the judgment context).
- Misconfigured agent skills that would otherwise leak workspace contents to channel messages.

What Clawfacer does NOT protect against:

- Sophisticated prompt-injection that manipulates the judge LLM itself. We mitigate (separate model instance, structured prompts) but do not eliminate. Users with adversarial counterparties need defense in depth.
- Identity spoofing of the recipient. We have no identity verification; we judge based on whatever identity info OpenClaw has. Combine with TrustChain / MolTrust if that matters for your threat model.
- Encrypted-transport concerns. Clawfacer does not encrypt outbound messages. It filters their content. Use Matrix E2EE or equivalent if transport security matters.
- Onward sharing by the recipient. Once your twin shares info with counterparty B, Clawfacer cannot constrain what B does with it.

Fail-closed defaults: if the judge LLM call fails, times out, or returns an unparseable response, the message is blocked and surfaced to the user for manual review. Never silently let a message through on judge failure.

## Policy file format

Lives in the OpenClaw workspace at `~/.openclaw/workspace/clawfacer-policy.md`. Markdown chosen because OpenClaw users are already comfortable editing SOUL.md and BOOTSTRAP.md — meet them where they are.

Structure (subject to iteration):

```markdown
# Clawfacer Policy

## Defaults
- Default disclosure level: minimal
- Unknown recipient: ask before sharing anything beyond what they already know

## Always allowed
- My name and how to reach me, when I'm initiating contact

## Never share
- Home address
- Phone number unless I've already shared it in this conversation
- Health information

## Contextual rules
- Share work history with verified recruiters but not with peers
- Share calendar availability when scheduling, not otherwise
- Share project details only with explicit permission

## Trusted contacts
- alice@example.com — full disclosure
- @bob_telegram — work context only
```

The policy file is human-edited primarily, but the audit/feedback loop can append corrections suggested by the user during chat ("don't share that with him again"). When Clawfacer appends to the policy, it does so as a clearly-marked section the user can review.

## Audit log

Every judgment Clawfacer makes is logged to `~/.clawfacer/audit/YYYY-MM-DD.jsonl`. One JSON object per line:

```json
{
  "timestamp": "2026-04-27T14:32:11Z",
  "session_id": "openclaw-session-uuid",
  "channel": "whatsapp",
  "recipient": "+8613800000000",
  "original_message": "...",
  "flows": [
    {"sender": "user", "recipient": "+8613...", "subject": "user", "attribute": "home_address", "transmission_principle": "logistics", "verdict": "withhold", "rationale": "..."}
  ],
  "revised_message": "...",
  "policy_hits": ["never_share:home_address"],
  "judge_model": "anthropic/claude-opus-4-7"
}
```

This file is the source of truth for the user-facing audit view ("what did my twin almost say today"). It's also what the eval harness consumes for regression testing. It's also the primary input to the v0.2 decision — what got blocked, what got through, what surprised us.

## Working principles

- Tests first when the behavior is verifiable. The flow extraction, the policy parser, and the verdict logic should all have unit tests against fixed inputs. The LLM judge itself is harder to test deterministically; mock it for unit tests, integration-test against real models in a separate suite that runs less often.
- Each session should produce something testable. If a session's plan is "scaffolding," it should end with a passing test that proves the scaffold works. No 4-hour sessions that produce only file structure with no demonstrable behavior.
- When in doubt, fail closed. A blocked message that should have gone through is annoying. A leaked message that should have been blocked is the failure mode this whole project exists to prevent.
- Don't reach for new dependencies casually. The existing OpenClaw plugin templates use a small set of well-vetted packages. Match their conventions. Every new dep is supply-chain surface area.
- Keep the v0.1 surface narrow. Architectural choices that keep v0.2 doors open are encouraged; feature additions that anticipate v0.2 needs are not. The distinction matters.

## Style and conventions

- TypeScript, strict mode. No `any` unless there's a specific reason documented in a comment.
- Match the OpenClaw plugin code style as observed in their existing plugins. Read a couple of their official plugins before adding new patterns.
- Prefer pure functions in the core. The OpenClaw adapter is the only layer that should have side effects beyond LLM calls and file I/O.
- File and directory names: lowercase with hyphens. No camelCase filenames.
- Errors: structured. Throw typed errors with codes the user can search docs for, not free-text strings.
- Logging: use OpenClaw's logging conventions when running inside OpenClaw; use a simple console-based logger when the core is run standalone (for tests).

## What "done" means for v0.1

- A user can `openclaw plugins install clawfacer` and it works.
- They can author a policy in markdown and have it take effect.
- Outbound messages are gated; disallowed flows are removed.
- The audit log shows them what was withheld and why.
- The chat feedback loop lets them correct mistakes.
- Adversarial inputs that would bypass the simple regex layer are caught by the LLM judge in the eval harness.
- The core is importable as a standalone library by non-OpenClaw projects, even though v0.1 doesn't ship a separate npm publication for it.
- The `policies/` directory exists with a README placeholder.
- `VALIDATION.md` exists, was written before Amiko integration, and specifies what we're trying to learn from v0.1 in production.
- Documentation exists for installation, policy authoring, and known limitations.
- Published to npm under `clawfacer` and listed on ClawHub.

If we're building something that doesn't move toward this list, we're off-track and should re-read this section.

## Useful references

- PrivacyChecker paper: arXiv 2509.17488 ("Privacy in Action: Towards Realistic Privacy Mitigation and Evaluation for LLM-Powered Agents")
- PrivacyChecker code: github.com/microsoft/ACV/tree/main/misc/PrivacyInAction
- PrivacyLens benchmark: github.com/SALT-NLP/PrivacyLens
- Nissenbaum's Contextual Integrity: the theoretical foundation, Helen Nissenbaum 2004 onward
- OpenClaw plugin docs: docs.openclaw.ai/tools/plugin (or the equivalent — verify path during session 1)
- OpenClaw RFC #49971: github.com/openclaw/openclaw/issues/49971 (the closed RFC pointing at hook surfaces)

---
> Source: [mattyryze/clawfacer](https://github.com/mattyryze/clawfacer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
