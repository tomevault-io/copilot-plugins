## projectauto-predefined-only

> Projectauto workflows must use the predefined-only trigger/condition/action contract


# Projectauto Predefined-Only Rule

- Do not add free-form condition expression text inputs to Projectauto workflow wizard or builder UIs.
- Do not add advanced JSON editors for Projectauto workflow node configuration.
- Projectauto workflow creation must validate against the catalog in `Modules/Projectauto/Workflow/Support/PredefinedRuleCatalog.php`.
- Condition nodes must use structured `config.condition_spec` metadata and may only compile the runtime expression from that spec.
- Finite schema fields must stay finite in the UI and backend by honoring `options` / `enum` metadata instead of falling back to free text.
- Workflow graphs must remain limited to one trigger, an optional `logic.if_else`, and cataloged action nodes.
- If a condition node exists, the trigger must link to that node and must not link directly to action nodes.
- If a new trigger or action is added, update the node registry, catalog metadata, validation, and docs together.

---
> Source: [kunwaso/upos612-metro](https://github.com/kunwaso/upos612-metro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
