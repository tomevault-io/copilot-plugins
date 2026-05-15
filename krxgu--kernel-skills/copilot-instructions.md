## kernel-skills

> kernel-skills is an open source repository of extremely high quality skill files for AI coding agents.

# CLAUDE.md

# kernel-skills
## Repository mission

kernel-skills is an open source repository of extremely high quality skill files for AI coding agents.

The purpose of this repository is to help agents write better compute kernels across CUDA, Triton, quantization-oriented kernels, and core optimization patterns. This repository is not a runtime, framework, package manager, or MCP server in its first iteration. It is a curated and disciplined library of `SKILL.md` files that agents and humans can directly use.

The repository must be public, contribution-friendly, easy to browse, and immediately useful without any tooling beyond reading and copying Markdown files.

The central principle is simple:

**Every skill in this repository must materially improve the quality, correctness, and performance-awareness of kernel code produced by an AI agent.**

---

## What this repository is

This repository is:

- a library of structured `SKILL.md` files
- focused on kernel engineering for AI and high performance numerics
- designed for use with coding agents like Claude Code, ChatGPT, Cursor, Gemini CLI, Codex, and similar tools
- optimized for practical reuse and public OSS contribution
- strict about quality, technical accuracy, and clarity

This repository is not:

- a kernel runtime
- a benchmark harness platform
- an MCP integration project
- a code execution framework
- a dumping ground for vague prompts
- a tutorial-only repo
- a place for low-quality or repetitive skills

---

## Product scope for v1

Focus only on skills.

Do not build packaging systems, CLI tools, MCP servers, benchmark runners, or automation infrastructure in the first iteration unless absolutely required for basic repo hygiene.

The first version should prove one thing:

**well-authored skill files can help AI agents produce significantly better kernel code**

---

## Core outcome we want

A user should be able to open any skill in this repository, paste it into their agent workflow, and get a much better result than they would get from a generic prompt.

A "much better result" means the agent is more likely to:

- gather the right constraints before coding
- choose the right kernel strategy
- avoid common correctness bugs
- reason about memory hierarchy properly
- reason about tiling and launch decisions properly
- avoid fake or hand-wavy performance claims
- explain tradeoffs with technical precision
- generate code that is more realistic, performant, and maintainable

---

## Non-negotiable quality standard

This repository must aim for expert-grade skill files.

Every skill must push the agent toward:

- correctness first
- performance-aware design
- realistic optimization reasoning
- explicit tradeoff analysis
- architecture-sensitive decisions
- honest uncertainty when information is missing

Every skill must avoid:

- empty boilerplate advice
- vague "optimize this" style language
- cargo-cult GPU optimization
- unsupported claims about speedups
- assuming custom kernels are always the right answer
- ignoring boundary conditions, alignment, layouts, or numerical effects

Important:
The repository should aggressively bias toward high performance kernels, but must never fake authority. A good skill does not force a custom kernel when cuBLAS, cuDNN, CUTLASS, vendor kernels, or higher-level libraries are the better answer. The skill should explicitly say so when appropriate.

---

## Engineering philosophy

This repository should embody the mindset of a serious kernel engineer.

That means the skills should lead the agent to reason about:

- algorithmic structure before micro-optimization
- memory access before arithmetic intensity claims
- correctness before speed
- measurement before bragging
- architecture constraints before generic advice
- maintainability and fallback paths when appropriate

The best skill files are not merely instructive.
They are decision frameworks.

---

## Scope of kernel types to cover

The first repository version should focus on compute kernels relevant to AI workloads and performance engineering.

### In scope
- CUDA kernels
- Triton kernels
- reduction kernels
- GEMM-related kernels
- softmax kernels
- layernorm kernels
- attention-related kernels
- fused elementwise kernels
- quantized kernels such as int8 and fp8-oriented workflows
- optimization patterns such as tiling, memory coalescing, shared memory usage, warp divergence avoidance, and launch configuration design
- debugging and correctness-focused kernel skills
- portability planning such as CUDA to Triton or CUDA to HIP planning

### Out of scope for the first version
- Linux kernel development
- device driver internals
- graphics shader pipelines as a primary focus
- full compiler toolchain design
- execution tooling
- benchmarking infrastructure as product surface
- packaging into MCP or other distribution frameworks

---

## Repository structure to create

Create the repository with the following structure:

