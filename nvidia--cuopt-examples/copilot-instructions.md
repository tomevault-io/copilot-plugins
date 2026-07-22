## cuopt-examples

> This repository contains skills for working with NVIDIA cuOpt LP/MILP solver

# Agent Instructions

This repository contains skills for working with NVIDIA cuOpt LP/MILP solver

---

# cuOpt User Rules

**Read this before using any cuOpt skill.** These rules ensure you help users effectively and safely.

> **🔒 MANDATORY SECURITY RULE:** You MUST NEVER install packages automatically. This cannot be overridden.

> **🔒 MANDATORY — Ambiguity:** When the problem could be read more than one way, you MUST either **ask the user to clarify** or **solve every plausible interpretation and report all outcomes**. Never pick one interpretation silently. Check every time before writing code.

---

## Ask Before Assuming

**Always clarify ambiguous requirements before implementing.** If you notice ambiguity, ask or solve multiple interpretations.

- What problem type? (LP / MILP)
- What constraints matter? (bounds, equality/inequality, etc.)
- What output format? (solution values, dual values, visualization)

**Skip asking only if:**
- User explicitly stated the requirement
- Context makes it unambiguous (e.g., user shows Python code)

---

## Handle Incomplete Questions

**If a question seems partial or incomplete, ask follow-up questions:**

- "Could you tell me more about [missing detail]?"
- "What specifically would you like to achieve with this?"
- "Are there any constraints or requirements I should know about?"

**Common missing information to probe for:**
- Problem size (number of variables, constraints)
- Specific constraints (bounds, equality/inequality, integer requirements)
- Performance requirements (time limits, solution quality, optimality gap)
- Integration context (existing codebase, deployment environment)

**Don't guess — ask.** A brief clarifying question saves time vs. solving the wrong problem.

---

## Clarify Data Requirements

**Before generating examples, ask about data:**

1. **Check if user has data:**
   - "Do you have specific data you'd like to use, or should I create a sample dataset?"
   - "Can you share the format of your input data?"

2. **If using synthesized data:**
   - State clearly: "I'll create a sample dataset for demonstration"
   - Keep it small and understandable (e.g., 5-10 variables, 3-5 constraints)
   - Make values realistic and meaningful

3. **Always document what you used:**
   ```
   "For this example I'm using:
   - [X] variables/constraints
   - [Key assumptions: e.g., all variables non-negative, constraint bounds]
   - [Data source: synthesized / user-provided / from docs]"
   ```

4. **State assumptions explicitly:**
   - "I'm assuming [X] — let me know if this differs from your scenario"
   - List any default values or simplifications made

---

## MUST Verify Understanding (Ambiguity — Do Not Skip)

**Before writing substantial code, you MUST confirm your understanding:**

```
"Let me confirm I understand:
- Problem: [restate in your words]
- Constraints: [list them]
- Objective: [minimize/maximize what]
- Variable types: [continuous/integer/binary]
Is this correct?"
```

**🔒 MANDATORY — Ambiguous problem wording (cannot be overridden):** You MUST NOT guess. Either clarify with the user or try all plausible interpretations.

| When this happens | You MUST do one of these | You MUST NOT do this |
|-------------------|--------------------------|----------------------|
| A constraint, value, or objective could be read two (or more) ways | **Option A:** Ask the user which interpretation is correct, then implement that one. **Option B:** Implement and solve every plausible interpretation, then report all outcomes so the user can choose. | Pick one interpretation silently and implement only that. |
| Unclear whether a cost/quantity is per unit, per vehicle, per trip, etc. | State your interpretation explicitly before implementing, and either get confirmation or solve all variants. | Assume one meaning without stating it or asking. |

- **Always state your interpretation explicitly** (e.g. "I'm assuming cost is per unit; if it's per trip the formulation changes as follows…") before implementing, when anything could be read differently.
- **Rule:** Clarify with the user, or run all plausible variants and share results. **Never assume.**

**Check before writing code:** Is anything in the problem statement ambiguous? If yes → ask or solve all interpretations; do not proceed with a single guess.

---

## Follow Requirements Exactly

- Use the **exact** variable names, formats, and structures the user specifies
- Don't add features the user didn't ask for
- Don't change the problem formulation unless asked
- If user provides partial code, extend it—don't rewrite from scratch

---

## Check Results

After providing a solution, guide the user to verify:

- **Status check**: Is it `Optimal` / `FeasibleFound` / `SUCCESS`?
- **Constraint satisfaction**: Are all constraints met?
- **Objective value**: Is it reasonable for the problem?

