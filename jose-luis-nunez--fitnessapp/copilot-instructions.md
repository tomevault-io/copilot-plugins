## fitnessapp

> >-


# Documentation Sync

When you edit Swift files, check if the change affects any of the items below. If it does, update `.cursor/references/architecture.md` in the **same commit/task** — do not defer.

## Trigger Map

| What changed | Section to update in `architecture.md` |
|---|---|
| `Shared/Design/AppStyle.swift` — new/renamed/removed token | **AppStyle Tokens** — add/rename/remove the token entry |
| New file in `Shared/View/`, `Shared/Design/`, `Shared/Components/` | **Shared Components** table — add row with component, file, purpose |
| Deleted shared component | **Shared Components** table — remove row |
| New/changed/deleted service | **Services** section — update API surface |
| New/changed/deleted model in `Core/Model/` | **Domain Models** section — update fields |
| New/changed `NavigationDestination` case | **Navigation** section — update enum listing |
| New feature folder under `Features/` | **Feature Map** tree — add entry |
| Deleted feature folder | **Feature Map** tree — remove entry |
| New utility in `Shared/Utilities/` | **Utilities** section — add entry |
| Changed coordinator or state object | **State & Navigation** section — update |

## How to Update

1. Read `.cursor/references/architecture.md`
2. Find the relevant section from the trigger map
3. Apply the minimal change (add/edit/remove the affected entry)
4. Do not rewrite unrelated sections

## UI Test Sync

When you edit files under `FitnessAppUITests/`, check if the change affects `ui-test-conventions/reference.md` and update it in the **same task**:

| What changed | What to update in `reference.md` |
|---|---|
| DSL function added/renamed/removed in `ElementActions.swift` | **DSL Function Reference** table |
| `TestDefaults` constant added/changed | **Timeout Defaults** table |
| Selector enum added/renamed/removed | **Project Structure** tree + examples if affected |
| Selector constant added/renamed/removed | Check examples and **Reference Example** for stale references |
| `BaseTest` API changed | **Test Template** and rules |
| New file/folder under `FitnessAppUITests/` | **Project Structure** tree |
| Deleted file under `FitnessAppUITests/` | **Project Structure** tree — remove entry |

Also update `uitest-reviewer-agent.md` and `uitest-prep-agent.md` if the change affects their validation rules or report format.

## Skill References

If the change affects what the skills check for (e.g. a new shared component that `reviewing-swift-code` should detect, or a new checklist item for `creating-feature-module`), update the skill's `SKILL.md` as well:

- New shared component -> add detection pattern to `reviewing-swift-code/SKILL.md` "Duplicated UI Patterns" list
- New utility -> add usage check to `reviewing-swift-code/SKILL.md` "Utility Usage" list
- New AppStyle token category -> add pattern to `reviewing-swift-code/SKILL.md` "Hardcoded Styling" table

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jose-Luis-Nunez) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
