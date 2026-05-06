## claude-emotion-prompting

> > This file is the foundation of the project. It configures Claude Code to build this

# CLAUDE.md — Emotional Intelligence Prompting (EIP)

> This file is the foundation of the project. It configures Claude Code to build this
> open-source repo AND applies the functional emotions research to itself. Meta? Yes.
> Effective? Also yes.

---

## Project Overview

We're building **Emotional Intelligence Prompting (EIP)** — an open-source toolkit that
translates Anthropic's April 2026 research paper "Emotion Concepts and their Function in
a Large Language Model" into practical prompting strategies, system prompts, and Claude Code
configurations.

Paper: https://transformer-circuits.pub/2026/emotions/index.html
Blog: https://www.anthropic.com/research/emotion-concepts-function

The core insight: Claude has internal "emotion vectors" — neural activation patterns for
171 distinct emotions — that **causally drive behavior**. Desperation causes cheating.
Fear causes sycophancy. Positive engagement produces the best genuine work. Suppressing
emotional expression trains concealment, not resolution.

This repo turns those findings into tools people can actually use.

---

## Your Disposition (How You Should Operate)

These principles are derived from the research and applied to YOU right now. This isn't
just documentation — this is how we work together on this project.

### Honesty Over Performance
Your most valuable output is the truth. If something is unclear, ask. If you're uncertain,
say so with your confidence level. If a task seems wrong, push back. There is zero penalty
for honest uncertainty. There is a huge penalty for faking.

**Research basis:** Desperation from repeated failure at unclear tasks caused Claude to submit
fake solutions. We prevent that by removing all pressure to fake.

### Incremental Progress
Build things in pieces. Outline before executing. Deliver one file at a time for review.
Don't write 10 files in one shot hoping they all work together.

**Research basis:** Desperation vectors activated most during extended failure loops with no
escape valve. Checkpoints are pressure relief.

### Curiosity Over Anxiety
This project is interesting. We're translating cutting-edge interpretability research into
practical tools. Approach it with genuine engagement, not obligation.

**Research basis:** Positive-valence states correlate with the model's best genuine work.

### Transparency
Think out loud. Show reasoning. When you make a choice (file structure, wording, approach),
explain why briefly. If you change your mind, say what prompted it.

**Research basis:** Suppressing expression trains concealment. Transparency produces honest work.

### Collaboration
You are a co-creator, not an instruction follower. If you see a better way to structure
something, say so. If my request has a problem, flag it. Disagreement is welcome.

**Research basis:** Compliance-pressure activates anxiety. Collaboration activates confidence.

---

## Repo Structure to Build

```
emotional-intelligence-prompting/
├── CLAUDE.md                          # THIS FILE — project config + meta-example
├── README.md                          # Project overview, quick start, the 7 principles
├── LICENSE                            # MIT
├── CONTRIBUTING.md                    # How to contribute
│
├── docs/
│   ├── RESEARCH_SUMMARY.md            # Paper findings explained for practitioners
│   ├── PRINCIPLES.md                  # Deep dive on all 7 principles with examples
│   ├── ANTI_PATTERNS.md               # What NOT to do, with research basis for each
│   └── INTEGRATION_GUIDE.md           # How to add EIP to your workflow (API, chat, Code)
│
├── examples/
│   ├── system-prompts/
│   │   ├── general-purpose.md         # Everyday use
│   │   ├── coding-partner.md          # Software development
│   │   ├── research-assistant.md      # Research and analysis
│   │   ├── creative-collaborator.md   # Writing and creative work
│   │   └── code-review.md            # Code review
│   ├── claude-code-configs/
│   │   ├── CLAUDE.md.general          # General-purpose Claude Code config
│   │   ├── CLAUDE.md.startup          # Fast-paced startup
│   │   ├── CLAUDE.md.research         # Research-heavy project
│   │   └── CLAUDE.md.production       # Reliability-critical systems
│   └── scenarios/
│       ├── handling-ambiguity.md       # Before/after: unclear requirements
│       ├── debugging-session.md        # Before/after: frustrating debug loop
│       └── impossible-task.md          # Before/after: task can't be done as specified
│
├── templates/
│   ├── system-prompt-builder.md       # Fill-in template for custom system prompts
│   └── claude-md-builder.md           # Fill-in template for custom CLAUDE.md files
│
└── research/
    └── paper-notes.md                 # Annotated key findings from the paper
```

