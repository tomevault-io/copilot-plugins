## user-management-portal-geographies

> Before making ANY changes to layout, structure, or PRO badge placement in these files:

# ConstructConnect User Management Portal - AI Instructions

## CRITICAL: Layout Guard Rails
Before making ANY changes to layout, structure, or PRO badge placement in these files:
- v2-subscription-overview-multi.html
- v2-modify-users-workflow-multi.html
- v2-subscription-overview.html
- v2-modify-users-workflow.html

**YOU MUST read and verify against LAYOUT-GUARD-RAILS.md first.**

## Common Mistakes to NEVER Repeat
1. ❌ Combining Pro and Starter cards into one card
2. ❌ Moving buttons outside of subscription cards
3. ❌ Placing PRO badge outside of zone-counter span
4. ❌ Changing button enable/disable logic without understanding context-aware requirements

## Required Actions
- Always check Figma design references when user mentions layout issues
- Read LAYOUT-GUARD-RAILS.md before ANY structural changes to multi-subscription files
- Verify PRO badge is INSIDE zone-counter span (lines 467-488)
- Verify buttons are INSIDE their respective subscription cards (lines 244-300)
- Never simplify button logic without preserving Pro/Starter separation

## Documentation
- MULTI-SUBSCRIPTION-FLOW-DOCUMENTATION.md contains complete implementation details
- LAYOUT-GUARD-RAILS.md contains patterns that must NOT be broken
- Both documents should be referenced when working on these files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/STaylor-Figma) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
