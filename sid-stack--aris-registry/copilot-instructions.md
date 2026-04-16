## aris-registry

> Enforces the ARIS Labs Sovereign Architecture and Stateless Intelligence standards to prevent UI regression and data persistence.


# ARIS Labs Engineering Protocol

## 1. The "Stateless" Directive
- **Logic:** All data processing must be volatile. Use RAM buffers, never persistent disk storage for solicitation text or user bid data.
- **Verification:** Always include a visual 'Stateless' indicator in the UI (Green pulse).

## 2. UI/UX Contract (The Three-Pane Terminal)
- **Structure:** [Pane 1: Agentic Logs] | [Pane 2: Audit Chat] | [Pane 3: Sovereign Brief].
- **Aesthetic:** Monochrome (Obsidian/White), 1px solid borders (#333), monospaced fonts for technical data.
- **Stability:** Never use absolute positioning for layout-critical elements. Use Flex/Grid.

## 3. High-Authority Routing
- Use `/audit` for Compliance Chat.
- Use `/verify` for SAM.gov Reps & Certs.
- Use `/intelligence` for Competitive SWOT/UEI analysis.

## 4. Git/Push Safety
- Before finishing: Run `pip freeze > requirements.txt`.
- Check for hardcoded secrets/keys.
- Ensure all new components are exported correctly to prevent 'Module Not Found' errors on push.---
trigger: manual
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sid-stack) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
