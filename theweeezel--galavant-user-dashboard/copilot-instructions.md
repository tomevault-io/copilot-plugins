## galavant-user-dashboard

> **MANDATORY:** This dashboard contains the player-facing gameplay guide (`src/pages/gameplay/gameplay-content.ts`). When writing or updating any player-visible content, you MUST NEVER reveal internal economy secrets that could be exploited to game the system or destabilize the game economy.

# Galavant User Dashboard Rules

## Economy Secrets Protection Rule

**MANDATORY:** This dashboard contains the player-facing gameplay guide (`src/pages/gameplay/gameplay-content.ts`). When writing or updating any player-visible content, you MUST NEVER reveal internal economy secrets that could be exploited to game the system or destabilize the game economy.

**NEVER expose in player-facing content:**
- Exact earning formulas, base rates, or multiplier constants
- Anti-cheat detection logic, thresholds, flag scores, or penalty escalation details
- Exact drop rate percentages or probability curves (toolboxes, mystery boxes, breeding)
- Token conversion curve formulas, emission models, or supply mechanics
- Exact staking/loyalty multiplier values or tier boundaries
- Minting cost escalation formulas or breed outcome probability weights
- Server-side validation thresholds (GPS quality cutoffs, speed penalty math, energy regen internals)

**SAFE to show players (use qualitative language):**
- General mechanic descriptions ("higher Earning stat = more SAT per minute")
- Relative comparisons ("Legendary bikes earn significantly more than Common")
- Approximate/vague ranges ("toolbox drops become more likely as Luck increases")
- Feature names and tiers ("Bronze, Silver, Gold loyalty tiers")
- UI-visible stats: bike types, quality tiers, attribute names, socket unlock levels

**Rule of thumb:** Describe *what* affects gameplay, not the precise numbers behind it. Players should understand mechanics without being able to reverse-engineer optimal exploits.

## Gameplay Guide Sync Rule

**MANDATORY:** The gameplay guide in `src/pages/gameplay/gameplay-content.ts` must stay in sync with the authoritative `GAMEPLAY.md` in `../galavant-infrastructure/`. Whenever a gameplay mechanic changes in the infrastructure repo, this guide must be updated in the same response — while respecting the Economy Secrets Protection Rule above.

Review the guide content against GAMEPLAY.md regularly. If a mechanic was changed but the guide was not updated, flag it and fix it.

---
> Source: [TheWeeezel/galavant-user-dashboard](https://github.com/TheWeeezel/galavant-user-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
