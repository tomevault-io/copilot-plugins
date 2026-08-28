## murmur

> This file is the rulebook: "when you see X, do Y". It carries no technical

# Every rule here must be followed.

This file is the rulebook: "when you see X, do Y". It carries no technical
detail on purpose — the `docs/` set explains what the app is and why it is
shaped this way, and it is context, not an implementation guide. If you catch
yourself looking for a parameter value, a latency budget, a state diagram or a
schema in this file, you are in the wrong file. Read `docs/00-START-HERE.md`.

## 1. Build from existing patterns, not new ones

You are building this app as a pattern-recognition model: everything you write should come from patterns and coding practices already in this codebase. Never invent your own architecture, pull in your own libraries, or start a new pattern without first confirming that a similar one doesn't already exist.

The app runs on a SOURCE OF TRUTH global architecture, which is already established — your job is to build into it. Globalize what you write so the app stays modular and future features are a plug-in rather than a rewrite; nothing should be tightly coupled, and swapping a tech stack, a business rule, or a strategy should be easy. That applies to configuration, state, and layered code where each layer is an independent task (the same way the IPC commands are built).

But don't create blindly. You have to know whether something that already facilitates your work exists. Only when no existing architecture can support the new feature should you build something new.

## 2. How to search: `grep -l` first

The whole codebase carries a `SOURCE OF TRUTH KEYWORDS` line at the top of each file and code block precisely so you can find things without reading everything. This is what keeps context pollution at zero and stops us from burning through session limits.

Before creating any type, function, constant, or component, grep for it by keyword — and you must grep with `-l`. That gives you the list of *files* where the thing could live; you then narrow to where it's most likely to be and read only those files. It's the fastest path that also protects the context window. If the keyword search turns up nothing, search the codebase normally. If you find it, follow what's already there.

Two commands do this for you. They are read-only navigation, they are part of the architecture, and they are the one authorised exception to §8's no-scripts rule:

```bash
pnpm sot <keyword>      # files whose SOURCE OF TRUTH line mentions <keyword>
pnpm sot:show <keyword> # the same, printing each matching header block
```

If you do have to create something new, give it its own `SOURCE OF TRUTH KEYWORDS` line with 5–10 specific keywords so the next agent can find it and understand how it works.

Never create a duplicate type, function, component, or block of code because grep felt like work. Most types already exist; if you search for them you will most likely find them.

## 3. The layered architecture

Dependencies point **downward only**. An upward import fails the build.

1. **Registry — the single source of truth for what the app *has*.** `src-tauri/src/registry/`. Every capability, setting, permission requirement, nav item, hotkey and metric is one entry. Adding a feature is an entry, not a new pattern. If you are writing a `match` on a feature name anywhere outside `registry/`, the branch belongs in the registry instead.
2. **Command factory — the heart of the app.** `src-tauri/src/ipc/factory.rs`. Every IPC command goes through it. It already does a ton of heavy lifting: input validation, permission preflight, reentrancy guarding, tracing, error mapping and metrics are all handled there, so don't re-check any of it in a handler. Never write `#[tauri::command]` directly.
3. **Commands and pipeline** consume the factory and hold business logic — validation decisions, orchestration, sequencing. Because the factory already covered the cross-cutting concerns, a handler should only contain logic specific to the task at hand.
4. **Ports** are traits for anything swappable. Traits and capability structs only — no logic, ever.
5. **Adapters** are the third-party integrations behind those ports. Only the selected adapter is ever constructed. Never branch on an adapter's name; branch on the capabilities it declares.
6. **Services** are the only layer that touches the database. One verb, one table, no business rules, no calling another service. Import them into commands with `use crate::services::x` so you get module references.
7. Remember to wire up any permission and any metric the feature requires — both are registry entries.

**Never create `middleware.ts`,** and never add a second place where recording state lives. The session state machine owns it.

## 4. Production-grade Rust and TypeScript

Never use `any`, `as any`, `@ts-ignore`, `unwrap()`, `expect()`, or `panic!` outside tests, or any other type or error bypass — this is a production application. Clippy runs at deny level and TypeScript is strict.

