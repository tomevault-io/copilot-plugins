## json-to-dart-advanced

> Context for AI assistants working on this project. Read this first.

# CLAUDE.md

Context for AI assistants working on this project. Read this first.

## What this is

A static, browser-only tool that converts JSON to Dart model classes. Inspired by https://javiercbk.github.io/json_to_dart/ but tuned for real-world Flutter codebases — the conventions match how Flutter teams actually write models by hand (named-constructor `fromJson`, nullable fields, defensive parsing, opinionated folder layout).

**Pure static site** — no backend, no build step. Push the folder to GitHub Pages and it runs.

## Tech stack

- HTML + CSS + ES modules (vanilla JS, no framework)
- JSZip (loaded from CDN) for ZIP downloads
- LocalStorage for project persistence
- `<dialog>` element for modals (modern browser only)

## File map

```
index.html              # UI shell — three tabs: Generator / Projects / Docs
styles/
  theme.css             # Color tokens (dark + light themes via [data-theme])
  main.css              # Layout
  components.css        # Buttons, cards, fields, modals, code preview
js/
  parser.js             # JSON value → typed model tree (type inference)
  naming.js             # Class names + folder layout (the api/param rules)
  generator.js          # Tree → Dart files (THE HEART — most logic lives here)
  convertService.js     # Returns the convert_service.dart string template
  downloader.js         # triggerDownload + buildGenerationZip
  projects.js           # localStorage project CRUD + buildProjectZip
  storage.js            # localStorage wrapper (single key: j2d_advanced_v1)
  docs.js               # Markdown docs generator (per-class + index)
  app.js                # UI controller — wires everything to DOM events
```

## Generation conventions (CRITICAL — don't violate)

These are the rules the user explicitly requested. They override default `json_to_dart` behavior.

### Naming

| Mode | Root class | Root folder/file |
|---|---|---|
| `api` (response) | `Api<Name>Model` | `api_<name>/api_<name>_model.dart` |
| `param` (request) | `Pm<Name>Model` | `pm_<name>/pm_<name>_model.dart` |

- Sub-classes are always `<Name>Model` (PascalCase + `Model` suffix).
- File naming: snake_case + `_model.dart`.
- **Folder rule:** a sub-class with its own nested objects gets its own folder under the parent. A leaf sub-class is a flat file in the parent's folder.

### Type inference (matches user's spec)

| JSON | Dart |
|---|---|
| `null`                 | `dynamic` (NOT `Null?`) |
| `[]` or all-null list  | `List<dynamic>?` |
| Mixed-type list        | `List<dynamic>?` |
| `[1, 2.0]`             | `List<double>?` (only int+double promote; everything else mixes → dynamic) |
| Whole number           | `int?` |
| Fractional number      | `double?` |

### Class shape

```dart
class FooModel {
  String? id;          // nullable fields
  String? name;

  FooModel({this.id, this.name});             // default constructor

  FooModel.fromJson(Map<String, dynamic> json) {  // NAMED CONSTRUCTOR, not factory
    id = json['id'];
    name = json['name'];
  }

  Map<String, dynamic> toJson() {
    final data = <String, dynamic>{};
    data['id'] = id;
    data['name'] = name;
    return data;
  }
}
```

- Nested objects in `fromJson`: wrap in `if (json['key'] != null) { field = ChildModel.fromJson(json['key']); }`.
- Lists of objects in `fromJson`: `if (json['key'] != null) { field = <ChildModel>[]; json['key'].forEach((v) { field!.add(ChildModel.fromJson(v)); }); }`.
- Object/list-of-object in `toJson`: ALWAYS wrapped in null check (otherwise `!.toJson()` crashes), regardless of mode.

### Param mode `toJson` — null-safe

In `param` mode, **every** assignment is wrapped in `if (field != null)`. The user wants empty-keyed payloads, never `{ "field": null }`.

### ConvertService option

When enabled:
1. Every `fromJson` primitive assignment uses `ConvertService.convert<Type>(json[...])`.
2. List-of-primitive assignments use `ConvertService.parse<Type>List(json[...])`.
3. Each generated file imports `package:<pkg>/utils/convert_service.dart` (or relative `utils/convert_service.dart` if no package name).
4. The downloaded ZIP includes `utils/convert_service.dart`.

Mapping:
- `String` → `convertString`, `int` → `convertInt`, `double` → `convertDouble`, `num` → `convertNum`, `bool` → `convertBool`
- `List<String>` → `parseStringList`, `List<int>` → `parseIntList`, `List<double>` → `parseDoubleList`, `List<num>` → `parseNumList`, `List<bool>` → `parseBoolList`

The bundled `convert_service.dart` includes nullable variants, `convertDateTime`, `convertMap`, and a generic `parseList<T>(json, fromJson)`. See `js/convertService.js`.

### Capabilities (each toggleable)

Static helpers added inside the class:

