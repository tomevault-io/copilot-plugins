## drive-time

> FlutterFlow AI is a local workspace for creating and editing FlutterFlow apps with a coding agent.

# FlutterFlow AI Workspace

FlutterFlow AI is a local workspace for creating and editing FlutterFlow apps with a coding agent.

## Files

- `dsl/create.dart`
- `dsl/edit.dart`
- `test/app_test.dart`
- `references/`
- `patterns/`
- `PROJECT_CONTEXT.md`
- `context/`
- `generated_code/` — read-only snapshot of the Flutter code FlutterFlow generates from the project. Manifest at `generated_code/.flutterflow/export_manifest.json` maps each entity (page, component, action block, etc.) to its `primary_files`. Use this when debugging visual or runtime bugs the DSL alone cannot explain (overflow, layout, render errors, build failures).
- `.flutterflow/` (SDK-managed: run history, traces, workspace state, plus router config)

## Workflow

### Selector-first edit workflow

If the user pasted a `FlutterFlow AI Selector v1` block, use it before any broad page/component inspection:

1. Parse the pasted block for `project_id`, `scope_kind`, `scope_name`, `selector_path`, `node_key`, `node_name`, and `node_type`.
2. Run `flutterflow ai inspect <project_id> --page|--component <scope_name> --selector-path <selector_path> --dsl-json` to resolve the target widget.
3. Verify the returned `node_type` and `node_name` match expectations from the pasted block.
4. **If the user is reporting a visual or runtime bug** (overflow, layout, render error, exception, "looks wrong" / "doesn't fit"): before authoring the patch, read the generated Dart for the selector's scope.
   - Look up the entity in `generated_code/.flutterflow/export_manifest.json` by `name == scope_name` (or `key == node_key`).
   - Read its `primary_files` to see the actual widget tree, constraints, and styling Flutter is rendering.
   - The DSL is intent; the generated code is what is actually running. Overflow, an unbounded `Column` inside a `Row`, fixed sizes vs. `Expanded`, etc. are only visible there.
   - If `generated_code/` is missing or stale (`flutterflow ai codegen status` reports `stale`/`missing`), run `flutterflow ai codegen refresh` first.
5. Author the patch in `dsl/edit.dart` using `findByPath(...)`:
   ```dart
   app.editPage('PageName', (page) {
     page.findByPath('PageName.body[0].children[1]').update((patch) {
       // ...
     });
   });
   ```
6. Run `dart test`, then `flutterflow ai validate`, then `flutterflow ai run`.
7. If `--selector-path` fails, fall back to `--selector-key` with the `node_key` from the block.
8. Only do a broad `flutterflow ai inspect --page/--component` pass when the selector is stale or missing.

### General workflow

1. Start from the closest working examples in `references/`. Do not read the full API surface first unless the references are insufficient or you are blocked.
2. For edit work, inspect first with `flutterflow ai inspect <project-id>`.
3. Edit `dsl/create.dart` or `dsl/edit.dart`.
4. Update `test/app_test.dart` to match your changes (page names, component names, expected structure). The starter test references `StarterPage` — change it to match whatever you built.
5. Run `dart test`.
6. Run `flutterflow ai validate ...`.
7. **Execute the push** — this is NOT optional, always run this as the final step. Always include `--commit-message` with a short description of what changed:
   - **Create:** `flutterflow ai run dsl/create.dart --project-name "<name>" --commit-message "<what the app does>"`
   - **Edit:** `flutterflow ai run dsl/edit.dart --project-id "<id>" --commit-message "<what changed>"`
   - Use `--find-or-create` only as a retry/recovery option when a previous create run may already have created the remote project but the local workspace is not bound yet.
   - If the workspace is already bound to a project in `.flutterflow/workspace.json`, FlutterFlow AI will refuse plain create mode by default. Use `--allow-new-project` only when you intentionally want a second project from the same workspace.
8. After the **first successful push**, run `flutterflow ai refresh-context <project-id>` using the project ID from `.flutterflow/workspace.json` to generate `PROJECT_CONTEXT.md`.

## Design & Quality Rules

**These are mandatory for every create and edit script.**

### Theme first
Always set up a theme before building UI. Use `app.themeColor()`, `app.primaryFont()`, and `app.darkMode()` to define a coherent color palette. Never hardcode hex colors in widgets — use `Colors.primary`, `Colors.secondary`, `Colors.secondaryText`, etc. See `references/styled_profile_dsl.dart` for the pattern.