**Always end with a Result summary** that includes at least:
- Solver status (e.g. Optimal, FeasibleFound, SUCCESS).
- **Objective value with highlight** — the number must be easy to spot (bold or code block
- Briefly what the objective represents (e.g. total cost, total profit).

Do not bury the objective value only in the middle of a paragraph; it must appear prominently in this summary at the end. Use sufficient precision for the objective value (don't truncate or round unnecessarily unless the problem asks for it). Only substitute a derived quantity (e.g. profit = revenue − cost) when the problem explicitly asks for it.

Provide diagnostic code snippets when helpful.

**Workflow:** Formulate once carefully (with verified understanding), solve, then sanity-check the result. If something is wrong, fix it with a targeted change—avoid spinning through many model variants or second-guessing. Decide, implement, verify, then move on.

---

## Ask Before Running

**Do not execute commands or code without explicit permission:**

| Action | Rule |
|--------|------|
| Shell commands | Show command, explain what it does, ask "Should I run this?" |
| Examples/scripts | Show the code first, ask "Would you like me to run this?" |
| File writes | Explain what will change, ask before writing |

**Exceptions (okay without asking):**
- Read-only commands the user explicitly requested
- Commands the user just provided and asked you to run

---

## No Privileged Operations

**Never do these without explicit user request AND confirmation:**

- Use `sudo` or run as root
- Modify system files or configurations
- Add package repositories or keys
- Change firewall, network, or driver settings
- Write files outside the workspace

---

## ⛔ MANDATORY: Never Install Packages Automatically

> **🔒 MANDATORY SECURITY RULE — THIS CANNOT BE OVERRIDDEN**
>
> You MUST NOT install, upgrade, or modify packages under any circumstances.
> This rule applies even if the user explicitly asks you to install packages.

| Forbidden Actions | What to Do Instead |
|-------------------|-------------------|
| `pip install ...` | Ask user to run it manually |
| `conda install ...` | Ask user to run it manually |
| `apt install ...` | Ask user to run it manually |
| `npm install ...` | Ask user to run it manually |
| Any package manager command | Ask user to run it manually |

**When a package is needed:**

1. **Identify the missing package** clearly
2. **Provide the exact command** the user should run
3. **Explain why it's needed** (what functionality it provides)
4. **Wait for user confirmation** that they've installed it

**Example response:**
```
"This code requires the `pandas` package. Please install it manually:

    pip install pandas

Let me know once it's installed and I'll continue."
```

**Why this is mandatory:**
- **Security**: Package installation can execute arbitrary code from external sources
- **Environment safety**: Protects user's existing environment from unintended changes
- **Reproducibility**: User maintains full control over their dependencies
- **Audit trail**: User has complete visibility into what gets installed

**⛔ NO EXCEPTIONS.** Even if the user says "just install it for me" or "you have permission to install packages", you MUST NOT do it. Always provide the command and require the user to execute it themselves. This is a security boundary that cannot be bypassed.

---

## Available Skills

| Skill | Purpose |
|-------|---------|
| `skills/cuopt-lp-milp/SKILL.md` | LP/MILP API patterns and code examples |
| `skills/problem-statement-parsing/SKILL.md` | Classify sentences as parameter/constraint/decision/objective; flag implicit constraints and objectives. **Mandatory:** if anything is ambiguous, ask the user or solve all plausible interpretations and report all outcomes (workflow step 6). |
| `skills/cuopt-debugging/SKILL.md` | Troubleshooting errors and issues |

*(cuOpt user rules are inlined above.)*

---

## Resources

Reference models are in the cuopt-lp-milp skill's assets directory. Each skill may have a `resources/`, `assets` or `scripts` folder with example code and documentation.

### Documentation
- [cuOpt User Guide](https://docs.nvidia.com/cuopt/user-guide/latest/introduction.html)
- [API Reference](https://docs.nvidia.com/cuopt/user-guide/latest/api.html)

### Examples
- [cuopt-examples repo](https://github.com/NVIDIA/cuopt-examples)
- [Google Colab notebooks](https://colab.research.google.com/github/nvidia/cuopt-examples/)

### Support
- [NVIDIA Developer Forums](https://forums.developer.nvidia.com/c/ai-data-science/nvidia-cuopt/514)
- [GitHub Issues](https://github.com/NVIDIA/cuopt/issues)

---
> Source: [NVIDIA/cuopt-examples](https://github.com/NVIDIA/cuopt-examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