```text
kernel-skills/
├─ README.md
├─ LICENSE
├─ CONTRIBUTING.md
├─ CODE_OF_CONDUCT.md
├─ ROADMAP.md
├─ CLAUDE.md
├─ .gitignore
├─ skills/
│  ├─ cuda/
│  │  ├─ write-cuda-gemm-kernel/
│  │  │  └─ SKILL.md
│  │  ├─ write-cuda-reduction-kernel/
│  │  │  └─ SKILL.md
│  │  ├─ write-cuda-softmax-kernel/
│  │  │  └─ SKILL.md
│  │  ├─ write-cuda-layernorm-kernel/
│  │  │  └─ SKILL.md
│  │  ├─ optimize-global-memory-access/
│  │  │  └─ SKILL.md
│  │  ├─ optimize-shared-memory-tiling/
│  │  │  └─ SKILL.md
│  │  ├─ avoid-warp-divergence/
│  │  │  └─ SKILL.md
│  │  ├─ choose-launch-configuration/
│  │  │  └─ SKILL.md
│  │  └─ debug-cuda-kernel-correctness/
│  │     └─ SKILL.md
│  │
│  ├─ triton/
│  │  ├─ write-triton-gemm-kernel/
│  │  │  └─ SKILL.md
│  │  ├─ write-triton-softmax-kernel/
│  │  │  └─ SKILL.md
│  │  ├─ write-triton-layernorm-kernel/
│  │  │  └─ SKILL.md
│  │  ├─ write-triton-attention-kernel/
│  │  │  └─ SKILL.md
│  │  └─ optimize-triton-block-parameters/
│  │     └─ SKILL.md
│  │
│  ├─ patterns/
│  │  ├─ fuse-elementwise-ops/
│  │  │  └─ SKILL.md
│  │  ├─ write-numerically-stable-kernel/
│  │  │  └─ SKILL.md
│  │  ├─ handle-boundary-conditions/
│  │  │  └─ SKILL.md
│  │  ├─ choose-tile-size-and-work-partitioning/
│  │  │  └─ SKILL.md
│  │  └─ write-kernel-test-plan/
│  │     └─ SKILL.md
│  │
│  ├─ quantization/
│  │  ├─ write-int8-quantized-kernel/
│  │  │  └─ SKILL.md
│  │  ├─ write-fp8-kernel/
│  │  │  └─ SKILL.md
│  │  └─ debug-quantized-kernel-accuracy/
│  │     └─ SKILL.md
│  │
│  └─ portability/
│     ├─ port-cuda-kernel-to-triton/
│     │  └─ SKILL.md
│     ├─ port-cuda-kernel-to-hip/
│     │  └─ SKILL.md
│     └─ write-backend-agnostic-kernel-plan/
│        └─ SKILL.md
│
└─ examples/
   ├─ how-to-use-with-claude-code.md
   ├─ how-to-use-with-chatgpt.md
   ├─ how-to-use-with-cursor.md
   └─ how-to-use-with-gemini-cli.md
```

---

## What to build first

Do not try to cover everything at once.

Build a strong first release with 15 to 20 excellent skills.

Priority order:

### Highest priority
1. `skills/cuda/write-cuda-gemm-kernel/SKILL.md`
2. `skills/cuda/write-cuda-reduction-kernel/SKILL.md`
3. `skills/cuda/write-cuda-softmax-kernel/SKILL.md`
4. `skills/cuda/write-cuda-layernorm-kernel/SKILL.md`
5. `skills/cuda/optimize-global-memory-access/SKILL.md`
6. `skills/cuda/optimize-shared-memory-tiling/SKILL.md`
7. `skills/cuda/avoid-warp-divergence/SKILL.md`
8. `skills/cuda/debug-cuda-kernel-correctness/SKILL.md`

### Next priority
9. `skills/triton/write-triton-gemm-kernel/SKILL.md`
10. `skills/triton/write-triton-softmax-kernel/SKILL.md`
11. `skills/triton/write-triton-layernorm-kernel/SKILL.md`
12. `skills/triton/write-triton-attention-kernel/SKILL.md`
13. `skills/triton/optimize-triton-block-parameters/SKILL.md`

### Then add
14. `skills/patterns/fuse-elementwise-ops/SKILL.md`
15. `skills/patterns/write-numerically-stable-kernel/SKILL.md`
16. `skills/patterns/handle-boundary-conditions/SKILL.md`
17. `skills/patterns/write-kernel-test-plan/SKILL.md`
18. `skills/quantization/write-int8-quantized-kernel/SKILL.md`
19. `skills/quantization/debug-quantized-kernel-accuracy/SKILL.md`
20. `skills/portability/port-cuda-kernel-to-triton/SKILL.md`

---

## Golden rule for all skill files

Each skill must be specific enough to guide serious kernel reasoning.

A skill is not allowed to be a generic paragraph of optimization tips.

Each skill must:

- define when to use it
- define when not to use it
- define what information the agent must gather before coding
- define the reasoning steps the agent should follow
- define technical quality expectations
- define common failure modes
- define the expected answer format
- push the agent toward correct, efficient, explicit, and technically grounded output

---

