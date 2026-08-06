## meow-agent

> > The canonical rulebook for anyone (human or AI) writing code in Meow Agent.

# AGENTS_2.md — Meow Agent Development Rules

> The canonical rulebook for anyone (human or AI) writing code in Meow Agent.
> Every change MUST conform to these rules. When a rule here conflicts with
> older docs (the now-removed AGENTS.md / SKILLS.md), **this file wins**.
>
> Companion docs:
> - **[DESIGN_2.md](./DESIGN_2.md)** — visual language, layout, spacing, components.
> - **[ARCHITECTURE_2.md](./ARCHITECTURE_2.md)** — runtime, LLM, agent, data flow.

---

## 0. The Five Non-Negotiables

1. **Accuracy over everything.** Never hallucinate a capability or claim success you cannot verify. Missing data / absent capability → say so honestly and stop. No guessing, no fabricating, no silent retry.
2. **Language-generic, always.** No per-language word lists, no per-case patches, no language-specific branches in engine, routing, or prompts.
3. **Reusable first.** Before writing a widget, helper, or prompt, find the existing one. Duplication is a defect.
4. **One source of truth per concern.** UI copy → `AppStrings`. Prompts → `prompt_*` files. Tool existence → `ModulePlugin`. Permissions → the gate maps. Never fork these.
5. **Verify before declaring done.** State re-check (snapshot probe, registry re-read, result-data keys) gates every mutation. The LLM's "done" is not proof.

---

## 1. Localization Rules (STRICT)

### 1.1 `isId` must NEVER appear in a screen or widget

`isId` is a private detail of `AppStrings` (`code == 'id'`). It exists so the
string class can pick a variant. It must not leak into presentation code.

**Banned in `lib/features/**/presentation/**` and `lib/app/widgets/**`:**
- `final isId = resolveLanguageCode(...) == 'id';`
- `AppStrings(isId ? 'id' : 'en')`
- a `bool isId` parameter threaded through a widget constructor
- `isId: s.isId` passed into a child widget

**Why:** every `isId` branch in a screen is a place a translation can silently
go wrong, and it spreads the language concept across the UI instead of keeping
it sealed inside `AppStrings`. The string class already knows the language —
ask it for the finished string, never for the language flag.

### 1.2 The only correct pattern

Resolve once at the top of `build` (or once per widget that needs copy), then
read finished strings:

```dart
@override
Widget build(BuildContext context, WidgetRef ref) {
  final langPref = ref.watch(appLanguageProvider);
  final s = AppStrings(resolveLanguageCode(langPref));
  // ...
  return Text(s.agentListTitle);   // finished string, no flag
}
```

If a child widget needs copy, **pass the finished `AppStrings s`** (or the
specific finished strings), never `isId`. A widget that needs three labels
takes three resolved `String`s or the `AppStrings` instance — not a bool.

### 1.3 No hardcoded natural-language strings

No Indonesian or English literal may appear in a widget tree. Every
user-facing string is a getter/method on `AppStrings`
(`lib/features/settings/data/app_language_provider.dart`). Add the getter there
**before** referencing it in UI.

Exempt (not translatable): brand name `MEOW AGENT`, route names, model-id
hints (`gpt-4o-mini`), emoji, and other non-language tokens.

### 1.4 New features add their strings first

Adding a screen/dialog/snackbar = add every label to `AppStrings` first, with
both `id` and `en` variants, then reference them. A getter that returns the
same text for both languages is fine when the term is a proper noun, but it
must still go through `AppStrings`.

### 1.5 Runtime (agent-facing) language is separate

The agent's spoken language is handled by the runtime via `DetectedLanguage` +
`ToolVerbalizer` / `NarrativeNarrator` / `LanguageRegistry` — NOT by
`AppStrings`. Never wire `AppStrings` into runtime prompt logic, and never wire
`DetectedLanguage` into UI copy. They are two different layers (see §3).

---

## 2. Prompt Rules (STRICT)

### 2.1 All prompt text lives in `prompt_*` files

Every LLM-facing string lives under `lib/services/agent_runtime/`:

| File | Owns |
|------|------|
| `prompt_constants.dart` | Central accessor (`PromptConstants.*`) + version + caching |
| `prompt_system.dart` | System rules, introduction gate |
| `prompt_analyze.dart` | Analyzer phase (intent, tool_groups, selectors) |
| `prompt_reflect.dart` | Reflector phase (strategy, impacts, slots) |
| `prompt_plan.dart` | Planner phase (goal tree) |
| `prompt_execute.dart` | Tool selector + reviewer |
| `prompt_context.dart` | Chat, compactor, repair, pending action, memory, workflow API context |
| `prompt_policy.dart` | Reusable policy blocks (Ask / Ground / Minimal / Recover / Voice) |
| `prompt_workflow.dart` | Workflow auto-execute prompts |
| `prompt_templates.dart` | Assembles the above into final prompts |

**Banned:** inline prompt strings in `runtime_engine.dart`, `workflow_runner.dart`,
module code, or any feature file. If you are writing a sentence the LLM will
read, it belongs in a `prompt_*` file and is exposed through `PromptConstants`.

### 2.2 Prompts are English-only

All prompt scaffolding and examples are authored in English. The LLM responds
in the user's language naturally via the separately-injected `DetectedLanguage`.

