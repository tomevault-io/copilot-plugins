## conference-app-2026

> A comment exists for exactly one reason: to state, in one line, a constraint the code cannot express and whose absence would invite a wrong edit — an API quirk, a required ordering, a dependency that looks unused but is load-bearing. Everything else is noise:

# Project instructions

## Code comments

A comment exists for exactly one reason: to state, in one line, a constraint the code cannot express and whose absence would invite a wrong edit — an API quirk, a required ordering, a dependency that looks unused but is load-bearing. Everything else is noise:

- Never restate what the code shows (what the next line does, a role the name already says, rationale that lives in docs/).
- Never repeat architecture-level knowledge at usage sites. A pattern documented in docs/ or enforced by a FIR checker is explained once, there; copies at every call site rot on the next design change.
- Keep a comment true from its own line. A fact borrowed from elsewhere — a declaration in another module, a version, a count — turns the comment false when that other place changes, with nothing nearby to prompt the edit; give the reason the line exists rather than the contents of what it refers to.
- This applies to NEW files too: do not open a class or file with a comment describing its layer's role, who calls it, or how it fits the architecture — that is docs/ material, and the class name plus its signature already carry the role.
- Write comments in English.

## Code style

- Prefer a callable reference (`foo::bar`) over a lambda whenever the lambda only forwards its arguments unchanged (`{ x -> foo.bar(x) }`, `{ foo.bar() }`). Keep the lambda where a reference cannot apply: `suspend` function types, receiver-typed parameters, `@Composable` lambdas, or any adaptation beyond Unit coercion.

## Wording

- OSS-quality wording everywhere (docs, comments, commit messages): no slang or in-house jargon, spell out acronyms on first mention, never reference symbols that no longer exist in the codebase.
- Documentation register: neutral, declarative, present tense — the tone of official library documentation. Prefer the standard rule vocabulary (`forbid` / `reject` / `require` / `must not`) over informal or policy-flavored words (`ban`, `block`); no exclamation marks, marketing adjectives, or first-person narration. When the user supplies wording, treat it as intent, not final phrasing — propose the conventional term if one exists.

## Sample content

Sample data ships with the repository and is held to the same standard as the rest of it.

- Sample values must not name a real person or organization, and must not read as one. This
  covers names, affiliations, handles, account ids, email addresses, and URLs that resolve to a
  real page or avatar.
- Keep placeholders recognizable as placeholders: `Speaker A`, `Session 1`, a role without an
  employer, `https://example.com/...`.
- Naming a technology is fine — a session title about Jetpack Compose or Kotlin Multiplatform
  describes subject matter. Naming an employer is not.
- The design file follows the same rule.

## Documentation (docs/)

- Every page is one of three kinds: rules (normative, `must` / `must not`), map (structure), or guide (procedural). Keep each page at one altitude — module and overview pages state responsibilities, never class lists; class-level detail lives on the page that owns the topic.
- One canonical home per topic; other pages link with positive phrasing ("For details, see X"), never negative ("this page does not cover…").
- Prefer a short simplified code snippet over dense prose; trim `@OptIn` / metadata noise; at most one "simplified for reading" caveat per page. Android examples take precedence.
- Readable length over exhaustiveness; delete low-value content; a table column whose value never varies is deleted.
- Headings are topic nouns — no status claims ("X works", "What was verified"), no emoji.
- State conclusions as facts; never narrate the investigation that produced them ("confirmed that…", "we measured…", "trying X gave Y") — work-log prose is noise to a new reader and is deleted, keeping only the resulting fact. Placeholders are `<Feature>` / `<Api>`, never `Xxx`. Checklists for agents belong in `.claude/skills`, not docs.
- Diagrams: small; color-code layers with a one-line legend; ELK layout for graphs with back edges; `sequenceDiagram` for temporal flows; repeated-focus (the same small diagram with the current node highlighted) for end-to-end walkthroughs.
- Sidebar label, page h1, and cross-page link texts must agree.

## Verification

- After code changes, verify all targets: `./gradlew :app-desktop:compileKotlinJvm :app-web:compileKotlinWasmJs :app-android:compileDevDebugKotlin :app-ios-kotlin:compileKotlinIosSimulatorArm64 :feature:sessions:jvmTest` (run from the repository root). Swift Export itself only runs under Xcode, which the iOS CI workflow covers.
- After docs changes, keep relative links valid; the VitePress site tooling lives in docs-site/ (run `npm install` there once, then `npm run docs:dev` from docs-site to serve the docs locally).

---
> Source: [DroidKaigi/conference-app-2026](https://github.com/DroidKaigi/conference-app-2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
