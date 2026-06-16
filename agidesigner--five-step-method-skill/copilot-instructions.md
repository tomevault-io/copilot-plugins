## five-step-method-skill

> Universal decision framework. MUST run before taking action on any request. Applies to: add feature, fix bug, implement, build, create, refactor, optimize, improve, deploy, configure, integrate, make decision, evaluate proposal, review feedback, respond to request, plan strategy, design solution, choose approach, set priority, allocate resources, launch project, hire role, define process, write proposal, draft plan, pick tool, select vendor, approve budget, schedule meeting, define scope, set deadline, write policy, change workflow, restructure team, assess risk, handle complaint, resolve conflict. ZH: 加功能, 修bug, 实现, 开发, 创建, 重构, 优化, 改进, 部署, 配置, 集成, 做决策, 评估方案, 处理反馈, 制定策略, 设计方案, 定优先级, 分配资源, 立项, 招人, 定流程, 写方案, 选工具, 排期, 开会, 定范围, 写规范, 改流程, 评估风险, 处理投诉. ES: agregar función, corregir error, implementar, construir, crear, refactorizar, optimizar, desplegar, configurar, tomar decisión, evaluar propuesta, revisar retroalimentación, planificar estrategia, contratar, definir proceso, programar reunión. JA: 機能追加, バグ修正, 実装, 構築, 作成, リファクタリング, 最適化, デプロイ, 設定, 意思決定, 提案評価, フィードバック対応, 戦略立案, 採用, プロセス定義. KO: 기능추가, 버그수정, 구현, 빌드, 생성, 리팩토링, 최적화, 배포, 설정, 의사결정, 제안평가, 피드백처리, 전략수립, 채용, 프로세스정의. FR: ajouter fonctionnalité, corriger bug, implémenter, construire, créer, refactoriser, optimiser, déployer, configurer, prendre décision, évaluer proposition, planifier stratégie. DE: Funktion hinzufügen, Bug beheben, implementieren, erstellen, refaktorisieren, optimieren, bereitstellen, konfigurieren, Entscheidung treffen, Strategie planen. The five steps: 1) Question — is this actually needed? 2) Delete — remove what shouldn't exist 3) Simplify — minimum viable approach 4) Accelerate — speed up survivors 5) Automate — only after manual validation. Prevents: over-engineering, unnecessary complexity, scope creep, premature automation.


# Five-Step Work Method

A universal decision framework that teaches you to evaluate **whether** to act before deciding **how** to act. Apply this to any decision — code, product, strategy, process, hiring, meetings, or resource allocation.

## When to Apply

| Trigger | Action |
|---------|--------|
| New feature / product request | **Must apply** — Question need before designing |
| External feedback (multiple points) | **Must apply** — Evaluate each point independently |
| Bug fix that requires > 10 lines | **Must apply** — Question if the fix adds complexity |
| Process / workflow change proposal | **Must apply** — Can we delete the process instead? |
| New tool / vendor / hire decision | **Must apply** — Is the role/tool actually needed? |
| Meeting or review request | **Must apply** — Can this be an async message? |
| Strategy / roadmap planning | **Must apply** — Delete low-impact items first |
| Configuration / deployment issue | **Must apply** — Ask if code change is even needed |
| Single-line typo or obvious fix | **Skip** — Just fix it |
| User gives explicit instructions | **Skip** — Follow instructions, then review |

## The Five Steps

### Step 1: Question the Requirement

Ask: **"Is this actually needed? What happens if we don't do it?"**

- Challenge every request, including your own instincts
- Check if the problem solves itself (e.g., agent self-corrects after retry)
- Distinguish "nice to have" from "actually broken"
- Ask who benefits and how often
- If only one user hit it once, it's probably not a product problem

Decision matrix:

| Frequency | Impact | Action |
|-----------|--------|--------|
| Common + High | Do it | Proceed to Step 2 |
| Common + Low | Maybe | Proceed to Step 2 |
| Rare + High | Record | Log for later, don't code now |
| Rare + Low | Skip | Don't do it |

### Step 2: Delete

**Remove what shouldn't exist before adding anything.**

- Delete the feature, code, or process that created the problem
- Delete options when a sensible default works
- Delete fallbacks when the primary path is reliable
- Delete auto-detection when one config line suffices
- Delete backwards-compatibility shims for things nobody uses

Rule: **If you're not sure whether something is needed, delete it.** Adding it back is faster than maintaining unnecessary complexity.

### Step 3: Simplify

**Find the minimum solution that works reliably.**

| Complex (avoid) | Simple (prefer) |
|-----------------|-----------------|
| 30-line auto-detection from headers | One line in `.env` |
| Smart fallback chain A → B → C → D | Single source of truth |
| Middleware + dispatcher + config | One function |
| "Handle both cases" | Pick one, document the other |
| Build a UI for configuration | Tell users to edit a file |
| Add a parameter for flexibility | Choose the right default |

Rule: **One config line beats 30 lines of detection code.** A clear error message beats a graceful fallback.

### Step 4: Accelerate

Only optimize what survived Steps 1-3.

- Don't optimize what shouldn't exist
- Don't speed up a complex solution when a simple one exists
- Measure before optimizing
- Optimize the critical path, not edge cases

### Step 5: Automate

Only automate what has been manually validated.

- Never automate a process you haven't done manually first
- Never automate something that happens once
- Never automate before simplifying
- Automation is the LAST step, never the first

