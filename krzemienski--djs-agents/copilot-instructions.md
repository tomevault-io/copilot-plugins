## djs-agents

> > Keep these rules pinned in the Cursor **Viewer**.

# 🛡️ Cursor Rules – Deep-Job-Search (Single-File Edition v3)

> **Purpose**
> Keep these rules pinned in the Cursor **Viewer**.
> They govern *every* task and prevent duplicate or unnecessary changes.

---

## 0 · Preparation / Code-Base Understanding
1. **Inventory first, code later.**
   - On every new chat, generate a quick file tree and store it with `Memory.put("repo_tree", …)`.
   - Build an **import graph / function map** (names → paths) and store it as `Memory.put("code_map", …)`.
2. **Sanity-check before editing.**
   - Before you add, rename, or delete anything, grep the current file(s) to confirm the code isn’t already present.
   - If something looks duplicated, **stop** and diff existing definitions.
3. **Summarise understanding.**
   - Use `Subsequential.think()` to outline:
     * what you’ve observed,
     * what you’re about to change,
     * why the change is required.
   - Commit only after the outline shows no logical contradictions.

---

## 1 · Environment
* **All commands** run through `./build-and-run.sh` **inside Docker**.
* Never execute scripts directly on the host.

---

## 2 · Folder & File Policy
* **NO new directories** – all logic lives in `deep_job_search.py`.
* Target end-state: only `deep_job_search.py` (+ Docker assets & docs) lives at repo root.
* `demo_multi_agent.py` & `run.sh` **must be deleted** once merged/obsolete.

---

## 3 · Branch & Commit Discipline
1. **One logical change → one branch** (`refactor/<slug>`, `feat/<slug>`, `fix/<slug>`).
2. **Commit template**
   ```
   [scope]: imperative summary

   WHY: context / ticket / reasoning
   WHAT: key edits
   HOW: tail of build-and-run log (exit 0, validation passed)
   ```
3. Do **not** commit if `./build-and-run.sh` exits non-zero.

---

## 4 · Acceptance Register
| Gate            | Command (inside Docker)                              | Must Pass                                  |
|-----------------|------------------------------------------------------|--------------------------------------------|
| Build & Execute | `./build-and-run.sh`                                 | exit 0                                     |
| MCP URL Check   | auto-invoked in script                               | each URL logs “✔ Apply button found”       |
| File Audit      | `ls demo_multi_agent.py run.sh`                      | *no such file or directory*                |
| Dir Audit       | `find . -maxdepth 2 -type d`                         | only `.` and hidden VCS dirs               |

---

## 5 · MCP Tooling
* **Browser.search / Browser.open** – confirm each job URL exposes an Apply mechanism.
* **Memory.put / Memory.get** – cache repo tree, code map, processed URLs.
* **Subsequential.think()** – keep your private reasoning; expose only conclusions.

---

## 6 · Guardrails
* If any acceptance gate fails, **stop**, fix, re-run before committing.
* Do **not** add test frameworks, coverage tools, or lint configs unless explicitly asked.
* Prioritise explicit logging over silent failures.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/krzemienski) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
