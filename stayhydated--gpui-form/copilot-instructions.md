## gpui-form

> This file is the working guide for contributors and coding agents in the `gpui-form` workspace.

# AGENTS.md

This file is the working guide for contributors and coding agents in the `gpui-form` workspace.

Use it to answer three questions quickly:

1. Where does this documentation belong?
1. Which crates are public entry points versus integration points versus internals?
1. What other surfaces must be updated in the same change?

## Project summary

`gpui-form` is a Rust form-generation ecosystem built on top of `gpui` and `gpui-component`, centered on `#[derive(GpuiForm)]`.

Its priorities are:

1. **Type safety**: generate strongly typed form state, metadata, and helper APIs at compile time.
1. **Ergonomics**: keep `#[derive(GpuiForm)]` and related attributes concise enough for normal application structs.
1. **Developer experience**: support inventory-driven prototyping, custom components, and layered crates that can be used directly when needed.

For most application code, start with `crates/gpui-form`.

Reach for `crates/gpui-form-prototyping-core` when you want to generate GPUI scaffolding from `GpuiFormShape` inventory data instead of wiring forms by hand.

## Audience labels

These labels describe the crate or surface itself, not the documentation file you are editing:

- **User-facing**: Normal entry points for application developers.
- **Public integration**: Public crates meant for extensions, lower-level runtime access, tooling, or deeper customization. These are not usually the default starting point.
- **Internal**: Workspace plumbing, parse-time and token-generation internals, examples, and maintenance surfaces.

## Documentation rules

### User-facing documentation

These surfaces are always user-facing:

- the root `README.md`,
- `examples/README.md`,
- crate-level `README.md` files under `crates/`.

Even for public-integration or internal crates, a `README.md` should explain:

- who the crate is for,
- what it does,
- what most users should use instead.

### Internal documentation

Only `docs/ARCHITECTURE.md` files are internal documentation.

Use them for:

- macro expansion and parse-time behavior,
- subsystem boundaries,
- data flow between facade, derive, runtime, schema, and prototyping layers,
- design rationale,
- internal relationships.

Do not put parser internals, token-emission detail, or subsystem design notes into READMEs.

## Synchronization rules

When changing a public derive attribute, supported component set, Koruma validation wiring, runtime re-export, custom component contract, prototyping workflow, or other user-visible API shape:

1. Update the root `README.md`.
1. Update `crates/gpui-form/README.md`.
1. Update the affected crate `README.md` files.
1. Update `examples/README.md` and the relevant example crates when showcased behavior changes.
1. Update the matching crate `docs/ARCHITECTURE.md` when boundaries or behavior change.

Keep these surfaces aligned in the same change unless there is a documented reason not to.

Additional rules:

- User-facing documentation should be example-first.
- Prefer Rust snippets over prose-only explanations when behavior changes.
- `examples/README.md` is the canonical index for runnable workspace examples.
- Keep the root `README.md` and `crates/gpui-form/README.md` aligned for install, quick-start, feature, and runtime re-export guidance.
- Keep supported-component docs aligned across the root `README.md` and `crates/gpui-form-derive/README.md`.
- Keep prototyping docs aligned across the root `README.md`, `crates/gpui-form-prototyping-core/README.md`, and `examples/prototyping` when inventory or codegen workflows change.

## Workspace map

### Main user-facing entry points

- `crates/gpui-form`
  Audience: **User-facing**
  Docs: [Architecture](crates/gpui-form/docs/ARCHITECTURE.md)
  Role: workspace facade, default entry point, and home of the public feature flags. Re-exports derive macros plus `core`, `runtime`, and `schema`, and keeps compatibility re-exports such as `custom`, `date_picker`, `infinite_select`, and `numeric`.

### Public integration crates

- `crates/gpui-form-core`
  Audience: **Public integration**
  Docs: [Architecture](crates/gpui-form-core/docs/ARCHITECTURE.md)
  Role: pure, non-GPUI helper logic such as numeric validation. Most application users should start with `gpui-form`.

