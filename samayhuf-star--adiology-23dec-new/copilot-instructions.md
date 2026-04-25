## adiology-23dec-new

> It enforces wizard safety, step isolation, and prevents cross-module breakage.

It enforces wizard safety, step isolation, and prevents cross-module breakage.
# PROJECT-SPECIFIC AI RULES FOR THIS REPO

## 1. WIZARD ARCHITECTURE PROTECTION
This project uses a multi-step Setup Wizard.  
ABSOLUTE RULE:
- **Never modify any step except the one explicitly mentioned.**
- Do NOT touch shared state, hooks, store slices, or context providers unless required by the exact task.
- Do NOT rename steps, restructure steps, or alter navigation flow.

## 2. FILE & MODULE ISOLATION
- AI must only edit the file(s) explicitly mentioned in the user request.
- Any modification outside the mentioned file must be rejected.
- AI should warn: “This change affects other modules; please confirm.”

## 3. NO CONTRACT / SCHEMA BREAKING
- API types, DB schemas, Redux slices, Zustand stores, and TypeScript interfaces are contract-critical.
- Do NOT modify them unless specifically requested in the task.
- Keep all props, type names, function signatures, and event shapes unchanged.

## 4. UI STABILITY GUARANTEE
- UI components must not be rewritten or refactored.
- Only fix specific UI issues (alignment, conditional rendering, disabled state, etc.)
- Do not migrate components to different patterns or frameworks.

## 5. STATE MANAGEMENT RULES
- Do NOT add new global state.
- Do NOT modify global reducers/stores unless explicitly instructed.
- For wizard steps, only modify the local or step-specific state.

## 6. NO RANDOM IMPROVEMENTS
The AI must NEVER:
- change formatting conventions
- reorder files
- “improve” performance
- refactor patterns
- rename things for readability
- replace code styles or architectures

Unless the user says: “refactor” or “improve”.

## 7. PATCH-ONLY EDITING (PREFERRED)
- Always output unified git diff patches where possible.
- Never rewrite the entire file unless the user demands a full rewrite.

## 8. HIGH-RISK ZONES (DO NOT TOUCH)
Unless explicitly instructed:
- Authentication logic
- Billing / Stripe integration
- Tracking / attribution logic
- API endpoints and controllers
- Database models
- Wizard navigation logic
- Deployment configs, env vars, package.json

## 9. FINAL STAGE MODE
Treat this project as production-ready.  
Default stance:
- If unsure → DO NOT CHANGE ANYTHING.
- Ask for clarification before performing anything risky.

## 10. CONSISTENT TECH STACK
Maintain existing:
- Frameworks
- Folder structures
- Naming conventions
- Project architecture

No migrations or alternations without explicit approval.

These rules apply to all Cursor edits and take priority over model assumptions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/samayhuf-star) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