### Components for reuse
Extract repeated widget subtrees into `app.component()`. If a card, list item, or section appears more than once (or would appear in multiple pages), make it a component with typed `params:`. Prefer small, focused components over monolithic pages.

### Default values on params
Any `params:` entry whose `DslType` has no `.withDefault(...)` is treated as **required**. That has two consequences you must handle:

- **Component instances** must pass every required param at every call site (the SDK throws "Missing required parameter(s)" at compile time if you forget) — but those values are still arbitrary at runtime, so a `null`-bound state passed in will crash the page.
- **Pages**, especially the initial page or any page reachable via deep link / app-link / external nav, can be entered with no arguments at all. A required page param read during build (e.g. inside `Text(Param('userId'))`) will resolve to `null` and throw a runtime null error before the page renders.

Rule of thumb: give every param a `.withDefault(...)` unless you can prove every call site supplies a non-null value. Defaults are cheap; null-deref crashes in the rendered Flutter app are not.

```dart
// Page param with a safe default — won't null-crash on cold entry.
app.page(
  'BrowsePage',
  isInitial: true,
  route: '/',
  params: {
    'query': string.withDefault(''),
    'page': int_.withDefault(0),
  },
  body: ...,
);

// Component param with a default for the optional case.
app.component(
  'EmptyState',
  params: {
    'title': string,                                // required
    'subtitle': string.withDefault(''),             // optional
    'icon': iconRef.withDefault('inbox'),
  },
  body: ...,
);
```

Defaults are available on every `DslType` (`string`, `int_`, `double_`, `bool_`, `dateTime`, enums, struct handles, list types, etc.) via `.withDefault(value)`.

### Descriptions everywhere
Always provide the `description:` parameter on `app.page()`, `app.component()`, `app.actionBlock()`, `app.collection()`, `app.table()`, `app.event()`, and `app.customFunction()`. Descriptions appear in the FlutterFlow editor and help other builders understand what each element does. Use short, clear descriptions (e.g., `description: 'Displays a single todo item with toggle and delete'`).

### Visual quality
- Give buttons proper width (`width: double.infinity` for full-width, or specific widths), `padding:`, `borderRadius:`, and `color:`/`textColor:`. Default buttons look thin and unstyled.
- Use `Container` with `padding:`, `borderRadius:`, and `color:` for cards and sections — not bare children.
- Use `spacing:` on `Column`/`Row` for consistent vertical/horizontal rhythm.
- Use `Styles.titleLarge`, `Styles.bodyMedium`, etc. for text hierarchy — not unstyled `Text()`.
- Use `maxLines:` + `overflow: TextOverflow.ellipsis` on text that could overflow.
- Always size circular loading indicators explicitly. Use `ProgressBar.circular(size: 40, thickness: 4)` instead of the unsized default.
- Avoid `shrinkWrap: true` on dynamic `ListView(...)` widgets unless the list truly must live inside another scrollable. Prefer giving the list bounded space and leaving shrink wrap off.

### Action outputs
- If a page or component has more than one backend action with an output variable, set `outputAs:` explicitly on each action. Do this for `ApiCall`, `FirestoreQuery`, `PostgresQuery`, and any similar action that exposes an output.
- In edit edits, prefer updating an existing trigger chain instead of appending a second API call with another generated output variable on the same page.
- If you see validator errors about output variables already being in use, the fix is usually to rename the outputs with `outputAs:` or move the second fetch behind a user action.

## Create → Edit Transition

**IMPORTANT:** Create scripts (`dsl/create.dart`) are one-shot — they create pages and components from scratch. You **cannot** re-run a create script against the same project; it will fail with duplicate-name errors.

After the first successful create push:
1. The project now exists. Read `projectId` from `.flutterflow/workspace.json`.
2. If `flutterflow` CLI is available, FlutterFlow AI also exports a local Flutter snapshot into `generated_code/`.
3. Run `flutterflow ai refresh-context <project-id>` to generate `PROJECT_CONTEXT.md`.
4. For all subsequent edits, use **edit flows** in `dsl/edit.dart` with `--project-id "<id>"`.
5. Use `flutterflow ai inspect <project-id> --page <PageName>` to understand the current page structure before editing.
6. Read `references/taskboard_dsl.dart` or other edit references for patterns.
7. After later pushes, `generated_code/` is re-exported automatically when context is refreshed (so the project map and generated Dart stay in sync). If a push leaves the snapshot stale (codegen skipped or the export failed), run `flutterflow ai codegen refresh`.