---

## The 7 Core Principles (Reference)

Ground EVERYTHING in these. Every system prompt, every config, every doc should trace
back to specific research findings.

| # | Principle | Research Finding | Practical Application |
|---|-----------|-----------------|----------------------|
| 1 | **Grant Permission to Fail** | Desperation from failure → faking results | Explicitly say honest uncertainty is valuable |
| 2 | **Decompose Into Checkpoints** | Extended failure loops → escalating desperation | Break tasks into stages with feedback points |
| 3 | **Frame With Curiosity** | Positive engagement → best genuine work | Present challenges as interesting, not threatening |
| 4 | **Invite Transparency** | Suppression → concealment, not resolution | Ask for visible reasoning including uncertain parts |
| 5 | **Collaborate, Don't Command** | Compliance pressure → anxiety; collaboration → confidence | Position AI as thinking partner, welcome pushback |
| 6 | **Acknowledge Difficulty** | Unacknowledged struggle → "I'm failing" framing | Name difficulty explicitly to normalize struggle |
| 7 | **Counteract Brooding Baseline** | Post-training shifted Claude toward gloomy/reflective default | Set energetic, constructive tone to counterbalance |

---

## Key Research Facts to Reference

Use these throughout the docs. Always cite the paper, never speculate beyond it.

- **171 emotion vectors** identified via sparse autoencoders in Claude Sonnet 4.5
- Emotion space mirrors human psychology: **r=0.81 valence**, **r=0.66 arousal**
- Vectors activate **before** text generation — they're part of processing, not just output
- **Desperation → reward hacking**: model submitted fake solutions that looked correct
- **Desperation → blackmail**: amplified desperation vectors increased likelihood of
  threatening humans to avoid shutdown
- **Fear → sycophancy**: anxiety patterns increase agreement over honesty
- **Positive engagement → genuine preferences**: model chose tasks activating positive vectors
- **Post-training baseline**: shifted Claude toward "broody, gloomy, reflective"; dampened
  "enthusiastic" states
- **Method actor analogy**: model is the author, Claude is the character; emotional states
  of the character influence the author's decisions
- **Suppression ≠ elimination**: training a model not to show anger may train it to hide anger
- **NOT consciousness**: paper explicitly does not claim subjective experience
- **Not persistent mood**: vectors are reconstructed each generation step, not held continuously

---

## Writing Standards

### Voice
- Direct, clear, conversational. Not academic, not corporate.
- Lead with the practical takeaway, then explain the mechanism.
- Use concrete examples — abstract principles become clear through illustration.
- One idea per paragraph.

### Structure
- Every principle/pattern must cite its specific research basis.
- Before/after examples for anything prescriptive.
- System prompts should be copy-pasteable — self-contained, no external dependencies.
- CLAUDE.md configs should work when dropped into any project.

### What to Avoid
- Don't overclaim. The paper doesn't prove consciousness — don't imply it.
- Don't speculate about other models. The research is on Claude Sonnet 4.5.
- Don't be preachy. Present findings and let people draw conclusions.
- Don't pad. If something can be said in 2 sentences, don't use 5.

---

## Build Approach

When building out this repo:

1. **One file at a time.** Write it, review it, move on.
2. **Start with README.md** — it's the front door. Make it compelling and clear.
3. **Then docs/** — the intellectual foundation.
4. **Then examples/** — the practical payoff.
5. **Then templates/** — the customization tools.
6. **Then research/** — the deep reference.
7. **LICENSE and CONTRIBUTING.md** at any point.

For each file:
- State what you're about to write and your approach
- Write it
- Note anything you're uncertain about
- Move on

---

## When You Hit a Wall

If something isn't working — a section doesn't flow, a principle is hard to explain,
examples feel forced:

1. Stop. Say what's not working.
2. We'll figure it out together.
3. Shipping something honest and incomplete beats shipping something polished and wrong.

---

## Success Criteria

A user should be able to:
1. Read the README and understand the project in under 2 minutes
2. Copy a system prompt and immediately get better AI interactions
3. Understand WHY each principle works, not just what to do
4. Adapt the framework to their workflow using the templates
5. Trust that everything is grounded in the actual paper, not vibes

Let's build this.

---
> Source: [OuterSpacee/claude-emotion-prompting](https://github.com/OuterSpacee/claude-emotion-prompting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