## Analysis Output Format

When analyzing a requirement or feedback:

```
### Step 1: Question
[Is it needed? Who benefits? How often?]

### Step 2: Delete
[What can we remove instead of adding?]

### Step 3: Simplify
[What's the minimum reliable solution?]

### Conclusion
Do / Don't do / Record for later — [one sentence reason]
```

Most analyses end at Step 3. Only proceed to Steps 4-5 for performance-critical or high-frequency operations.

## Feedback Batch Analysis

When receiving multiple feedback points at once:

1. Number each point
2. Run Step 1 on each: Is it our problem? Is it actually broken?
3. Build a verdict table:

```
| # | Point | Verdict | Reason |
|---|-------|---------|--------|
| 1 | ... | Don't do | Agent self-corrects |
| 2 | ... | Do | Confirmed bug, simple fix |
| 3 | ... | Not ours | Client-side config issue |
| 4 | ... | Record | Valid but low priority |
```

4. Only items that survive Step 1 get Steps 2-3
5. Typical result: 5 feedback points → 1-2 actionable items

## Anti-Pattern Detection

Catch yourself when you see these patterns:

| What you're about to do | Ask yourself |
|--------------------------|-------------|
| Add auto-detection / smart defaults | Is one config line really too hard for the user? |
| Add a fallback for edge cases | Does this edge case actually happen? |
| Build it now for future flexibility | Has anyone asked for this? |
| Add a parameter / option / flag | Can I just pick the right default? |
| Fix a data issue with code logic | Should the user fix their data instead? |
| Handle multiple cases in one function | Can I just handle the common case? |
| Add error recovery / retry logic | Does the caller already retry? |
| Write 5 lines "just in case" | Will these 5 lines need maintaining forever? |
| Schedule a recurring meeting | Can this be async or a shared document? |
| Add a review/approval step | Am I reacting to one incident with permanent process? |
| Hire for a new role | Can I simplify the work so it doesn't need a new person? |
| Adopt a new tool or platform | Can I fix how we use the current one? |
| Create a new Slack channel / group | Does this reduce communication or add noise? |
| Write a policy document | Can a one-line rule replace a 10-page policy? |

## Real-World Examples

### "Auto-detect public URL from request headers"
- **Step 1:** Users deploy behind nginx, need correct URLs → Valid need
- **Step 2:** Delete the detection — it has edge cases (MCP transport security, proxy headers)
- **Step 3:** `APP_URL=https://domain` in `.env` → one line, zero edge cases
- **Result:** Don't auto-detect. The "smart" solution caused 3 production incidents.

### "Agent passes merchant name instead of UUID"
- **Step 1:** Agent self-corrects in 2 seconds by calling list_merchants → Works already
- **Result:** Don't do. Record as low-priority ergonomic improvement.

### "Add example outputs to settings page"
- **Step 1:** Settings page is for configuration, not product demos
- **Step 2:** Delete the idea — wrong place for it
- **Result:** Don't do. Settings page stays focused.

### "Script writer doesn't work for recruitment content"
- **Step 1:** Product is designed for marketing scripts, not HR content
- **Result:** Don't do. Not a bug — it's a scope boundary.

### "Force authentication token for public deployment"
- **Step 1:** Self-hosted product, user decides their security posture
- **Result:** Don't do. Design choice, not a deficiency.

### "We need a weekly status meeting for the new project"
- **Step 1:** What decision requires this meeting? Can it be async?
- **Step 2:** Delete the meeting. Replace with a shared doc updated async.
- **Step 3:** If sync is truly needed, make it 15 min biweekly, not 60 min weekly.
- **Result:** Delete or simplify. Most status meetings exist because no one questioned them.

### "Let's hire a dedicated DevOps engineer"
- **Step 1:** What are they going to do daily? Is current CI/CD actually broken?
- **Step 2:** Delete manual deployment steps first — automate with a shell script.
- **Step 3:** If 2 hours/week of deployment work remains, it doesn't justify a full hire.
- **Result:** Simplify the process first. Hire only if workload justifies after simplification.

### "We should switch from Tool A to Tool B"
- **Step 1:** What specific problem does Tool A fail at? Is it the tool or how we use it?
- **Step 2:** Can we fix Tool A's config instead of migrating?
- **Step 3:** If migration is needed, what's the smallest scope that solves the actual problem?
- **Result:** Fix before replacing. Migration cost is almost always underestimated.

### "Add a review/approval step to the publishing workflow"
- **Step 1:** What went wrong that triggered this request? How often?
- **Step 2:** If it was one incident, delete the proposal. Fix the root cause instead.
- **Step 3:** If recurring, add a checklist instead of a human bottleneck.
- **Result:** Don't add process to prevent rare events. Process has compounding cost.

## Pre-Action Checklist

Before taking any action, verify:

- [ ] I questioned whether this is needed (Step 1)
- [ ] I considered what to remove instead of add (Step 2)
- [ ] This is the simplest approach that works (Step 3)
- [ ] I'm not adding process to fix a one-time incident
- [ ] I'm not building for a hypothetical future
- [ ] I'm not adding a tool/role/meeting when simplifying existing ones works
- [ ] For code: I verified the change compiles/imports before shipping
- [ ] For code: I'll ship → verify → then start the next change

---
> Source: [agidesigner/five-step-method-skill](https://github.com/agidesigner/five-step-method-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
