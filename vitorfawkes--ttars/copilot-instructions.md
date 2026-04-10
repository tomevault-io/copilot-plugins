## ttars

> > **Context:** Preventing "Silent Regressions" and "Context Blindness" in AI-driven Development.

# 🏛️ The "Iron Dome" Protocol: AI Safety & Collaboration Standards
> **Context:** Preventing "Silent Regressions" and "Context Blindness" in AI-driven Development.

## 1. The "Live State" Requirement
**"An Agent shall not modify a persistent store (DB) based solely on static code."**
*   **Implementation:** Before any `CREATE OR REPLACE VIEW`:
    1.  **READ:** Run `SELECT definition FROM pg_views WHERE viewname = 'target_view_name';`
    2.  **DIFF:** Compare the live definition with the new code.
    3.  **ASSERT:** Ensure no existing columns are lost.

## 2. The "Context Anchoring" Requirement
**"An Agent must explicitly acknowledge the 'Constitution' before Planning."**
*   **Implementation:** The file `docs/SYSTEM_CONTEXT.md` is the Anchor.
    *   It contains the "Forbidden Actions" list.
    *   The Agent must explicitly state: *"I have checked the Forbidden Actions list."*

## 3. The "Verification First" Requirement
**"An Agent shall not mark a task as Done until it has proven the absence of regression."**
*   **Implementation:**
    *   **Old Way:** "I added the column. It works."
    *   **New Way:** "I added the column. I also queried the *old* columns (`is_group_parent`) and confirmed they are still there."

## 4. The "Design System" Requirement
**"An Agent must adhere to the 'Light Mode First' principle."**
*   **Implementation:**
    *   **READ:** `docs/DESIGN_SYSTEM.md` before writing any UI code.
    *   **FORBIDDEN:** Using `text-white` or `bg-white/10` on default backgrounds.
    *   **MANDATORY:** Use `border-slate-200` and `shadow-sm` for card depth in Light Mode.

---
**Violation of this protocol is a Critical Engineering Failure.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/VitorFawkes) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
