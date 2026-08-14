## alakazam-studio

> One file, one contract. Every agent tool reads this; nothing else overrides it.

# Working in this repo

One file, one contract. Every agent tool reads this; nothing else overrides it.
Where a rule here conflicts with a pattern you find in the code, the rule wins —
existing code may predate it.

## What this is

A standalone Creator Studio: describe a world, edit it as a state machine, play
it. It is a static browser app with three runtime dependencies. It has no server
of its own, no database and no accounts. Everything it talks to is something the
user points it at, and it defaults to a provider that needs no key and no
network.

The promise the whole repo exists to keep: **clone it, run it, author and play a
world with no account, no key, no `.env`.** Any change that weakens that is
wrong, however good it looks.

## The gate

`npm run check` — typecheck, conventions, unit (`test:unit`), build, e2e.
Identical locally and in CI. Do not open a PR that has not passed it, and never
bypass it: if the gate is broken, fix the gate.

Compiler baseline is strict plus `noUncheckedIndexedAccess`,
`exactOptionalPropertyTypes`, `noImplicitOverride`,
`noPropertyAccessFromIndexSignature` and `verbatimModuleSyntax`. These are not
negotiable and they are not stylistic: every one of them has caught a real
defect in this codebase.

## Zero tolerance

**No unsafe assertions.** `as T` (except `as const`) and non-null `!` are the
two places "the types say this is safe" is an unverified claim. Narrow with a
guard, or fix the source type. Existing violations are grandfathered per-file
with a measured budget in `scripts/check-conventions.mjs`; those numbers may only
go down.

**Validate at every boundary.** Anything from a fetch, a `JSON.parse`, a
WebSocket frame, a CLI arg or `localStorage` is untrusted until it has been
narrowed. A parse failure drops the item; it never throws into the UI.
(Held by review and by the boundary tests — `api-narrow`, the store's op
refusals — not by a checker rule: "is this narrowed?" is a judgment a regex
cannot make. The other rules in this section are machine-enforced.)

**No secret in a `VITE_` variable.** Vite inlines them into the bundle at build
time, so a key placed there is published to every visitor rather than configured
by one operator. Keys are entered in Settings and live in that browser.

**No encoded judgment.** Taste, meaning and editorial decisions come from a
language model — a caller-supplied parameter or a real model call — never from a
keyword list, a threshold ladder, or a curated table standing in for
understanding. Code does structure, I/O and deterministic transforms. The test:
"am I encoding a decision a person would think about?"

## Proof, not claims

"It should work" and "it should render" are the same failure.

A change to behaviour is verified by running it and reading the output. A change
to anything visual is verified by driving the app to the state that matters,
screenshotting it, and **looking at the screenshot**. Every serious defect found
in this repo so far was found by using the thing — filming it, driving a browser,
installing from cold — and none by reading the code.

**Every state that matters is addressable by URL.** `?section=`, `?world=`,
`?play=`, `?settings=1` — see `src/studio/url-state.ts`. A state you can only reach
by clicking is a state nobody can verify twice. If you add a state worth looking
at, give it a parameter in the same change.

Tests assert the user-facing claim, not the implementation. `tests/no-key.spec.ts`
is the central one: it starts from an empty browser with no key and no network.

## A guard is not evidence the thing it guards exists

Before trusting any safety system, grep the **call sites** of the feature it
protects. This repo shipped a capability system — probe, badge, standing warning,
four provider implementations, a documented protocol — built around a method
nothing ever called. Then it happened twice more: `heldCommands` had no key
handler, and the entire bring-your-own-LLM layer was reached only by the Settings
test button.

"Bring your own X" is a claim about call sites, not about configuration surface.
Presets, key storage and a connection test are all perfectly consistent with the
feature never running.

When you find one instance of a defect class, sweep for the class.

## Structure

A concept owns everything about itself: its type, its schema, its component, its
server surface, its tests. Callers invoke; they never reimplement. If code is
about X, it lives with X.

Smells: a parent holding a child's logic; a sibling importing a sibling; a
"helper" in the parent for the child; rendering X inline instead of using X's own
component; the same threshold hardcoded twice. Compound names are a signal to
decompose — two words is probably a subdomain, three means the modelling is
wrong.

Cross-cutting code does **not** get a `lib/` by default. Ask "what noun owns
this?" — there almost always is one, and it deserves a directory with the same
ownership discipline as any feature.

## Documentation

Markdown is for WHY. Never for WHAT or HOW — that is what the code is. Before
writing any doc, ask whether it will be wrong in a week; if so it belongs in
code, or nowhere.

The spec lives next to the implementation, so they change in the same diff. No
inline comments; the surviving prose is the doc comment on a module's public
face, which says what it owns and what belongs elsewhere. Comments explain **why
the non-obvious choice was made**, especially when it encodes a defect somebody
already paid for.

Rules are general, not task notes. State the principle and stop; never bake in a
detail from the task in front of you.

## Delete aggressively

Inside the repo there is no backwards compatibility to keep: delete the old
version and update every call site in one pass. Dead code, stale docs and unused
dependencies are ongoing costs, and git history is the archive.

At the edges — anything a user's stored worlds depend on, or a documented URL, or
the WebSocket protocol third parties implement — the contract is real. Change
those deliberately, with a migration.

## When you change something

State structurally why it belongs where you put it ("X lives in the store because
the store owns persistence"), not syntactically ("I avoided `as`"). Exhaust
existing state before adding new state: a missing check almost always beats a new
flag.

If a rule here keeps getting broken, the fix is a check, not a longer paragraph.
Add it to `scripts/check-conventions.mjs` — the rule registry generates its own
`--help`, so enforcement and documentation cannot drift apart.

---
> Source: [Alakazam-studios/alakazam-studio](https://github.com/Alakazam-studios/alakazam-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