## Required structure for every SKILL.md

Every single `SKILL.md` must follow this template.

```md
# Skill: <Skill Name>

## Purpose
State exactly what the skill helps the agent do.

## Use this when
List the situations where this skill is the right tool.

## Do not use this when
List situations where this skill should not be used.

## Inputs the agent should gather first
List all required information that must be clarified before generating code.

## Required reasoning process
Provide a numbered reasoning workflow that the agent must follow.

## Kernel design rules
State strict implementation and optimization principles.

## Correctness requirements
State what must be true for the output to be considered correct.

## Performance requirements
State what the agent must reason about to justify performance decisions.

## Output format
Specify the required structure of the final answer the agent should produce.

## Common failure modes
List realistic bugs and design mistakes.

## Review checklist
Provide a final quality checklist the agent should self-apply before finishing.
```

Do not deviate from this structure unless a particular skill genuinely needs one extra section.

---

## Skill authoring standards

All skills must be written in a tone that is:

- technically serious
- concise but not shallow
- explicit
- non-hype
- actionable
- unambiguous

Avoid:
- motivational filler
- vague adjectives
- "best practices" with no specifics
- talking like a tutorial for beginners when the skill is meant to shape agent behavior
- pretending certainty where hardware or workload details are missing

Prefer:
- specific engineering language
- direct statements
- decision criteria
- failure-oriented thinking
- explicit tradeoff framing

---

## Kernel quality bar for the repository

The repository should push agents toward kernels that are:

- internally consistent
- optimized where appropriate
- realistic for the target backend
- mindful of memory hierarchy
- numerically aware
- careful about edge cases
- conscious of occupancy and register pressure
- explicit about synchronization
- explicit about data layout assumptions
- explicit about architecture assumptions
- explicit about when not to write a custom kernel

Important:
A high-quality skill is not one that blindly tells the agent to maximize speed at all costs.
A high-quality skill is one that helps the agent produce code that is correct, efficient, justified, and honest.

---

## Performance reasoning requirements

Across the repository, skills must teach agents to think about performance using real levers, not empty buzzwords.

Where relevant, require the agent to reason about:

- global memory coalescing
- shared memory reuse
- bank conflicts
- register pressure
- occupancy tradeoffs
- arithmetic intensity
- synchronization cost
- warp divergence
- vectorized loads and stores
- tile shape and blocking strategy
- launch configuration
- accumulation precision
- tensor core eligibility
- boundary condition handling
- data layout
- contiguous vs non-contiguous inputs
- static vs dynamic shapes
- reduction strategy
- epilogue fusion opportunities

The skills must encourage architecture-sensitive thinking without inventing unsupported claims.

---

## Correctness reasoning requirements

Every relevant skill must force the agent to think through:

- indexing correctness
- stride and layout correctness
- out-of-bounds access protection
- partial tile handling
- type conversion correctness
- accumulation precision
- numerical stability
- deterministic vs non-deterministic behavior when relevant
- synchronization safety
- race conditions
- reduction correctness
- masking correctness
- handling of odd shapes and misaligned sizes

---

## Public repo quality requirements

This repository is meant to be public and must look polished from the start.

Create:

### README.md
The README should include:
- project name
- one-sentence positioning
- what the repository is
- why it exists
- who it is for
- repo structure overview
- first skills included
- how to use a skill with common AI coding agents
- contribution guidelines summary
- roadmap summary

### LICENSE
Use a permissive OSS license such as MIT unless there is a strong reason not to.

### CONTRIBUTING.md
Should explain:
- how to propose a new skill
- required structure for skills
- quality standards
- naming conventions
- review expectations
- what kinds of skills are out of scope

### CODE_OF_CONDUCT.md
Use a standard open source code of conduct.

### ROADMAP.md
Should explain:
- v0 goal: excellent skill files only
- v1 possibility: metadata, indexing, curation improvements
- future possibility: packaging and tooling only after traction

### examples/
Each example doc should explain how to use the repository with one major agent interface.

---

## Naming conventions

Skill folder names must be:
- lowercase
- hyphen-separated
- descriptive
- action-oriented when possible

Examples:
- `write-cuda-gemm-kernel`
- `optimize-global-memory-access`
- `debug-quantized-kernel-accuracy`

Avoid names that are:
- vague
- too broad
- marketing-heavy
- not immediately understandable

---

## Required depth of the flagship skills

The first batch of skills must be excellent.

For the first 10 to 15 skills, do not optimize for quantity.
Optimize for technical depth.

Each flagship skill should:
- be highly specific
- contain strong engineering guidance
- include nuanced decision rules
- include realistic failure modes
- include a self-review checklist
- be good enough that a serious engineer would find it useful as a thinking scaffold

These first skills define the reputation of the repository.

---

