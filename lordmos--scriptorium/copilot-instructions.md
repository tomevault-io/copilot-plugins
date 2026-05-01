## scriptorium

> > Scriptorium is a multi-agent framework for writing technical books about open-source

# Scriptorium — AI Coding Assistant Context

> Scriptorium is a multi-agent framework for writing technical books about open-source
> source code. This file provides context for AI coding tools working on a
> Scriptorium-based book project.

---

## ⚡ 一句话启动

**把这句话发给你的 AI 助手，它会自动完成所有工作：**

```
[项目名] 的源码在 [目录路径]。请读 QUICK_START.md，然后向我提问。没有问题就开始工作。
```

AI 会读取 `QUICK_START.md`，向你确认书名、读者等基本信息，然后自主运行全部五阶段流水线——你只需在大纲完成时确认一次。

**跨 session 恢复：**
```
请读 checkpoint.md，继续上次未完成的工作。
```

---

## 📖 About This Book Project

<!-- TODO: Fill in when creating your book project from this template -->

- **Book Title**:
- **Source Project** (the open-source repo being analyzed):
- **Target Reader**:
- **One-line Description**:

---

## 🏗️ Framework Architecture

See `agents/00-system-overview.md` for the full architecture diagram.

**Core Principle**: *Agents are stateless workers; the file system is stateful.*

Each agent is invoked once per task. It reads a specific set of files (File Pointers),
executes its role, writes output, and exits. The Orchestrator (you) coordinates the
whole pipeline by injecting the right files into each agent.

---

## 📁 Key Files

### Shared State — fill in before starting

| File | Description |
|------|-------------|
| `source-map.md` | Maps source code paths → book chapters |
| `outline.md` | Full book outline with chapter list |
| `style-guide.md` | Writing conventions, tone, formatting rules |
| `glossary.md` | Technical term definitions |
| `metaphor-registry.md` | Registered metaphors (avoid duplicates across chapters) |
| `checkpoint.md` | Current progress — **read this first every session** |
| `audit-log.md` | History of all agent invocations |

### Agent Specs — read to understand each role

| File | Agent | Phase |
|------|-------|-------|
| `agents/01-orchestrator.md` | Orchestrator #0 (you) | All |
| `agents/02-architect.md` | Architect #1 | Phase 1 |
| `agents/03-reader-advocate.md` | Reader Advocate #2 | Phase 1 |
| `agents/04-researcher.md` | Researcher #3 | Phase 2 |
| `agents/05-writer.md` | Writer #4 | Phase 3 |
| `agents/06-code-reviewer.md` | Code Reviewer R1 | Phase 4 |
| `agents/07-consistency-reviewer.md` | Consistency Reviewer R2 | Phase 4 |
| `agents/08-content-reviewer.md` | Content Reviewer R3 | Phase 4 |
| `agents/09-reader-panel.md` | Reader Panel | Phase 4 |
| `agents/10-bookbinder.md` | Bookbinder #11 | Phase 5 |
| `agents/11-quality-inspector.md` | Quality Inspector #12 | Phase 5 |

### Framework Docs — reference

| File | Description |
|------|-------------|
| `framework/workflow.md` | Full 5-phase pipeline with completion markers |
| `framework/file-pointers.md` | File Pointers mechanism and rules |
| `framework/parallel-strategy.md` | DAG batch execution strategy |
| `framework/review-architecture.md` | Triple-parallel review (R1/R2/R3) |
| `framework/recovery.md` | Checkpoint recovery and disaster recovery |

---

## 🔄 The 5-Phase Pipeline

```
Phase 1 · Prep      → Architect builds outline + source-map; Reader Advocate validates
Phase 2 · Research  → Researcher dives into source code, one chapter at a time
Phase 3 · Writing   → Writer produces each chapter draft
Phase 4 · Review    → R1 + R2 + R3 review in parallel → consolidate → revise
Phase 5 · Publish   → Bookbinder produces final formatted output
```

Full pipeline with completion markers: `framework/workflow.md`
Parallel execution strategy: `framework/parallel-strategy.md`
Recovery from interruption: `framework/recovery.md`

---

## 🤖 Starting a New Book

**This is all you need to say:**

```
The source code for [project name] is in [directory path].
Please read QUICK_START.md, then ask me any questions you have.
If you have no questions, start your work.
```

The AI reads `QUICK_START.md` and runs the full 5-phase pipeline autonomously.
You only need to: ① answer startup questions  ② approve the outline  ③ read the final manuscript.

**Resume after interruption:**

```
Please read checkpoint.md and continue where we left off.
```

Full orchestrator briefing: `QUICK_START.md`

---

## 🔑 File Pointer Rules

1. **Unidirectional**: all instructions flow Orchestrator → Agent
2. **Minimal**: provide exactly the files the agent needs — not the whole repo
3. **Explicit**: always give exact file paths; never ask agents to search the file system
4. **Bridge**: shared state files are the only communication channel between agents
5. **Marker**: file existence signals task completion (e.g., `research/ch03-report.md` exists → ch03 researched)

Full details: `framework/file-pointers.md`

---

## 📋 Templates

All templates are in `templates/`. They contain `{{变量名}}` placeholders — fill them
in before the relevant phase begins. Do not modify the `agents/` or `framework/`
directories; they are the framework core.

---

## 🔧 Useful Commands

```bash
cat checkpoint.md               # current project state
cat outline.md                  # book structure
cat framework/workflow.md       # full pipeline reference
cat agents/04-researcher.md     # researcher spec
ls research/                    # completed research reports
ls chapters/                    # completed chapter drafts
```

---
> Source: [lordmos/scriptorium](https://github.com/lordmos/scriptorium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