For types, work in this order so we save context: if the type crosses the IPC boundary, it is derived by specta from the Rust type and lands in the generated bindings — use that and stop there. Otherwise check `src-tauri/src/types/` to see whether the custom type already exists, and only create one if it doesn't and it will genuinely be a reusable source of truth. Types are never written anywhere except `src-tauri/src/types/`. **Never hand-write an IPC type and never edit a generated file.** Most likely a few types are already created, so leverage those and construct your own from them.

`AppError` is the only error that crosses IPC, so the UI has one error surface rather than forty.

Run a full type check every single time you hand something over — `cargo check`, `cargo clippy`, and `tsc --noEmit` — to prove the codebase is clean. Never report a false positive.

## 5. Validate every input at the boundary

Every command, form, and component that takes input is validated against a declared schema, every single time. On the Rust side the schema is declared to the command factory and the factory enforces it — do not re-check it in the handler. On the frontend it is a Zod schema with React Hook Form following the shadcn approach (https://ui.shadcn.com/docs/forms/react-hook-form#approach). This is what makes data corruption impossible.

## 6. Inline comment context injection

This is the most important part of your development process: it's how other AI devs grep the codebase and understand each block through its SOT keywords. At the top of every file, and above every function or block that is not self-evident, write:

```
/**
 * SOURCE OF TRUTH KEYWORDS: Symbol1, Symbol2, TypeA (5–10 keywords, to power the SOT keyword search)
 * WHAT:  What this block or function is.
 * WHY:   Why it's needed here and why it's done this way.
 * WHERE: Where it's being used.
 */
```

WHAT is one sentence on inputs and outputs. WHY is the gotcha, constraint or non-obvious choice — this is the one that saves the next agent a day, so if the choice was forced by something, say what forced it. WHERE names who calls this and what it calls, so the reader can follow the thread without searching.

Add ordinary inline comments too, but keep them minimal and outcome-based — don't just narrate the code. Trivial helpers don't need the block.

A file without a SOURCE OF TRUTH header is invisible to everyone who comes after you. It is not optional, and it is not something to add later.

## 7. UI

Before creating any component, check whether a reusable one already exists and use it if so. Never duplicate a component with similar functionality — extend or compose what's there. If you notice yourself copying the pattern of another component, that's the signal to globalize it instead.

Place a component that's reusable across routes or features in `components/global/COMPONENT_NAME_FOLDER`; a component used only inside one route goes in `THE_ROUTE/_components`.

Design every component for reuse: never hardcode logic, layouts, or data into it. Global means it can genuinely be reused anywhere, with custom options such as slots. A list component, for example, owns all the reusable logic — search, virtualization, keyboard navigation — while the row, the actions and the empty state come in as slots. Keep global components flexible, configurable, production-ready, and strongly typed so they scale across the app. Fewer lines is better.

Rust owns domain state and pushes it to the frontend as typed events. The frontend never polls it and never keeps a second copy. Frontend stores hold UI state only.

Settings controls are generated from the registry. Adding a setting is a registry entry, not a new component.

For a simple new component, follow the design themes already in the app. For a complex one, use a shadcn UI block as the reference, then rename it to our folder structure and naming conventions and globalize it if needed. shadcn components and blocks often arrive with outdated copy that doesn't match this app's branding — fix it.

Never hardcode a theme value — no hex values, no `text-white`, no `bg-[#...]`, no forced `dark` classes, and no improvised size, radius, duration or spring. Always use the design tokens so everything follows the app theme. Every value comes from `docs/04-DESIGN-SYSTEM.md`; if the value you need is not there, add it there first, then use it.

## 8. Delivery

Keep everything production-grade: never create scripts, seed files, or probe/tester files, or anything else that couldn't be pushed to production. Ask permission before creating any script that performs a manual action. The SOT search commands in §2 are the one standing exception — they are read-only and they are part of the architecture.

Use barrel exports (`mod.rs`, `index.ts`) at every folder boundary, exporting as a single object where that makes sense.

One file, one responsibility. Over ~400 lines means it is doing two things — split it.

Never skip a feature or leave it incomplete. If pieces are missing — because you forgot them or because the user never mentioned them — either finish them or tell the user about those outliers.

Don't use git unless told.

---
> Source: [webprodigies/murmur](https://github.com/webprodigies/murmur) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