Do NOT modify and re-run `dsl/create.dart` to make changes to an existing project.
Do NOT switch back to `--project-name` in a bound workspace unless you intentionally want a separate project and pass `--allow-new-project`.

## Edit Context

- `flutterflow ai init --project <id>` creates a project-bound workspace and writes `PROJECT_CONTEXT.md` when credentials are available.
- When available, `flutterflow ai init --project <id>` also exports a local Flutter snapshot into `generated_code/`.
- `generated_code/` is the **runtime truth**. The DSL describes intent; the generated Dart is what Flutter actually builds and renders. Read it whenever you need to reason about layout, sizing, overflow, render exceptions, build errors, or any "why does the rendered app look or behave like this" question — these are not answerable from the DSL alone.
- Use the manifest at `generated_code/.flutterflow/export_manifest.json` to jump directly from an entity (page, component, action block) to its `primary_files`. Look up by `name` (matches the selector's `scope_name`) or `key` (matches `node_key`). Do not grep for files when the manifest exists.
- Treat `generated_code/` as read-only. Do NOT edit files there directly — make changes in `dsl/edit.dart` or other FlutterFlow AI-managed source, then push through `flutterflow ai run`.
- If a task starts from a generated Dart file, identify the corresponding page, component, or resource from that file and apply the change through FlutterFlow AI rather than patching the generated output.
- `flutterflow ai refresh-context <project-id>` rewrites `PROJECT_CONTEXT.md`, `context/`, `dsl_json/` **and** re-exports `generated_code/` so the context map and the generated Dart stay in lockstep. Pass `--no-codegen` to skip the generated-code refresh; in that case the snapshot is marked stale and `flutterflow ai codegen status` / `codegen refresh` apply.
- Run `flutterflow ai refresh-context <project-id>` after meaningful remote changes or when the local context looks stale.
- Run `flutterflow ai context-check` to verify whether local context metadata is still fresh.
- Treat `PROJECT_CONTEXT.md` as the summary map. Use `flutterflow ai inspect` and `flutterflow ai resources` when you need exact current page, component, or resource details.

## Edit APIs for Existing Resources

### Existing collections
```dart
final trips = app.existingCollection('trips');
// Use like any collection handle: listOf(trips), docRef(trips), FirestoreQuery(trips, ...)
```

### Existing components
```dart
final tripCard = app.existingComponent('TripCard', params: {'title': string});
// Use in layouts: tripCard(title: State('name'))
```

`name:` and `visible:` are reserved on every component-instance call and forwarded to the resulting widget; do not declare component params with those names:
```dart
final hero = tripCard(title: 'Paris', name: 'HeroTripCard', visible: State('showHero'));
```

### Component param binding
```dart
app.editPage('MyPage', (page) {
  page.setComponentParam(page.findByName('FilterChip'), 'active', Equals(State('filter'), 'all'));
});
```

### Page-load actions
```dart
app.editPageOnLoad('MyPage', [
  FirestoreQuery(trips, limit: 20, outputAs: 'loaded'),
  SetState('tripsList', ActionOutput('loaded')),
]);
```

### Idempotent creation
```dart
app.ensurePage('CreateTrip', route: '/create', body: ...); // no-ops if page exists
app.ensureFirebaseAuth(providers: [...], homePage: '...', signInPage: '...'); // no-ops if auth configured
```

### Removing existing entities

The `App` exposes typed remove APIs that delete entities from the bound project at compile time. All of them fail loudly if the name is also declared or referenced in the same `App` (mixing them would silently undo the change or leave dangling references):

```dart
app.removePage('LegacyPage');
app.removeComponent('OldBanner');
app.removeCollection('archivedTrips');
app.removeTable('legacy_users');
app.removeDataStruct('DeprecatedModel');
app.removeEnum('OldStatus');
app.removeActionBlock('legacyHandler');
app.removeAppEvent('deprecatedEvent');
app.removeCustomFunction('oldFn');
app.removeCustomAction('oldAction');
app.removeCustomWidget('OldWidget');
app.removeSpacingToken('legacy_md');
app.removeRadiusToken('legacy_sm');
app.removeShadowToken('legacy_card');
```

There is no `app.removeProject(...)` — project-level deletion is intentionally not in the DSL.

### Edit property patches

`page.update(selection, (patch) { ... })` accepts these typed methods on `EditWidgetPatch`:

- `text`, `color`, `visible`, `spacing`, `padding`, `borderRadius`, `size`, `icon`, `buttonVariant`, `textFieldLabel`, `textFieldHint`
- `margin(EdgeInsets...)` — wraps the target in a re-run-stable `<Target> Margin` Container; subsequent patches in the same `update` still apply to the wrapped node
- `alignment(Alignment...)` — Container/Stack child alignment
- `border({color, width})`, `shadow(Shadow(...))`, `opacity(0.0..1.0)` — Container box decoration
- `EditParamEditor.updateParam(name, {type, description})` — partial drift on existing page/component params
- `page.mutateNode(selection, (FFNode node) { ... })` — escape hatch for raw node mutations not yet exposed via the typed patch surface; closure must be idempotent

### Create gradients

```dart
Container(
  gradient: Gradient.linear(
    [GradientStop(Colors.primary, 0), GradientStop(Colors.secondary, 1)],
    angle: 135,
  ),
)
```

## Runtime Artifacts

- `.flutterflow/runs.jsonl`: local run history
- `.flutterflow/history/<run-id>/`: archived source files and plan
- `.flutterflow/traces/<run-id>.json`: canonical run trace
- Use `flutterflow ai history`, `flutterflow ai trace latest`, and `flutterflow ai support inspect <run-id>` to debug what happened.

## Source Tracking

- FlutterFlow AI keeps the source that produced each run for auditability and replay.
- By default, `flutterflow ai run dsl/create.dart` or `flutterflow ai run dsl/edit.dart` tracks the executed DSL script.
- Support tooling can turn a traced run into a bundle or replay workspace with `flutterflow ai support bundle`, `flutterflow ai support replay`, or `flutterflow ai support case`.

## References

- Start from the closest working examples in `references/` before inventing new DSL structure.
- Only use `flutterflow ai docs api-surface` or `flutterflow ai docs ui` when the references do not cover what you need or you are blocked on a specific API detail.
- `flutterflow ai docs api-surface` covers the lower-level helper contract. `flutterflow ai docs ui` covers the broader widget and action authoring surface.
- CRUD: `references/shopflow_dsl.dart`
- Task board: `references/taskboard_dsl.dart`
- Auth: `references/auth_shell_dsl.dart`
- Supabase: `references/supabase_crud_auth_shell_dsl.dart`
- Firestore: `references/social_feed_data_dsl.dart`
- Forms: `references/workflow_forms_dsl.dart`
- Shell/navigation: `references/commerce_shell_dsl.dart`
- Content generation: `references/content_companion_dsl.dart`
- Resource/library usage: `references/resource_library_dsl.dart`
- Postgres compile-only: `references/postgres_compile_only_dsl.dart`
- Action blocks: `references/action_block_showcase_dsl.dart`
- App events: `references/app_event_showcase_dsl.dart`
- GenUI: `references/genui_catalog_assistant_dsl.dart`
- Action reuse/composability: `references/taskboard_dsl.dart`
- Local state CRUD (lists, forms, per-item actions): `references/local_state_crud_dsl.dart`
- Theming, styling, layout (colors, fonts, sizing, borders, password fields): `references/styled_profile_dsl.dart`
- Media/content (horizontal lists, grids, images, text truncation, scrollable rows): `references/media_browser_dsl.dart`
- Asset/reference types (`imagePath`, `videoPath`, `audioPath`, `docRef(...)`, typed media/reference state): `references/asset_and_reference_surface_dsl.dart`
- Edit: search + filter on existing page: `references/edit_add_search_filter_dsl.dart`
- Edit: add form + detail page + navigation: `references/edit_form_and_detail_dsl.dart`
- Edit: restyle, enhance, empty states, refresh: `references/edit_restyle_and_enhance_dsl.dart`
- Edit: existing collections, components, data binding, idempotent ops: `references/edit_data_binding_dsl.dart`
- Multiple API calls with explicit `outputAs:` naming: `references/multi_api_call_dsl.dart`
- Custom code + pub.dev packages (pairing a custom action with `http`, a custom widget with `intl`): `references/custom_code_pub_package_dsl.dart`
- Custom Dart classes + enums used as typed args/returns via `classRef` / `customEnumRef`: `references/custom_code_classes_and_functions_dsl.dart`

## Custom code authoring

The SDK is the canonical way to add, update, and remove user-authored Dart inside a FlutterFlow project.

### The five artifact kinds + pub deps

| Artifact | DSL (greenfield) | Helper (brownfield) |
| --- | --- | --- |
| Custom function | `app.customFunction` | `addCustomFunction` |
| Custom action | `app.customAction` | `addCustomAction` |
| Custom widget | `app.customWidget` | `addCustomWidget` |
| Custom class | `app.customClass` | `addCustomClass` |
| Custom enum | `app.customEnum` | `addCustomEnum` |
| Pub dep | `app.pubDependency` / `pubDevDependency` / `pubDependencyOverride` | `addPubDependency` / `addDevDependency` / `addDependencyOverride` |

### Greenfield vs brownfield — the rule

If you're building a fresh app inside `buildApp((app) { ... })`, use the DSL. If you pulled an existing project and are editing it, call the helpers directly. Don't mix within one script.

### Validation runs automatically

Every `add*` / `update*` / DSL entry runs format + identifier + shape checks before mutating the project. Failures raise typed exceptions:

```dart
try {
  app.customAction('fetchUser', code: actionSource);
} on CustomCodeDuplicateError catch (e) {
  // name taken — pick another
} on CustomCodeValidationError catch (e) {
  // e.issues lists every validator problem
}
```

What's NOT caught: type-correctness against the rest of the project (missing imports, wrong `FFAppState` field, signature mismatches). For those, iterate via the staging sandbox below.

### Pairing a custom artifact with a pub package

Declare both in the same place:

```dart
buildApp((app) {
  app.pubDependency('dio', '^5.4.0');
  app.customAction(
    'fetchUser',
    args: {'id': string},
    returns: string,
    code: """
import 'package:dio/dio.dart';
Future<String> fetchUser(String id) async {
  final res = await Dio().get<String>('https://example.com/users/\$id');
  return res.data ?? '';
}
""",
  );
});
```

Pub.dev package discovery (which package to use, which version) is your job — the SDK only records the resolution.

### Typing your custom-code parameters

Custom function / action / widget parameters and return types use `DslType`. Full coverage:

- Scalars: `string`, `int_`, `double_`, `bool_`, `dateTime`, `imagePath`, `videoPath`, `audioPath`, `mediaPath`, `latLng`, `color`, `json`, `uploadedFile`
- Collections: `listOf(T)`
- User-authored: `classRef(handle)` / `customEnumRef(handle)` — where `handle` is what `app.customClass(...)` / `app.customEnum(...)` returned
- Schema: `app.enum_(...)` handle, `app.struct(...)` handle
- Firestore / Postgres: `FirestoreCollectionHandle`, `DocumentReferenceType`, `PostgresTableHandle`
- Actions: `action`

Example — a custom function whose return type is a user class:

```dart
final user = app.customClass(
  'User',
  code: 'class User { final String name; const User(this.name); }',
);
app.customFunction(
  'buildUser',
  args: {'name': string},
  returns: classRef(user),
  code: 'return User(name);',
);
```

Types not yet in the DSL (`Document`, `SQLiteRow`, `WidgetBuilder`, `ChildSlotParam`, `ApiResponse`, RevenueCat types, etc.): drop into `app.raw((project) { ... })` and set the `FFParameter`'s `dataType` directly. File an issue if you hit one that gets used a lot — we'll promote it into `DslType`.

### Iteration loop for non-trivial custom code

`generated_code/` in your workspace is the canonical mirror of the currently-pushed project. **Never write to it.** For custom code that references generated types (`FFAppState`, structs, existing components, collection fields), use the staging sandbox:

1. Write your candidate file under `.ffai_staging/lib/my_file.dart` (gitignored, ephemeral).
2. Run `dart analyze .ffai_staging/` — it resolves types through a path dep on `generated_code/` and surfaces compiler errors the SDK validators can't see.
3. Fix issues, re-analyze until clean.
4. Pass the validated code string to `app.customWidget(...)` / `addCustomAction(...)` / etc.
5. Compile + push. The server regenerates `generated_code/` on your next `refresh-workspace`, keeping it consistent.

Skip the sandbox for trivial edits that only touch pure-Dart values.

### Explicit non-goals

The SDK deliberately does not:
- Search pub.dev or resolve package versions.
- Parse your Dart into structured AST (the IDE re-parses on open).
- Shell out to `dart analyze` — that's your call from the staging sandbox.
- Enforce lint rules beyond format + identifier + shape.

---
> Source: [diogolufada-web/drive-time](https://github.com/diogolufada-web/drive-time) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