- No per-language example sets.
- Never enumerate language-specific words ("semua/setiap" / "all/every") as a
  fixed list. Describe the concept semantically ("any word meaning all/every in
  any language").
- The `tool_groups` enum and `capabilityHints` are English-only closed sets.
- Bulk/predicate selectors are structural, never language-dependent.

### 2.3 No redundant prompt copy

A spec (e.g. the narrative one-sentence rule) is documented once and referenced,
not copy-pasted across phases. When you refactor a phase, delete the prompt
constants it no longer uses — dead prompt constants are a defect.

### 2.4 Dynamic values via parameters

Inject dynamic content with typed parameters
(`String fooPrompt(String name)`), never by string-concatenating in the caller.

---

## 3. Layering Rules

Two language systems, never crossed:

```
UI copy            ──▶  AppStrings (id/en)              ──▶  widgets
agent spoken lang  ──▶  DetectedLanguage + Verbalizer   ──▶  chat bubbles
prompt scaffolding ──▶  prompt_* (English only)         ──▶  LLM
```

- UI never reads `DetectedLanguage`. Runtime never reads `AppStrings`.
- Prompts never embed user-facing localized copy; they request structured
  output and let the verbalizer render language.

---

## 4. Code Reuse & Consistency

### 4.1 Reuse the design system

Use the existing widgets (see DESIGN_2.md §components) instead of rebuilding:
`MeowCard`, `MeowInput`, `MeowDropdown`, `MeowPrimaryButton`,
`MeowSecondaryButton`, `MeowSection`, `MeowAgentIcon`, `showMeowConfirmDialog`.
New shared UI goes in `lib/app/widgets/` and is exported from `widgets.dart`.

- Read theme tokens via `context.cs` / `context.extras` — never hardcode a hex
  color in a widget.
- Match the spacing scale and radii in DESIGN_2.md; don't invent new ones.

### 4.2 Tools and modules are one-file additions

Adding a tool/module = ONE `ModulePlugin` file + one line in
`runtime_module_plugins.dart`. There is no central dispatch switch, registry
map, or catalog map to hand-edit (see ARCHITECTURE_2.md §4).

### 4.3 Permissions are gated, never fail-open

Every new tool that mutates state or touches sensitive data MUST have a gate
entry in `tool_permission_requirements.dart` (exact map) — or be covered by a
prefix rule in `toolPermissionPrefixRequirements`. A tool absent from both
**fails open** (the policy allows it).

`test/tool_permission_coverage_test.dart` enforces this: every registered tool
must be gated OR in the documented `intentionallyUngated` allowlist. Run it
after adding any tool. Prefer a gate entry over expanding the allowlist.

### 4.4 Risk comes from the registry, not the LLM

`risk` and `requiresConfirmation` are read from the `ToolDefinition`, never
trusted from model output. Never route a security decision through LLM text.

### 4.5 Error handling

- Tools return `ToolExecutionResult` — never throw across the dispatch boundary.
- Native (Kotlin) returns `Map<String, Any?>`, wrapped in try/catch, never
  crashes the app, logs with `Log.e(TAG, msg, e)`, and never blocks the main
  thread.
- LLM JSON calls go through `LlmJsonCaller` (one repair retry); handle a `null`
  result as a real failure path, don't assume success.

### 4.6 Every mutating tool has a `verificationProbe`

A tool that reports `success: true` for a mutation without a `verificationProbe`
is a gap. Use `tool_result_data` (assert keys) or `snapshot_contains` /
`snapshot_absent` (re-read state).

---

## 5. Testing Requirements

| Suite | File | When to run |
|-------|------|-------------|
| Golden end-to-end | `test/runtime_golden_test.dart` | after any runtime refactor |
| Module drift guard | `test/module_plugin_test.dart` | after adding/changing a module |
| Permission coverage | `test/tool_permission_coverage_test.dart` | after adding/gating any tool |
| Per-tool unit | `test/<module>_test.dart` | with every new tool |

Minimum per new tool: success path, empty/null input (no crash), permission
missing (safe fallback), registered with correct risk/confirmation metadata,
and `verificationProbe` present for mutations.

Real-LLM flow tests (`test/runtime_real_llm_test.dart`) read credentials from
`.env` (gitignored). Never hardcode or commit credentials. These tests no-op
without `.env`, so a green run without creds proves nothing — run them with a
real provider when validating multi-turn flows.

---

## 6. Distribution & Permission Philosophy

1. **Not targeting Play Store** (direct APK / sideload). No store-policy
   constraints on Accessibility Service, background services, or system
   integrations.
2. **Accessibility-based automation is allowed** — implement freely.
3. **Everything is opt-in.** The app never force-enables or silently activates a
   permission. Pattern: **Present → Explain → Request → Respect.** Degrade
   gracefully when denied.
4. **User stays in control.** Any permission can be revoked at any time;
   `ModulePermissionReconciler` flips dependent toggles off when the OS
   permission is revoked.

---

## 7. Pre-Commit Checklist

- [ ] No `isId` anywhere under `presentation/` or `app/widgets/`.
- [ ] No inline natural-language literal in a widget tree.
- [ ] New UI copy added to `AppStrings` (both `id` + `en`) before use.
- [ ] No inline prompt string outside `prompt_*` files.
- [ ] New prompt text is English-only, no per-language word lists.
- [ ] Reused existing widgets / theme tokens; no duplicated styling or hex colors.
- [ ] New tool: one `ModulePlugin` file + registered in `runtime_module_plugins.dart`.
- [ ] New tool: gated (exact or prefix) or justified in the allowlist.
- [ ] Mutating tool: has a `verificationProbe`.
- [ ] `flutter analyze` clean; relevant test suites pass.

---

## 8. The One Question

Before merging, ask:

> "Is this consistent with what's already here — same widgets, same string
> system, same prompt home, same gate — and does it tell the truth about what
> the agent can do?"

If no: align it before shipping.

---
> Source: [Ibnuard/meow-agent](https://github.com/Ibnuard/meow-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