- `crates/gpui-form-component`
  Audience: **Public integration**
  Docs: [Architecture](crates/gpui-form-component/docs/ARCHITECTURE.md)
  Role: GPUI-facing runtime implementations for infinite select, date picker, and custom component helpers. Most users should go through `gpui_form::runtime` via the facade.

- `crates/gpui-form-schema`
  Audience: **Public integration**
  Docs: [Architecture](crates/gpui-form-schema/docs/ARCHITECTURE.md)
  Role: schema metadata, component behavior metadata, and inventory registry types used by derives and prototyping. Most application users should not need it directly unless they are extending metadata or tooling.

- `crates/gpui-form-derive`
  Audience: **Public integration**
  Docs: [Architecture](crates/gpui-form-derive/docs/ARCHITECTURE.md)
  Role: proc macros for `#[derive(GpuiForm)]`, `SelectItem`, `InfiniteSelect`, and `CustomComponentState`. Most users should depend on `gpui-form` rather than this crate directly.

- `crates/gpui-form-prototyping-core`
  Audience: **Public integration**
  Docs: [Architecture](crates/gpui-form-prototyping-core/docs/ARCHITECTURE.md)
  Role: prototyping and code-generation helpers that consume `GpuiFormShape` inventory data and generate scaffolded form code.

### Internal crates

- `crates/gpui-form-codegen`
  Audience: **Internal**
  Docs: [Architecture](crates/gpui-form-codegen/docs/ARCHITECTURE.md)
  Role: parse-time component parsing, per-component layout emission, and token generation used by `gpui-form-derive`.

- `crates/gpui-form-component-story`
  Audience: **Internal**
  Role: Storybook launcher, story UI, and story-only localization assets for the runtime components in `gpui-form-component`.

### Examples and shared surfaces

- `examples/README.md`
  Canonical index of runnable workspace examples.

- `examples/i18n`
  Shared localization assets used by the example crates.

- `examples/some-lib`
  Shared example domain types and source structs that derive `GpuiForm`.

- `examples/some-lib-custom-components`
  Example external custom component state types and UI widgets.

- `examples/some-lib-forms`
  Storybook-like GPUI example app for browsing generated forms.

  Run with `cargo run -p some-lib-forms`.

- `crates/gpui-form-component-story`
  Storybook-like GPUI example app for browsing the reusable runtime components.

  Run with `cargo run -p gpui-form-component-story`.

- `examples/prototyping`
  Prototype generator that reads `GpuiFormShape` inventory data and emits form scaffolding.

  Run with `cargo run -p prototyping`.

## Working rules by change type

### When editing docs

- Keep READMEs user-facing.
- Move macro expansion details, parser internals, and subsystem design into `docs/ARCHITECTURE.md`.
- Prefer examples over prose-only explanations.
- Sync the root `README.md`, `crates/gpui-form/README.md`, and `examples/README.md` when the primary workflow changes.

### When editing Rust crates

- Use `cargo` for build, test, and run tasks.
- Keep shared package metadata and dependency versions in the workspace root `Cargo.toml`.
- Prefer `workspace = true` for shared dependencies in workspace crates.
- Use local `path` dependencies only where the workspace already relies on them, mainly in the workspace root and example crates.
- Treat `crates/gpui-form` as the public compatibility boundary unless you are intentionally changing lower-level crate APIs too.

### When editing components or generated form metadata

- When adding or changing a component, update `gpui-form-codegen` component parsing and layout, `gpui-form-schema` runtime behavior metadata, and `gpui-form-prototyping-core` `FieldCodeGenerator` mapping together.
- Update user-facing docs for supported components and usage syntax in the same change.
- Keep facade re-exports and lower-level runtime and type surfaces aligned when generated code paths change.

### When editing prototyping or generated outputs

- Prefer changing the source generator or inventory metadata over hand-editing generated output.
- Keep `examples/prototyping` aligned with `gpui-form-prototyping-core` when shape metadata or emitted layout changes.

### When writing tests

- Prefer focused crate-level tests near the changed subsystem.
- For macro or token-generation changes, test emitted behavior at the derive and codegen boundary rather than only the lowest-level helper.

---
> Source: [stayhydated/gpui-form](https://github.com/stayhydated/gpui-form) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