- `parseItem(json)` — single-object safe parse, returns `Class()` on failure.
- `parseItems(json)` — handles **both** `[{}, {}]` AND `{ "data": [{}, {}] }`.
- `copyItem(source)` — deep copy via `fromJson(toJson())`.
- `copyItems(list)` — list version.
- `toString()` override — picks first existing field of: `name`, `title`, `label`, `id`; else first String field; else first field.

## Where the rules are enforced

- **Naming + folder layout** → `js/naming.js` (`nameForNode`, `planFileLayout`, `relativeImport`).
- **Type inference** → `js/parser.js` (`primitiveDartType`, `inferListElement`, `mergeObjectSamples`).
- **Class rendering** → `js/generator.js`. Specifically:
  - `renderFromJsonAssignment` — handles primitive/object/list/dynamic + ConvertService swap.
  - `renderToJsonAssignment` — handles null-check wrapping.
  - `renderParseItem`, `renderParseItems`, `renderCopyItem`, `renderCopyItems`, `renderToStringOverride`.

## Persistence model (LocalStorage)

Single key: `j2d_advanced_v1`. Shape:

```js
{
  theme: "dark" | "light",
  projects: { [id]: Project },
}

// Project
{
  id, name, description, createdAt, updatedAt,
  modelSets: [
    { id, featureName, mode, rootKey, options, json, files: [{path, content, className}], createdAt }
  ]
}
```

`featureName` is used to place models under `lib/features/<feature>/models/...` in the project ZIP export.

## Verification approach

There is no test suite. To verify generator changes:

```bash
# Smoke test (script not committed — write ad-hoc):
node -e "
import('./js/parser.js').then(p => import('./js/generator.js').then(g => {
  const tree = p.parseJson({ id: 1, items: [{ name: 'a' }] }, 'home');
  const files = g.generate({ tree, mode: 'api', rootKey: 'home', packageName: null,
    nullSafeToJson: false, useConvertService: false, capabilities: {} });
  for (const f of files) console.log('---', f.path, '---\n' + f.content);
}));
"
```

Or run the UI:
```bash
python3 -m http.server 8080
# open http://localhost:8080
```

The **Load sample** button in the Generator tab seeds a representative JSON that exercises: nested objects, nested lists with object elements that themselves have nested lists, null fields, empty lists, and mixed types.

## Known limitations / gotchas

- **Class-name collisions:** two siblings keyed `user` would both produce `UserModel`. Generator does not de-dup or namespace. Document if you change this.
- **Object samples are merged across array elements** (so `MenuModel` includes fields from both `{title, sub_menu: [...]}` and `{title}`). Intentional — gives a forgiving union shape.
- **Saved model sets are read-only.** No edit-in-place yet (see roadmap).
- **`<dialog>` element** is used for modals — Safari < 15.4 won't work. Modern-browser only is acceptable here.
- **JSZip is loaded from a CDN** (cdnjs). If you self-host, vendor it into `vendor/` and update the script tag in `index.html`.

## When changing things

- **Adding a new option:** update `js/app.js` `readOptions()` AND the HTML form in `index.html` AND the generator's destructured params. Persist via `lastResult.options` so save-to-project captures it.
- **Adding a new capability:** add a renderer to `generator.js`, wire a checkbox into `index.html`, read it in `app.js readOptions()`. Make sure the static method ends up inside the class, not after the closing brace.
- **Tweaking ConvertService:** edit `js/convertService.js`. The Dart string is returned by a single function — keep it Dart-formatted (the user runs `dart format` on output).
- **Style changes:** prefer editing `styles/theme.css` (tokens) over component CSS. Both light and dark themes must stay legible — test the toggle.

## What NOT to do

- Don't switch to a framework (React/Vue/etc.) — the static-site, no-build property is load-bearing for "publish to GitHub Pages by pushing the folder".
- Don't split the convert_service.dart template into another file — keeping it as a JS string keeps the build step at zero.
- Don't add backend calls. Everything is local. The user pastes potentially private API responses — sending them anywhere would break trust.
- Don't rename existing classes/methods to be "cleaner" without checking the user's stated conventions above. The naming was specifically requested.

## Roadmap items the user mentioned

- Edit saved model sets in place.
- Custom field renaming overrides.
- `Equatable` / `==` / `hashCode` capability.
- "Compare against existing model" diff view.

## Output style reference

Generated output should feel hand-written, not codegen'd. Concretely that means:
- Nullable fields, named-constructor `fromJson` (NOT a factory), explicit `forEach` loops for lists of objects.
- No `json_serializable` / `freezed` annotations — output must compile standalone with zero dependencies (except `dart:convert` if `ConvertService` is bundled).
- Comments are sparse. The user wants clean files they can drop into existing apps without explaining where each line came from.

---
> Source: [Mahshook-HITEK/json_to_dart_advanced](https://github.com/Mahshook-HITEK/json_to_dart_advanced) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