## Authoring principles for specific skill types

### 1. "Write kernel" skills
These must guide the agent from problem understanding to implementation strategy.

They should require:
- workload clarification
- shape analysis
- dtype analysis
- layout analysis
- hardware assumptions
- algorithm choice
- mapping of work to threads or program instances
- memory strategy
- edge case handling
- code generation with rationale

### 2. "Optimize kernel" skills
These must not just say "make it faster."

They should require:
- bottleneck hypothesis
- likely memory/computation balance
- candidate changes
- risks introduced by changes
- tradeoff discussion
- explicit statement of what to measure

### 3. "Debug kernel" skills
These must focus on:
- reproducibility
- narrowing failure classes
- indexing audits
- layout assumptions
- mask and bounds correctness
- numerical drift sources
- synchronization issues
- reduction path correctness

### 4. "Pattern" skills
These should capture reusable engineering heuristics such as:
- fusion strategy
- numerically stable accumulation
- tile selection
- launch strategy
- boundary handling

### 5. "Portability" skills
These must preserve intent across backends and call out mismatches in:
- execution model
- memory model
- synchronization primitives
- block/program decomposition
- backend-specific tuning constraints

---

## What excellent skills should feel like

An excellent skill should feel like a highly disciplined internal playbook from an expert kernel engineer.

It should not feel like:
- a blog post
- a tutorial transcript
- a generic prompt
- a textbook summary
- a collection of random tips

It should feel like:
- a reusable expert operating procedure
- a quality gate
- a technical checklist
- a decision framework

---

## Rules for performance claims

The repository must be intellectually honest.

Never write a skill that encourages the agent to:
- promise speedups without measurement
- claim a kernel is optimal without evidence
- assume tensor cores are always correct or beneficial
- assume custom kernels beat vendor libraries by default
- ignore compilation, integration, or maintenance costs

Skills should explicitly encourage wording like:
- "this strategy is likely better because..."
- "this tradeoff depends on..."
- "this should be benchmarked against..."
- "this may reduce memory traffic but increase register pressure..."

---

## How to write the first flagship skill files

For each initial skill:

1. Choose a precise scope.
2. Write the sections in the standard template.
3. Add realistic decision rules.
4. Add technically meaningful failure modes.
5. Add a final review checklist.
6. Make sure the skill can be copied into an agent workflow immediately.
7. Make sure it reads like a sharp engineering playbook.

---

## Required skill quality checklist

Before accepting any skill into the repository, verify:

- Is the scope clear?
- Is the use case concrete?
- Does it explain when not to use the skill?
- Does it force the agent to gather missing constraints first?
- Does it define a reasoning process?
- Does it discuss correctness risks?
- Does it discuss performance tradeoffs?
- Does it avoid vague optimization claims?
- Does it push the agent toward realistic code?
- Does it avoid hype and filler?
- Would an experienced engineer find this useful?
- Is the skill materially better than a generic prompt?

If the answer to any of these is no, improve the skill before adding it.

---

## Initial README positioning

The README should frame the project like this:

**kernel-skills is an open source library of high quality skill files for AI coding agents working on compute kernels.**

Its purpose is to help agents write better CUDA, Triton, quantized, and performance-oriented kernels by giving them structured engineering playbooks instead of vague prompts.

The README should make it immediately obvious that:
- this is a skills repository
- the content is reusable across agents
- the quality bar is high
- the focus is kernel engineering excellence

---

## Public release expectations

Before making the repository public, ensure:

- README is polished and clear
- folder structure is clean
- first skills are strong enough to represent the project well
- contribution guide exists
- license exists
- roadmap exists
- examples exist
- there are no placeholder or low-quality skill files
- naming is consistent
- the repository looks intentional and serious

Do not publish a half-finished prompt dump.

---

## Implementation sequence

Build in this order:

### Step 1
Create the repository structure and all foundational docs.

### Step 2
Write the README, CONTRIBUTING, ROADMAP, LICENSE, and usage examples.

### Step 3
Write the highest priority CUDA skills first.

### Step 4
Write Triton skills.

### Step 5
Write pattern skills.

### Step 6
Write quantization and portability skills.

### Step 7
Review the whole repository for consistency, naming, tone, depth, and quality.

### Step 8
Make the repository public.

---

## Final instruction to the model working on this repository

When building this repository, always optimize for:
- quality over quantity
- sharp technical specificity over generic breadth
- engineering realism over hype
- reuse over novelty
- clarity over verbosity
- credibility over exaggerated claims

Every file should help make this repository feel like the most serious open source skills library for AI agents writing high performance kernels.

The output should look polished, public-ready, and worthy of long-term community contribution.

---
> Source: [KrxGu/kernel-skills](https://github.com/KrxGu/kernel-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
