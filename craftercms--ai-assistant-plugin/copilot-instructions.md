## crafterq-form-panel-contract

> Preserve AI Assistant form control — agents accordion, plugin registration, non-regression contract


# AI Assistant form control (Studio Form Engine) — required behavior

**Before merging form-control / plugin entry changes:** from `sources/` run `yarn package` (runs Rollup **and** `scripts/verify-crafterq-form-pipeline.mjs`). If verify fails, the accordion or Studio registration will break in production.

**CSS/layout-only changes:** Prefer `AiAssistantFormControl.tsx` (`GlobalStyles`, `sx`, etc.). Do **not** delete, bypass, or replace the agent pipeline in `main.js` when doing styling.

## Frozen implementation rules (`sources/control/ai-assistant/main.js`)

These exist because they were removed or “simplified” multiple times and broke authors. **Do not delete or replace with shortcuts.**

1. **Agent sources** — Union agents parsed from **Redux `uiConfig.xml`** and **`get_configuration` `/ui.xml`** (by stable key). Never return early after the first source if it has “some” agents but fewer than the repo file.

2. **Agent cache** — TTL cache is allowed, but **never treat a cached empty array as a hit** (`[].length === 0` must miss cache).

3. **XML `<agent>` collection** — Use **`cqNearestAgentsAncestorForAgent`** (parent walk). **Never** use **`Element.closest('agents')`** on `DOMParser` XML documents; it can skip every agent.

4. **Form Properties bag** — Use **`cqNormalizePropertyList`** + **`cqFormFieldPropertiesFromRender`** merging **`self.properties`**, **`config.properties`**, **`config.configuration.properties`**, **`config.props.properties`**. Match toggles with **`cqPropertyEntryName`** (`name` or `id`). Keep explicit-false-only disable + **`cqVisibleAgentsFromProperties`** fallback when filter would drop all.

5. **Async order** — Compute **`cqLoadAgentsForSite`**, **`cqFormFieldPropertiesFromRender`**, **`cqVisibleAgentsFromProperties`** **inside** **`importPlugin(...).then(...)`** immediately before `root.render`, so Redux/ui.xml can hydrate during the import gap.

6. **React prop shape** — `AiAssistantFormControl.tsx` must keep **`normalizeAgentsProp`** (array or numeric-key object).

7. **Parity** — `fetchAiAssistantUiAgents.ts` must use the same **parent-walk** agent collection as `main.js` (no `closest('agents')`).

8. **Canonical file** — Edit **`sources/control/ai-assistant/main.js` only**; `yarn package` copies it to `authoring/.../control/ai-assistant/main.js`.

## Plugin bundle (`sources/index.tsx`)

- **`PluginDescriptor.id`** must be **`aiAssistantStudioPluginId`** (`org.craftercms.aiassistant.studio`) — same as **`craftercms-plugin.yaml`**. Studio skips `registerPlugin` on duplicate **internal** id; a mismatch leaves **`craftercms.components.aiassistant.Helper`** unregistered.

- Export **`export default plugin`** **and** **`export { plugin }`** for `module.plugin ?? module.default`.

## UX (unchanged without product approval)

- **Accordion** — `AiAssistantFormControlPanel.tsx`: one row per enabled agent; one expanded; chat only in expanded row. Do not move this layout for feature work; put logic in `AiAssistantChat.tsx` / `aiAssistantApi.ts` / `main.js` / types.

- **Form shell width** — `cqIsStudioFormWidthShell` / 95% widen behavior stays unless deliberately replaced.

- **Authoring form context** — `getAuthoringFormContext` + form appendix in chat stays.

Do not revert to “filter by stable keys only”, “one shared chat strip”, or “single ui.xml source only” without an explicit product decision.

---
> Source: [craftercms/ai-assistant-plugin](https://github.com/craftercms/ai-assistant-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
