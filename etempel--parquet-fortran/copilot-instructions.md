## parquet-fortran

> This file is forward-looking: it captures durable conventions, gotchas, and guardrails to help

# Instructions for Claude

This file is forward-looking: it captures durable conventions, gotchas, and guardrails to help
maintain this library and develop new features going forward. It is not a changelog or session
log — do not add entries describing how or when a specific feature was implemented, what a past
session investigated, or a chronological record of development. Only add generalizable guidance
that will still be correct and actionable for a future task, independent of which session
produced it (git history/commit messages are the right place for "what happened when").

**If checked out inside a larger workspace** (e.g. alongside sibling Fortran projects under a
shared `fortran/` directory), check whether `../../fortran/CLAUDE.md` exists and read it too — it
captures conventions shared across those projects (workflow guardrails, documentation/FORD
conventions, generic Fortran/fpm gotchas, testing & coverage conventions) that apply here as well,
unless this file says otherwise. This repository is also developed and used completely standalone,
so that file won't always exist — treat it as supplementary, not required.

## Contents

This file is a reference, not a start-to-finish read — jump to the note you need. Keep this ToC
in sync when adding, removing, renaming, or reordering a heading (see "Documentation structure"'s
working rules).

- [Workflow & guardrails](#workflow--guardrails)
  - [Only modify files inside this repository](#only-modify-files-inside-this-repository)
  - [Report before implementing on analysis/audit requests](#report-before-implementing-on-analysisaudit-requests)
  - [Only apply low-blast-radius renames/refactors](#only-apply-low-blast-radius-renamesrefactors)
  - [`feature_*.md` planning documents](#feature_md-planning-documents)
  - [NEVER delete a `feature_*.md` document — the maintainer archives them](#never-delete-a-feature_md-document--the-maintainer-archives-them)
  - [The `feature_risks.md` standing-risks register](#the-feature_risksmd-standing-risks-register)
  - [Never splice a file with an unanchored `index()` — use Edit, or assert both ends](#never-splice-a-file-with-an-unanchored-index--use-edit-or-assert-both-ends)
  - [Don't run the GitLab CI pipeline yourself](#dont-run-the-gitlab-ci-pipeline-yourself)
  - [The CI-environment Docker image: ask for it, never build it](#the-ci-environment-docker-image-ask-for-it-never-build-it)
  - [Don't commit or push on the main/default branch yourself](#dont-commit-or-push-on-the-maindefault-branch-yourself)
- [Documentation conventions](#documentation-conventions)
  - [New features require tests and docs](#new-features-require-tests-and-docs)
  - [Documentation structure](#documentation-structure)
  - [CONTRIBUTING.md is project-wide workflow only](#contributingmd-is-project-wide-workflow-only--a-tools-own-detail-goes-in-its-header)
  - [A guide page describes the CURRENT state, never a former one](#a-guide-page-describes-the-current-state-never-a-former-one)
  - [Checking documentation links](#checking-documentation-links)
  - [FORD doc-comment conventions](#ford-doc-comment-conventions)
  - [FORD config gotchas](#ford-config-gotchas)
- [Source code structure & conventions](#source-code-structure--conventions)
  - [One program unit per file; filename == unit name](#one-program-unit-per-file-filename--unit-name)
  - [Some `src/*.f90` files are generated — edit the generator, never the output](#some-srcf90-files-are-generated--edit-the-generator-never-the-output)
  - [Nested submodule tree](#nested-submodule-tree)
  - [Group interface bodies into commented `interface` blocks](#group-interface-bodies-into-commented-interface-blocks)
  - [A module procedure cannot implement its own submodule's spec-declared interface](#a-module-procedure-cannot-implement-its-own-submodules-spec-declared-interface)
  - [A separate module procedure must be IMPLEMENTED before it is CALLED in the same submodule](#a-separate-module-procedure-must-be-implemented-before-it-is-called-in-the-same-submodule)
  - [Naming conventions](#naming-conventions)
  - [`parquet_random` is a LEAF; `parquet_sampling` is where anything more goes](#parquet_random-is-a-leaf-parquet_sampling-is-where-anything-more-goes)
  - [Each `parquet_random` generic reads its OWN word space, and a missed tag is silent](#each-parquet_random-generic-reads-its-own-word-space-and-a-missed-tag-is-silent)
  - [Public numeric arguments: provide both int32 and int64 kinds](#public-numeric-arguments-provide-both-int32-and-int64-kinds)
  - [A new process-global parameter goes in `parquet_settings`](#a-new-process-global-parameter-goes-in-parquet_settings-and-a-design-doc-must-say-so)
  - [Role-A MAMLs live in `table_types/`, not `schemas/`](#role-a-mamls-live-in-table_types-not-schemas)
  - [MAML fixture directory: `schemas/`](#maml-fixture-directory-schemas)
  - [Reading MAML source files: shared helper, line-length limit, CRLF handling](#reading-maml-source-files-shared-helper-line-length-limit-crlf-handling)
  - [Error stop messages: include file/schema context](#error-stop-messages-include-fileschema-context)
  - [Filter evaluation: a Null is UNKNOWN, a NaN is a VALUE](#filter-evaluation-a-null-is-unknown-a-nan-is-a-value--the-two-behave-oppositely)
  - [The row-group statistics screen: every uncertainty must DECLINE](#the-row-group-statistics-screen-every-uncertainty-must-decline)
  - [Guard mutating public procedures against being called twice](#guard-mutating-public-procedures-against-being-called-twice)
  - [Implicit finalizers must never route through a path that can throw/abort](#implicit-finalizers-must-never-route-through-a-path-that-can-throwabort)
  - [A written column's nullability is a CONTRACT with the array beside it](#a-written-columns-nullability-is-a-contract-with-the-array-beside-it)
  - [Automatic BYTE_STREAM_SPLIT for float columns in the writer](#automatic-byte_stream_split-for-float-columns-in-the-writer)
  - [A character ARRAY is trimmed on the way into a column; a character SCALAR is not](#a-character-array-is-trimmed-on-the-way-into-a-column-a-character-scalar-is-not)
  - [Validity is per ELEMENT, and a vector row is not one bit](#validity-is-per-element-and-a-vector-row-is-not-one-bit)
  - [Auto-threading: `omp_in_parallel()` picks a DEFAULT](#auto-threading-omp_in_parallel-picks-a-default-and-that-is-not-the-guard-claudemd-warns-about)
  - [`parquet_table` concurrency: one file owns the OpenMP plumbing](#parquet_table-concurrency-one-file-owns-the-openmp-plumbing-and-guards-key-on-ownership)
  - [A `parquet_table` pointer does not survive a ROW-structural mutation](#a-parquet_table-pointer-does-not-survive-a-row-structural-mutation)
  - [New `parquet_table` state goes on the CACHE](#new-parquet_table-state-goes-on-the-cache--never-as-an-allocatable-component-of-the-type)
  - [Assembling a `parquet_column` from pieces: preallocate and `%paste`](#assembling-a-parquet_column-from-pieces-preallocate-and-paste)
  - [`parquet_column`'s TYPED accessor tier: never reach storage through a binding](#parquet_columns-typed-accessor-tier-never-reach-storage-through-a-binding)
  - [`parquet_string_column`'s typed tier: the same rule, and why concurrency makes it worse](#parquet_string_columns-typed-tier-the-same-rule-and-why-concurrency-makes-it-worse)
  - [A `parquet_schema` built in code must be parsed before anything reads its fields](#a-parquet_schema-built-in-code-must-be-parsed-before-anything-reads-its-fields)
- [Element-domain modules (`parquet_strings`, `parquet_temporal`)](#element-domain-modules-parquet_strings-parquet_temporal)
  - [The `parquet_strings` module](#the-parquet_strings-module)
  - [The `parquet_temporal` module (date/time/timestamp)](#the-parquet_temporal-module-datetimetimestamp)
- [Build & compiler notes](#build--compiler-notes)
  - [The three machines available for testing](#the-three-machines-available-for-testing)
  - [Writing benchmarking instructions for another machine](#writing-benchmarking-instructions-for-another-machine)
  - [Compiler & language gotchas](#compiler--language-gotchas)
    - [General Fortran & language gotchas](#general-fortran--language-gotchas)
    - [gfortran-specific gotchas](#gfortran-specific-gotchas)
    - [ifx-specific gotchas](#ifx-specific-gotchas)
    - [flang-specific gotchas](#flang-specific-gotchas)
    - [nagfor-specific gotchas](#nagfor-specific-gotchas)
    - [C++ side (`src/parquet_wrapper.cpp`) gotchas](#c-side-srcparquet_wrappercpp-gotchas)
  - [NAG's "explicitly imported but not used" warnings: most are FALSE POSITIVES](#nags-explicitly-imported-but-not-used-warnings-most-are-false-positives)
  - [Arrow's own type singletons have thread-unsafe lazy state on first concurrent use](#arrows-own-type-singletons-have-thread-unsafe-lazy-state-on-first-concurrent-use)
  - [gcovr <7.1 cannot parse gcov output for a 10,000+ line file](#gcovr-71-cannot-parse-gcov-output-for-a-10000-line-file)
  - [gcovr 8.4+ drops coverage for module-contained Fortran subroutines](#gcovr-84-drops-coverage-for-module-contained-fortran-subroutines)
  - [`-fPIC` blocks inlining on ELF](#-fpic-blocks-inlining-on-elf-so-the-same-fortran-can-be-twice-as-slow-on-linux-as-on-macos)
  - [Verifying the bind(C) boundary](#verifying-the-bindc-boundary)
  - [If `src/parquet_wrapper.cpp` is ever split into multiple translation units](#if-srcparquet_wrappercpp-is-ever-split-into-multiple-translation-units)
  - [A hand-run `gfortran` without `-J` leaves a `.mod` in the repo root](#a-hand-run-gfortran-without--j-leaves-a-mod-in-the-repo-root-and-a-global-gitignore-hides-it)
  - [Stale `fpm` build cache](#stale-fpm-build-cache)
  - [Keeping `tools/prep_fpm_publish.sh` in sync](#keeping-toolsprep_fpm_publishsh-in-sync)
  - [Manual (never-`fpm test`) large-scale/benchmark tools](#manual-never-fpm-test-large-scalebenchmark-tools)
  - [Measuring whether Arrow memory was actually freed](#measuring-whether-arrow-memory-was-actually-freed-rss-cannot-answer-the-pool-counter-can)
  - [A `shared_ptr` parameter on a per-row helper](#a-shared_ptr-parameter-on-a-per-row-helper-is-the-first-thing-to-suspect-in-parquet_wrappercpp)
  - [Instrument phases before optimising a multi-phase operation](#instrument-phases-before-optimising-a-multi-phase-operation)
- [Testing & coverage](#testing--coverage)
  - [Running a single test suite/test](#running-a-single-test-suitetest)
  - [Error scenarios are pre-run in parallel](#error-scenarios-are-pre-run-in-parallel)
  - [Tests run concurrently: never share a fixture file path between two tests](#tests-run-concurrently-never-share-a-fixture-file-path-between-two-tests)
  - [An intermittent test failure has THREE causes](#an-intermittent-test-failure-has-three-causes-and-the-third-is-not-concurrency-at-all)
  - [Every `check()` call needs its own message](#every-check-call-needs-its-own-message)
  - [Verifying a change with mutation testing](#verifying-a-change-with-mutation-testing)
  - [A test that asserts a REFUSAL must say what to assert when the refusal lifts](#a-test-that-asserts-a-refusal-must-say-what-to-assert-when-the-refusal-lifts)
  - [A test that asserts THREADING must skip without OpenMP](#a-test-that-asserts-threading-must-skip-without-openmp)
  - [A test must not assert a compiler's `ERROR STOP` spelling or exit status](#a-test-must-not-assert-a-compilers-error-stop-spelling-or-exit-status)
  - [A static check that enumerates names goes stale silently](#a-static-check-that-enumerates-names-goes-stale-silently)
  - [A `tools/*.sh` check must run under bash 3.2, and must never exit 0 having stopped early](#a-toolssh-check-must-run-under-bash-32-and-must-never-exit-0-having-stopped-early)
  - [Measuring test coverage](#measuring-test-coverage)
  - [Coverage tooling never drives design](#coverage-tooling-never-drives-design)
  - [Fortran gcov attribution artifacts](#fortran-gcov-attribution-artifacts)
  - [`src/parquet_wrapper.cpp`: GCC vs Clang gcov attribution](#srcparquet_wrappercpp-gcc-vs-clang-gcov-attribution)
  - [Regression tests for "sized/typed from the first element" bugs](#regression-tests-for-sizedtyped-from-the-first-element-bugs)
  - [A Fortran-side debug hook has to be PUBLIC, so prefer a C++ one](#a-fortran-side-debug-hook-has-to-be-public-so-prefer-a-c-one)
  - [Guarding a hard Arrow int32-only ceiling](#guarding-a-hard-arrow-int32-only-ceiling)

## Workflow & guardrails

### Only modify files inside this repository

Never edit, create, or delete files outside this library's own directory tree (e.g. files
under a different project checkout, dotfiles like `~/.zprofile`, or other paths elsewhere on
the machine) — even when doing so would streamline a task (such as setting a computer-specific
environment variable). If something outside this repository genuinely needs to change, tell
the user what's needed and let them make that change themselves.

### Report before implementing on analysis/audit requests

When asked to analyze, audit, or review something (naming conventions, documentation
duplication/coverage, test coverage, etc.), report findings and a proposed plan first and
wait for confirmation before editing any files. Only proceed straight to editing when
explicitly asked to implement/fix/add something directly.

### Only apply low-blast-radius renames/refactors

When renaming or refactoring existing (non-new) code for consistency, only apply the
renames/changes that are low blast-radius (few call sites, no public API/doc impact).
For anything with wider knock-on effects (public API, many call sites, cross-file
conventions), report it as a proposed change and wait for confirmation instead of applying
it directly.

### `feature_*.md` planning documents

`feature_*.md` files in the repo root
are design/planning documents for features not yet implemented — they are git-ignored
(`.gitignore`'s `feature_*.md` entry), so they never reach a commit and exist purely as scratch
design memory between sessions.

**`feature_risks.md` is the one exception and is TRACKED** (`.gitignore` carries an explicit
`!feature_risks.md` negation) — it is a committed document, not scratch memory, so everything below
about writing for a future session applies to it doubly, and it must also read correctly for a
contributor who has never seen a planning document at all. See
[The `feature_risks.md` standing-risks register](#the-feature_risksmd-standing-risks-register) for
its structure and the rules for editing it. A *new* `feature_*.md` file is scratch by default: adding
another negation is a deliberate decision to publish that document, not a formatting choice.

**Every `feature_*.md` design or implementation document must carry a settings analysis** — does
this feature introduce any process-global parameter, does each one pass the admission test, and if
so what are its knob name, default, validation and environment variable? Answer "none" explicitly
when the answer is none; a missing section is indistinguishable from the question never having been
asked. See
[A new process-global parameter goes in `parquet_settings`](#a-new-process-global-parameter-goes-in-parquet_settings-and-a-design-doc-must-say-so)
for the admission test and what follows from it.

**Whenever asked to write or update a `feature_*.md` file, write it to be fully self-explaining
without relying on the current session's conversation for context** — a future session opening
the file has no memory of this one. Concretely: don't reference "this conversation," "as discussed
above" (meaning the chat, not the document), tool-call artifacts (e.g. a clarifying-question
option that was offered but not visibly quoted), or any other detail that only makes sense to
someone who was present for the conversation that produced the file. Quote the user's own
decisions/wording directly in the document rather than alluding to them. Cross-references to
other files in the repo (source, other `feature_*.md` docs, `CLAUDE.md` sections) are fine, since
a future session can read those too.

### NEVER delete a `feature_*.md` document — the maintainer archives them

**Do not delete, move, rename or `git rm` any `feature_*.md` file, and do not consolidate several
into one by removing the originals.** This holds even when asked to "clean up", "archive", "remove
the obsolete planning documents" or "merge these into one" — a request phrased that way is a request
for the *content* work (write the consolidated document, extract what is still open), not for the
deletion. **The maintainer archives these files by hand, to a secure location outside this
repository.** Leave the originals in place and say they are ready to be archived.

**Why this is a hard rule rather than a preference: there is no undo.** `.gitignore` carries
`feature_*.md`, so these files are never in a commit — no `git checkout`, no `git reflog`, no
history. A deleted one exists nowhere except a backup someone happened to take, and a backup in a
session's own scratch directory disappears with the session. That makes deletion the single most
irreversible action available in this repository, more so than anything in the source tree.

**What to do instead**, when a consolidation or cleanup is genuinely wanted:

1. Write the new document, and put in it only what is still open. That is the useful half of the
   work and it is not restricted at all.
2. **Leave every original where it is.** List them, say what was carried across from each, and hand
   the list to the maintainer to archive.
3. If a document's content has been fully superseded, say so *in that document* — a banner at the
   top pointing at its replacement. A superseded document that is still readable costs nothing; a
   deleted one that turns out to have held a measurement nobody re-derived costs a campaign.

**Two facts that make this less costly than it sounds.** These files are already invisible to git,
so leaving them in the tree pollutes no commit and no published package — being untracked, they never
enter the disposable branch `tools/prep_fpm_publish.sh` builds the tarball from, which is also why
that script's `REMOVE_PATHS` names only the *tracked* `feature_risks.md` and none of the others. And
source comments in this repository routinely cite planning documents that
are no longer present (`feature_ifx.md`, `feature_table_parallel.md`, `feature_string_parallel.md`
and others are cited from `tools/` and `src/` while absent from the tree); such a citation is
attribution for where a decision was measured, and it stays meaningful whether or not the file is
still here. So there is no tidiness argument that outweighs the irreversibility.

### The `feature_risks.md` standing-risks register

`feature_risks.md` (repo root, **tracked** — see the previous section) records the properties of the
shipped code that a future change can break **without any test failing and without an abort**: a
wrong answer, a stale pointer, a corrupted heap, a silently skipped row group. It is the companion
to read *before editing an area*, not a to-do list, and it is where the reasoning behind a
non-obvious invariant lives when that reasoning is too long for a code comment and too specific for
this file.

Five rules govern it, and all five are easy to break by treating it as an ordinary document:

- **Only MAJOR risks, and by preference the ones the tests do not fully cover.** *Major* means the
  failure a future change causes is a **wrong answer, lost or corrupted data, a stale pointer, a
  hang, or a broken frozen contract** — something a user of the library would suffer. That is the
  admission test, and most things that feel worth writing down fail it. A **cost** does not qualify,
  however large and however silent: a 7x slowdown ships a correct library, and its home is the
  source comment beside the code, or this file when the lesson is project-wide. Neither does a
  **loud** failure — one that breaks a test, aborts, or fails the lint stage announces itself, which
  is the opposite of what this register is for. Neither does a property of the **test harness**, the
  build or the benchmarking method; those belong here in CLAUDE.md. Where a rejected entry carries
  something genuinely worth keeping (a one-command check, a measurement that justifies a constant),
  **move it to the code rather than dropping it** — `src/parquet_argsort_engine.f90`'s header
  carries the comparator inlining-budget check for exactly that reason.
- **Every risk is `Risk-N`, and the number is permanent.** Numbering runs from `Risk-1` upward
  across the whole file, independent of which section the entry sits in. Moving an entry between
  sections **never** renumbers it, so a reference from this file, from `feature_table.md`, from
  `tools/check_source_conventions.py` or from a code comment stays valid for good. Numbers of
  deleted entries are **not** reused. Never renumber to make a section contiguous.
- **Four sections, and an entry moves between them as its status changes**: *1. New risks* (where a
  newly identified one lands, before anyone has decided whether it is testable — empty is the
  healthy state), *2. Risks with a proposed testing scenario*, *3. Risks not testable*, and
  *4. Risks already covered, kept for what they still forbid*. A new risk takes the next unused
  number and goes in section 1.
- **Section 4 is pruned, not archived.** A covered entry stays only if it still forbids something —
  a rule for the next contributor, a trap not visible in the code, or a test whose *design* has to
  be copied rather than merely kept passing. An entry that has become "this works and is tested" is
  deleted outright; the test is the record at that point, and a register that accumulates solved
  problems stops being read. **Prune whenever you are in the file anyway**, rather than waiting for
  a cleanup round: the register is only read if it is short enough to read, and every entry that
  does not earn its place is charged to everyone who opens it.
- **A verdict is checked against the suite, never inferred.** Before marking anything "proposed",
  grep the tests for what actually asserts it — several entries once marked proposed turned out to
  be covered already. Before marking one "covered", name the test.

**When implementing a proposed test from it, update the entry in the same change**: move it to
section 4 (or delete it, per the pruning rule), name the test that now covers it, and say what the
test's shape is protecting if that is the interesting part. An entry that still reads "proposed"
after the test exists is worse than no entry, because the next reader will write the test again.

**When a new silent-failure property is discovered** — typically while fixing a bug whose symptom
appeared far from its cause — add it as a new `Risk-N` in section 1 rather than only writing a code
comment. This file (CLAUDE.md) is for rules that apply project-wide; `feature_risks.md` is for a
specific property of a specific area, with its test status attached.

### Never splice a file with an unanchored `index()` — use Edit, or assert both ends

A script that locates a replacement range by searching for two markers —

```python
start = s.index("<first marker>")
end   = s.index("**Proposal.**")        # finds the FIRST one in the file, not this section's
s = s[:start] + new + s[end:]
```

— silently **duplicates** everything between the wrong `end` and `start` whenever `end < start`. It
does not raise, the file still parses, and the damage reads later as an unrelated merge artifact.
Confirmed instance: a splice in `feature_optimise.md` duplicated **132 lines** — a section's tail, a
whole section, and the head of a third — and went unnoticed across sessions, after which the stale
duplicate was read as authoritative. (A second planning document was damaged around the same time by
a genuine unresolved three-way merge, which is a *different* mechanism with the same outcome: silent
structural damage to a file nothing validates. Both were found only by cross-checking headings.)

**Rules, in order of preference:**

- **Prefer the `Edit` tool over a hand-rolled splice.** It fails on a non-unique `old_string`, which
  is exactly the check the shape above omits.
- When a script really must replace a range, **assert both ends**: that each marker occurs the
  expected number of times, that `end > start`, and that the text being discarded is what you think
  it is. Print the discarded byte count.
- **After any scripted edit to a structured document, re-derive its structure and compare** —
  `grep "^#" file | sort | uniq -d` catches a duplicated heading instantly, and a ToC-versus-headings
  cross-check catches a deleted one. Both take one command and both would have caught these.

**A script that validates SEVERAL replacements and writes ONCE at the end is the same hazard wearing
different clothes: a late failure silently discards the earlier edits.** The natural shape —
`assert s.count(old_i) == 1` per edit, then a single `write()` at the bottom — leaves the first two
replacements applied *in memory only* when the third assertion fails. The file is untouched, the
traceback scrolls away, and every check still passes, because a document missing two paragraphs is
still a valid document. Confirmed on `doc/pages/types/supported-data-types.md`, where two paragraphs
went unwritten and were found only by rendering the page afterwards. Either write after each
replacement, or verify afterwards — and for a `doc/pages/` page, **verifying means rendering it**:
`ford docs.md`, then grep `ford-doc/page/<group>/<name>.html` for one distinctive phrase per edit,
with whitespace collapsed on both sides (a source line break survives into the rendered `<p>`, so a
search string spanning one reports a correct paragraph as missing). That is the only check that sees
a lost edit; `fpm test`, `check_doc_anchors.py` and `check_source_conventions.py` cannot.

**A `feature_*.md` file has NO recovery path** — it is git-ignored, so there is no `git checkout` and
no history. The only copy of a damaged section is whatever a session transcript happens to hold
(`~/.claude/projects/<project>/*.jsonl`, searchable for the text), which is how both incidents were
repaired. Take a copy before a scripted rewrite of one, and be aware that a "clean up this document"
request is a one-way door without it.

### Don't run the GitLab CI pipeline yourself

The user runs `.gitlab-ci.yml` on their own GitLab server — don't attempt to execute it
(e.g. via `gitlab-runner`, docker, or otherwise) as part of verifying changes. Verify
locally instead (`fpm build`/`fpm test` with the same `FPM_FFLAGS`/`FPM_CXXFLAGS`/
`FPM_LDFLAGS` the CI job sets, minus anything CI-environment-specific like the apt installs).

**In practice that means running plain `fpm test` and setting nothing at all.** Two separate
reasons, and both are easy to get wrong in the direction of "add the flag to be safe":

- **`-fopenmp` is not needed and never was.** `fpm.toml`'s `openmp = "*"` metapackage injects it
  into the compile *and* link flags, for this package and its dependencies alike. Confirm with
  `fpm build --show-model | grep -o 'fortran_compile_flags="[^"]*"'`, which shows `-fopenmp` even
  when `FPM_FFLAGS` carries nothing but `-I` paths; confirm end-to-end by running
  `error_scenarios concurrent_calls_into_shared_reader` from a build with no `-fopenmp` anywhere,
  which still aborts (exit 134) because several threads really do enter the reader at once.
  **This holds for gfortran and ifx, and NOT for flang** — on fpm 0.13.0 alpha the metapackage
  contributes no `-fopenmp` at all under `FPM_FC=flang-mp-22`, which the same `--show-model` command
  shows directly (the flags line carries only `-cpp` and the `-I` paths, where gfortran's carries
  `-fopenmp`). So the "setting nothing at all" advice above is compiler-specific; on flang the build
  fails at `use omp_lib` rather than quietly running serially. See the flang note under
  [The three machines available for testing](#the-three-machines-available-for-testing).
- **`FPM_FFLAGS` clobbers what the environment exported, but it does NOT suppress fpm's profile
  flags — the missing `-O`/`-fcheck=bounds` comes from omitting `--profile`.** Setting it on a
  command line replaces whatever the environment already exports (a dev machine typically puts
  Arrow-adjacent `-I` paths there — clobbering those is the Fortran-side twin of the
  `fatal error: 'arrow/api.h' file not found` failure below), and that half is worth avoiding. But
  the profile half is a **separate mechanism**: fpm applies profile flags only when `--profile` is
  given, and appends `FPM_FFLAGS` to them rather than instead of them.

  Verified on fpm **0.13.0 alpha** by reading the compile line
  `fpm build --verbose` actually emits, in a throwaway two-file project, all four ways
  (independently reproduced on two machines):

  | invocation | Fortran compile line gets |
  |---|---|
  | `--profile release`, `FPM_FFLAGS` **set** | `-I<env> -O3 -Wimplicit-interface …` — **additive** |
  | `--profile debug`, `FPM_FFLAGS` **set** | `-I<env> -Wall -Wextra -g -fcheck=bounds …` — **additive** |
  | no `--profile`, `FPM_FFLAGS` set | `-I<env>` and nothing else |
  | no `--profile`, `FPM_FFLAGS` unset | **nothing else either** |

  The last row is what identifies the real cause: with no `--profile` there is no `-O` *whether or
  not* `FPM_FFLAGS` is set. **So a plain `fpm test` has no bounds checking on ANY machine, not only
  on one that exports `FPM_FFLAGS`, and `fpm test --profile debug` is what turns it on — including
  on a machine that exports `FPM_FFLAGS`, which does not lose the debug flags.**

  Re-check the table above against a newer fpm before assuming it still holds. Note the corollary
  that is easy to get backwards: a `--profile release` measurement taken on a machine that exports
  `FPM_FFLAGS` is **not** an `-O0` measurement and must not be discarded as one.

Setting `FPM_CXXFLAGS`/`FPM_LDFLAGS` to CI's values has the same replacing behaviour on the C++
half, which on a dev machine is where Arrow's include and library paths come from — the build then
fails with `fatal error: 'arrow/api.h' file not found`, which looks like a missing dependency
rather than a flag problem. CI can set them because its own image puts Arrow on the default search
path.

So: **`fpm test`** for the ordinary check, **`fpm test --profile debug`** when you want the
bounds/`-Wall` checks (worth doing at least once for anything touching allocation or array
shapes — see the `clone_new_cache` note under "Compiler & language gotchas" for a bug only that
build could see), and `tools/coverage.sh` rather than a hand-rolled `--coverage` when measuring
coverage.

### The CI-environment Docker image: ask for it, never build it

A local run can only ever prove a change works on *this* machine's compiler. Some real failures
are specific to the CI toolchain and are invisible locally — the metadata-array copy in
`clone_new_cache` (see "Compiler & language gotchas") segfaulted on CI's gfortran while running
clean locally under `-fcheck=all`, `--coverage -fopenmp` and the full suite. When a CI failure
cannot be reproduced locally, or when a change needs checking against a compiler that isn't
installed here, **ask the maintainer for the CI-environment Docker image**: it reproduces the
GitLab CI environment and carries **gfortran, ifx and flang**, so all three can be exercised
against the current working tree.

Rules for using it:

- **Never build or rebuild the image yourself** — not via `tools/build_ci_test_image.sh`, not via
  `docker build`/`docker run` against a base image, not by any other route. Building it is the
  maintainer's job, and only the maintainer's.
- **Only ever run the library against an image the maintainer has already made available.** That
  means building and testing this repository inside it, nothing else.
- **Any change the image itself needs** (a different compiler version, another package, a changed
  `before_script` step) is a **request** to the maintainer, who will generate a new image. Do not
  work around a missing tool by installing it into a running container either — say what is
  needed and wait.
- Asking for the image is not a substitute for the local verification described above; it is what
  to reach for *after* a local run has come back green and the failure persists on CI.

### Don't commit or push on the main/default branch yourself

The user always commits and pushes their own changes on `main` — even after explicitly asking
for a feature/fix to be implemented, do not run `git commit`/`git push` on `main` yourself
unless they separately, explicitly ask for that specific commit. Leave finished work
uncommitted in the working tree for them to review and commit. (This is specific to the
main/default branch; it doesn't apply to work you've been asked to do inside your own
throwaway branch/worktree, if any.)

## Documentation conventions

### New features require tests and docs

Whenever asked to implement a new feature in this repository, always:

1. Add unit test coverage for it (in the relevant `test/*.f90` suite; add abort/error-path
   coverage via `test/error_scenarios.f90` + `test/test_errors.f90` +
   `tools/run_error_scenarios.sh` if the feature has failure modes that `error stop`).
2. Update documentation — see [Documentation structure](#documentation-structure) for what
   goes where. In brief: every new public procedure/type gets its own `!>`(leading)/`!!`(trailing) doc-comment
   (picked up automatically by the FORD-generated API reference — no hand-maintained table to
   update); user-facing behavior/how-to goes in the relevant `doc/pages/*.md` guide page; touch
   README.md only if the landing-page story changes (a new feature bullet, a new limitation, a
   setup change); update CONTRIBUTING.md if it affects contributor
   workflow.

Do this without being asked separately each time — it applies by default to any
"implement/add feature" request in this repo, not just when explicitly reminded.

**CHANGELOG is active as of the 1.0.0 release.** `CHANGELOG.md` has a published `[1.0.0]`
section — every user-facing change from here on (new feature, behavior change, bug fix affecting
documented behavior) gets an `[Unreleased]` entry added at the top of the file, following the
existing [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format already used there. Do
this without being asked separately, the same way tests/docs are added by default for a new
feature (see "New features require tests and docs" above).

**`[Unreleased]` is written for someone upgrading from the last release, not as a development
log.** Everything in it is read against `[1.0.0]` (or whatever the newest published section is),
so an entry only earns its place if it describes a difference a reader of that release would
actually see. Six rules follow, and they apply to every future entry — not just when someone
asks for a cleanup:

- **State WHAT changed, never WHY — and keep it to a line or two.** An entry is an inventory item:
  the feature added, the behaviour changed, the bug fixed. Nothing else. No rationale, no
  measurement, no design discussion, no account of what it replaced, what the alternative was, or
  what it cost to build. That reasoning is worth writing down and it has a home — the **code comment
  beside the code** it explains, a `doc/pages/` page when a *user* has to act on it, or this file
  when it is a project-wide rule — and the changelog is none of them. A reader opens it to decide
  whether to upgrade and what will differ when they do; every explanatory sentence buries that under
  something they did not ask for. **The test: would this sentence still be here if the change had
  been obvious?** If yes, it is justification, and it goes. A bullet that has grown past two or three
  lines has almost always failed this test rather than genuinely needing the room.
- **`### Changed` and `### Fixed` are for functionality that is in the RELEASED version.** A fix
  or behavior change to something that is itself still sitting in `[Unreleased]` is invisible to
  every user — there is no released behavior for it to differ from — so it does not get its own
  entry. Fold whatever the reader needs to know into that feature's own `### Added` bullet
  instead, and drop the rest. This is the rule that keeps the section from filling up with the
  history of how an unreleased feature was built.

  Stated the other way round, because that is the form it is usually needed in: **a feature
  implemented after 1.0.0 never earns a `### Changed`/`### Fixed` entry for its own subsequent
  changes and fixes**, however many rounds it goes through before it ships. It has exactly one
  bullet — under `### Added` — and that bullet is kept current instead. `### Changed`/`### Fixed`
  are reserved for the behaviour a reader of 1.0.0 (or of whatever the newest published section
  becomes) can actually observe changing under them.
- **`### Added` records overall features, not each part of one.** A sub-feature, a new
  type-bound procedure, or a helper that exists to serve a feature already listed belongs *in
  that feature's bullet*, not as a bullet of its own. Several closely related additions that
  share a purpose (say, a set of new metadata-only reader queries) go in one bullet together.
- **Leave out what is not user-facing at all**: test coverage, internal refactors and file
  reorganizations explicitly marked "no functional change", tooling that changes nothing a
  consumer of the library can observe. A genuinely new contributor-facing tool wired into CI can
  have one short line; its subsequent tweaks cannot.
- **Every minor change in a release shares ONE bullet — one for the whole section, not one per
  subsection — placed last, under `### Fixed`: "Many other minor fixes and improvements."** A small
  fix, a tightened message, a widened argument kind, a new overload of something already listed, a
  contributor-facing tool, a documentation correction — none of these earns a line of its own, and a
  reader upgrading does not scan for them. That bullet is one line and **is never expanded into a
  sub-list**: the moment it starts enumerating, it has become the development log this section
  exists not to be. If a change genuinely needs the reader to know about it by name, it is not minor
  and belongs in a bullet of its own; that judgement is the only thing this rule asks for.
- **One `### Added`/`### Changed`/`### Fixed` per release section, in that order.** Appending a
  second `### Added` after `### Fixed` is easy to do by accident when adding an entry to a long
  section, and it silently splits the list a reader is trying to read as one.

Re-read the whole `[Unreleased]` section when adding to it, rather than appending to the end: an
entry frequently belongs inside a bullet that is already there, and a fix being added may
supersede text further up.

### Documentation structure

User- and contributor-facing docs are split across three layers — keep new content in the right one:

- **README.md** — the lean *landing page*: what the library is, features, one quick example,
  install / prerequisites / environment variables, "important behavior", a compact **API
  overview** index (plain-text procedure/type names, no per-procedure detail), limitations, and
  license/contributing pointers. Keep it short — do **not** let it grow back into a manual;
  deep-dive/how-to content goes in `doc/pages/`, and the per-procedure reference is generated
  by FORD, not hand-written here.
- **`doc/pages/<group>/*.md`** — the full *user guide*, organised **two layers deep**: six group
  directories (`io/`, `types/`, `schema/`, `tables/`, `utilities/`, `operating/`), each holding
  one file per topic plus its own `index.md` (an orientation paragraph and a described bullet
  per page), rendered as FORD nested narrative pages. `doc/pages/index.md` is the guide's
  landing page — orientation prose, one described bullet per group, and a flat
  every-page-at-a-glance list. Every `index.md` carries one `ordered_subpage:` frontmatter entry
  per child (a page filename, or a bare group-directory name), mirrored by its body bullets;
  `check_doc_page_index_consistency` (`tools/check_source_conventions.py`, run in CI's lint
  stage) enforces all of those pairings, including that every group directory has an `index.md`
  — FORD **silently skips** a group without one (exit 0, whole group absent from the site).
  Cross-group links between pages use `../<group>/<name>.html#anchor`; `../../index.html` is the
  README front page while `../index.html` is the guide's own landing page — one level apart,
  entirely different pages. Never a hand-maintained per-procedure API table here either — link
  to the generated reference instead.
- **FORD-generated API reference** — every public procedure/type/module gets its own `!>`
  (leading) doc-comment plus a trailing `!!` tag on every dummy argument/function result (see
  the "FORD doc-comment conventions" section below); FORD turns these into the browsable
  modules/procedures/types reference automatically. This is the *only* place the per-procedure
  reference lives — there is no hand-written equivalent to keep in sync.
- **CONTRIBUTING.md** — *contributor-facing*: building/testing this repo, project conventions,
  and repo-maintenance tooling (see its own Contents ToC for the full topic list, kept current
  there rather than duplicated here).

Working rules:

- A new public procedure gets a `!>`(leading)/`!!`(trailing) doc-comment (see "FORD doc-comment
  conventions" below), and that is the whole obligation. The FORD-generated reference is the only
  complete enumeration of the public surface, and it is derived from source rather than maintained
  by hand. **README.md carries no per-procedure index** — it lists features and points at the guide.
  Do not reintroduce one: the hand-maintained "API overview" it used to carry had drifted to 14
  missing names before a lint check was written for it, and that check was retired together with
  the section.
- Every section heading must appear in that file's own **Contents** ToC (README.md/CONTRIBUTING.md/CLAUDE.md);
  `doc/pages/*.md` pages don't need one — FORD generates in-page navigation from headings itself.
- Moving content between README.md and a `doc/pages/*.md` page turns in-page `#anchor` links
  into cross-file `doc/pages/<page>.md#…` / `README.md#…` links — repoint them, and fix
  now-stale relative wording ("above", "below", "this README"). Re-run `tools/check_doc_anchors.py`
  afterward (see "Checking documentation links" below).
- **A performance figure on a guide page is APPROXIMATE and MACHINE-FREE.** A reader cannot
  reproduce a number quoted to three significant figures against a compiler and a machine they do
  not have, so a page carrying one is stating something it cannot support. **No compiler names, no
  machine descriptions, and no decimals on a ratio** — `"4.17x (gfortran) and 3.93x (ifx)"` becomes
  `"several times"`. Name the tool that measures it instead (`bench/benchmark_table.sh`,
  `bench/benchmark_random.sh`, `bench/benchmark_colindex.sh`, …) so a reader can get their own
  number. **Two things are NOT covered**, and stripping them makes the page worse: a **contract
  number** — a threshold, an acceptance rate, a documented bound, a growth factor, or a share with
  real cross-machine provenance — stays exact; and a **parity claim** ("peak memory is the same at
  every thread count") is the point of the sentence rather than a speed measurement. The test is
  whether the number is *evidence for a design choice the reader has to make*, or *a snapshot of one
  machine's throughput*. This applies to performance figures only: a compiler named in a
  **correctness** or **build** context (a miscompilation to avoid, a flag to pass, a reproducibility
  guarantee across compilers) is information the reader needs and stays.
- **A page has ONE name, and the two index lists quote it under one rule.** The frontmatter
  `title:` is the page's canonical name — FORD uses it for the browser tab, the `<h1>` and its own
  self-links — so **`doc/pages/index.md`'s flat every-page list carries the whole title** (that list
  has no descriptions; the link text is all a reader gets), while **a group `index.md`'s bullet
  carries the title or an initial prefix of it** (a one-line description follows after the dash, so
  repeating the title's tail is redundant). Backticks are ignored on both sides — no frontmatter
  `title:` in the guide carries one. Enforced by `check_page_titles_match_their_list_entries`
  (`tools/check_source_conventions.py`). **The corollary is the useful half: a page whose flat-list
  entry says more than its own `<h1>` is under-named**, and the fix is to lengthen the title rather
  than shorten the entry — six pages differed this way before the rule was settled, including one
  titled "Random numbers" for a page half about sampling.
- **A count written out beside the list it counts, or a module list the code also owns, needs a
  check.** `doc/pages/index.md` carries three such claims — the page count, the entry-module
  enumeration, and the page titles above — and each had drifted from what it described while every
  test stayed green. All three are now compared against the thing they describe by
  `tools/check_source_conventions.py`; a fourth of this shape should get the same treatment rather
  than a careful review.
- **Optional arguments are shown in square brackets** when a signature is written out in prose or
  in a table — `call t%get_file_metadata(key, value, [found])`, `%ncols([resident_only])`. This
  applies to *descriptions* of a call, never to a runnable code example inside a ```fortran fence,
  where brackets would not compile. Adopted after the fact rather than in one sweep: apply it to
  any signature you write or edit, and retrofit a whole page the next time that page is touched
  for another reason (`doc/pages/tables/table.md` is retrofitted; the others are not yet). The first
  bracketed signature on a page should carry a one-line note saying what the brackets mean.
- **A reference bullet carrying more than about three distinct claims becomes its own `###`
  subsection.** Most `doc/pages/` reference pages are shaped as a bullet list with one bullet per
  procedure, and a bullet in that shape grows without anything pushing back: `schema/building-schema-in-code.md`'s
  `%add_field` bullet reached **1779 characters carrying nine claims** before anyone noticed, with
  `%init` at 1349 and `%get_field` at 1334 beside it. Wrapping such a bullet at the page's margin
  makes it *diffable* and leaves it just as unreadable; splitting it is the actual fix. Keep the
  procedure's call form as the subsection's opening line, in the same bold-and-brackets style the
  bullet used, then one paragraph per claim. Two mechanics decide whether it renders: a bullet's
  continuation *paragraph* needs **4-space** indentation (2 spaces silently ends the list — see
  below), which is exactly why promoting to a heading usually beats indenting; and the promotion
  changes the page's heading set, so re-run `tools/check_doc_anchors.py` afterwards, which resolves
  inbound `#anchor` links from README.md, CONTRIBUTING.md, CHANGELOG.md, this file and every
  root-level `*.md` as well as from `doc/pages/`. Leave the short bullets alone — this is a rule
  about essays that have grown inside a list, not a ban on lists.
- **A fenced code block is NEVER indented, not even to sit under the bullet it belongs to.**
  python-markdown — the engine FORD drives — does not recognise a ``` fence carrying any leading
  whitespace: the fence is emitted literally inside a `<p>`, the enclosing list is closed before it,
  and because the block has become prose a blank line inside the example splits it in two. The
  published page shows the example as running text with a stray ``` in it, while the source looks
  right, FORD exits 0 and no test reads the generated HTML. Measured against python-markdown 3.3.4
  and 3.4.4 with the extensions `fpm.toml`'s `[extra.ford]` configures: **2 spaces broken, 4 spaces
  broken** (that one becomes an indented code block *containing* the ``` line), **column 0 correct**.
  So put the fence and its content at column 0 — the list closes before it and reopens after, which
  is only cosmetic — or restructure the bullet into a subheading; and dedent any bullet-continuation
  prose that follows the fence, or it is orphaned at an indent belonging to a list that has closed.
  Enforced by `check_no_indented_code_fence` (`tools/check_source_conventions.py`, run by both
  `tools/run_lint_check.sh` and CI's `lint` stage); six blocks across three pages had been published
  this way before it existed.
- **A list item's continuation PARAGRAPH needs 4-space indentation; 2 spaces silently ends the
  list.** Same engine, same silence, different construct from the fence rule above — and 2 spaces is
  what a `- ` bullet's own wrapped continuation *lines* use, so splitting a long bullet into
  paragraphs at the indent already in front of you is the natural mistake. python-markdown closes the
  `<ul>` at the blank line, emits the rest of that bullet as top-level `<p>`, and renders every
  following bullet as **literal `- ` in running prose**. Confirmed on `doc/pages/io/writing.md`'s
  `chunk_size` bullet: `ford docs.md` exited 0, `check_no_indented_code_fence` passed, the source
  looked right, and only reading the generated HTML found it. So indent a continuation paragraph 4
  spaces, or leave the bullet as one paragraph — and when a page gains a multi-paragraph list item,
  render it once and check the following bullet is still an `<li>`.

  **4 spaces is NECESSARY BUT NOT SUFFICIENT: the paragraph also needs a BLANK LINE after it,
  before the next bullet.** Get the indent right and omit that blank line and the list still
  breaks — python-markdown takes the following `- ` lines as *lazy continuation* of the indented
  paragraph and swallows them into it, so they render as literal `- ` in running prose exactly as
  the 2-space mistake does. Measured on `doc/pages/operating/troubleshooting.md`: one correct
  4-space paragraph, no blank line before the next bullet, and the rendered page had **2 `<li>`
  where the source had 8** — six bullets absorbed. Every check was green (`check_doc_anchors.py`,
  all 38 source-convention checks including `check_no_indented_code_fence`), and only the render
  showed it. **The cheap probe is to count, not to read**: compare `<li>` in the generated HTML
  against `^- ` in the source, since a broken list still looks perfectly ordinary in both the
  source and the rendered page. Note the safe move is usually neither indent — a bullet needing a
  second paragraph is a bullet that wants promoting to its own `###` heading.
- **Code-fence tags: `fortran`, `bash`, `yaml` or bare — never `maml`.** `fortran` for library
  code, `bash` for a shell command, a bare fence for program output and for a plain-text diagram.
  A MAML block takes **either** a bare fence or ```` ```yaml ```` — both are established practice
  (`schema/maml-format.md`, the MAML reference page, uses each), so neither is to be "corrected"
  into the other. ```` ```maml ```` appears nowhere and must not be introduced: Pygments has no
  `maml` lexer, so it changes no rendering while making whichever page adopts it the only outlier.
- **Diagrams: plain text, not Mermaid.** This project's GitLab does not reliably render Mermaid
  diagrams, so draw flows as plain-text/ASCII inside a normal code fence (renders identically
  everywhere) — see the MAML→header flow in `doc/pages/schema/maml-format.md`'s "The MAML metadata format".
- **Badges:** README.md carries three dynamic `gitlab.4most.eu` badges (CI pipeline, test
  coverage, API documentation) alongside the static license/language/fpm ones. These are
  GitLab-specific — `tools/prep_github_mirroring.sh` swaps them for a single GitHub Pages
  documentation badge when mirroring (see CONTRIBUTING.md's "Mirroring to GitHub").

### CONTRIBUTING.md is project-wide workflow ONLY — a tool's own detail goes in its header

`CONTRIBUTING.md` answers one question: *how do I work on this repository?* The conventions everyone
follows, the build/test/lint/CI/release/publish workflow, and decisions with repository-wide
consequences. It is read start to finish by someone who has just arrived, so **its length is a cost
paid by every reader**, and every paragraph added to it is charged to all of them.

**A per-tool, per-script or per-file detail is not project-wide and does not belong there.** How one
script is invoked, what its environment variables do, what its modes are, what its output columns
mean, why one arm of one benchmark exists, what a figure measured on one machine — all of that goes
in **that file's own header comment** (`tools/*.sh`, `tools/*.py`, `bench/*`, `src/*.f90`), next to
the code it describes and in front of whoever is about to change it. This is already the convention
rather than a new one: every program under `bench/` and all but one script under `tools/` already
carry such a header, so moving a paragraph there is usually merging rather than writing.

**The test is: would the person who invalidates this sentence be looking at this file when they do
it?** If someone adds a mode to a benchmark, they are editing the benchmark — so a description of its
modes kept anywhere else is a second copy waiting to go stale. That is not hypothetical. The "Other
tools/ helpers" section had reached **909 lines, 64% of the whole file**, and by the time it was
reviewed it disagreed with `bench/benchmark_table.sh`'s own header about which output line to read
(the header said RSS, `CONTRIBUTING.md` said the Arrow pool counter — and only the second is right)
and with `bench/benchmark_sort_engine.sh`'s about which sort engine ships. Both were found by moving
the prose next to the code, not by review.

Rules:

- **A tool gets ONE row in the index table** — its name and a one-line purpose — and nothing more.
  Never a paragraph, never a worked example, never a table of environment variables. A new tool
  therefore needs no `CONTRIBUTING.md` edit beyond that row, and a tool whose behaviour changes needs
  none at all. `check_contributing_is_an_index` (`tools/check_source_conventions.py`) enforces this:
  it fails if a `tools/` or `app/` path is mentioned outside the index table and a small allow-list
  of genuine workflow references.
- **Never write out a count or a list the repository owns** (see
  [Documentation structure](#documentation-structure)). Point at the source instead: "the
  `new_testsuite(...)` array in `test/run_tester.f90`", not a copy of it. Six such facts had gone
  stale at once before this rule existed, including a suite list that named 10 of 27, an executable
  count that read 24 for 31, and "sixteen structural invariants" for 44.
- **Before adding a paragraph, decide which of four files it belongs in**: this file (a rule for
  future work anywhere), the tool's own header (how this tool works), `feature_*.md` (a campaign's
  measurements), or `doc/pages/` (anything a *user* of the library needs). `CONTRIBUTING.md` is the
  residue — what a contributor needs before they know which area they are working in.
- **If you are explaining what a benchmark's output means, you are writing the script's header.**
  That is the single most reliable signal that a paragraph is in the wrong file.

### A guide page describes the CURRENT state, never a former one

**A sentence in `doc/pages/`, README.md or a `!>` doc-comment is a defect when a reader cannot
evaluate it without knowing a state of the code they have never seen** — however true it is. The
reader is using the library as it ships today; the history is in git, in `CHANGELOG.md` and in this
file, all of which have an audience that wants it. A user-facing page has no such audience.

Three shapes, all of which have shipped here and all of which read as perfectly ordinary prose:

- **A justification that appeals to a former API.** `types/string-columns.md` explained why a null
  reads back as `""` with *"purely by choice (to keep this behavior unchanged from before `get`
  became a subroutine)"*. The behaviour is right and the reason is unusable: `get` has only ever been
  a subroutine as far as any reader can tell, so the justification reduces to "because it used to be
  something else". Give the reason that still applies — here, that an unallocated result would be
  indistinguishable from one the callee never reached.
- **"now" or "have always" attached to behaviour.** *"…what they have always done, and what they now
  use internally"* asserts a change (implying they once did not) and a claim about history, where the
  page needed only the fact: *"…which reach these same two bulk forms internally"*. **"now" is the
  single highest-yield word to grep for**, and most of its occurrences are innocent — "the column is
  now empty", "`col%is_null(i)` is now `.true.`" describe a state after an operation, not a release.
  Read every hit; do not replace them mechanically.
- **Provenance — how or where something was discovered.** *"a library-wide compiler caveat … found
  and root-caused via this same type's accessors"* tells a reader nothing they can use. That belongs
  in this file, which is exactly where it already was.

**What is NOT in this class**, since over-applying the rule costs real information: a performance
comparison between two *current* APIs ("about six times faster than a loop of `%append_string`"); a
"before/after" that means *within one call* ("result element k is the old element `perm(k)`", "the
column is now empty"); a documented deprecation a user can still encounter; and `CHANGELOG.md`, whose
whole job is to describe change. Code comments in `src/` are also exempt — their reader is a
maintainer, for whom "this was consolidated into one place" is useful context.

**This class is MANUFACTURED by ordinary correct work, so a page being finished is no evidence
against it.** Every instance found in the 2026-08-13 sweep was written by a careful edit: someone
reviews a page, the review produces a code or documentation change, the change updates the page —
and the updating sentence explains *what changed*, because at that moment the change is the salient
thing. Six weeks later it is the only thing on the page a reader cannot check. So this is not a
one-off cleanup that can be declared done: **re-check it whenever you touch a page for any other
reason**, and expect the pages most actively maintained to carry the most of it.

**Grep for it, but never sweep it mechanically — most hits are innocent and the ratio is brutal.**
Across the eight pages swept, fourteen hits, **three real**. The eleven survivors were the same words
meaning something else entirely: "no longer belongs to any one row group" and "the column is now
empty" (state after an operation), "most recently used to read this column" and "a value used to
*detect* a Null" ("employed to", not "formerly"), "originally Null" and "the old element `perm(k)`"
(within one call), "could not even resolve minutes" (hypothetical). A find-and-replace pass would
have damaged every one of them. Three greps find the candidates; a human decides:

```bash
grep -nE "used to|previously|formerly|no longer|in the past|historically|originally" doc/pages/<page>
grep -nE "ha(ve|s) always|as before|unchanged from|for (backward )?compatibility|root-caused" doc/pages/<page>
grep -nE "\bnow\b" doc/pages/<page>
```

**When this is found on an already-reviewed page, fix the sentences and nothing else.** A page-wide
rewrap on a later targeted round produced 49 hunks for four one-sentence fixes on
`types/string-columns.md` and was reverted: the reflow buried the change and would have turned a
review into a re-read. Rewrap on the round that first reviews a page (see `feature_doc.md` §6.2's
Pass 4), never on a follow-up.

### Checking documentation links

After editing headings or `#anchor` links in README.md/CONTRIBUTING.md/CHANGELOG.md/CLAUDE.md/docs.md/
`doc/pages/*.md`, run `tools/check_doc_anchors.py` to verify every in-page and cross-file anchor
link still resolves against GitHub's actual heading-slug rules. It exits nonzero and lists any
broken link. It scans `doc/pages/*.md` too, resolving both the raw `other.md#anchor` form (used by
README.md/CONTRIBUTING.md linking *into* `doc/pages/`) and the FORD-rendered `name.html#anchor` /
`../index.html#anchor` form those pages use to link to each other / back to README.md (see
`resolve_link_target` in the script).

### FORD doc-comment conventions

Every public/private procedure, type, dummy argument/function result, and type-bound procedure
binding in `src/*.f90` carries a `!>`(leading)/`!!`(trailing) doc-comment. `ford docs.md` should
run clean (besides the expected environment-only "Graphviz not installed" warning) — keep new
code to the same standard.

**A clean `ford docs.md` run does not by itself verify doc-comment coverage.** FORD's
undocumented-entity warnings are opt-in (`-w`/`--warn` on the command line, or a `warn:` key in
`fpm.toml`'s `[extra.ford]`) and are **not** enabled in this project's `fpm.toml` — a plain
`ford docs.md` only proves nothing is *broken* (bad cross-references, malformed metadata, parse
failures), not that everything is documented. A completely undocumented new procedure can be added
and `ford docs.md` will still "run clean." To actually check coverage, run `ford --warn docs.md`
instead — expect it to be very noisy (several thousand warnings), dominated by categories that are
*not* required by this project's conventions and can be ignored:

- `Undocumented variable` for local variables (`i`, `idx`, `res`, ...);
- `Undocumented moduleprocedure` for the abbreviated `module procedure NAME ... end procedure NAME`
  form (exempted by the bullet below);
- `Could not extract source code for proc ...` — the separate FORD limitation documented under
  "FORD config gotchas" below, not a documentation gap at all;
- **`Undocumented interface` and `Undocumented proc`** — these fire on interfaces that *are*
  documented, so they are noise too. Verify before believing either one:
  `parquet_prefetch_columns_array` and `table_materialize` both carry full `!>`/`!!`
  doc-comments and both appear in this list. The count
  scales with how many interface bodies exist, so **adding N documented procedures raises it by
  roughly N** — and a large chunk of the entries are FORD's own unnamed `'unknown'` placeholders,
  which name no entity at all and cannot be acted on even in principle.

**The one category that is genuinely load-bearing is `Unknown entity`,** the use-association
accessibility limitation documented under "FORD config gotchas". **Never quote a stored figure for
it — re-derive.** It rises by one for each name any module re-exports or hides with an accessibility
statement naming a **use-associated** name, which is the expected cost of keeping a sibling module's
plumbing out of a user's namespace, so a rise of exactly that size is not a regression. Restructuring
that gives several entry modules their own settings re-exports moves it by dozens at once, entirely
legitimately. **Do not read a jump as a regression without breaking it down per module first:**

```bash
ford --warn docs.md 2>&1 | tr '\n' ' ' | tr -s ' ' | sed 's/Warning: Unknown entity/\n&/g' \
  | grep -o "attribute '[^']*' in module '[^']*'" | sort | uniq -c
```

(the `tr`/`sed` dance is needed because FORD wraps a warning across two lines, so a plain
`grep -c` on the raw output undercounts). Compare *that*
number across a change, not the total: it is the only one that moves for a real reason. When a
before/after total does move, break the delta down by category
(`grep "Warning" | sed -E 's/.*Warning: //'`) rather than treating the raw count as a regression —
a stage that adds a few dozen procedures will add several hundred warnings while documenting every
one of them.
The "before/after regression" check further down needs `ford --warn docs.md`'s output for the
same reason — comparing two plain `ford docs.md` runs compares two counts that are both
structurally 0 (Graphviz-only) and cannot detect a coverage regression — but compare it
per-category, per the paragraph above, not as a single total.

Keep new code to the same standard:

- Leading `!>` = predoc (documents what follows); trailing `!!` = postdoc (documents what
  precedes). **`!<` is not a FORD marker at all** (that's Doxygen).
- Every dummy argument/function result gets its own trailing `!!` tag wherever the argument list
  is actually written out — the spec in `parquet_core.f90`, and any submodule body that *restates*
  the full interface (`module subroutine name(args)` with the arguments redeclared, e.g.
  `parquet_maml_base_add_col_qc.f90`). The abbreviated `module procedure name ... end procedure
  name` form (no restated arguments) is exempt — nothing to tag, and the spec in `parquet_core.f90`
  is the canonical doc location for it.
- Every type-bound procedure binding (`procedure ::`, `generic ::`, `final ::` inside a type's
  `contains` block) needs its own short trailing `!!` description, separate from documenting the
  procedure it binds to (easy to forget since the bound procedure's own doc feels like it
  "covers" the binding), e.g.
  `procedure :: add => parquet_filter_add !! Appends one AND-combined rule clause.`.
  **Accepted exception:** the two `generic :: add_metadata => ...`/`generic :: add_metadata =>
  schema_add_metadata_...` bindings (`parquet_core.f90`'s `parquet_column_info`/`parquet_schema` type
  bodies) document with a leading `!>` above the binding instead, since both are long multi-line
  continuation lists where a trailing `!!` would be awkward. Every other binding in both type
  bodies still uses the trailing form — don't extend this exception to a new binding without a
  similar multi-line-continuation reason.
- Don't start a doc-comment's first line with a bare `word:` (e.g. "qc: min: ..."). FORD reads
  that as an attempted metadata key: it either warns (unrecognized key) or, worse, silently
  swallows the line if it happens to match a real key (`date:`, `author:`, `version:`, ...).
- **A rationale/gotcha explanation written directly above a procedure header is still that
  procedure's required doc-comment — it must use `!>`, not a plain `!` block.** The two look
  interchangeable at a glance (both are multi-line comment blocks sitting just above code), but
  only `!>` is picked up by FORD; a plain `!` block there silently leaves the procedure
  undocumented even though a human reader sees an explanation right above it. Plain `!` stays
  reserved for the *interface-block group banners* described below (`! ---- ... ----`) — never
  for a procedure's own doc, even a short one-off private helper.
- **After a file split/relocation refactor, verify FORD coverage didn't regress with a
  before/after diff, not just a clean `ford docs.md` run on the new state alone**: `git stash` the
  changes, run `ford --warn docs.md` (not plain `ford docs.md` — its warning count is always 0
  regardless of coverage, see above, so it cannot detect a regression this way), note the warning
  count, `git stash pop`, and re-run — matching counts confirm the refactor didn't silently drop
  doc-comments FORD would have warned about anyway. **A pure refactor is the only case where the
  raw totals should match**; a stage that *adds* procedures raises the total by design, so there
  compare `Unknown entity` and the per-category breakdown instead (see above). A relocated
  private helper legitimately
  disappearing from its old module's FORD
  page (private procedures don't get individual page entries) is expected and not a regression by
  itself — cross-check with `grep 'public ::'` in `parquet_core.f90` before treating an "not found on
  page" result as a problem.

### FORD config gotchas

- `md_extensions = ["markdown.extensions.toc"]` in `fpm.toml`'s `[extra.ford]` is
  **load-bearing** — without it, no heading gets an `id`, silently breaking every anchor link
  site-wide with no error.
- `preprocessor = "cpp -traditional-cpp -E"` is required, not optional — `src/parquet.f90`'s
  version-string logic uses real cpp macros, not just comment-style directives.
- `doc/pages/*.md` files deliberately have **no top-level heading in their body** (title comes
  from frontmatter only) — adding one back would reintroduce the duplicate-heading bug
  `doc/user.css` fixes on the front page.
- Two working CI doc-publish paths: `.github/workflows/docs.yml` (GitHub Actions → GitHub Pages)
  and `.gitlab-ci.yml`'s `readthedocs` job (GitLab CI → gitlab.4most.eu's readthedocs-style
  docserver). GitLab Pages isn't available on gitlab.4most.eu — the docserver replaces it.
- **FORD does not resolve `doc/pages/*.md` links written in README.md's body text** (`docs.md` is
  `{!README.md!}`, embedding README.md's raw markdown verbatim as the front page). FORD's own
  navbar correctly links to `page/index.html`, proving it knows the real mapping
  (`doc/pages/<name>.md` → `page/<name>.html`, `#anchor` preserved) — it just doesn't apply that
  resolution inside embedded markdown body content. Both CI docs jobs run
  `tools/fix_ford_page_links.sh ford-doc` right after `ford docs.md` to fix this in the generated
  output; README.md's source keeps the `doc/pages/*.md` form since that's what's correct for
  browsing the repo directly on GitLab/GitHub. This is host-independent (unlike
  `tools/prep_github_mirroring.sh`), so both CI jobs need it, not just one. `doc/pages/*.md` files
  linking to *each other* don't have this problem — they already use FORD's native
  `page/*.html`-relative form directly in source.
- `tools/generate_parquet_maml.sh`'s `end module` template line must keep emitting
  `! GCOVR_EXCL_LINE` (added deliberately, not FORD/gcovr's default) — a careless edit there
  will silently drop it from every regenerated file (`src/parquet_maml_base.f90` and any
  downstream project's generated `parquet_maml` module).
- **FORD 7.0.13 never renders per-argument docs for members of a named, multi-specific generic
  interface** (e.g. `parquet_get_metadata`'s 12 `module procedure` entries) — every member's card
  under "Module Procedures" on the generic's page permanently shows "Arguments: None". This is
  **not fixable from source** — rewriting the submodule's abbreviated `module procedure NAME` into
  a fully restated `module subroutine NAME(args)` has no effect on the rendered output. Solo public
  procedures that aren't generic members (e.g. `parquet_get_string_length`) are unaffected.
  Workaround in place: each of the 12 public generics' own leading `!>` doc-comment (the block
  immediately above `interface <name>`) spells out every distinct argument name/role in prose,
  since that comment does render on the generic's page. Do not re-attempt the submodule-restatement
  fix without first checking a newer FORD release against upstream issue
  (https://github.com/Fortran-FOSS-Programmers/ford/issues/738).
- **FORD 7.0.13 cannot resolve a `use`-association accessibility statement** — an
  `Unknown entity '<name>' with attribute '<public|private>' in module '<m>'` warning, and the one
  FORD number worth tracking across a change. **Compare the PER-MODULE census, not the total**, and
  derive both with the command above rather than quoting a stored figure; a census written down here
  went stale twice and was once *derived* rather than measured, under-reporting a whole module's
  group.

  **What it keys on is USE-ASSOCIATION, and nothing else.** A module re-exports or hides a name it
  imported, and FORD cannot follow that; a name *defined* in the module itself never warns. The
  sharpest illustration is a pair of settings accessors on the **same** `public ::` statement, where
  the getter warns and the setter does not, purely because the getter lives in
  `parquet_settings_base` and the setter in `parquet_settings`. So the count is driven by two
  deliberate rules — every entry module re-exports the settings knobs its own code reads (see
  "Nested submodule tree"), and both facades hide the cross-module plumbing Fortran has no package
  scope for. Neither has a spelling that avoids the warning, and the alternatives are a narrow
  import that cannot configure itself and a public namespace full of internals.

  **Confirmed not fixable from source**: neither a `!>` doc-comment on the line, nor an explicit
  `use ..., only:` list, nor a bare unrestricted `use` changes anything — all three were tried
  against FORD 7.0.13. Nothing is lost from the generated site either; the symbols get their own
  type and module pages, they just do not appear as belonging to the re-exporting module on its own
  page. Re-test against a newer FORD before attempting a source-side fix again.
- **The `parquet` facade's re-exports do not appear on `module/parquet.html` either, and that is
  the accepted cost of S10.** The facade re-exports its siblings with *bare* `use` statements
  (no `public ::` list), which produces no warning at all — but FORD equally does not list the
  re-exported names as belonging to `parquet`, so the page a user is told to `use` is now
  **empty**: version reporting used to be the facade's own code and kept the page at ~33 KB, and it
  moved out (`parquet_get_version` to the leaf `parquet_version`, `parquet_get_arrow_version` to
  `parquet_settings`) so that an Arrow-free tier could report a version at all. **The maintainer
  accepted this** and the guide points
  readers at the site-wide `lists/procedures.html`/`lists/types.html` instead, where every name does
  appear — `doc/pages/index.md` says so explicitly. This is a cosmetic gap in the generated
  reference, not a broken link. Re-test against a newer FORD before assuming it still holds.
- **FORD 7.0.13 cannot extract a "Source Code" section for a `module procedure NAME ... end
  procedure NAME` implementation** (the abbreviated form; `fpm.toml` sets `source = true`), which
  is why `ford --warn docs.md` reports roughly 240 "Could not extract source code for proc ..."
  warnings, including for a large share of the public surface (`parquet_get_nrows_int32`/`_int64`,
  every `parquet_write_<type>_column`, every `parquet_get_metadata_<type>`, ...). **Confirmed
  fixable in principle, but not worth doing**: rewriting one such procedure
  (`parquet_get_string_length` in `parquet_read.f90`) from the abbreviated form into a fully
  restated `module subroutine NAME(args) ... end subroutine NAME` does make FORD extract its
  source correctly — but it also makes FORD start requiring that submodule body to carry its own
  doc-comments (it otherwise reports a new `Undocumented interface` warning), which directly
  conflicts with this project's own deliberate convention (see "FORD doc-comment conventions"
  above) that the abbreviated form is *exempt* from restating docs precisely so they aren't
  duplicated between `parquet_core.f90`'s spec and every submodule body. Applying this fix across all
  ~240 affected procedures would mean restating and re-documenting most of
  `parquet_read.f90`/`parquet_write.f90`/`parquet_metadata*.f90` — a large, high-blast-radius
  rewrite for a convenience feature (an auto-generated source listing on each procedure's page),
  and it was left undone; the maintainer should decide before anyone attempts it project-wide.

## Source code structure & conventions

### One program unit per file; filename == unit name

Every `src/*.f90` file defines exactly one `module` or `submodule`, and the filename (sans
`.f90`) equals that program unit's name — e.g. `parquet_read_numeric.f90` ⇒
`submodule (parquet:parquet_read) parquet_read_numeric`. This is a near-universal fpm/Fortran
convention and is load-bearing for navigation and for the publish tooling (which flips
`module-naming` to `"parquet"`). Do not put two program units in one file, and do not name a
file differently from the unit it defines. (fpm does not hard-fail on a mismatch by default —
`module-naming = false` here — but treat it as a firm rule.)

### Some `src/*.f90` files are generated — edit the generator, never the output

Several source files are emitted by a `tools/` script from a single source of truth (a kind table, a
schema, a template) and are **committed** rather than regenerated at build time, so `fpm build` stays
dependency-free. They look like ordinary hand-written source when opened, which is exactly the trap:
editing one directly appears to work, passes tests, and is then silently reverted the next time anyone
regenerates.

**Always check the top of a `src/*.f90` file before editing it.** Every generated file opens with a
banner naming its generator (`GENERATED FILE -- DO NOT EDIT BY HAND` / `automatically generated`). If
that banner is there, make the change in the generator's own input instead — the kind/field table or
template inside the script — and re-run it. Currently generated: `src/parquet_maml_base.f90` (from
`tools/generate_parquet_maml.sh` + the `.maml` files under `schemas/`), and `src/parquet_columns.f90`,
`src/parquet_columns_access.f90`, `src/parquet_columns_mutate.f90` (from
`tools/generate_parquet_columns.py`, whose kind table is the single place a supported column kind is
declared), and `src/parquet_tables.f90`, `src/parquet_tables_access.f90`,
`src/parquet_tables_addcol.f90`, `src/parquet_tables_materialize.f90` (from
`tools/generate_parquet_tables.py`, which imports that same kind table), and `src/parquet_sorting.f90`,
`src/parquet_sorting_keys.f90`, `src/parquet_sorting_argsort.f90`, `src/parquet_sorting_permute.f90`,
`src/parquet_sorting_select.f90`, `src/parquet_sorting_search.f90`, `src/parquet_sorting_unique.f90`,
`src/parquet_sorting_reduce.f90`, `src/parquet_argsort.f90` and `src/parquet_argsort_kernel.f90`
(all from `tools/generate_parquet_sorting.py`, which emits TEN files across the two sorting tiers and
imports the nine SCALAR rows of that same kind table
and adds the three types that are not `parquet_column` storage kinds at all), and
`src/parquet_ziggurat.f90` (from `tools/generate_parquet_ziggurat.py`, whose 771 constants are the
unique solution of one equation rather than a table anyone chose — see its `--self-test`). `src/parquet_table_example.f90` is emitted by
`tools/generate_user_table_code.py` from `table_types/maml_example4.maml` (see "Role-A MAMLs live in
`table_types/`" below) — it ships as a worked example and nothing else in the library uses it, but it
is committed and `--check`ed exactly like the rest. **`src/parquet_tables.f90` is
the one most likely to be edited by mistake**, because it is the table layer's module spec — every
type-bound binding and every interface body lives there, so adding a `parquet_table` procedure means
editing the generator's literal template text, not the file it emits. Treat this list as a snapshot —
trust the banner, not the list, and add new generated files here when they appear.

**`src/parquet_argsort_engine.f90` is the ONE sorting file that is hand-written**, and the split
matters in both directions: its four procedures' **interfaces** live in the generated
`src/parquet_argsort.f90`, so changing a signature means editing the generator's
`emit_engine_interfaces()` while changing the body means editing the engine file directly. Everything
else across BOTH sorting tiers — including `sort_key_buf`, every module-level `save` variable and
every `parquet_debug_*` declaration — is generated, so a new debug hook is a generator change even
though its implementation is not. (It was `src/parquet_sorting_engine.f90` until the argsort tier was
split out; a source comment or planning document naming the old path means this file.)

**`src/parquet_sorting_oracle.f90` is hand-written too, and is NOT part of that rule.** It is the
C++ sort engine behind procedure pointers, a separate module rather than a submodule of either
tier, and the generator does not know about it. It is also the only file in the sorting area that
imports `parquet_bindings` — which is the whole point, since that import is what the two tiers must
never acquire.

**Grep for the banner carefully: the engine's own header begins "NOT a generated file".** A
case-insensitive search for "generated file" therefore reports it as generated, which is the exact
inversion of the truth and would send someone to edit the generator for a file the generator does not
emit. Read the line, don't match it.

Working rules for this class of file:

- **A generator must emit the project's own conventions**, or it multiplies a single template omission
  across every kind it emits: `!>`/`!!` doc-comments on everything public (only `ford --warn docs.md`
  would reveal their absence — see "FORD doc-comment conventions"), the `! GCOVR_EXCL_LINE` markers, and
  the 132-column line limit.
- **Prefer a generator that can verify its own output.** `tools/generate_parquet_columns.py --check`
  re-derives the files in memory and exits nonzero if the committed copies have drifted — run it after
  any change in that area, and give a new generator the same mode.
- **Sibling generators should share one source of truth rather than each carrying its own copy.** A
  second generator over the same kind table imports it from the existing script instead of duplicating
  it; two drifting copies of a kind list is a much worse failure than one slightly awkward import.
  **A CONSUMER-FACING generator cannot do that**, because it is copied into projects where this
  repository's files do not exist — so it bakes the copy in and has `--self-test` cross-check it
  against the real source *when that source is present*, which is always here and never downstream.
  `tools/generate_user_table_code.py` carries `parquet_table`'s ~264 type-bound procedure names
  that way (a field name colliding with one cannot become an accessor), so **adding a binding to
  `parquet_table` fails the lint stage until that list is updated** — which is the point: the
  staleness is closed by a test rather than by remembering.
- **A generator whose output is USER-EDITABLE needs marker-delimited windows, and three rules
  that make them safe.** Most generated files here are machine-owned; `tools/generate_user_table_code.py`
  emits a module a user is expected to extend, which is a different problem. The windows are the
  generator's **input**, not decoration — it lifts them out of the existing file, re-emits
  everything else, and puts them back; it also *reads* one of them to write the matching
  `%clone`/reset statements. So: a malformed marker set (missing, duplicated, unbalanced) must be
  refused **before** anything is rewritten, since that is the one state in which regenerating
  destroys user code; `--check` belongs in CI, because an edit outside a window otherwise works
  perfectly until the next regeneration silently deletes it; and the header carries a digest of the
  source input, so `--check` can say "you edited generated text" rather than "this file is stale"
  — without it both look identical and the message has to guess. See `feature_risks.md` Risk-32.
- **A generator that is maintainer-only needs NO entry in `tools/prep_fpm_publish.sh`** (downstream
  projects consume the committed output, and `tools/` is an allow-list, so anything not named in
  `TOOLS_KEEP` is stripped automatically). One that is consumer-facing — like
  `tools/generate_parquet_maml.sh`, which downstream projects run on their own schemas — **must be
  added to `TOOLS_KEEP`**, or the published tarball will not contain it. See "Keeping
  `tools/prep_fpm_publish.sh` in sync".

### Nested submodule tree

`src/*.f90`'s `parquet_core`/`parquet_read`/`parquet_write`/`parquet_metadata` files form a nested
submodule tree (not flat siblings under `parquet_core`), split by data-type family for read/write
and by format for metadata:

```
parquet                         (module — the FACADE; see below. Holds NO code at all.)
parquet_io                      (module — the I/O FACADE: re-exports parquet_core + parquet_settings
                                 and nothing else. No code at all.)
parquet_core                    (module — core API + cross-subtree private-helper interfaces)
├─ parquet_read                 (submodule — reader lifecycle, queries, shared read helpers)
│   ├─ parquet_read_numeric     (int32/int64/float32/float64/logical, all access modes)
│   ├─ parquet_read_string
│   └─ parquet_read_temporal    (date/time/timestamp)
├─ parquet_write                (submodule — writer lifecycle, shared write helpers)
│   ├─ parquet_write_numeric
│   ├─ parquet_write_string
│   └─ parquet_write_temporal
└─ parquet_metadata             (submodule — parse/build orchestration + shared metadata helpers)
    ├─ parquet_metadata_base    (format-agnostic column_info/table_metadata plumbing)
    ├─ parquet_metadata_get     (parquet_get_metadata queries)
    └─ parquet_metadata_maml    (MAML-specific: section schema + all validation)

parquet_bindings                (module — independent C++ interop)
parquet_settings_base           (module — LEAF: every knob's state, getters, setters and the three
                                 emit channels. Imports iso_fortran_env + omp_lib only.)
parquet_settings                (module — settings_base + parquet_bindings: print, reset, env, the
                                 ONE C++ push parquet_push_settings_to_cpp, and
                                 parquet_get_arrow_version)
parquet_version                 (module — LEAF: cversion and parquet_get_version. Imports
                                 parquet_settings_base only, for the emit channel.)
parquet_strings                 (module — independent element domain; + settings_base)
parquet_temporal                (module — independent element domain)
parquet_random                  (module — the generator AND the four distributions; reaches only
                                 parquet_expkey and parquet_ziggurat, both leaves — enforced)
parquet_expkey                  (module — LEAF: the frozen -log(u) transform)
parquet_ziggurat                (module — LEAF, GENERATED, data only: the normal's layer tables)
parquet_argsort                 (module — the ARGSORT TIER: pf_argsort over the six intrinsic
                                 types, pf_sort_threads, sort_key_buf, the sorting knobs and the
                                 oracle's procedure pointers. Reaches settings_base ONLY.)
├─ parquet_argsort_engine       (submodule — HAND-WRITTEN: comparators + radix/merge/counting)
└─ parquet_argsort_kernel       (submodule — GENERATED: extractors, engine drivers, the twelve
                                 intrinsic pf_argsort specifics, the thread rule)
parquet_sorting                 (module — the FULL tier: extends pf_argsort with the five
                                 down-tier element types and adds every other pf_ operation)
parquet_sorting_oracle          (module — TEST-ONLY: the C++ sort engine behind procedure
                                 pointers. Never re-exported by any facade.)
parquet_sampling                (module — permutations/subsets/resampling/weighted draws;
                                 uses parquet_random + parquet_argsort + parquet_expkey)
parquet_maml_base               (module — generated)
└─ parquet_maml_base_add_col_qc (submodule)
parquet_wrapper.cpp             (C++ TU)
```

**The ADVERTISED ENTRY MODULES are the rows of the table in
`doc/pages/operating/choosing-a-module.md`, which is the authority — do not re-derive the list from
this tree by subtracting exceptions.** That is how this paragraph used to read, and it was wrong in
both directions: it swept in `parquet_maml_base`, whose only user-facing names are three types the
facade imports with an `only:` list, and it read as though the list lived here rather than on the
page. The same list is `ENTRY_MODULES` in `tools/check_module_footprints.sh`, which had drifted from
the page in the direction that measures nothing — `parquet_tables` and `parquet_settings` were both
absent, so the number the page printed for `parquet_tables` had never been measured by anything.
Every one of the twelve is covered by the semantic-versioning promise **in its own right**. That is wider than it sounds: a change to
`parquet_column`'s bindings is a breaking change even when nothing reachable through `use parquet`
moves. Two properties follow, and both are enforced rather than intended:

- **Eight tiers keep their FORTRAN graph clear of `parquet_bindings`** — `parquet_version`,
  `parquet_temporal`, `parquet_strings`, `parquet_random`, `parquet_argsort`, `parquet_sampling`,
  `parquet_columns` and `parquet_sorting`. **There is now one check per tier**
  (`tools/check_source_conventions.py`), each walking that tier's closure **including submodules**
  and failing if it ever reaches `parquet_bindings`. One check per tier rather than a few for the
  group is deliberate and was arrived at the hard way: three of these tiers used to be covered only
  *transitively*, because `parquet_sorting` happens to import `parquet_columns` which imports
  `parquet_temporal` — real coverage that would evaporate silently the day that import went — and
  `parquet_version` was covered by nothing at all, since no checked module imports it.
  `tools/check_argsort_standalone.sh` proves it the other way round, by compiling the argsort tier with
  a bare compiler and no Arrow at all. **This never made the PACKAGE Arrow-free** — `link` is a
  package-level key in `fpm.toml`, so `parquet_wrapper.cpp` is compiled and `-larrow` linked
  whichever module a consumer names. Say which of the two you mean.
- **A `use` line added anywhere can multiply what a consumer compiles**, because fpm prunes at
  module granularity and never prunes a submodule separately. `tools/check_module_footprints.sh`
  builds a throwaway consumer per entry module and diffs the compiled set against the committed
  `tools/module_footprints.txt`. When it fails, **updating the expectation is almost never the
  fix** — move whatever needed the import up a tier instead.

**Where a new procedure goes is therefore a TIER decision, not a filing one.** Anything reaching a
reader, a writer or `parquet_bindings` belongs at or above `parquet_core`. Anything a
`use parquet_sampling` program needs belongs in `parquet_argsort` or lower. And **a module
re-exports, getter and setter both, every settings knob its own code reads** — including
`verbosity`/`message_stream` when it can emit — which is what lets a narrow import be configured
without naming `parquet_settings` and putting the C++ boundary back. `test/test_module_surface.f90`
holds one single-import module per tier and is where that rule is asserted; **do not add a second
library `use` to any module in that file**, or it silently stops testing anything.

**`parquet` and `parquet_io` are BOTH facades and neither holds any code.** `src/parquet.f90`
re-exports `parquet_io`, `parquet_tables`, `parquet_columns`, `parquet_strings`, `parquet_temporal`,
`parquet_sorting`, `parquet_random`, `parquet_sampling`, `parquet_settings`, `parquet_version` and
three types from `parquet_maml_base`, so a user writes exactly one `use parquet`. `src/parquet_io.f90` re-exports `parquet_core` and `parquet_settings` and is the
supported face of the reader/writer surface for a program that never builds a `parquet_table`.
Five rules follow, and all five are easy to violate by reflex:

- **The core API's spec file is `src/parquet_core.f90`, not `src/parquet.f90`.** Every
  `public ::` line, every interface body, every shared `parameter` reached by host association
  from a submodule lives there. A note elsewhere in this file saying "declared in `parquet.f90`"
  and meaning the reader/writer spec means `parquet_core.f90`.
- **The facade must not gain library logic**, and since version reporting moved out it holds
  none: no `contains`, no parameter, nothing needing cpp. A new public procedure goes in
  `parquet_core` (or the relevant sibling) and is re-exported for free. The cost of holding
  nothing is that `module/parquet.html` renders empty — accepted, see the FORD note above.
- **`parquet_version` is the one module the facade re-exports that NO sibling does.**
  `parquet_get_version` is deliberately unavailable from `parquet_io`, `parquet_tables`,
  `parquet_settings` or any Arrow-free tier: it is not a setting, nothing in the library reads it,
  and making every tier carry it would grow every tier's graph for a compile-time constant. A
  program on a narrow import writes `use parquet_version` (two files, no C++ boundary). Do not
  "fix" this into consistency with the settings re-export rule — and note the one bare
  `use parquet_version` line in `src/parquet.f90` is what `test_facade_covers_every_layer`
  (`test/test_examples.f90`) and `test_module_surface_version` (`test/test_module_surface.f90`)
  exist to pin from both sides.
- **The linked ARROW version is a different question with a different home.**
  `parquet_get_arrow_version` lives in `parquet_settings`, beside `parquet_get_arrow_threads`,
  because reading it means calling into C++ — so it reaches `parquet_io`, `parquet_tables` and
  the facade, and no Arrow-free tier. Its two `bind(C)` interfaces are named `c_get_arrow_version`
  /`c_get_parquet_version` in `parquet_bindings` for the collision reason under "Naming
  conventions"; the `bind(C, name=)` values, and so `parquet_wrapper.cpp`, are untouched.
- **The facade uses bare `use <sibling>` with DEFAULT-PUBLIC accessibility**, deliberately — that
  is what re-exports a whole module without maintaining a ~120-name `public ::` list. Its eleven
  `private ::` statements (19 names: the compression pair, the three emit channels and the
  suppression query, the affinity clamp, the C++ settings push, the validity block width, and the
  ten `parquet_column_*` accessors) are the only thing
  keeping implementation details out of the user's namespace, so anything new the facade imports
  for its own use needs its own `private ::` line. `parquet_bindings` is never re-exported.
  `parquet_maml_base` is imported with an `only:` list rather than in full, because its other
  public names are this library's own embedded MAML fixtures, not user API.
- **`parquet_core` is internal and documented as such** (README's API-stability bullet,
  `doc/pages/tables/table.md`, and the module's own doc-comment). `use parquet` and `use parquet_io`
  are the two supported spellings of that surface and both carry the semantic-versioning promise;
  `parquet_core` carries none. Sibling modules must `use parquet_core`, never `use parquet` —
  the facade uses *them*, so the reverse is a circular dependency and will not compile.
- **The two facades must hide the SAME names.** `parquet_io`'s `private ::` list covers the
  cross-module plumbing `parquet_core` and `parquet_settings` are forced to make public for want of
  package scope — `parquet_split_name_list`, `parquet_parse_sort_key`, the compression resolver and
  the emit channels. `src/parquet.f90` must NOT repeat the first two: `parquet_io` has already
  hidden them, so a `private ::` there names a symbol that is not accessible at all, which nagfor
  reports as an implicitly-typed local rather than as anything to do with accessibility.

Two regression tests, and each catches what the other cannot. `test/test_examples.f90`'s
`test_facade_covers_every_layer` imports nothing but a bare `use parquet`;
`test/test_module_surface.f90`'s `test_module_surface_io` imports nothing but `use parquet_io` and
round-trips a file through it. Both break the *build* rather than an assertion when a re-export is
dropped, which is how they earn their keep.

Reserved for future element-domain work (not yet implemented): `parquet_map`/`parquet_list`/
`parquet_struct` (independent modules, like `parquet_temporal`) plus their own
`parquet_read_*`/`parquet_write_*` type-family children — see CONTRIBUTING.md's "Features
considered but not implemented" for scope/status.

**Placement rule for a new read/write specific or shared helper:** type-generic code (used by
more than one of numeric/string/temporal) belongs in the parent (`parquet_read`/`parquet_write`)
as an ordinary contained procedure — descendants reach it by host association, no interface
needed. Type-specific code belongs in the matching child, also as an ordinary contained
procedure. A new *public* generic's specifics, and any type-bound binding target, must keep
their interface declared in `parquet_core.f90` itself regardless of family (see "A module procedure
cannot implement its own submodule's spec-declared interface" below for what breaks if you
relocate one incorrectly) — never assume a helper is safe to relocate purely from its call sites
without also checking those two disqualifiers, plus whether it's itself `public ::`-exported.

**A known gfortran 15.2.0 ICE to watch for when adding a new cross-subtree call into this
tree:** calling `parquet_parse_protected_cols` (declared in `parquet_core.f90`) directly from a
submodule nested **two levels** under `parquet` (e.g. `parquet:parquet_metadata:
parquet_metadata_maml`) crashes the compiler with an internal compiler error — isolated to this
one procedure's exact argument shape (an assumed-length `character` array paired with a
deferred-length allocatable `character` array result, i.e. `character(len=*), intent(in) ::
lines(:)` + `character(len=:), allocatable, intent(out) :: names(:)`) called from 2+ levels of
nesting. Worked around via a thin relay: `parquet_metadata.f90` (the level-1 parent, where
calling it already works) exposes a plain contained subroutine
`parquet_parse_protected_cols_relay` that just forwards to it, and the grandchild calls that
relay instead of reaching two levels up directly. If a *different* procedure with a similar
deferred-length-character-array-result shape hits the same ICE from deep nesting, use the same
relay pattern — this bug is narrow (only this one procedure's exact shape is known to trigger
it; other similarly-shaped procedures at the same nesting depth compile and run cleanly).

### Group interface bodies into commented `interface` blocks

Declare the `module subroutine`/`function` interface bodies in `parquet_core.f90` (and in any
submodule spec that hosts relocated interfaces) as **several small `interface … end interface`
blocks grouped by concern**, each introduced by a one-line plain-`!` banner (e.g.
`! ---- Read column specifics (by type x access mode) ----`) — not one monolithic block.
Fortran allows arbitrarily many interface blocks, so this costs nothing and keeps the
declarations navigable. The banner **must be a single-bang `!` comment, never `!>`**: a `!>`
immediately above the first `module subroutine` in a block is a predoc and FORD would attach it
to that procedure. Keep each new interface body in the group that matches the submodule
implementing it. Do not give the blocks `generic-spec` names (e.g. `interface foo`) purely for
labeling purposes — that declares an actual named generic interface (a callable overloaded
entry point requiring every member to be a distinguishable-argument-list overload of the
others, and unable to mix `subroutine`/`function` specifics under one name), not a label. The
plain-`!` banner is the only naming mechanism for these groups.

### A module procedure cannot implement its own submodule's spec-declared interface

A `module procedure`/full-restated `module subroutine` body must live in a **descendant** of
whichever spec declared its interface — never in the same submodule that declares it. This
matters when relocating a private helper's interface (per the Placement rule in "Nested
submodule tree" above): if the helper's *body* already lives in the same file the interface is moving into
(e.g. relocating an interface from `parquet_core.f90` into `parquet_metadata`'s own spec, when the
body already lives in `parquet_metadata.f90` itself, not one of its children), keeping it a
`module procedure` no longer works — gfortran reports errors like "Symbol ... has already been
host associated" or, for a still-public name, a `public ::` failure. Convert that one procedure
to a **plain contained procedure** instead (drop `module`, restate the full signature with its
own `!>`/`!!` doc-comment, since there's no longer an interface to hold it): callers in the same
file are unaffected, and descendants still reach it by host association (confirmed on gfortran 15
for both a module→submodule and a submodule→sub-submodule chain). This is independent of, and
does not need, the interface remaining declared anywhere — see `parquet_parse_col_map`
(`parquet_metadata.f90`) and `parquet_check_read_row_count` (`parquet_read.f90`) for worked
examples.

### A separate module procedure must be IMPLEMENTED before it is CALLED in the same submodule

NAG 7.2 binds a separate module procedure's name to an implicit **external** procedure at the first
call site, so the later `module procedure` statement implementing that same name is rejected:

```
Error: MAKE is not the interface of a separate module procedure
       detected at MAKE@<end-of-statement>
```

**gfortran, ifx and flang all accept either order**, so nothing in the ordinary fleet enforces this
and a violation can sit in the tree indefinitely. Confirmed instance: `parquet_metadata_maml.f90`'s
`parquet_validate_maml_file` called `parquet_load_maml_file` 64 lines above where that procedure was
implemented. Moving the implementation above its caller is the entire fix and costs the other
compilers nothing -- it is a pure relocation, with no line of the body changed.

Reproducer (fails under NAG; compiles when the two bodies are swapped):

```fortran
submodule (mm:mid) leaf
contains
    module procedure driver
        out = make(fn)          ! CALLS make ...
    end procedure driver

    module procedure make       ! ... and implements it LATER: rejected
        r = len_trim(fn)
    end procedure make
end submodule leaf
```

**This does NOT apply to an ordinary contained procedure.** A plain `subroutine`/`function` defined
inside the same submodule is reached by host association and may appear in any order -- which is why
`parquet_load_maml_file` may still sit above `parquet_read_maml_source_lines` and call it. The rule
is specific to a *separate module procedure*, i.e. one whose interface is declared in an ancestor.

**The error names the implementing statement, never the call that caused it**, so the first move on
seeing it is to grep the file for earlier references to that name. More generally, when a submodule
error resists explanation, compile a **minimal leaf submodule implementing only the failing
procedure** against the real module files: if that compiles, the ancestors, the interface and the
procedure's result type are all exonerated in one step and the cause is elsewhere in the file.
Growing a synthetic reproducer toward the real code proved nothing across five rounds here (the
result type, its allocatable components, its submodule-implemented type-bound procedure, the
ancestor's re-export, and the parent calling it all compile clean); the minimal-leaf test found it
immediately.

### Naming conventions

Follow these when adding new public API, types, or internal helpers:

- **Public module-level API** (anything in `src/parquet_core.f90`'s `public ::` list — functions,
  subroutines, types) always carries the `parquet_` prefix, e.g. `parquet_get_metadata`,
  `parquet_open_reader`, `parquet_schema`.
- **One prefix per module, applied to everything public in it — and `parquet_` is not the only
  one.** `parquet_` is for the parquet-file-facing modules (the reader/writer/schema/table/element
  domains: everything listed under "Nested submodule tree"). **`pf_`** — for parquet-fortran, the
  library as a whole — is for *library-wide utility* modules whose subject is not a parquet file at
  all. There are four: `parquet_sorting` (a general-purpose sorting API over plain Fortran arrays),
  whose procedures are `pf_sort`, `pf_argsort`, `pf_permute`, … and whose type is `pf_sort_keys`;
  `parquet_argsort` (the tier below it: `pf_argsort` over the six intrinsic element types, plus
  `pf_sort_threads`), which shares that vocabulary because it shares the generic — see the tier note
  under "Nested submodule tree";
  `parquet_random` (counter-based random numbers over nothing but a seed and an index), whose
  procedures are `pf_random_at`, `pf_random_int_at`, `pf_random_fill_draws`, … with the frozen contract
  identifier `pf_random_algorithm`; and `parquet_sampling` (drawing from a population rather than
  drawing a number), whose procedures are `pf_random_perm_at`, `pf_random_subset`,
  `pf_random_resample`, `pf_weighted_subset`, `pf_weighted_permutation`, … with the type
  `pf_weighted_draw` and the second frozen identifier `pf_random_perm_algorithm`. **The
  `parquet_random`/`parquet_sampling` split is a dependency boundary, not a filing decision** —
  see the leaf rule below. Note the one deliberate exception in each: a **test-only debug
  hook keeps the project-wide `parquet_debug_*` spelling** rather than the module's own prefix
  (`parquet_debug_random_uses_int128`), because that convention is what marks a procedure as a debug
  hook across the whole codebase and is the more useful signal at the call site.
  **Do not "correct" a `pf_` name to
  `parquet_`** — nothing in `tools/check_source_conventions.py` enforces either prefix, so the rule
  lives here and nowhere else. Two things this rule is *not*: it is not a licence to mix prefixes
  inside one module (pick one and apply it to every public name there), and it is **not** a reason
  to rename the existing library-level `parquet_`-named procedures (`parquet_get_version`,
  `parquet_kind_name`), which are deliberately left alone — renaming them would be a public API
  break for a naming preference.
  Note the constraint that shapes such a module's own name: **a module cannot share its name with a
  procedure it declares**, which is why the module is `parquet_sorting` and not `parquet_sort` (see
  the `parquet_strings`/`parquet_string` bullet further down).
- **Type-bound procedures** (`schema%init`, `schema%add_field`, `reader%...`) do *not* need a
  `parquet_` prefix — the type itself namespaces them. If the natural short name collides
  with another type's backing implementation, keep the short name as the type-bound binding
  target and give the private module procedure a distinguishing name (e.g.
  `parquet_column_info`'s binding `set_column_available => set_available`, kept short only to
  avoid colliding with `parquet_schema`'s own `set_column_available` impl).
- **`maml_` prefix** is reserved specifically for MAML-parsing/building internal helpers
  (e.g. `maml_push_line`, `maml_line_exists`) — don't reuse it for unrelated internal code.
- **Other private module-level helpers** (in `parquet_metadata.f90`, `parquet_read.f90`,
  `parquet_write.f90`) generally keep the `parquet_` prefix too, even though private —
  matches the existing majority convention in those files; only give it a bare, unprefixed
  name if it's a small, obviously-local helper (rare; check for existing precedent first).
- **Private module-level types** (not part of the public API, e.g. `maml_section_schema`)
  drop the `parquet_` prefix — this is intentional, not an inconsistency to "fix".
- **C++ bindings** (`parquet_bindings.f90` interfaces, `parquet_wrapper.cpp`) mirror the C++
  side's own naming (still generally `parquet_`-prefixed for the `extern "C"` surface) —
  don't rename these to match Fortran-side conventions. **One exception, and the rule for
  creating another:** `parquet_core.f90` does an unrestricted `use parquet_bindings`, so a public API
  procedure cannot share a name with a binding. When that collides, keep the public name and give
  the *Fortran-side interface* a `c_`-prefixed one while leaving `bind(C, name="...")` — and thus
  the linked symbol and `parquet_wrapper.cpp` — untouched; `tools/check_bindc_boundary.py` keys on
  the `bind(C, name=)` value, so it follows the rename with no change. There are four instances:
  `c_reader_set_filter` (bound to `parquet_reader_set_filter`, whose Fortran name belongs to the
  public post-open filter setter), `parquet_get_thread_pool_capacity` (bound to
  `parquet_get_max_threads`, renamed the other way round for the same collision with
  `parquet_get_arrow_threads`), and `c_get_arrow_version`/`c_get_parquet_version` (bound to
  `parquet_get_arrow_version`/`parquet_get_parquet_version`, the first of which is now
  `parquet_settings`' public linked-version query).

- **A new module holding several related element/handle types** (as opposed to one module per
  type) should be named after the *domain* those types belong to, not any single type inside
  it — e.g. `parquet_temporal` for `parquet_date`/`parquet_time`/`parquet_timestamp`. See "The
  `parquet_temporal` module" below for the reasoning and the sibling modules (`parquet_map`,
  `parquet_list`) this leaves room for.

- **A module cannot share its name with a type (or a procedure) it declares** — gfortran rejects
  it outright. This has bitten twice: it is why the `parquet_strings` module is plural while its
  type is `parquet_string` (see "The `parquet_strings` module" below for that instance), and it
  constrains a generated table type's MAML, where `dataset:` names the module and `table:` derives
  the type. Check the pair whenever you name a module after what it holds.

When in doubt, grep for an existing analogous name before inventing a new convention.

### `parquet_random` is a LEAF; `parquet_sampling` is where anything more goes

**`src/parquet_random.f90` imports `iso_fortran_env` and nothing else, and that is a hard
constraint rather than a tidy accident.** Two checks compile that one file standalone — with no
dependency resolver, no fpm, and no Arrow install anywhere — and they are the only evidence for two
properties nothing else can reach:

- **`tools/check_random_kernels.sh`** builds the arms of the route (e) `#ifdef` fork and asserts
  they agree. The fork exists because gfortran 14 and 15 have both been observed *miscompiling* the
  unprotected Philox round at `-O3`, silently, with no warning under `-Wall -Wextra`. The wrapping
  arm ships wherever the compiler has no 128-bit integer kind (ifx), and **no other check compiles
  it at all**, because every other compiler in the fleet takes the protected arm.
  **Run it under `FC=nagfor` as well, not only gfortran.** A run there covers the `PF_SAFE64` arm as
  nagfor's OWN build, which is the only place that happens — every other family can merely *simulate*
  that arm with `-D__NAG_COMPILER_RELEASE=1`, and a simulation is a statement about the simulating
  compiler's codegen. Skipping it is what let a wrong-answer defect in `ult` reach the golden vectors.
  A nagfor run covers **two** arms: `safe64` as its shipped build, and the wrapping kernel through
  the script's forced half, which reaches it by pre-expanding the source with an external `cpp` —
  nagfor has no `-U` at all (`-u` means IMPLICIT NONE), and the fork deliberately has no
  `-DPF_FORCE_*` door to open. The script asserts a nagfor run really compiled `safe64`, rather than
  reporting green on whatever it got. The int128 arm is out of reach there and stays with the
  families that have a 128-bit kind. **What that forced half established is worth knowing:** nagfor
  compiles the wrapping kernel correctly at `-O0`, `-O2`, `-O3` and `-O4`, so its `PF_SAFE64` is a
  choice bought for `-C=intovf`, not a correctness necessity — while gfortran's wrapping arm is the
  one that fails, under `-flto`.
- **`tools/check_exp_key.sh`** does the same for `src/parquet_expkey.f90`'s frozen `-log(u)`
  transform, which is a leaf for the identical reason. **Run this one under `FC=nagfor` too**, and
  know which of its arms carry the weight there: nagfor generates C, so contraction and
  reassociation happen in the BACKEND and are reached with `-Wc,` — and its default backend line
  carries **`-march=nocona`**, a pre-FMA target, so without the `-Wc,-march=native` pairing the
  whole FMA class is invisible on that compiler. Measured by defeating `ek_rnd`'s `volatile`
  barrier: the plain `-O0`/`-O2`/`-O3`/`-O4`/`-float-store` ladder still reproduced the frozen
  value, while `-O4 -Ounsafe` and the `-Wc,-ffast-math` arms caught it — the fast-math-plus-native
  pairing with a different wrong fingerprint again. An optimisation ladder alone proves almost
  nothing under nagfor.

**The failure mode is a check that quietly stops being able to run, which is the worst kind.** When
the weighted draw was first written it went into `parquet_random`, bringing `use parquet_sorting`
with it — hence `parquet_bindings`, hence the whole of `parquet_wrapper.cpp` and Arrow. Every
configuration in the kernel check then died on a missing `parquet_sorting.mod` and the script
exited saying it proved nothing, while `fpm build` and `fpm test` stayed perfectly green throughout
(the library obviously has Arrow). Nothing else noticed, and nothing else could have.
(The draw now lives in `parquet_sampling` and takes its sort from the Arrow-free `parquet_argsort`
tier, so the edge is gone — but the failure mode is unchanged and is exactly what
`tools/check_argsort_standalone.sh` was written to catch one tier up.)

So:

- **A `use` added to `parquet_random` is a design decision, not a detail.** Enforced by
  `check_parquet_random_stays_leaf` (`tools/check_source_conventions.py`), in two clauses: every
  project module reached, transitively, must itself reach nothing but **compiler-supplied** modules
  (`iso_fortran_env`, `iso_c_binding`, `ieee_arithmetic`, `omp_lib`); and every such module must
  appear in the `SRC` list of **every `tools/*.sh` that compiles `parquet_random`**. That second
  list is **derived by globbing**, not enumerated — an enumerated one goes stale in the direction
  that stops checking, and did: it named two scripts while `tools/check_random_ubsan.sh` was a
  third, still listing `src/parquet_settings_base.f90` (removed at the `parquet_sampling` split)
  and neither module `parquet_random` had gained since. `tools/check_exp_key.sh` is correctly
  outside that set and must stay outside it — it compiles `src/parquet_expkey.f90` ALONE, which is
  the whole reason the frozen transform can be swept across compilers, and a separate clause fails
  if it ever grows a `parquet_random` entry.
- **The rule is NOT "imports nothing outside `iso_fortran_env`", and that wording is a trap.**
  `parquet_settings_base` imports `omp_lib` inside `#ifdef _OPENMP` — and was the FIRST entry in
  `check_random_kernels.sh`'s `SRC` list for months, passing throughout, because that script
  compiles with no `-fopenmp` so the import never happens. The strict wording condemns a module with
  a demonstrated track record for a reason unrelated to the hazard, which is reaching
  `parquet_bindings` and hence Arrow. So the walk stops at compiler-supplied modules, not at
  `iso_fortran_env` alone.
- **The second clause exists because a module can be admissible and still break the scripts.** They
  are plain ordered compiles with no dependency resolver, so a missing `SRC` entry is a hard failure
  and a stale one is worse — the script then silently compiles a different set than the rule
  believes. Having the check derive the closure and compare it against both lists is what stops the
  rule and the scripts drifting apart.
- **If the FIRST clause fires, adding the module to the script's `SRC` list is the WRONG fix** — it
  makes the check pass while destroying the property it measures. Move whatever needed the import
  into `src/parquet_sampling.f90` instead. (If the *second* clause fires, adding to both `SRC` lists
  is exactly the right fix — that is what it is asking for.)
- **`parquet_sampling` no longer carries an Arrow link edge, and the way it lost one is the
  pattern to copy.** `pf_weighted_permutation` sorts its keys, and taking that sort from
  `parquet_sorting` used to bring `parquet_bindings` — and with it the whole C++ wrapper — into
  every consumer's build, for one specific over a `real64` array. The fix was not to duplicate a
  sorting algorithm but to split the tier: `parquet_argsort` holds `pf_argsort` over the six
  intrinsic types and reaches nothing but `parquet_settings_base`, so `use parquet_sampling` went
  from 24 compiled files to 8. **When a leaf needs one procedure from a heavyweight module, move
  the procedure down a tier rather than copying it or accepting the edge.**
- **The split is a dependency boundary, so it decides placement**: anything drawing a *number* goes
  in `parquet_random`, anything drawing from a *population* goes in `parquet_sampling`. A new
  procedure that needs a thread-count rule, a sort or a settings knob belongs in the latter by
  construction.
- **A sibling module sees only PUBLIC names**, so `parquet_sampling` reaches the generator through
  `pf_random_int_at`/`pf_random_key`/`pf_random_fill_draws` rather than the private
  `int_at_impl`/`key_from`/`fill_draws_i32` those wrap. Each is an exact pass-through; keep it that
  way, or a value changes with only the golden vectors to report it.

### Each `parquet_random` generic reads its OWN word space, and a missed tag is silent

One `(seed, stream)` pair names **four** independent sequences of 32-bit words, not one. Which
sequence a reader walks is a two-bit domain tag OR-ed into the block index — `DOM_REAL64`,
`DOM_REAL32`, `DOM_INT_NARROW`, `DOM_INT_WIDE` in `src/parquet_random.f90`. That is what makes two
values taken at different `(generic, draw)` coordinates independent; before it,
`pf_random_int_at(seed, i, 1, 6, 2)` was a deterministic function of `pf_random_at(seed, i)`.

**Adding a generic means deciding which space it reads, and adding a `random_block` call site means
naming a domain.** Every one of the 27 existing call sites carries an explicit tag, including the
`DOM_REAL64` ones where the tag is zero and could have been omitted — that is deliberate, so an
untagged site is visible by inspection rather than by measurement. **`DOM_REAL64` must stay zero**:
it is what keeps every uniform, every distribution and the reader's `sample_fraction=` mapping at
the values they have always returned.

**Two identities are exempt and must survive**: `pf_random_at` is the top 53 bits of
`pf_random_bits_at`, and `pf_random_exp_at` is `-log(1 - u)` for that same `u`. Both are contract
and both are asserted.

The failure mode — a reader left in the wrong space still returns uniform, in-range,
well-distributed values, and nothing announces it — is `feature_risks.md` **Risk-133**, which also
records the subtler half: the tag is safe only because no call site passes a block index above the
one its own draw addresses.

### Public numeric arguments: provide both int32 and int64 kinds

When adding a public procedure argument that holds a row count / size / index (any integer a
caller might naturally declare as a plain `INTEGER`), make it generic over **both**
`integer(int32)` and `integer(int64)`, following the existing `parquet_get_nrows_int32`/
`parquet_get_nrows_int64` overload pattern. An `integer(int64)`-only dummy forces callers
with a default-kind `INTEGER` variable into a `Type mismatch ... passed INTEGER(4) to
INTEGER(8)` compile error.

**The rule only applies when the value can legitimately exceed int32.** It exists so a caller is
never forced to widen a variable the library could have accepted as-is — not as a blanket style
requirement on every integer argument. An argument whose value is bounded below `huge(1_int32)` by
the format, by Arrow, or by the library's own guards stays a single default-kind `integer`, and
adding a second kind for it would be noise. `parquet_open_writer`'s `chunk_size` is the worked
example: it is a row-group row count, and a row group cannot hold more than int32 rows (see
"Guarding a hard Arrow int32-only ceiling"), so there is deliberately no `_int64` form and its
absence is not a defect. When declining the rule on these grounds, say so in the argument's own
doc-comment, so the next reader does not "fix" it.

Fortran constraint that shapes this: an *optional* dummy that differs only by kind cannot be
the sole disambiguator between specific procedures in a generic interface (a call omitting it
is ambiguous). So when such an argument is optional, carry the argument-absent case as its
own separate specific rather than an optional dummy — see `parquet_open_reader`'s split into
`parquet_open_reader_base` (no `nrows`) plus `parquet_open_reader_nrows_int32`/`_int64`
(required `nrows`), all under one generic interface.

For a *required* (non-optional) argument, this ambiguity constraint doesn't apply — Fortran can
disambiguate two specifics differing only by a required argument's kind without any special
handling, so just add the second kind-specific directly (no base/kind-suffixed split needed),
sharing one private `_impl` worker between the two (mirrors `add_col_qc_impl`'s existing
shared-worker pattern) — see `parquet_read_array_row_mode`'s `row_index` (12 specifics: 6 data
types x `integer(int32)`/`integer(int64)` row_index, each pair delegating to one
`parquet_read_<type>_array_row_mode_impl`).

### A new process-global parameter goes in `parquet_settings`, and a design doc must say so

**Any parameter that is global to the library and that a user could reasonably want to change belongs
in `src/parquet_settings.f90`** — not as a `parameter` buried in the module that happens to use it,
and not as a new argument threaded through a call chain. That module is the single place a program
looks to find out what the library will do, and the single place `parquet_print_settings`,
`parquet_reset_settings` and `parquet_settings_from_env` can reach.

**The admission test is one sentence: a setting may change how FAST, how LARGE or how LOUD the
library runs; it may never change what the library ANSWERS.** A program-wide default for something
like null ordering, quality-control enforcement or a numeric tolerance is deliberately absent and
must stay absent — it would make the same call return different results in different programs, with
nothing at the call site to hint at it. Anything expressible as an argument to a specific call (a
writer's `compression=`, a sort's `threads=`) belongs there instead, and an explicit argument always
wins over a setting.

Most internal constants fail that test and should stay where they are. Vocabulary (accepted token
lists), mathematical facts, format ceilings imposed by Parquet or Arrow, container implementation
details, and input-sanity bounds are **not** settings. The last of those is worth stating outright:
the `parquet_max_*` limits are published as read-only constants precisely because making them
settable would convert a guard against runaway input into a way to overflow a parser's own stack.

**Whenever a `feature_*.md` design or implementation document is written, it must contain a settings
analysis** — a short, explicit section answering: does this feature introduce any process-global
parameter, does each one pass the admission test, and if so what is its knob name, its default, its
validation and its environment variable. Say "none" when the answer is none; an absent section reads
as "not considered". This is a standing obligation on every future feature document, in the same way
tests and docs are standing obligations on every feature.

Five rules apply to a knob once it is admitted, and each exists because breaking it fails silently:

- **A round-trip test is not a test of a setting.** Set-then-get passes just as happily against a
  value that is stored and never read. Every knob needs three assertions: its default, its round
  trip, and an **observed effect with a negative control** — something measurable that differs
  between the default and the set value, *and* the same observation at the default showing the other
  outcome. See `feature_risks.md` Risk-41, and `tools/check_source_conventions.py`'s
  `check_settings_are_read` for the static half.
- **Every knob must be resettable, printable, documented and reachable from the environment.** Three
  of those four are enforced statically: `check_print_settings_documented` and
  `check_env_covers_every_setting` both take their knob list from `parquet_print_settings`' own
  printed rows, and `check_settings_are_read` works from the `cfg_*` declarations instead. Resettable
  is the one a lint check cannot see, so it is asserted by a test (`test_reset_all_knobs`) that sets
  every knob to a non-factory value first. The point of sharing the printed rows is that a new knob
  fails several checks at once rather than needing several people to remember several lists.
- **A knob whose value the C++ side needs is MIRRORED, and the mirror has rules**: values cross the
  `bind(C)` boundary already **resolved** (no tokens, no "0 means default" sentinels — Fortran
  decides, C++ obeys), one push function per group rather than one per knob so a reset cannot
  half-restore, and the C++ globals' initialisers must equal the Fortran defaults because they are
  what applies before the first push. See `feature_risks.md` Risk-42.
- **Do not add a second way to set the same thing.** A test-only `parquet_debug_set_*` override for
  a value that is now a real setting is a second writer, and the two can disagree; three such hooks
  were retired for exactly this reason. Observation hooks (`parquet_debug_get_*`) are fine and are
  usually how a knob's effect is asserted at all.
- **Renaming a public setting is a semantic-versioning event.** The `use parquet` surface is covered
  by the promise in README.md, so check whether the name appears in a *published* CHANGELOG section
  before renaming it — and never edit a published section, which records what that release actually
  shipped. A Fortran-side rename is free on the C++ side: `bind(C, name=...)` decouples the two, and
  `tools/check_bindc_boundary.py` keys on the bound name.

### Role-A MAMLs live in `table_types/`, not `schemas/`

Two directories hold `.maml` files, and they are read for opposite purposes. `schemas/` describes
files the library **writes** (and is globbed wholesale into `src/parquet_maml_base.f90` by
`tools/generate_parquet_maml.sh base`, so anything dropped in there becomes a compiled-in fixture).
`table_types/` holds **Role-A** schemas — the input to `tools/generate_user_table_code.py`, which
turns each one into a named `parquet_table` extension type. A Role-A MAML put in `schemas/` by
mistake is not an error and does not fail anything; it just silently becomes an embedded base
fixture it was never meant to be, and shows up in `parquet_maml_base`'s generated accessor list.

Both generators take `--dir=` with those names as defaults, so a downstream project gets the same
split without being forced into it.

### MAML fixture directory: `schemas/`

`.maml` example/fixture files live in `schemas/`. `tools/generate_parquet_maml.sh` accepts
`--dir=<name>`/`--dir <name>` (default `schemas`) so downstream projects embedding their own
MAML schemas aren't forced to match this project's convention — see
`doc/pages/utilities/embedding-maml-schemas.md` for the user-facing how-to.

### Reading MAML source files: shared helper, line-length limit, CRLF handling

Any code path that reads a `.maml` file's lines from disk (`parquet_load_maml_file`,
`parquet_load_qc_maml_file`, or a future one) should go through the shared
`parquet_read_maml_source_lines(filename, context, lines, nlines)` subroutine in
`parquet_metadata_maml.f90` rather than writing its own read loop. It handles two things a
hand-rolled loop easily misses:

- **A per-line length cap.** Lines are read into a fixed `character(len=maml_max_line_len)`
  buffer (`maml_max_line_len = 1024`, declared once in `parquet_metadata.f90` and host-associated
  to descendants — don't hardcode `1024` again elsewhere). A line longer than this is detected via
  non-advancing read + `size=`/`iostat_eor` (not silently truncated with `iostat == 0`, which was
  the original bug this helper fixes) and aborts with a clear message naming the offending line
  number, rather than letting a truncated `keywords:`/`protected_cols:` line validate and write
  incomplete metadata into the output file.
- **CRLF transparency.** A trailing `char(13)` (from a Windows-edited `.maml` file) is stripped
  before the line is returned — plain Fortran `trim()` does not remove it, so without this a
  CRLF file produces a confusing "invalid data_type"-style error for a value that looks visibly
  correct to a human reading the file.

If a future MAML-adjacent feature needs to read a `.maml`-like file's lines directly, reuse this
helper (or extend it) instead of duplicating the read loop — that's exactly the class of bug it
was introduced to close off project-wide.

### Error stop messages: include file/schema context

New `error stop` messages in the read/write/schema-building paths should append the relevant
file and, where applicable, schema/maml name using the existing helpers — `writer_context_suffix`
(`parquet_write.f90`), `reader_filename_suffix` (`parquet_read.f90`), `maml_name_suffix`
(`parquet_metadata.f90`) — rather than naming only the offending column/field, so a failure is
identifiable when several readers/writers/schemas are in play at once. These are only
meaningful once the reader/writer/schema knows its file/name (i.e. post-open), so the
guard-clause "…has not been opened" messages are exempt.

### Filter evaluation: a Null is UNKNOWN, a NaN is a VALUE — the two behave oppositely

`eval_filter_clause` (`parquet_wrapper.cpp`) gives a Null row `kUnknown` for every comparison and a
NaN row an ordinary `kTrue`/`kFalse`, and that difference is deliberate in both directions. IEEE
makes every comparison against a NaN false, so a NaN row is **excluded** by `>`/`>=`/`<`/`<=`/`==`
but **survives** `/=` and any negated comparison — precisely where a Null row does the opposite
(`kUnknown` negates to `kUnknown`). Both halves are load-bearing and neither is a rounding error to
be "harmonized":

- **Nullness is governed solely by `is_null`/`is_not_null`.** They are the only clauses that answer
  `kTrue`/`kFalse` for a Null row. `is_nan`/`is_not_nan` deliberately do **not** join them — a Null
  row is `kUnknown` for both, so `x is_not_nan` means "is a real number", not "is not a NaN,
  whatever else it may be". Making either one two-valued on Null would quietly let Null rows into
  filters that never mention nullness, which is the back door the Kleene design exists to close.
- **`is_nan`/`is_not_nan` are restricted to `FLOAT`/`DOUBLE`/`HALF_FLOAT`** and `error stop` on
  anything else. `DECIMAL*`/`UINT64` also reach the comparison arms as doubles but can never *hold*
  a NaN, so accepting them would answer a constant for what is almost certainly a mistyped column.
- **A bare `nan` comparison value is rejected; `inf` is not.** `parse_double_strict` is `strtod`, so
  both parse — but `x == nan` can only ever match nothing and `x /= nan` everything non-null.
- **`is_nan` is expressible without the operator**, as `not (x >= 0 or x < 0)` (every non-NaN real
  satisfies exactly one disjunct; a NaN neither; a Null is unknown for both). `test_filter.f90` uses
  that equivalence as an independent oracle for the operator — a good pattern to copy for any future
  operator that is sugar over the existing grammar, and a ready-made expression for exercising the
  NaN path of anything that reasons about clauses without evaluating them.

**The last point has a sharp consequence for the row-group statistics screen** — see the next
section, which states the rule as implemented.

### The row-group statistics screen: every uncertainty must DECLINE

`screen_row_groups` (`parquet_wrapper.cpp`) reads each row group's footer statistics and skips the
row groups a filter provably cannot match. It is the only thing in the reader whose failure mode is
a **silent wrong answer** rather than an abort: a wrongly pruned row group's rows simply never
appear, with nothing to notice. Six rules keep the failure direction at "prune nothing", and every
one of them is the kind a later simplification would delete:

- **Every gate returns `kScreenAnything` (`{may_true, may_false, may_unknown} = all true`), never a
  guess.** Statistics absent, an unusable ordering, an unsupported type, an unparseable literal —
  all decline. A declining leaf prunes nothing and, per the combinators, cannot make anything else
  prune either.
- **`AND`'s `may_true` is an over-approximation and must stay one.** `a.may_true && b.may_true`
  says "some row satisfies `a`, and some row satisfies `b`" — not necessarily the *same* row.
  Row-group statistics are per-column marginals with no joint information, so nothing better is
  available; tightening it is a bug.
- **For a `FLOAT`/`DOUBLE` leaf, `may_false` is unconditionally `nn > 0`, and `/=`'s `may_true` is
  too.** Parquet excludes NaN from min/max and records no NaN count, so the bounds can rule a NaN
  neither in nor out — and a NaN is an ordinary value that compares *false* (see the previous
  section), so it makes every comparison false while sitting outside `[min, max]`. `NOT` consumes
  `may_false`, so the ordering-derived form prunes row groups that do match: `{1.0, 2.0, NaN}` under
  `not (x > 0.5)` matches the NaN row while `min = 1.0 > 0.5` claims nothing can be false. Reachable
  without `is_nan` at all, since `not (x >= 0 or x < 0)` *is* `x is_nan`.
- **The screen walks the SAME postfix node list as `evaluate_nodes`**, with the same stack shape,
  and takes each leaf's family from the same Arrow schema expression the evaluator dispatches on.
  Drift between the two is the second-biggest risk after the rules themselves; keep them adjacent
  and keep the walks structurally identical.
- **Two guard pairs are individually redundant and jointly load-bearing**, exactly as
  `column_has_nulls_from_footer` records for its own: `is_stats_set()` + a null `statistics()`
  (removing both segfaults on `test/fixtures/no_stats.parquet`), and the `sort_order() ==
  SortOrder::UNKNOWN` check + the `sort_order()` SIGNED/UNSIGNED check a few lines below it
  (removing both mis-reads an unsigned column's bounds as signed). Do not delete either half on
  the strength of a coverage report. (Note: `ColumnDescriptor` has no `can_use_min_max()` method
  in any Parquet C++ release found on this machine — verified against versions 19, 22, 23, and 24;
  the first guard reads `sort_order()` directly instead.)
- **A row group's mask segment is all-false when it is pruned**, which is why nothing downstream
  needs to learn a new concept — a pruned row group simply *is* an empty one, which
  `row_group_effective_rows` and every row-group-scoped operation already handle.

Two structural notes for anyone extending this. `live_mask` is `filter_mask` restricted to the live
row groups' rows and is what `apply_row_transform` filters with, because every whole-column decode
now reads only those row groups; `filter_mask` stays the canonical full-length object everything
row-group-indexed uses. And **the unpruned path deliberately still calls `ReadColumn`** in
`get_single_chunk_array` rather than routing through `read_live_row_groups` for symmetry: measured
at 3.4% of a whole filtered read (reproducible to 0.1%), because `ReadTable` reconstructs a Table
and its schema per call. Do not "unify" those two branches without re-measuring.

Testing this needs both halves: an **A/B equality** against
`parquet_debug_set_disable_statistics_prescreen(1)` over the same fixture, *and* an assertion on
`parquet_debug_get_row_groups_pruned()` — equality alone passes just as happily against a screen
that never prunes. Both hooks are process-global, which is why `test/run_tester.f90` excludes the
`filter_screen` suite from its per-test parallelism.

### Guard mutating public procedures against being called twice

When adding a new type-bound procedure or public subroutine that mutates a `parquet_writer`/
`parquet_reader`/`parquet_schema`'s state, consider whether a caller reusing the same object
across two calls (a loop, a copy-paste mistake, a refactor) could silently corrupt state, leak a
resource, or crash instead of getting a clean error. Default to a check-before-mutate guard: test
the relevant flag/allocated-component first and `error stop` with a message like
`"<procedure>: already called for this <writer/row group/...>"` before any mutation happens,
rather than silently overwriting state or leaving the object in an inconsistent, only-later-
surfacing-as-a-crash condition. See `parquet_write_row_mask_impl`/`parquet_write_chunk_row_mask_impl`/
`parquet_new_row_group_impl` (`parquet_write.f90`) for the established pattern. Not every mutating
procedure needs this — e.g. a procedure whose second call is genuinely idempotent (rebuilds the
same state from the same inputs, like `parquet_validate_user_maml`) doesn't need a guard — but
default to adding one unless a call is provably idempotent.

### Implicit finalizers must never route through a path that can throw/abort

A `FINAL` procedure (e.g. `writer_finalize`/`reader_finalize`) can run at unpredictable points
(variable reuse via an `intent(out)` re-open, scope exit, an early `RETURN`) with no way for a
caller to see or handle a failure. Never have a finalizer call a close/cleanup path that performs
completeness or validity checks capable of throwing a C++ exception across the `extern "C"`
boundary (which crashes the whole process — see "`src/parquet_wrapper.cpp`: GCC vs Clang gcov
attribution"'s notes on uncaught exceptions) or issuing an `error stop`: an implicit finalizer
should always succeed silently, freeing/abandoning resources without validating the object's
completeness. If the normal close path (e.g. `close_parquet_writer`) has such checks, give the
finalizer its own dedicated "abandon" entry point that skips them entirely — see
`abandon_parquet_writer`/`writer_finalize` (`parquet_wrapper.cpp`/`parquet_write.f90`) for the
pattern — rather than trying to have the finalizer conditionally decide when it's "safe" to call
the real close. Apply the same pattern to any future finalizable type (e.g. a `parquet_reader`-side
completeness check, if one is ever added).

### A written column's nullability is a CONTRACT with the array beside it

Whether a column's Arrow field says `nullable` is decided in one place, `build_field`
(`parquet_wrapper.cpp`), and the rule differs by write path. Getting it wrong is not a cosmetic
metadata slip: it is `feature_risks.md` **Risk-82**, whose failure mode is a file whose definition
levels disagree with its own schema — which this library's own reader cannot see, because it answers
from the data's null count rather than the flag.

**Where the flag comes from.**

- **A whole-column write decides from the VALUES** (`has_any_null`): it has seen every one before it
  writes anything, so a column containing no Null is written non-nullable whether or not a mask was
  passed. Note `has_any_null` answers `false` for a null pointer, so "no mask at all" lands here too.
- **A streamed write decides from mask PRESENCE on the FIRST row group** (`resolve_chunk_nullability`)
  — nullable iff that row group passed an `is_valid` mask, whatever its entries were. It cannot use
  values: the field is fixed when the first row group locks the file's schema, long before the later
  row groups exist. Every later row group must then use the same masked/unmasked form, and a
  mismatch is a hard error in both directions.
- **A protected column is always non-nullable**, on every path. That is the only way to declare a
  *streamed* column of the always-nullable kinds null-free, and it is why C++ has a per-writer
  `protected_columns` registry at all — Fortran erases an all-`.true.` mask for the mask-carrying
  kinds, but temporal and `parquet_string_column` have no mask to erase.

**The always-nullable kinds are `date`, `time`, `timestamp` and `parquet_string_column`, and `date`
is the one a new rule will miss.** Their nulls live inside the element rather than in a caller's
mask, so "was a mask passed" cannot predict whether row group 7 holds a Null; they stay nullable
when streamed unless protected. **`date` does not reach `stash_temporal_column_chunk`** — it is
int32-backed, so `parquet_write_date_column_chunk` goes through the generic
`append_typed_column_chunk` and needs its `always_nullable` argument set explicitly. A rule written
for "temporal columns" that only touches the temporal helper silently excludes dates.
There are **five** chunk-write sites in total (`append_typed_column_chunk`,
`parquet_write_string_column_chunk`, `parquet_write_string_array_column_chunk`,
`parquet_write_string_column_chunk_buffers`, `stash_temporal_column_chunk`); a new one must call
`resolve_chunk_nullability` or it silently reverts to whatever default it passes.

**The safety invariant, and it is the whole reason the scheme is sound: a field declared
non-nullable must NEVER receive an array containing nulls.** It holds by construction — an absent
mask reaches the builder as a null `valid_bytes`, which cannot produce a null — and that is exactly
what the always-nullable exceptions protect.

**A field and its array must AGREE, and Arrow enforces it at close time from far away.**
`FixedSizeListBuilder` stamps its finished array with a type whose child field is nullable (it
derives the type from the value builder, which has no say), so declaring the child non-nullable in
`build_field` makes `arrow::Table::Validate()` reject the write with *"Column data for field N … is
inconsistent with schema"* — at `parquet_close_writer`, naming a field index rather than the call
that caused it. `align_array_to_field` restamps the array's type from the field's; it must be
applied **wherever a field and an array are stored together**, which is `append_column`'s two
branches plus every chunk site's `pending_chunk_arrays` assignment. Restamping is safe because the
difference is pure metadata, and it is a deliberate no-op when the types already match.

**The child field's name must stay `item`.** That is what Arrow's own
`FixedSizeListType(DataType)` constructor supplies (`arrow/type.h`), and it appears in the Parquet
schema's leaf paths — renaming it changes how every other tool addresses the column.

**And the general lesson, which is not about Arrow at all: when a parameter STOPS being ignored,
every call site that omitted it becomes a suspect.** `build_field`'s `nullable` argument was
discarded for vector columns for as long as the function existed, so
`parquet_append_string_array_column` simply never passed one — correct, until it wasn't. The moment
the argument became live, that call would have declared a non-nullable element field for a loop that
appends nulls. Grep every call site of a parameter whose meaning you have just changed, and check
what the defaulted ones now mean.

### Automatic BYTE_STREAM_SPLIT for float columns in the writer

`apply_float_byte_stream_split` (`parquet_wrapper.cpp`) is called at both places `WriterProperties`
get built (the first-row-group path in `parquet_finish_row_group`, and `close_parquet_writer`'s own
`WriteTable` path) and, for every `float32`/`float64` field, calls `disable_dictionary(name)` *and*
`encoding(name, Encoding::BYTE_STREAM_SPLIT)` on that same column. This is automatic and type-based
— not exposed as a public argument — because dictionary encoding rarely helps floating-point data
(samples are usually near-unique) while byte-stream-splitting each value's bytes across separate
per-position streams compresses substantially better under most codecs; every other column type is
left at the writer's normal defaults (dictionary enabled, no BSS).

**The `disable_dictionary` + `encoding(..., BYTE_STREAM_SPLIT)` pair must always be applied
together, on the same set of columns, never one without the other.** Confirmed directly from
`parquet/properties.h`: `Builder::encoding(path, type)`'s own doc comment states it "is only
applied if dictionary encoding is disabled" for that column — requesting BYTE_STREAM_SPLIT while
dictionary stays enabled for that column is a **silent no-op**, not an error, and produces a file
that still uses ordinary dictionary encoding despite the (ineffective) BSS request. If a future
change relocates or refactors this logic, keep both calls paired and keep calling
`apply_float_byte_stream_split` at **both** `WriterProperties::Builder` construction sites listed
above — the debug/test-only fixture writers elsewhere in this file (`parquet_debug_write_*`) are
deliberately not included, since they bypass the normal schema-driven writer path entirely.
Verified empirically (not just from the header comment) via a scratch program writing a
float32/float64/int32 file and inspecting `pyarrow.parquet.ParquetFile(...).metadata`'s
per-column `encodings`: the float columns report `('RLE', 'BYTE_STREAM_SPLIT')` with no
`RLE_DICTIONARY`, while the int32 column keeps `('PLAIN', 'RLE', 'RLE_DICTIONARY')` — this
library has no reader-side API to introspect a file's physical encoding, so `pyarrow` (or another
external tool) is the only way to confirm this end-to-end; a pure test-drive/Fortran test can only
confirm the *data* round-trips correctly, not which encoding was used to store it.

### A character ARRAY is trimmed on the way into a column; a character SCALAR is not

Every element of a `character(len=*)` array shares one declared length, so a shorter value is
blank-padded by Fortran and the padding cannot be what the caller meant. A `character(len=*)`
**scalar** is exactly as long as the caller wrote it. So:

- **Array arguments trim** — `parquet_column%set_all`/`%append_values`, `parquet_table`'s
  `%add_column`/`%set`/`%set_slice`, and `%set_element`'s *vector* form (whose `value(:)` is an
  array). `refill_string_store` (`src/parquet_columns_string.f90`) is where the rule is implemented
  and explained.
- **Scalar arguments do not** — `%set_at`'s scalar form, `%set_element` on a `PK_STRING` column, a
  row handle's `%set`.
- **`parquet_string_column`'s own API never trims by default**, because it takes bytes the caller
  controls exactly and offers explicit `trim=`/`strip=`.

Do not "harmonize" these into one behaviour: the asymmetry *is* the rule, and it is what makes
`%get` into a `character(len=:), allocatable` come back sized to the longest real value rather than
to whatever width the caller happened to declare. `%add_column` documented the trimming from 1.0.0
and did not do it until this was fixed, so the doc-comments now state the rule rather than just the
behaviour.

### Validity is per ELEMENT, and a vector row is not one bit

`parquet_column`'s validity API comes in a **row** form and an **element** form, and the storage has
always been `width * nrows` bits. Every query and mutation exists in both shapes — `is_null(i)` /
`is_null(i, e)`, `set_null(i)` / `set_null(i, e)`, `clear_null(i)` / `clear_null(i, e)` — with the
element index bounded by `width` (`check_element`), never a flattened `(row-1)*width + element`
position.

**The row and element forms are deliberately asymmetric where they differ, and that asymmetry is the
rule to preserve:**

- A row **QUERY** answers about the row as a whole: `is_null(i)` is `.true.` when **any** element of
  row `i` is null, and `row_validity` builds that summary. Costs O(width) with an early exit.
- A whole-row **MUTATION** acts on every element: `set_null(i)`, `clear_null(i)` and `append_nulls`
  mark the entire row. Naming only a row says the row is missing.
- **`modify_nulls=.false.` protects individual null ELEMENTS**, not whole rows: a vector row with one
  null element still has its other elements written.

Each operation acts at the granularity the caller named — that one sentence generates all three.

**Shapes must match on every paired API.** A rank-1 `values` takes a rank-1 `is_valid`; a rank-2
`values` takes a rank-2 `is_valid`, shaped `(width, nrows)`. This holds for `parquet_read_column`,
`parquet_write_column` (both always did), and now for the table's `%get`/`%col`/`%get_slice`/`%set`.
There is no rank-1 form for a vector column and no widening anywhere. The **standalone** mask APIs
are the deliberate exception, because they have no values to match: `%get_valid_mask` and
`%set_null(mask)` accept **either** rank, where rank-1 is the row summary ("which rows are
complete?") and rank-2 the true element state. Both are unambiguous because the mask is a required
argument there — do not "fix" that asymmetry.

**When walking the bitmap in bulk, iterate the SET BITS, not all 64 positions of a nonzero word.**
`row_validity`/`element_validity` use `trailz` + `ibclr`. Testing every position instead makes a
densely-null wide column cost `width` times more — measured at 2.7x on a width-16 column that is
half null, against 2.2x *faster* than the pre-element-null implementation with the bit-scan. The
zero-word skip is what keeps the null-free path free and must stay.

**Do not reintroduce widening in a new read or write path.** A per-element null read from a file is
stored as such (`set_validity` writes the whole mask in one pass rather than replaying `width*nrows`
setter calls), and a table write hands the element mask straight to the writer. The round-trip test
in `test/test_table.f90` (`test_element_null_round_trip`) is what catches a regression, in both
directions at once.

### Auto-threading: `omp_in_parallel()` picks a DEFAULT, and that is not the guard CLAUDE.md warns about

Two places decide on their own how many threads to use — `parallel_prefetch_ok`
(`parquet_tables_read.f90`, for the table's internally-parallel read) and `pf_sort_threads`
(`parquet_argsort_kernel.f90`, for every sort). **Both resolve to serial inside an OpenMP parallel
region**, and both do it with the same two lines:

```fortran
if (omp_get_max_threads() <= 1) return   ! or: n = 1
if (omp_in_parallel()) return            ! nested regions are the caller's business
```

This does **not** contradict "Never key a guard on `omp_in_parallel()` alone" below. That rule is
about a guard that *refuses* an operation, which under test-drive's own `!$omp parallel do` fires
across the entire suite. These refuse nothing — they choose a default, and an explicit request
(`threads=8`) is still honoured inside a parallel region. Keep the distinction when adding a third
such decision, and reuse `pf_sort_threads` rather than writing a fourth copy of the rule:
`omp_get_max_threads()` reads an ICV, not the current team size, so inside an 8-thread region it
answers 8 and a missing check means 8x8 threads.

**Every thread count the library resolves must also be clamped to `omp_get_num_procs()`, and that
clamp has ONE home: `parquet_clamp_to_affinity` (`src/parquet_settings_base.f90`).** Four resolvers
reach it — the sort's `resolve_thread_count`, `pf_sort_threads` and the bulk random draws through
`parquet_auto_thread_count`, `prefetch_thread_count` and `parquet_string_threads` — and a fifth must
call it rather than copy it. Three things make this worth a rule:

- **`omp_get_max_threads()` is NOT reduced by `OMP_PROC_BIND` + `OMP_PLACES=cores`; `omp_get_num_procs()`
  is.** A process bound before `main` reports 64 from the first and 2 from the second, so a resolver
  reading only the ICV opens 64 threads on 2 processors and time-shares them, which is slower than
  not threading at all. Prefetch and the string bulk paths shipped in exactly that state, unclamped
  and unwarned, while sorting had clamped for months — because the clamp had been written where it
  was needed rather than where it belonged.
- **The warning must fire from the CLAMP, not from the operation.** It once lived in
  `resolve_thread_count` alone, which the automatic path reaches *after* `pf_sort_threads` has
  already clamped silently — so the warning was unreachable for the one job it existed to catch, a
  program with `OMP_NUM_THREADS=64`, no explicit `threads=`, and nothing said. A clamp that is
  silent in one place and loud in another cannot be reasoned about; make the clamp itself the only
  thing that speaks.
- **A clamp is untestable without an override, and its tests are then VACUOUS rather than absent.**
  A process cannot narrow its own affinity after starting, and on an ordinary machine
  `omp_get_max_threads()` and `omp_get_num_procs()` agree — so every assertion about the clamp holds
  just as well with the clamp deleted. Confirmed by mutation. `parquet_debug_set_affinity_procs`
  exists for this, with `parquet_debug_reset_affinity_warning` beside it because a once-per-process
  message is a single-shot observable that a negative control has to be able to re-arm.

### `parquet_table` concurrency: one file owns the OpenMP plumbing, and guards key on OWNERSHIP

`src/parquet_tables_parallel.f90` holds the table's lock, the append/read counters and the shared
refusal every structural mutation goes through, so that `#ifdef _OPENMP` and `use omp_lib` appear in
exactly one file. The two exceptions are `unsafe_first_touch`/`record_open_thread`, which stayed in
`parquet_tables_read.f90` next to the materialization path they guard. All three implement the SAME
ownership test and must agree: *a table this very thread opened inside the current parallel region is
thread-private and exempt; anything else may be shared.*

Rules a change here must not break:

- **Never key a guard on `omp_in_parallel()` alone.** test-drive runs its own tests inside
  `!$omp parallel do`, so that fires suite-wide (see "Tests run concurrently"). Ownership is the
  precise question, and refusing a thread-private table would make the whole slice regime unusable.
- **`table_append_table` and `table_append_row` each take the lock exactly once and then call
  `append_table_worker`; neither calls the other.** An OpenMP simple lock is not recursive, so a
  second acquisition on one thread deadlocks rather than failing to build. A new internal caller
  goes to the worker.
- **A lock is a HANDLE, not a value.** `%clone` builds a fresh one (`clone_new_cache`), never a copy
  of the source's, and `table_finalize` destroys it — via `table_destroy_lock`, which validates
  nothing and cannot abort, because a finalizer must always succeed silently.
- **The read path stays free of atomics.** `table_check_no_append` (one atomic read) sits in
  `table_resolve`, the single choke point every value accessor goes through; the `readers_active`
  counter is taken only around the *long* windows (a lazy first touch, and `materialize_marked`),
  never around a resident read or a per-element accessor. Two atomics per cell would dominate a
  `%get_element` loop. That asymmetry is deliberate and is documented on the cache fields.
- **`table_resolve(..., writing=.true.)` is how a write declares itself**, which is what gives every
  generated `%set`/`%set_element` specific the string-column rule from one place. A new write
  specific inherits it by copying its neighbour's call.
- **Validity is allocated lazily, so the FIRST null races** — guarded at the table layer (where
  thread ownership is reachable), never in `parquet_columns`, which is a standalone module with no
  thread knowledge. `%ensure_validity` is the escape hatch. Three dispatch classes, and only two
  can race: bitmap kinds (`ensure_bitmap`), string kinds (`parquet_string_column`'s own
  `ensure_validity_cap`), and temporal kinds — which allocate nothing and must NOT be refused.
- **The internally-parallel `%prefetch` gives each thread its own reader** and is gated by
  `parallel_prefetch_ok`. Every clause there is a correctness or cost rule, not a tuning knob; the
  sharpest is that an **unseeded `sample_fraction=` would make each per-thread reader draw a
  different subset**, so columns read by different threads would hold different rows — a silent
  wrong answer. Do not relax that gate without re-reading its own comment.
- **Arrow's own per-column threading is left enabled inside that region.** Measured both ways on a
  24-column x 900k-row file with 8 OpenMP threads: 0.037-0.040 s nested, 0.046-0.047 s with
  `use_threads=.false.`. Nesting the two is faster, so the "one level of parallelism only" instinct
  is wrong here. Re-measure before changing it.
- **Disjoint ROWS are not disjoint BITS.** The one shipped concurrency bug found this way came from
  reasoning that each thread owned its own row group, so its writes could not collide.
  `parquet_column`'s validity is a bit-packed `integer(int64)` map — `parquet_validity_block_bits`
  elements to a block — and updating a block is a read-modify-write, so the threads filling adjacent
  row groups both read, modify and write the block their boundary falls in. One update is lost, the
  column still validates, and a row's null flag is simply wrong. **Rows, elements and bits are three
  granularities, and only the last is what a read-modify-write actually touches**; a row-group size
  is chosen for I/O and has no reason to be a multiple of 64, so the collision is the normal case
  rather than a corner. `feature_risks.md` Risk-64 has the fix (trim each range to whole blocks, and
  put only the ragged ends in a `critical`) and the measurement that ruled out simply serialising the
  paste. **Corollary:** when a sibling module needs an internal layout fact in order to be *correct*,
  publish the constant rather than copy it — `parquet_columns` exports
  `parquet_validity_block_bits` for exactly this caller, because a second copy of that number could
  drift with nothing to report it.
- **Every new guard needs a NEGATIVE control**, not just an error scenario. A guard that fires
  unconditionally passes every abort test ever written for it while breaking the permitted case;
  `test_table_private_mutation_allowed` (`test/test_openmp.f90`) is the pattern.

### A `parquet_table` pointer does not survive a ROW-structural mutation

`%col` hands back a live pointer into a column's storage, and `%filter_rows`, `%sort_by`, `%top_n`,
`%delete_rows`, `%truncate`, `%append` and `%append_null_rows` all reallocate that storage
(the rebuilds — `delete_by_mask`, `reindex`, `gather` — exact-fit, while `append` grows
geometrically and so may not reallocate at all on a given call; it bumps the generation counter
regardless, so a pointer is to be treated as dead either way). A pointer taken
before one of them therefore points at freed memory afterwards, and **Fortran offers no way to
detect this** — the code compiles, and usually appears to work.

Two consequences for future work here:

- **Any new mutation that changes the row set inherits this**, so it belongs in
  `parquet_tables_rowmutate.f90` next to the others, and its doc-comment should say it detaches.
  The file/`%col` split (`..._mutate.f90` never changes the row set, `..._rowmutate.f90` always
  does) is what keeps the rule checkable by looking at which file a procedure is in.
- **The file split tracks the ROW SET, not pointer stability, and `%compact`/`%reserve` are the
  two places that differ.** Everywhere else the two coincide, which is why the split reads as
  though it decided both: a row-structural change reallocates storage, so it detaches *and*
  invalidates every pointer, while nothing in `..._mutate.f90` did either. `%compact` and
  `%reserve` reallocate storage *without* changing the row set — so they live in
  `..._mutate.f90` (they detach nothing, and a column not yet read is still readable afterwards)
  and yet they do invalidate every outstanding `%col` pointer and row handle. So: **pointer
  invalidation is the union of "changes the row set" and "reallocates storage"; only the first
  half decides which file a procedure goes in.** A new procedure in `..._mutate.f90` that
  reallocates must say so in its doc-comment, and should advance `%generation()` only when it
  actually reallocated — that counter is the documented way a caller finds out whether its
  pointer died, so bumping it unconditionally forces a needless re-fetch on every call that
  changed nothing.
- **A row-structural mutation skips a column that is not resident** (`table_mutable_column`)
  rather than refusing to run, which is what lets a lazy table drop rows without first reading
  every column it has. The skipped column is then unreadable for good, and the detach guard
  (`table_check_not_detached`) is the only thing that reports it — so every path that would read
  from the file after a mutation must run that guard. There are five today (`table_touch`,
  `table_resolve_width`, `materialize_marked`, `table_reload`, `table_row_group_bounds`);
  a sixth that forgets it will read through a deallocated reader.

**"Detached" means "had a file and can no longer read it", never simply "was mutated".** A table
built by `parquet_new_table` has no file to lose, so growing or reordering it must leave
`%is_detached` answering `.false.` — otherwise every from-scratch table would report itself
detached the moment it was filled. `table_detach` therefore only sets the flag when
`cache%file_backed` is still true, and never clears it.

### New `parquet_table` state goes on the CACHE — never as an allocatable component of the type

`parquet_table` itself is deliberately **five scalars and one pointer, with no allocatable
components at all**; every piece of real state (`reader`, `cols`, `rg_bounds`, `source_file`, the
read-time transform) lives in `parquet_table_cache`, behind that pointer. The file header of
`parquet_tables_lifecycle.f90` states this for the column store and gives one reason (a `%col`
pointer must outlive the dummy argument). There is a second, sharper reason, and it applies to
*any* component, not just the column store:

**`parquet_table` is FINALIZABLE, so every allocatable component it gains makes the compiler
generate a deeper recursive walk for its `intent(out)` entry and its `FINAL` — and this project has
three confirmed compiler bugs in exactly that machinery on exactly this type.** Two are documented
above and in `parquet_tables_lifecycle.f90` (gfortran leaving an OpenMP `private()` copy
uninitialized; `%detached` surviving an `intent(out)` reset). The third: hanging a
`type(parquet_schema), allocatable` off `parquet_table` — for the composed read-time transform,
which really is per-table state — **segfaulted ifx inside its own runtime**, in a block-local table
opened inside an `!$omp parallel do`, at the `intent(out)` entry of `parquet_open_table`. The
backtrace named no library code at all: unnamed RTL frames with a self-recursive PC, bottoming out
in libc, i.e. the runtime's own nested-derived-type descriptor walker following a bad descriptor
into `free()`. `parquet_schema` is the deep one (`maml` + `cinfo` + `metadata`, each holding
allocatable arrays of derived types with their own allocatable components), but the rule is not
about that type specifically.

So: **put new table-level state on `parquet_table_cache`**, where it costs nothing structurally —
the cache is a plain, non-finalizable type reached through a pointer, freshly `allocate`d per open,
so its default initializers are reliable and nothing walks it on procedure entry. Reserve
`parquet_table`'s own body for plain scalars (`regime`, `row_lo`, `row_hi`, `row_count`,
`detached`). Note the ordering consequence in `open_table_impl`: anything stored on the cache has to
be assigned *after* `allocate(table%cache)`, not before.

### Assembling a `parquet_column` from pieces: preallocate and `%paste`

**A column's storage grows GEOMETRICALLY, and a REBUILD is exact-fit — the two halves are
different procedures and it is worth knowing which you are in.** `grow_storage`
(`parquet_columns_mutate.f90`, generated) is two lines over `ensure_capacity`, whose rule is
`newcap = max(need_rows, self%cap + self%cap/2_int64)` — 1.5x — so `%append` in a loop is amortised
O(1) per row, and building a column a row at a time is a reasonable thing to do. `gather_storage`
and the other rebuilds behind `%filter_rows`/`%sort_by`/`%top_n`/`%delete_rows`/`%truncate`
allocate **exact-fit** and say so in their own comment (*"Only grow_storage ever creates slack"*),
which is what makes `%shrink_to_fit` — and `parquet_table`'s `%compact` — a no-op on any column
that has not been appended to. The price of the growth half is that an appended-to column can hold
up to 1.5x the storage its rows need. `feature_risks.md` **Risk-67** records the consequence that
matters most — `size(storage)` is `cap`, not `nrows`, so every read must be bounded by `1:nrows`.

**When the final row count is known before the pieces are, `init` the column once at full size and
`%paste` each piece into place.** Not because appending is quadratic — it is not — but because
`%paste` avoids the intermediate copies altogether rather than merely amortising them, and it
allocates once. `%paste(src, at [, from] [, count])` overwrites an existing row
range without reallocating or changing `nrows`; `from`/`count` copy a sub-range of the source, so
trimming a piece needs no `keep` mask and no `%delete_by_mask` either. `materialize_slice`
(`parquet_tables_read.f90`) is the worked example. Two things to know before using it:

- **`%paste` REPLACES the pasted range's validity, it does not merge it** (`%append`'s rule, where
  the destination rows are always fresh, is merge). A valid source element clears a null the
  destination already had. Preserve this in any change: merging instead would leave a stale null
  sitting on top of a value that really was read, and the table path cannot catch it, because there
  the destination is always freshly `init`'d — only `test/test_columns.f90` covers that difference.
- **The string kinds are excluded and abort.** A `parquet_string_column` is a packed
  variable-length store with no fixed row slots, so it cannot be overwritten in place — but it also
  does not need to be, because `ensure_offsets_cap`/`ensure_data_cap`/`ensure_validity_cap`
  (`parquet_strings.f90`) already grow it **geometrically** (1.5x). Keep the grow-and-append shape
  for those two kinds, and don't "fix" the exclusion.

If a future kind gains its own storage, decide which of these two shapes it has before adding it to
the paste path.

### `parquet_column`'s TYPED accessor tier: never reach storage through a binding

**`parquet_tables` must never call a type-bound procedure on a `parquet_column`.** Every per-cell
path reaches storage through the `parquet_column_*` generics that `parquet_columns` exports —
`parquet_column_get_at`, `_set_at`, `_get_elem`, `_set_elem`, `_data_ptr`, `_string_column`,
`_is_null`, `_set_null`, `_clear_null` — never through `%values%get_at(...)` or any sibling. The
generics are public and the specifics beneath them are private; `src/parquet.f90` privatises all
nine again, so none of it reaches a `use parquet` program and the public API is unchanged.

**Why, in one paragraph, because the reason is not visible in the source.** When ifx passes a
`type(parquet_column)` actual to a `class(parquet_column)` dummy whose callee is in another
compilation unit, it constructs the runtime class descriptor: one full type-descriptor record per
allocatable component — 21 records, **178 stores** — emitted as straight-line code in the **caller's
prologue**, unconditionally, ahead of any branch. A `select case` arm that never executes still pays
it. Measured at **~35 ns per call**, which was **78%** of what a resolved-handle `%get(i, value)`
cost; removing it took that read from **48.24 ns to 14.38 (3.36x)** and the whole table layer's
overhead above a raw storage read from 43.10 ns to 9.34. gfortran never emitted the block and still
gained **1.23-1.32x**, because the re-homing takes a polymorphic dummy off the hot path on any
compiler. Three conditions are jointly necessary: a non-polymorphic actual, a callee in a separate
compilation unit, and a type carrying deep deallocation/finalisation information.

**The shape, and the one arrow that must not be flipped.** The implementation lives at the `type`
end and the binding is a one-line forwarder onto it:

```fortran
module procedure parquet_column_get_at_f64          ! the IMPLEMENTATION, type(parquet_column)
    call parquet_column_check_kind(col, PK_FLOAT64, "get_at")
    value = col%f64(i)
end procedure parquet_column_get_at_f64
!
module procedure get_at_f64                          ! the binding, now a forwarder
    call parquet_column_get_at_f64(self, i, value)   ! class -> type: legal, free
end procedure get_at_f64
```

A `class` actual passed to a `type` dummy costs nothing; a `type` actual passed to a `class` dummy
builds the descriptor. **Writing it the other way round — a typed wrapper that calls the binding —
relocates the conversion into the wrapper and buys exactly zero**, and the symptom is a measurement
that refuses to move rather than anything failing.

Four rules for anyone working here:

- **Adding a per-cell accessor means adding BOTH halves** — a typed specific under the right generic
  (in `tools/generate_parquet_columns.py`, which emits the interfaces, the bodies and the
  forwarders) and a `private ::` line in `src/parquet.f90` if it introduces a new generic name.
- **A typed body must call the TYPED guards** (`parquet_column_check_kind`/`_check_index`/
  `_check_element`/`_check_width`) and no `class(parquet_column)`-dummy helper at all. The guards
  are not the only such helper: `ensure_bitmap` had to be converted to a `type` dummy for exactly
  this reason, because the typed `set_null` forms call it. Check what a new body reaches for.
- **`check_no_type_bound_column_access` (`tools/check_source_conventions.py`) is the enforcement**,
  and it covers all three shapes above. **This is deliberately NOT a `feature_risks.md` entry**:
  that register is for properties whose breach is a wrong answer, and a reintroduced binding call
  answers correctly and merely costs 35 ns — so a static check, not a risk entry, is its home.
- **Keep the tier even if a future ifx stops emitting the block.** It costs gfortran nothing (it
  gains, in fact), and re-flattening it would re-expose the library to the next compiler that makes
  the same codegen choice. The one-command check is `objdump -dr --no-show-raw-insn <obj> | awk
  '/<parquet_tables_mp_col_fetch_f64_>:/,/^$/' | grep -c 'R_X86_64.*\.bss'`, which must read **0**.

**Scope deliberately stops at per-cell.** `mat_*`/`matchunk_*`, `add_column_*`, `set_arr_*` and the
once-per-table procedures still convert, and that is fine — the cost is per call, not proportional
to the work, so it amortises away on a per-column path. A library-wide sweep went 262 -> 138
procedures carrying the block, and everything left is per-column or cold. `parquet_sorting` is
excluded on the same grounds (it binds a sort key once per column). Widening scope needs a
measurement, not an assumption.

### `parquet_string_column`'s typed tier: the same rule, and why concurrency makes it worse

**`parquet_columns` must never call a type-bound procedure on a column's `str` component.** A
`parquet_column`'s string storage is `type(parquet_string_column), allocatable :: str`, so
`col%str%is_null(i)` hands a non-polymorphic actual to a `class` passed-object dummy — the same
conversion the section above removes one type further up. Every access goes through the
`parquet_string_column_*` names `parquet_strings` exports instead. They are public there so
`parquet_columns` can reach them, and `src/parquet.f90` privatises every one again, so the
`use parquet` surface is unchanged. `check_no_type_bound_string_column_access`
(`tools/check_source_conventions.py`) is the enforcement, and it covers both halves: no
`%str%<binding>(...)` in `parquet_columns*`, and no binding call on a `type` dummy inside the tier
itself.

**The measurement that widened the scope, since the section above says a measurement is what it
takes.** `parquet_string_column` has three allocatable components, so the descriptor block here is
three records and **24 stores** rather than 21 and 178 — about a seventh of the per-call cost, which
is why a serial reading of it says "not worth it". That reading is wrong, and the reason is *where*
the stores go rather than how many there are: **ifx emits the block into `.bss`, not onto the
stack**, so it is one set of process-global cache lines that every thread writes on every call.

```
                     `%has_nulls` on one 200000-row date column, 8 rounds
    threads     2        8       16       64      384
    before   0.07 s   3.28 s   8.93 s   16.4 s   38.9 s      <- work per thread is CONSTANT
    after    0.06 s   0.07 s   0.06 s   0.10 s    0.27 s
```

Two things are worth reading off that table. The scan is **read-only and touches no shared state of
its own**, so nothing in the source suggests it should scale at all badly — the contention is
entirely in code the compiler emitted. And the block is emitted **ahead of the `select case` that
picks the kind**, so a `date` column pays for the string arm in full without ever entering it. The
whole `fpm test` run went from **39 s wall / 93 min CPU to 12 s / 9 min** on the change to one
procedure, before the rest of the sweep.

Practical notes:

- **`perf` names the symptom precisely and the cause not at all**: 94% of cycles sit in
  `parquet_column_is_null_row`, and annotation puts them on the `movq $0x4e0,...(%rip)` stores in
  its prologue. `nm` is what settles it — a descriptor temporary shows as a **`b` (local `.bss`)**
  `var$NNN` symbol, so it is shared; a stack temporary has no symbol at all.
- **No compiler flag avoids it.** `-auto`, `-assume recursion`, `-standard-semantics`, `-O1` and
  `-O3` all emit the identical 30 static descriptors (ifx 2026.1.1). The fix has to be in the source.
- **The shape is the same as the section above and the arrow must not be flipped**: the
  implementation lives at the `type` end and the binding is a one-line forwarder onto it. A typed
  wrapper calling the binding relocates the block into the wrapper and buys nothing.
- **A private module helper of `parquet_strings` should simply take a `type` dummy.** Six did not
  (`reindex_apply`, `delete_by_mask_serial`/`_parallel`, `gather_apply_serial`, …) and each one put
  the block back inside the typed procedure that called it. None was a binding target, so dropping
  the `class` cost nothing.
- **Scope, and what is deliberately left.** Three procedures still carry the *`parquet_column`*
  block — `deep_copy`, `move_from` and `clear`, all through `out%clear()`/`out%init()` on a `type`
  dummy. They are once per column, which is the boundary the section above draws, and no
  measurement has been taken that moves them across it.
- The one-command check, per procedure:
  `objdump -d <binary> | awk '/<parquet_columns_mp_parquet_column_is_null_row_>:/,/^$/' | grep -c 'var\$'`,
  which must read **0**.

### A `parquet_schema` built in code must be parsed before anything reads its fields

`schema%init` + `schema%add_field` build the schema's MAML **text** only; `schema%cinfo` stays
unpopulated until `parquet_parse_maml(schema)` runs. Calling `%get_num_fields`/`%get_field_name`/
`%is_column_set` before that reads uninitialized state — in a loop bounded by `%get_num_fields()` this
becomes a runaway allocation and an **OOM kill**, with no error message and nothing pointing at the
schema. Any new procedure that walks a caller-supplied schema should therefore guard with
`if (.not. schema%is_parsed()) error stop "<procedure>: this schema has not been parsed; call
parquet_parse_maml(schema) after building it with %init/%add_field"` before touching `%cinfo`. Note
`%is_parsed()` is the correct check, not `%is_init()` — the latter is `.true.` for exactly the
unparsed from-scratch schema this guard exists to catch.

## Element-domain modules (`parquet_strings`, `parquet_temporal`)

### The `parquet_strings` module

`src/parquet_strings.f90` is a **near**-independent module (`use parquet_strings`) providing
`parquet_string_column` (Arrow-LargeUtf8-style
offsets+data+bit-packed-validity string storage) and `parquet_string` (a non-owning handle to
one element). User guide: `doc/pages/types/string-columns.md`.

- **"Independent" is a direction, not a fact, and the exceptions are enumerated.** It depends on
  `iso_fortran_env`/`iso_c_binding`, and on exactly two things beyond them: `parquet_settings` (for
  `parquet_output_is_suppressed`, because `verbosity="silent"` governs solicited output wherever it
  lives, and for `parquet_get_string_threads`, because a thread cap is a setting for the same reason
  every other thread cap is), and `omp_lib` under `#ifdef _OPENMP`, since its bulk rebuilds thread
  internally. **Adding a third dependency is a decision, not a detail** — the value of this module
  being reachable without the Arrow/Parquet C++ stack is what the rule protects, and each of the two
  exceptions above was argued before it was taken. Consequence worth knowing when working here:
  because the module reaches no `bind(C)` surface at all, the C++-side debug-hook convention is
  unavailable to it, which is why its test hooks are public Fortran procedures (see "A Fortran-side
  debug hook has to be PUBLIC, so prefer a C++ one").
- **The module is `parquet_strings` (plural) on purpose.** A module and a type cannot share a
  name in gfortran (`public :: parquet_string` binds to the module, and the type declaration
  then conflicts). The user-facing *type* is `parquet_string`, so the *module* had to differ —
  do not "fix" the plural back to `parquet_string`.
- **It is wired into the library.** `parquet_string_column` is a specific of
  `parquet_write_column`/`parquet_write_column_chunk`/`parquet_read_column`/
  `parquet_read_column_chunk`, going through new C++ entry points that pass offsets+data+validity
  directly (`parquet_append_string_column_buffers` write-side, the buffer-fill counterpart
  read-side — see `parquet_bindings.f90`) rather than the legacy fixed-width, space-padded block
  (`parquet_append_string_column`/`parquet_read_string_column`) every other string path still
  uses. This write path does **not** trim (the column stores bytes verbatim by default; the
  padded path trims because padding is indistinguishable from real trailing spaces). Scope is
  scalar 1-D string columns only — there is no vector/matrix `parquet_string_column` specific;
  vector/matrix string columns stay on the legacy padded path. The two interop hooks
  `raw_buffers` (export c_loc pointers for a writer) and `append_buffers` (bulk-append one row
  group from C buffers, int32/int64 offsets + validity merge) are what this integration is built
  on.
- **`allow_null=.true.` on `get`/`to_string` returns an empty string, not unallocated** — see the
  gfortran note in "Compiler & language gotchas" below.
- **A struct-nested leaf read through the compact buffer-handoff path needs its validity bitmap's
  element offset threaded through explicitly — the offsets/data buffers do not need this.**
  `extract_string_buffers` (`parquet_wrapper.cpp`) reports the source Arrow array's own
  `data()->offset` as an extra out-argument (`validity_offset` in `parquet_bindings.f90`'s
  `parquet_read_string_column_buffers`/`_chunk_buffers`), and `append_buffers` (this module) takes
  a matching optional `validity_offset_bits` argument to start its bit-walk at the right bit —
  because Arrow never pre-rebases a validity bitmap for a sliced array the way it does the
  offsets/data buffers (`raw_value_offsets()[0]` already accounts for slicing on those two).
  Precondition for any future non-scalar `parquet_string_column` specific (a vector/matrix column,
  or any new call site reached through `unwrap_struct_path`'s struct-path resolution): don't drop
  this argument or assume a fresh `offsets(1)==0` guard alone is sufficient — a sliced source whose
  removed leading elements are all empty strings passes that guard while still needing the
  validity offset to avoid misaligned nulls.
- **`get_single_chunk_array`'s whole-column struct-path result is not retained by `column_cache`
  the way a plain column's result is — anything that returns a raw pointer into it across the
  `bind(C)` boundary must pin it itself.** `column_cache` only ever stores the *pre*-
  `unwrap_struct_path` array; a dotted struct-field path additionally builds a fresh, uncached
  array (a new combined-validity buffer) on every call. `parquet_read_string_column_buffers` learned
  this the hard way (a confirmed, 100%-reproducible use-after-free: every row of a struct-nested
  string leaf read via the compact `parquet_string_column` path came back `Null`, because the
  freshly built validity buffer was freed the instant that function returned to Fortran, before
  Fortran's `append_buffers` read the pointer) — fixed by pinning the returned array in
  `ParquetReaderHandle::last_whole_column_buffers_array`, mirroring `last_chunk_buffers_array`'s
  existing pattern for the row-group-scoped chunk read. Any future whole-column function that hands
  a raw buffer pointer back across the `bind(C)` boundary for a column reachable via a struct path
  must pin its array the same way — don't assume `column_cache` alone covers it.

### The `parquet_temporal` module (date/time/timestamp)

`src/parquet_temporal.f90` provides `parquet_date`/`parquet_time`/`parquet_timestamp` — one
element each (unlike `parquet_string_column` above, which owns a whole column) — fully wired
into `parquet_read_column`/`parquet_write_column` and every chunked/row-mode/element-mode
counterpart. User guide: `doc/pages/types/date-time.md`.

- **Domain-grouped module naming, not one-module-per-type — `parquet_temporal` is the precedent
  for future sibling modules.** The name groups `parquet_date`/`parquet_time`/`parquet_timestamp`
  under their shared *domain* rather than any single type, leaving an obviously-parallel name for a
  future `parquet_map`/`parquet_list` module (Parquet `MAP`/variable-length `LIST` support — see
  CONTRIBUTING.md's "Features considered but not implemented") to grow into, so this one module
  never accumulates every future element type. Follow the same pattern: one module per *domain* of
  related types, named after the domain (`temporal`, `map`, `list`), not after any single type
  inside it.
- **These three types carry their own null state — no `is_valid=`/`null_value=` argument
  anywhere on their read/write path, unlike every other supported type.** A default-initialized
  element is null; write gathers validity from the elements themselves; a null-containing column
  reads without the error-on-Null the numeric/string readers apply by default. This is a
  deliberate, documented deviation (see `doc/pages/types/date-time.md`'s "Null values are part of the
  element" section and `supported-data-types.md`'s callout in its own "Null values" section) —
  not an oversight to bring in line with the rest of the library. A future `parquet_map`/
  `parquet_list` module should make its own considered choice here rather than assuming either
  convention by default.
- **Parquet's physical format has no seconds-resolution `TIME`/`TIMESTAMP` encoding at all**
  (only milliseconds/microseconds/nanoseconds) **and no `DATE64` physical representation**
  (`DATE` requires an `int32` day count) — confirmed empirically, not just from the spec: even
  with `ArrowWriterProperties::store_schema()`, Arrow's writer silently coerces a `SECOND`-unit
  `TIME`/`TIMESTAMP` array to `MILLI` on write, and a `date64()` array is always coerced to
  `date32()`. A MAML `time[s]`/`timestamp[s]` token is therefore rejected at `add_field`/parse
  time (`error stop`, not a silently-wrong stored unit) — see `apply_temporal_unit_token` in
  `parquet_metadata.f90`. `parquet_unit_seconds` still exists as a constant, but only for
  `set_unix`/`to_unix` (Unix-time interop), never as a file column's own declared/stored unit.
  The `DATE64` decode branch in `parquet_wrapper.cpp`'s `convert_date_values` is kept as
  defensive dead code (in case a future Arrow/Parquet version changes this) and `GCOVR_EXCL`'d
  rather than chased with an unbuildable fixture.
- **`civil_from_days`'s final `if (m <= 2) y = y + 1` line is easy to drop when hand-transcribing
  or re-deriving this algorithm** (e.g. to compute an expected civil date for a boundary-value test
  by hand/in a scratch script) — Howard Hinnant's algorithm computes a year-of-era relative to a
  March-based year, and this trailing correction is what shifts a January/February result back
  onto the actual calendar year; omitting it silently produces a year that is off by exactly one
  for any date whose month is January or February (confirmed by re-deriving the function from
  `parquet_temporal.f90`'s source without this line and getting a consistent one-year error only
  on Jan/Feb dates, e.g. `civil_from_days(0)` coming out as `1969-01-01` instead of `1970-01-01`).
  If you ever need to independently re-verify a `days_from_civil`/`civil_from_days` boundary value
  outside the Fortran source (by hand or in another language), re-read both functions' full bodies
  in `parquet_temporal.f90` first rather than reconstructing them from memory, and sanity-check the
  result via a *round-trip* (`days_from_civil(civil_from_days(z)) == z`) rather than trusting a
  single-direction computation.
- **Legacy `INT96` timestamp test fixtures**: `enable_deprecated_int96_timestamps()` is a method
  on `parquet::ArrowWriterProperties::Builder`, *not* `parquet::WriterProperties::Builder` (easy
  to guess wrong — the name doesn't indicate which builder). See
  `parquet_debug_write_datetime_fixture` in `parquet_wrapper.cpp` for the working pattern if a
  future debug fixture needs another legacy/foreign Arrow encoding.

## Build & compiler notes

### The three machines available for testing

Three physical machines are available for building, testing and benchmarking this library, and they
differ in ways that matter — architecture, SIMD width, Fortran compiler, core count and Arrow
version. **Recorded here because a performance claim is only meaningful with the machine attached**,
and because this project already has one case (CLAUDE.md's materialize note) where the same change
measured 1.84x under one toolchain and parity under another. There is **no Docker** on machine A, so
the CI-environment image is not a route to a second toolchain; anything needing one runs on B or C.

| | **A — laptop** | **B — `bunyip.to.ee`** | **C — desktop** |
|---|---|---|---|
| CPU | Apple M1 Pro, 8 cores | 2 x AMD EPYC 9654, **192 physical / 384 logical**, 2 sockets, 2 NUMA nodes | Intel i7-10700K, 8 physical / 16 logical |
| arch / SIMD | **arm64, NEON (128-bit)** | **x86-64 Zen 4, AVX-512** (f/bw/dq/vl/vnni/bf16/…) | **x86-64 Comet Lake, AVX2 (256-bit)** |
| RAM | 32 GB | **1132 GB** | 128 GB |
| OS | macOS (arm64) | RHEL 9.7, kernel 5.14 | macOS (x86_64) |
| Fortran | gfortran 15.2 (MacPorts), **flang 22.1.8** | **ifx 2026.1.1**; gfortran 15.2.1 (gcc-toolset-15, what `activate_gcc.sh` now selects); gfortran 14.2.1 (gcc-toolset-14); system gfortran 11.5 — *see the warning below* | gfortran 15.2 (MacPorts), **flang-mp-22** |
| C++ | Apple clang 21, **`g++-mp-15`** (GCC 15.2) | **icpx 2026.1.1**; g++ 14.2.1 (toolset) or 11.5 (system) | Apple clang, MacPorts GCC |
| Arrow / Parquet | 25.0.0 | **24.0.0** | 25.0.0 |
| fpm | 0.13.0 alpha | 0.13.0 alpha | 0.13.0 alpha |

**Activating an environment on machine B.** B carries two complete toolchains and neither is
implicit — a shell there starts in the ifx environment:

- **ifx** (the default): `source /storage/projektid/qmost/activate_qmost_env.sh`. Sets `FPM_FC=ifx`
  and exports `FPM_CXXFLAGS`/`FPM_LDFLAGS` carrying Arrow's paths.
- **gfortran**: `source /opt/fortran/activate_gcc.sh`, plus the `scl` line below. It sets
  `FPM_FC=gfortran` and ends with `scl enable gcc-toolset-15 /bin/bash`.

**Which toolset that script selects has CHANGED, and the number is worth checking rather than
copying from here.** It used to end with `scl enable gcc-toolset-14`; it now selects **15**
(gfortran/g++ 15.2.1), which is what made UBSan possible on this machine — gcc-toolset-14 shipped
only a 32-bit `libubsan`, and 15 has the x86-64 one. Read the last line of the script rather than
trusting this paragraph, and use whatever toolset it names in the `enable` line below.

**Both activations need help to survive being sourced NON-INTERACTIVELY, and the gfortran one needs
two commands rather than one:**

```bash
source /opt/fortran/activate_gcc.sh </dev/null >/dev/null 2>&1
source /opt/rh/gcc-toolset-15/enable      # REQUIRED -- the line above is not sufficient
```

Without the redirection the script's trailing interactive subshell exits immediately with no tty and
everything after it runs under the *system* toolchain, while the variables it exported beforehand make
the shell look correctly configured; without the second line `gfortran` stays at **11.5.0**, which is
the miscompilation hazard below. Both were confirmed on machine B by a run that checked
`gfortran --version` before building — which is the only thing that catches either.

**A third hazard, and it is the one a benchmarking driver walks straight into: a sourced script
inherits the CALLER'S POSITIONAL PARAMETERS.** A driver invoked as `./run.sh ifx` that then sources
an activation script hands that script `$1="ifx"`, and a script which inspects `$@` takes a
different path. Demonstrated on machine B against `activate_qmost_env.sh`:

```bash
bash -c 'source activate_qmost_env.sh; echo $FPM_FC'          # -> ifx
bash -c 'source activate_qmost_env.sh; echo $FPM_FC' _ ifx    # -> (empty)
```

With `FPM_FC` unset, fpm fell back to the **system gfortran 11.5.0** — the compiler this project's
floor exists to exclude. **That run failed safe only by luck of a second mechanism**: the same
inactive environment left `FPM_CXXFLAGS` unset, Arrow's headers were not found, and the C++ half
would not compile. **Had Arrow been on the default search path it would have produced a complete set
of plausible numbers from a miscompiling compiler**, with nothing in the output saying so — the
benchmark wrapper's own banner reads `fortran : gfortran (fpm default)`, which is easy to read past.

The fix is one line — `set --` before sourcing anything — plus a hard assertion that `FPM_FC` is the
toolchain that was asked for and that `FPM_CXXFLAGS` is non-empty. **Assert, do not merely print**:
all three of these hazards are invisible in a log that a human skims and fatal to every number below
them.

**`ld.lld` is NOT on the `PATH` the ifx activation sets, and `-ipo` cannot link this library without
it.** `activate_qmost_env.sh` adds `/opt/intel/oneapi/compiler/2026.1/bin`; the linker lives one
directory further down, at `/opt/intel/oneapi/compiler/2026.1/bin/compiler/ld.lld`. Without it an
`-ipo` build of this library dies with **8087 undefined `<module>_mp_<proc>_` references**, which
reads as a defect in this project and is not (the system `ld`'s gold plugin cannot parse oneAPI's
bitcode). Extend `PATH` explicitly, and note that a tool gated on `command -v ld.lld` will silently
decline here — see "A wrapper must fail rather than degrade" below.

**DANGER on machine B: the system `gfortran` is 11.5.0, which is BELOW this project's minimum of 13
and will silently miscompile it.** In the ifx environment `/usr/bin/gfortran` (11.5.0) is what
`gfortran` resolves to, so anyone overriding `FPM_FC=gfortran` there without first sourcing
`activate_gcc.sh` gets a compiler that miscompiles the optional allocatable-`character` argument in
`schema%add_col_qc` — surfacing as a spurious "column not found" abort at runtime, far from the
cause (see "Compiler & language gotchas" for the underlying bug). **Always source
`activate_gcc.sh` before building with gfortran on B**, and check `gfortran --version` reports the
toolset's version (15.2.1 as of 2026-08-17, 14.2.1 before that) rather than 11.5.0 before trusting a
result from there. The check is "not 11.5.0", not a specific number — the toolset has moved once
already.

**Machine B's gfortran environment exports `-ffree-line-length-none` in `FPM_FFLAGS`.** That is
directly contrary to this project's enforced 132-column limit, so **a build on B cannot be used to
verify line length** — a too-long line compiles cleanly there and fails elsewhere. Check line length
on A or C, or with `tools/run_lint_check.sh`.

**Machines A and C cannot be told apart from a compiler listing — only `uname -m` separates them.**
Both run macOS, both carry MacPorts gfortran 15.2 and flang 22.x, and both are on Arrow 25.0.0; they
differ in *architecture*, A being Apple M1 Pro (arm64, NEON-128) and C Intel (x86-64, AVX2). So a
report that identifies its machine by listing compilers has not identified it. This has already gone
wrong once: a gfortran 15.2 result taken on C was first written up as "macOS arm64", which would have
turned a **confirmation of an existing `feature_risks.md` entry** into a spurious claim about a
**new architecture**. Have every run print `uname -m` — `tools/machine_report.sh` does, and takes
seconds — and read it before writing any figure down.

**What each machine is good for:**

- **A** — everyday development, and the only machine with a local `flang`. Quiet, so it gives the
  most stable small deltas.
- **B** — anything about **ifx**, about **threading at scale** (384 threads), or about **AVX-512**.
  It is also the only machine that can compare **gfortran against ifx with everything else held
  constant**, which is the cleanest compiler experiment available. Its size cuts both ways: a large
  NUMA machine is *bad* for small measurements (a cold destination is dominated by page faults —
  this project has recorded 5.6x run-to-run variation there at one size), so use it for scaling
  questions and take small deltas on A.
- **C** — x86-64 with the **same gfortran and Arrow as A**, so an A-vs-C comparison isolates
  **architecture alone** (NEON-128 against AVX2-256). That is the only clean single-variable
  vector-width experiment available and it is worth remembering it exists.

**Arrow differs**: B is on 24.0.0, A and C on 25.0.0. Irrelevant to anything measured purely in
Fortran, a confound for anything going through `src/parquet_wrapper.cpp` or an end-to-end read/write
timing. Do not compare those figures across B and C.

**Never override `FPM_FFLAGS`/`FPM_CXXFLAGS`/`FPM_LDFLAGS` on any of the three** — on all of them
those variables carry Arrow's (and other libraries') include and link paths, and setting them on a
command line *replaces* rather than appends, producing `fatal error: 'arrow/api.h' file not found`,
which reads like a missing dependency rather than a flag mistake. Append when a flag must be added:
`FPM_FFLAGS="${FPM_FFLAGS:-} -flto"`.

**A minimal mixed Fortran/C++ `bind(C)` program links under LTO on all three** — `-flto` for
gfortran + g++, **`-ipo`** (not `-flto`) for ifx + icpx, and `-flto` for `flang-mp-22` +
`clang++-mp-22` on machine C.

**That is NOT the same as "the library links".**
Machine B's `--lto-probe` passed while the real `-ipo` build of this library failed with the 8087
undefined references described above. The probe links objects **directly**; the actual failure
mechanism is a static ARCHIVE of bitcode objects being read by the system linker, which the probe
never builds. So the probe answers "can these two compilers emit and consume LTO objects at all",
never "can this library be built that way" — see "A pre-flight probe is evidence only if it
reproduces the real build's structure" below, which is the generalised form of the same trap.

**A flang build here is SERIAL-ONLY, and that is a property of the installation rather than of this
code.** MacPorts' `flang-mp-22` ships **no `omp_lib.mod` at all** — confirmed with a three-line
standalone program (`use omp_lib` under `-fopenmp` fails identically, so this is nothing to do with
fpm or with this project), and the only `omp_lib.mod` anywhere under `/opt/local` belongs to GCC. So
on machines A and C:

- **with** `-fopenmp`, a flang build cannot get past the first `use omp_lib` inside an
  `#ifdef _OPENMP` — nothing to be done from this repository;
- **without** it — which is what fpm actually does under flang, since the metapackage contributes no
  `-fopenmp` there — every OpenMP block is preprocessed away and **the whole project builds**:
  `FPM_FC=flang-mp-22 fpm build` compiles every `src/` file and links all 17 `app/` executables, and
  `nm` finds no `materialize_marked_parallel` in the object, against 2 occurrences in the gfortran
  one.

**The suite does now pass under flang, serially: `fpm test` reaches 1768 passed, 0 failed, 15
skipped, exit 0.** Getting there needed two unrelated fixes and both generalise, so read them before
concluding that a future flang failure is a portability defect in this library:

- **Fifteen tests assert that something THREADED, and now skip rather than fail** — the sort
  engine's Design A/B tests, `string_parallel`'s eight A/B rebuilds, and one shared-table error
  scenario. Their assertions are not merely untestable without OpenMP, they are *vacuous*: both arms
  run the same serial code and the equality holds for the wrong reason. See
  [A test that asserts THREADING must skip without OpenMP](#a-test-that-asserts-threading-must-skip-without-openmp).
- **The `table_parallel` crash was stack exhaustion, not a race**, and it is fixed at the root — see
  the character-temporary bullet under "Compiler & language gotchas". Earlier versions of this note
  called it "unexplained and undiagnosed"; it is neither, and nothing about it was specific to
  threading despite the suite it appeared in.

**"Compiles and links", "runs the suite" and "is supported" are still three different claims**, and
only the first two are now true. Every flang run here is of the SERIAL paths, so it exercises none of
the threading this library ships; a green flang suite is evidence about portability and about the
serial code, never about the parallel code. Keep quoting which of the three you mean.

**And `--profile release` does NOT link under flang, which is an LLVM defect rather than
anything here.** The release profile carries `-flto`, and the link dies with `LLVM ERROR: Unsupported
stack probing method` followed by `flang: error: unable to execute command: Abort trap: 6`. The
default profile is unaffected. So a flang check of this repository must use the default profile, and
a release-profile failure there is not a portability signal — re-test it against a newer LLVM before
treating it as one.

**Two things had to be fixed in this repository before any of that worked, and both will recur.**

- **A type may not carry more than 255 bindings ahead of a SPECIAL binding, because flang sorts the
  binding table by NAME and stores a special binding's index in one byte.** `parquet_table` has ~285
  bindings, and its `generic :: assignment(=)` guard used to be backed by `table_assign_guard` — which
  sorts under "t", lands past 255, and killed flang with an internal compiler error
  (`CHECK(bindingIndex <= 255)`, `runtime-type-info.cpp`) naming neither the type nor the line. The
  binding is now `assign_guard` (implementation unchanged), and **its alphabetical position is
  load-bearing**: `tools/generate_parquet_tables.py` says so at the declaration, because gfortran and
  ifx are indifferent and nothing will warn if it is renamed back. Any *future* special binding on a
  large type — another defined assignment, a defined operator — needs an early-sorting name too.
- **An unguarded `use omp_lib` is a compile failure, not a graceful fallback to serial.**
  `bench/benchmark_sort_tail.f90` had one, as did a since-deleted radix probe. Both guarded the
  `use` with `#ifdef _OPENMP` and supply serial shims for the `omp_*` functions they call under
  `#ifndef _OPENMP` (`omp_get_wtime` via `system_clock`, the rest returning 1/0). The `!$omp`
  directives themselves need no guarding — the preprocessor removes them. This is the same defect
  recorded below for `materialize_marked_parallel`, and it is worth grepping for whenever an `app/`
  program gains a timer.

So **"flang can build this" and "flang can build this with threads" are different claims here**, and
any flang measurement taken on these machines is of the serial paths. Do not read a flang build
failure as a portability defect without first checking whether the toolchain has the module.

**That serial build only became possible once a latent portability bug was fixed**, and the shape is
worth keeping in mind for any future OpenMP-dependent procedure: `materialize_marked_parallel`
(`src/parquet_tables_read.f90`) had an **unguarded** `use omp_lib` while all 21 other OpenMP uses in
that file sat inside `#ifdef _OPENMP`, so a no-OpenMP build failed to compile instead of taking the
serial fallback the guards exist to provide — invisible on gfortran and ifx, where the metapackage
always supplies the flag. **The whole procedure is now guarded, not just its `use`**: its body calls
`omp_get_thread_num`/`omp_get_max_threads` throughout and drives an `!$omp parallel` region, so
nothing of it survives OpenMP's removal. Two details to copy rather than rediscover — the guard opens
**above** the `!>` doc-comment block, so a doc-comment is never left behind to attach itself to the
next procedure; and **the caller is guarded to match**, because `parallel_prefetch_ok` already
answering `.false.` without OpenMP makes the branch dead but does *not* stop the reference from
having to resolve.

**But linking is not optimising: an LTO build across this library's STATIC LIBRARY also needs a
plugin-capable ARCHIVER, and without one it silently does nothing.** Under `-flto`/`-ipo` an object
holds intermediate representation rather than finished code. An archiver that cannot read that IR
indexes only what it can see, so the linker takes the machine-code half of a fat object and performs
no cross-module optimisation — **the build succeeds, the tests pass, and the measurement is of an
ordinary build with bigger objects.** There is no error and no warning. GCC ships `gcc-ar` (a
wrapper adding `-plugin liblto_plugin`); `fpm` archives with plain `ar` unless `FPM_AR` says
otherwise. **On Linux, binutils `ar` usually loads the plugin itself; on macOS the
default `ar` is Apple's cctools `ar`, which cannot** — so both macOS machines need
`FPM_AR=gcc-ar-mp-<N>` for a GCC LTO build to mean anything. The tell is the archive size: machine
C's `libparquet-fortran.a` came out **14.2 MB under `-flto` against 5.2 MB plain** while still full
of ordinary text symbols under `nm`, and measured no gain anywhere.

**On Intel, do NOT go looking for `xiar`: it does not exist in oneAPI 2026.1 at all** (`find
/opt/intel/oneapi -name xiar` finds nothing on machine B), having been retired in favour of
`llvm-ar` in the same `bin/compiler` directory as `ld.lld`. Two consequences, and the second was
measured rather than assumed: a warning telling someone to source `setvars.sh` so that `xiar`
appears names **a fix that cannot work**, and the archiver is not the load-bearing half on Intel
anyway — with `ld.lld` on `PATH` and plain `ar` archiving, `-ipo` on machine B linked cleanly and
still halved the temporal-validity figure, i.e. real interprocedural optimisation happened. `ld.lld`
is what `-ipo` needs; the archiver choice is a refinement. **A tooling warning is a claim like any
other**: B spent two extra probe runs falsifying this one rather than letting it qualify a whole
campaign's numbers, which was the right call and is the pattern to copy.
`tools/fpm_lto.sh` selects the archiver for both compiler families and refuses to build when it
cannot find one; `tools/machine_report.sh` reports which archivers exist. **Check the archiver before
reporting any LTO result, in either direction** — "LTO changes nothing here" is exactly what a
disabled LTO build looks like.

**On the two macOS machines the C++ half is Apple clang unless something says otherwise, so the
normal build is MIXED-FAMILY.** fpm derives its C/C++ compiler from the *Fortran* compiler's family
when `FPM_CC`/`FPM_CXX` are unset — gfortran gives `gcc`/`g++`, and on macOS a bare `gcc`/`g++` is
Apple clang, not MacPorts GCC. Machine C builds MacPorts gfortran 15.2 against Apple clang 21 that
way, which is deliberate (`FPM_CXXFLAGS` carries `-stdlib=libc++`, which MacPorts `g++` would
reject). Read `tools/machine_report.sh`'s `fc`/`cc` lines rather than assuming a matched pair, and
say which pair a figure came from — the same-family (`gfortran-mp-15` + `g++-mp-15`) and
mixed-family builds are different experiments.

**A mixed-family build cannot do CROSS-LANGUAGE LTO at all, whatever the archiver, because the two
halves emit incompatible IR.** Verified on machine A by inspecting the `-flto` objects directly: the
Fortran one carries GCC's `__wrapper_sects`/`__wrapper_names`/`__wrapper_index` Mach-O sections
(GIMPLE, in a *fat* object that also has a real `__text`), while `file` reports the C++ one as
`LLVM bitcode, wrapper`. No linker optimises across GCC GIMPLE and LLVM bitcode. **So the archiver
advice above is necessary but not sufficient on macOS** — machine A re-measured `-flto` with
`FPM_AR=gcc-ar-mp-15` genuinely in effect (confirmed: fpm really does invoke it, and the archive went
from 4.6 MB / 2830 text symbols to 13.8 MB / 2144) and still measured **no change on any item at a 3%
noise floor**, including the one item where machine B's same-family builds show a clean 2x. Two
consequences: an LTO figure from either macOS machine says nothing about an LTO figure from a
same-family toolchain, and "we set the archiver, so LTO was working" is not a conclusion the archiver
alone supports. Check what the *objects* contain (`file`, `otool -l | grep sectname`) before claiming
either way.

**But "mixed-family" blocks CROSS-LANGUAGE LTO only — the Fortran half can still be optimised
against itself, and on this project that is where the LTO-shaped wins actually live.** The paragraph
above is easy to read as "LTO on macOS does nothing", and machine A disproved that within one
campaign: `parquet_column%get_at` calls `check_kind`/`check_index` in a *sibling submodule*, both
pure Fortran, both GIMPLE — and under `-flto` on machine A the call cost **2.449 ns against 2.829
shipped**, landing within noise of a hand-inlined variant (2.481). The linker inlined two
cross-program-unit Fortran calls on the machine whose LTO "settles nothing". So: a null LTO result
on a mixed-family build settles nothing about anything **crossing the language boundary**, and says
plenty about Fortran-internal call overhead. Say which of the two a measurement was about before
quoting it either way, and note that this is the class `feature_optimise.md` §2.3's "LTO buys
nothing outside S7-4" verdict was never tested against.

### Writing benchmarking instructions for another machine

Measurements regularly have to be taken somewhere other than the machine the work is being done on —
a different architecture, a different compiler, a different core count. There is **no Docker and no
shared filesystem**, so this always means: someone carries instructions to that machine and runs
them by hand. Four rules make that repeatable, and they apply to *any* machine, not only the three
listed above.

- **Start from `bench/benchmark_template.md`.** It is the committed, machine-agnostic template:
  what the instruction-writer must fill in, the steps the runner follows, the rules that apply to
  every run, and the report skeleton. Do not write a run sheet from scratch, and do not edit the
  template with one campaign's details.
- **The instantiated instructions go in `feature_*.md`** (repository root), which is git-ignored —
  it is working material, carried between machines **by hand**, and must never reach the published
  package. **One instructions file is prepared per campaign**, named for the question rather than for
  a machine (`feature_benchmark_stage7.md`); **each machine then gets its own copy suffixed with the
  machine's name** (`..._A.md`, `..._B.md`), writes its report into that copy, and returns it.
  Machines run independently and may run concurrently — there is nothing to merge. **The analysis
  happens once, when every copy is back**, reading them side by side; that is why each report must
  carry full provenance even when it feels redundant, and why a machine is not asked to compare
  itself against another.
- **Everything the run EXECUTES belongs in the repository** — the program and its wrapper, both
  under `bench/` — so it travels by `git pull` and cannot drift between machines. Never paste code
  into the instruction file for someone to copy. Whatever is being measured must be **committed and
  pushed first**, which inverts this file's usual "leave it uncommitted on `main`" rule; for a
  before/after comparison push both commits and have the same command run on each.
- **The report is written back into the machine's own copy of the instructions file**, which is then
  returned under the name it was given — instructions and report together, since the instructions
  are what make the report interpretable later. **A run that is compared against an earlier one at a
  different commit must first check what changed** (`git diff --stat <old>..<new> -- app src tools`)
  and record the answer; without it, figures get filed against a build they were not taken on. The template's §5 skeleton says what a report must contain — provenance from
  `tools/machine_report.sh`, whether it built and passed *before* any timing, deviations, verbatim
  output, and what the run does not settle.

**Three things a multi-machine campaign reliably teaches, all confirmed by three reports coming back
at once:**

- **Verdicts can agree everywhere while MAGNITUDES span 3–6x, and a sub-conclusion can INVERT.** In
  one campaign every machine passed every gate, and yet the headline quantity was **2.8x** worse
  under ifx than gfortran on the same machine and **3.1–6.3x** worse under flang — while the two
  control arms were *cheaper* under ifx, so it was not a code-quality gap but one specific path. In
  the same campaign one sub-conclusion reversed outright on an allocator axis (25–30% under glibc,
  52–63% under macOS malloc), which was the difference between "fix the accessor" and "fix the
  allocation". **So "the verdicts agreed, one machine would have done" is wrong twice over**: the
  size of a win is what sizing work needs, and the inverted item was invisible to any single
  machine.
- **A NOISE FLOOR measured on a proxy is evidence about the proxy.** One machine's earlier entry
  reported a 5% floor and concluded it could not settle a 5% threshold; the figure had been taken on
  a different tool whose arms are ~30 ms bandwidth-bound array loops, while the campaign's own arms
  are small-working-set scalar loops that reproduced to **0.19%** — wrong by 25x, in the direction
  that disqualifies a perfectly good machine. Measure the floor with the harness the campaign
  actually runs.
- **"Loaded" is not "noisy", and a busy machine should not be disqualified without a number.** One
  report ran with load1 between 5.7 and 24 on 8 cores, with an unrelated process at 122% CPU
  throughout, and still measured a **1.5%** floor — because the arms are single-threaded and every
  figure is best-of-N. Say what the load was, measure the floor, and let the floor decide.

`tools/machine_report.sh` is the companion: a read-only toolchain report (CPU, SIMD, memory, every
compiler on `PATH`, **what fpm will actually use**, Arrow, the already-exported `FPM_*` variables,
load) plus an optional `--lto-probe` that link-tests a minimal mixed Fortran/C++ program per
toolchain in seconds. Have every run start with it; "the environment was already set up" is not
reproducible, and its `fc`/`cc` lines are the only authoritative answer to which compilers fpm picks
— fpm derives C and C++ from the Fortran compiler's *family*, so the first `g++` on `PATH` is
frequently not the one that builds `src/parquet_wrapper.cpp`.

**Two failure modes worth designing against, both already encountered.** An environment activation
script may not survive being sourced non-interactively (one ends by spawning an interactive
subshell, which exits immediately with no tty, leaving the *system* toolchain active while its
exported variables make the shell look correctly configured) — so a run sheet must verify the
compiler version rather than assume activation worked. And any wrapper that builds two
configurations must give each its own `FPM_BUILD_DIR` **outside** `build/`, or
`tools/run_error_scenarios.sh`'s `find … -name error_scenarios | head -n 1` can test the wrong
binary and report a false green.

**That build-tree name must vary by COMPILER as well as by configuration, and an existing wrapper
probably gets this wrong.** A since-deleted LTO wrapper named its trees for the configuration alone
(`test_run/s7-plain`, `test_run/s7-lto`) regardless of `FPM_FC`, so running it a second time under a
different Fortran compiler dropped that compiler's `error_scenarios` binary into the *same* tree —
fpm keeps the objects apart in its own per-compiler subdirectory, but the `find … | head -n 1`
lookup does not, and one of the two binaries is then chosen arbitrarily. The trap is the same one
the separate trees exist to close, re-opened along an axis nobody was thinking about. Until a
wrapper includes the compiler in its tree name, **run a second toolchain through it without
`--test`**, or point `FPM_BUILD_DIR` somewhere of your own. Both remote machines hit this
independently — one deleted the trees by hand between toolchains, the other declined to run a second
compiler at all rather than risk it.

**A pre-flight probe is evidence only if it reproduces the real build's STRUCTURE.** A cheap probe
that link-tests a few objects answers a question about the *compilers*; it cannot answer a question
about the *library* unless it is built the same way. `tools/machine_report.sh --lto-probe` passed on
machine B while the real `-ipo` build of this library failed to link, because the failure lives in a
static archive of bitcode objects and the probe never archives anything. The failure mode is the
worst available: a probe that passes is read as clearance, so the campaign proceeds and the eventual
error looks like a defect in the library rather than in the check. So when adding or trusting a
probe, ask what the real build does that the probe does not — an archive, a mixed compiler family, a
particular linker — and either reproduce it or say plainly what the probe does not cover.

**A wrapper must FAIL rather than degrade when it cannot engage the configuration it was asked
for.** A since-deleted LTO wrapper gated `-fuse-ld=lld` on `command -v ld.lld`, warned when it was
absent, and continued — so on machine B, where the documented activation script does not put it on
`PATH` (see the machine table above), the campaign's headline fix silently did nothing and the run
reproduced the exact failure the fix existed to remove. `tools/fpm_lto.sh` refuses instead, which is
the behaviour to copy. A warning on stderr, hundreds of lines above
the eventual error, is not a defence. **Two rules follow:** resolve a companion tool relative to the
compiler that owns it (`"$(dirname "$(command -v ifx)")/compiler/ld.lld"`) rather than trusting
`PATH`, and when a requested configuration cannot be assembled, exit nonzero — a run that could not
do what was asked is not a slower run, it is a different one.

**When tooling is fixed mid-campaign, every figure taken before the fix is suspect.** A measurement
is of whatever the tooling actually did, not of what it was meant to do, so a number carried forward
across a tooling change is a number filed against a build that never ran. Re-run it or label it, and
prefer re-running: this is the same reasoning as the "what changed since the earlier commit" step,
applied to the harness instead of the source.

### Compiler & language gotchas

Split six ways: the compiler-independent rules first, then one group per compiler in the fleet, then
the C++ half. **File a new gotcha by which compiler exhibits it** — a rule that binds regardless of
compiler goes in the general group even when one compiler is what exposed it, and a rule that exists
because of one compiler's codegen goes in that compiler's group even when the fix is portable.

#### General Fortran & language gotchas

Compiler-independent rules. Anything specific to one compiler is in its own section below —
[gfortran](#gfortran-specific-gotchas), [ifx](#ifx-specific-gotchas), [flang](#flang-specific-gotchas),
[nagfor](#nagfor-specific-gotchas) — and the C++ half is in
[C++ side](#c-side-srcparquet_wrappercpp-gotchas).

- **132-column line limit is enforced — do not reintroduce `-ffree-line-length-none`.**
  `src/*.f90` and `test/*.f90` are held strictly within the standard 132-column free-form limit,
  including comments (both whole-line and trailing end-of-line) — a comment pushing a line past
  132 columns is a violation just like code would be. When a line runs long, wrap it with `&`
  continuations (code/strings) or split it across multiple `!`-prefixed comment lines — don't reach
  for the compiler flag again, and don't add per-file/per-line suppressions.
- **Every `submodule (parquet) name` file needs its own `implicit none`** (right after the
  `submodule` line, before `contains`) — a submodule's `implicit none` is *not* inherited from
  the ancestor module. Contained procedures *within* a module/submodule do inherit their host's
  `implicit none` via ordinary host association, so it does not need repeating inside each
  individual function/subroutine — one `implicit none` per submodule file is sufficient.
- **`sign(1.0, x)` is NOT a portable test for a NEGATIVE ZERO — use the sign bit.** F2018 16.9.180
  makes `SIGN(A, B)` with a zero `B` **processor-dependent**: a processor that does not distinguish
  negative zero returns `|A|`, so `sign(1.0_real64, -0.0_real64)` is entitled to be `+1.0`. gfortran
  and flang distinguish it, **ifx does not**, so the idiom reported a *correct* merge as broken on
  one compiler out of three. The portable form asks the question directly:

  ```fortran
  neg = (x == 0.0_real64) .and. transfer(x, 0_int64) < 0_int64
  ```

  `ieee_is_negative` from `ieee_arithmetic` is equally correct if the module is already imported.
  Two adjacent traps: the sign of a negative zero in a *constant expression* is a second,
  independent risk, so build the value at runtime, which costs nothing and rules the question out;
  and a fixture whose whole discriminating power is one bit **must assert its own precondition
  first**, or a compiler that loses the sign produces a failure message blaming the code under test.
- **A function returning an unallocated `allocatable` cannot yield an unallocated LHS via
  `x = func()`.** Intrinsic assignment from an unallocated allocatable function result leaves the
  LHS *allocated* (an empty string/array), even for a fresh target. So an API cannot signal "absent"
  purely by returning an unallocated result — provide an explicit flag/sentinel instead (this is why
  `parquet_strings`' `allow_null` returns `""`, guarded by `is_null()`).
- **Never blank a deferred-length allocatable character ARRAY with `arr = ""` — assign element by
  element.** Intrinsic assignment to an allocatable reallocates it whenever the RHS's length
  differs, and that rule applies to a whole-array assignment from a scalar too: `arr = ""` keeps
  the shape but reallocates every element to **length zero**. A later `arr(i) = name` then writes
  the declared width into a zero-length allocation, so the names come back blank *and* the heap is
  corrupted. **gfortran keeps the length and hides both symptoms; ifx follows the standard and
  shows them** — as blank strings in an error message, followed some tests later by
  `free(): invalid next size (fast)` in unrelated code, which reads like a completely different bug.
  `parse_read_maml_remap` (`src/parquet_tables_maml.f90`) is the worked example; it blanks with an
  explicit `do i = 1, count` loop, because an array *element* is not itself an allocatable variable
  and so only blank-pads. The hazard does **not** apply to a deferred-length allocatable *scalar*
  (`suffix = ""` is the intended idiom), nor to an array assignment whose RHS carries the right
  length already (`values_c = pack(values, mask)` in `parquet_write_string.f90`).
- **A list-directed `read(text, *, iostat=ios) n` is NOT a strict parse, and silently accepts a
  wrong value.** It rejects `"5abc"` and `"3.9"` (`iostat` 5010) and `""` (`iostat` -1) as you would
  hope — but it accepts **`"5 6"` with `iostat == 0`, yielding 5**. So parsing any caller-supplied
  text this way (an environment variable, a config line, a command-line argument) turns a typo or a
  shell variable that expanded to two words into a plausible wrong value applied silently, which is
  far worse than a clean failure. Parse strictly by hand instead: trim, allow one optional `+`/`-`,
  require at least one digit and **nothing else** to the end of the string, and only then let `read`
  do the conversion. `env_int64` (`src/parquet_settings.f90`) is the worked example, and
  `settings_env_two_numbers` (`test/error_scenarios.f90`) is the regression test.
- **A compiler that WRAPS an overflowing integer expression may still use the overflow's
  undefinedness to delete a branch somewhere else — "it wraps" is not "it is safe".** These are two
  different claims and this project conflated them once, at the cost of a silent wrong answer.
  `parquet_random`'s integer draw needed `hi - lo + 1` as an unsigned 64-bit pattern; that overflows
  above `2**63`, and ifx was measured wrapping it faithfully, so it shipped as a documented wrapping
  site. ifx does wrap it — and, because `hi >= lo` holds by construction, also concludes that absent
  overflow the result must be positive, and therefore **deletes the `if (s < 0)` branch two
  functions away** that handles exactly the wide-width case. The draw then took the narrow-width
  path and returned a plausible, uniform-looking, wrong value still inside `[lo, hi]`.

  **So: a wrapping measurement is evidence about one expression's result, never about an inference
  the optimiser may draw wherever that value flows** — and a branch on the result's *sign* is the
  most inviting target there is. Fix at the root rather than at the branch: compute the pattern
  without overflowing (32-bit halves cost nothing off a hot path — see `sub64`/`add64` in
  `src/parquet_random.f90`), which leaves no undefinedness to reason from. Where an overflowing site
  is genuinely kept for speed, it must be guarded by a comparison against an independent
  overflow-free implementation, because nothing else will notice. See `feature_risks.md` Risk-94.
- **`transfer(source, mold, size)` into a longer target leaves the trailing bytes undefined**, not
  blank-padded. To place a short string into a longer fixed-length slot, assign normally (which
  blank-pads); reserve `transfer` for exact-size byte moves.
- **A PER-ELEMENT `transfer` into a `character(len=1)` array allocates a temporary each time, and
  SEQUENCE ASSOCIATION is the way out.** `dst(a:b) = transfer(str(lo:hi), dst, n)` is the obvious way
  to copy a `character(len=*)` scalar's bytes into a packed `character(len=1), allocatable` payload,
  and it costs an allocation per call: measured at **21.6 ms of a 55.5 ms** 1M-element column fill.
  The fix is not a better `transfer` but a change of view — a contiguous `character(len=w)` array's
  bytes are one `w*n` block, and passing it to a `character(len=1), intent(in) :: src(*)` dummy hands
  that block over verbatim, after which each element's copy is an ordinary **section-to-section
  assignment between two `character(len=1)` arrays**: no temporary, no intermediate buffer, one pass.
  `pack_character_bytes` (`src/parquet_strings.f90`) is the worked example, and the same trick is what
  lets the read/write paths hand a caller's `character(len=*)` array straight to a
  `character(kind=c_char) :: data(*)` `bind(C)` dummy with no staging buffer at all.

  **Three things to know before reaching for it.** The actual argument must be **contiguous** —
  declare the public dummy `contiguous`, which costs a copy-in for the rare strided caller and makes
  the association safe. It flattens rank freely, so a rank-2 `values(:,:)` associates with the same
  assumed-size dummy in column-major order, which is how a matrix specific reaches a rank-1 bulk
  entry point **without `reshape`** — `reshape` would copy the whole array to produce the flattening
  association gives for free. And **do not hand-roll `len_trim` over the byte view** once you have
  it: replacing the intrinsic with a trailing-blank scan measured **3x slower** (32.5 ms against
  9.5), so lengths stay on the element view and only the payload copy uses the byte view.
- **An array-section assignment whose two sides are the SAME array costs a heap temporary per
  iteration.** A scalar byte loop compacting a buffer in place —
  `do k = lo, hi; w = w + 1; a(w) = a(k); end do` — reads like something waiting to be replaced by
  `a(w+1:w+n) = a(lo:hi)`. It is not: the compiler cannot prove the two ranges do not overlap, so it
  must evaluate the right-hand side into a temporary first, which is one allocate/free **per
  element**. Measured on 4 M elements: `trim_all` 0.0261 s → 0.1439 s (**5.5x slower**),
  `delete_by_mask` 0.0118 s → 0.0886 s (**7.5x**). Nothing fails; only a benchmark notices.
  **The rule is about aliasing, not about sections** — same array, use the loop; different arrays,
  use the section, where it is a single `memcpy` and is worth having. This is why a threaded rebuild
  that writes into fresh buffers wins twice over: it removes the race *and* the aliasing. See
  `compact_all_serial`/`delete_by_mask_serial` (`src/parquet_strings.f90`), whose loops carry a
  comment saying they may not be tidied up.
- **Intrinsic assignment to or from a FINALIZABLE type runs the finalizer — twice per iteration in
  the obvious loop.** `dest(i) = obj%make(i)` finalizes `dest(i)` before overwriting it *and*
  finalizes the function result afterwards, so a loop that only needs to set a couple of components
  pays two finalizer calls per element for them. `parquet_string_column%view_all` did exactly that
  (`data_string(i) = self%view(i)`, setting a pointer and an integer) and got **3.9x** faster by
  writing the two components directly, which also dropped a redundant bounds check. **That example
  is historical in one respect**: `parquet_string` has since had its finalizer removed outright (it
  was a non-owning handle whose finalizer only nulled a borrowed pointer), which is the *other* way
  to fix this shape and the better one when a type does not actually own anything — the measurement
  and the rule both still stand for the types that do. This library exposes four finalizable types
  (`parquet_writer`, `parquet_reader`, `parquet_string_column`, `parquet_table` — grep `final ::`
  to re-derive the list rather than trusting this one, which has already gone stale twice), so
  check for this shape before threading any loop that assigns one of them. Every other finalizer note
  in this file is about *correctness*; this one is purely about cost.
- **Passing an UNALLOCATED allocatable to an `optional` dummy makes that dummy ABSENT** (F2018
  15.5.2.12; verified on gfortran before relying on it). This is load-bearing, not a curiosity:
  `parquet_column%row_validity` and every `mat_*`/`matchunk_*` return or hold an unallocated mask for
  a null-free column and pass it straight on as `is_valid=`, so the callee sees no argument at all and
  takes its own no-mask fast path. It means a procedure can decline to supply an optional argument
  *at runtime*, without the caller writing an `if (present(...))` fork or duplicating the call — but
  it also means **a caller cannot tell "absent" from "the producer had nothing to say"**, so any
  procedure returning such an array must document that `allocated()` is part of its contract.
- **Never write a function that returns `character(len=:), allocatable` — use a subroutine with
  an `intent(out)`/`intent(inout)` allocatable `character` argument instead.** This is a fixed
  project-wide convention, not just advice: gfortran has a confirmed, still-open compiler bug
  (GCC [PR113797](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=113797); related:
  [PR97977](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=97977)) where the codegen for *receiving*
  such a function's result uses a hidden length-tracking variable that isn't always properly
  thread-local, silently corrupting memory when the function is called concurrently. Plain
  non-`character` allocatable function results (numeric scalars/arrays, allocatable arrays of a
  derived type) are not affected and need no action.

  Already applied throughout `src/*.f90` — apply the same conversion to any new occurrence.

  **This is not a hypothetical risk — it has actually caused silent, hard-to-trace memory
  corruption in this project.** `top_level_of` (`parquet_tables_read.f90`, added alongside the
  table-mutation work) was written as exactly this forbidden shape and slipped past review.
  Confirmed via ThreadSanitizer: two OpenMP threads calling it concurrently (from
  `table_release_one`/`materialize_marked`, both reachable from ordinary `%prefetch`/
  `%materialize_all` use) raced on gfortran's hidden length-tracking temporary (TSan named it
  `slen.49.1`), corrupting memory that then surfaced later as an unrelated-looking `ERROR STOP`
  immediately followed by a SIGSEGV, in a completely different part of the table code, several
  investigation rounds later (a ThreadSanitizer-caught Arrow-singleton race — see the next
  section — was found and fixed FIRST, from a plausible-looking but ultimately secondary TSan
  report, before this one, the actual dominant cause, was found in a follow-up TSan run). Fixed
  by converting it to a subroutine per this rule; both call sites updated to
  `call top_level_of(name, top)`. **Lesson for any future concurrency investigation in this
  codebase: fixing one TSan-caught race does not mean the reported failure is resolved — rerun
  the sanitizer after every fix, since more than one independent race can be masking behind the
  same flaky symptom, and a small, easy-to-miss helper like this one can be the dominant cause
  even when a more "interesting"-looking external-library race is also genuinely present.**

  **Design note for the "assign result back into the input variable" pattern** (e.g.
  `col = schema%get_col_qc(col)`): a subroutine can't alias the same actual argument to separate
  `intent(in)`/`intent(out)` dummies, so make the single argument `intent(inout)` instead — it
  holds the input on entry and the result on exit, preserving `call schema%set_col_qc(col)`'s
  single-variable ergonomics (see `schema_set_col_qc`/`parquet_maml_file%set_col_qc`).
- **`intent(out)` on a finalizable type resets every component for free — don't convert one to
  `intent(inout)` without auditing every component first.** `parquet_writer`/`parquet_reader` both
  have a `FINAL` procedure, and Fortran resets *every* component of an `intent(out)` dummy to its
  default initializer (deallocating every allocatable, zeroing every scalar) on procedure entry.
  `parquet_open_writer`/`parquet_open_reader`'s own bodies only explicitly (re)initialize a
  handful of fields (e.g. `handle`, `filename`) and lean on this implicit reset for the rest
  (mask state, row-group tracking flags, `qc`, cached metadata, ...). If either is ever converted
  to `intent(inout)` (e.g. to add a "not already open" guard), that implicit reset goes away, and
  a caller reusing the same variable across two open calls would silently carry over stale state
  from the previous use unless the open body is rewritten to unconditionally reset every single
  component itself — a much larger, easier-to-get-subtly-wrong change than it first appears.
  Prefer keeping `intent(out)` and solving misuse-prevention some other way.
- **That reset is NOT free when the dummy is POLYMORPHIC, and on an elemental setter it costs
  several times the work the setter does.** `class(t), intent(out)` makes the compiler
  default-initialise the element *through the runtime* on entry, because the dynamic type is not
  known statically — so an `elemental` setter pays it per element, and a type-bound procedure has
  no way out, since its passed-object dummy must be polymorphic. Measured on `parquet_temporal`'s
  read path over 4M rows: the construction loop cost **12.3 ms** with `intent(out)` and **3.4 ms**
  with `intent(inout)` (3.6x; timestamp 17.2 → 5.7 ms), taking a whole date-column read from
  23.6 ms to 14.8 ms. Isolating it on a layout-identical twin gives the shape exactly — `class` +
  `intent(out)` is the slow cell, while `class` + `intent(inout)`, `type` + `intent(out)` and
  `pure` + `type` are all within noise of a plain store, so **neither `class` nor `impure` is the
  cause on its own; the combination is.** Note the fourth cell does not exist: a pure procedure may
  not have a polymorphic `intent(out)` dummy at all, so "make it `pure`" is not available as a fix
  for a type-bound setter.

  **The fix is `intent(inout)`, and it moves an obligation from the compiler to the source**: every
  component must then be assigned by hand, or a *reused* element silently keeps its previous value
  in whatever was missed. Two rules follow, and the second is the one that protects correctness
  rather than speed:

  - A setter that assigns every component on every path may take `intent(inout)`. If it has an
    empty body that relied on the reset (a `%set_null`), give it an explicit body matching the
    declared initialisers.
  - **A setter with a caught-failure path that `return`s without assigning must KEEP
    `intent(out)`** — that reset is what makes a failed `%parse`, or a null-propagating
    `%set(date, time)`, yield a *null* element instead of a stale one. Converting one of those for
    speed is a silent correctness regression.

  Enforce it statically rather than by review: `check_temporal_setters_assign_all`
  (`tools/check_source_conventions.py`) derives both the component list and the setter list from
  the source, so a component added later fails on every setter of that type at once. See
  `feature_risks.md` Risk-70. **Do not reach for the whole-array elemental call as an alternative
  fix** — measured after the change, `call arr%set_unix(vals, unit)` was **3.2x slower** than the
  indexed loop it would replace, so the win is in the dummy's intent, not in the call shape.
- **A component and its parent cannot both be actual arguments of one call.** Passing `t%cache` and
  `t%cache%reader` to the same procedure argument-associates two dummies with overlapping storage,
  which Fortran forbids as soon as either is defined (F2018 15.5.2.13) and which compilers optimise
  against. Neither gfortran nor ifx diagnoses it. This shapes API design rather than being a bug to
  fix afterwards: `table_open_reader_with_transform(cache, filename, [rdr])` takes the reader as an
  **optional** argument precisely because of it — absent means "open `cache%reader`", reached through
  the cache, and `rdr` names any OTHER reader. Where that leaves two near-identical code paths, a
  local `type(...), pointer` resolving to one or the other is the way out, with `target` on both
  dummies; a pointer to a component of a plain (non-`target`) dummy is not permitted, which is why
  `table_materialize` writes its own two arms out instead.
- **`.and.` does not short-circuit, and `-fcheck=bounds` is what tells you.** Fortran may evaluate
  both operands, so `size(a) == size(b) .and. all(a == b)` compares two differently-sized arrays
  whenever the sizes differ — an out-of-bounds read that a plain `fpm test` runs straight past and
  `fpm test --profile debug` aborts on. Nest the tests instead (`same = .false.; if (size(a) ==
  size(b)) same = all(a == b)`). This has bitten twice: once in `parquet_close_writer`'s
  mask-consumed check, once in a test comparing two sample draws. Both times the guarded form looked
  obviously safe.

  **It has now bitten a THIRD time, in the same test file, and the third instance says something the
  first two did not: the mismatch can be the NORMAL case rather than an edge case.**
  `test_sample_seed_is_full_width` asserted `.not. (n_lo == n_hi .and. all(lo_bits(1:n_lo) ==
  hi_bits(1:n_hi)))` — and the whole point of that test is that two seeds select *different* row
  counts, so the `all()` compared sections of different extent on almost every run. It sat there
  green under gfortran, ifx and flang.
  **`fpm test --profile nagdeb` is the sharpest detector available for this class** — nagfor's
  `-C=array` names the array, the section and both extents (*"Rank 1 of HI_BITS(1:N_HI) has extent
  102 instead of 99"*), deterministically, at every thread count. `--profile debug` on gfortran
  catches it too; nothing else in the fleet does. **A regex cannot find these reliably** — a scan
  tight enough to avoid false positives found nothing, and a loose one reported only fixed-index
  accesses — so the checked build is the check.

  **And when this fires under threads it can present as a SEGFAULT rather than as the runtime
  error**: several test-drive threads reporting a fatal error at once is the `abort()`-from-many-
  threads hazard documented above, so the first symptom may be `exit code 11` with no message. Run
  the suite again single-threaded (`OMP_NUM_THREADS=1`) before concluding anything about a
  segfault under `nagdeb` — that is what turned this one from "an intermittent nagfor bug" into a
  one-line source defect.

  **The same non-short-circuiting is also a PERFORMANCE hazard, and there it is compiler-dependent
  in a way that hides on gfortran.** `if (cheap_test .and. expensive_call() == 0)` may evaluate the
  expensive half unconditionally. Measured instance: `table_resolve`'s
  `if (name == PARQUET_ROW_INDEX .and. table_find(self, name) == 0)` costs **4.3 ns under gfortran**
  -- a bare string comparison, i.e. it short-circuits -- and **33.9 ns under ifx**, which is a string
  comparison plus very nearly a second complete `table_find` (ifx prices one at 31.4 ns). So ifx
  performs **two full name lookups per accessor where one is needed**, on every value accessor in
  the table layer, worth **27% of a per-cell read**. Whether the optimiser short-circuits is
  compiler-specific, so measuring it on one compiler settles nothing. **Nest the test rather than
  relying on the optimiser** whenever the second operand is more than a comparison:

  ```fortran
  if (cheap_test) then
      if (expensive_call() == 0) then
          ...
      end if
  end if
  ```
- **cpp runs over every source file, so `/*` anywhere — including inside a Fortran comment —
  breaks the build.** `fpm.toml` declares `[preprocess.cpp]`, which applies to *all* sources, not
  just `.F90` ones. Writing a glob like `tools/*.sh` in a comment opens a C block comment and the
  file fails to compile with a confusing `unterminated comment` pointing at the *last* line of the
  comment block, not the offending one. A trailing `\` at the end of a comment or string literal is
  a cpp line-continuation for the same reason, and silently splices the next source line. Neither is
  a Fortran error, so nothing about the message suggests the real cause. Write `tools/ *.sh`, "a
  shell wrapper under `tools/`", or anything else that avoids the two-character sequence.
- **A type-bound procedure cannot share a name with a data component of the same type**
  (`Error: Procedure 'x' at (1) has the same name as a component of 'my_type'`). When a natural
  accessor name collides with the component it reports (`%nrows()` over an `nrows` component), rename
  the **component** — it is private implementation detail — and keep the short binding name, which is
  the public surface. Renaming the binding instead pushes an internal detail into the API.
- **`-ftrapv` traps only the WRAPPING arm of `src/parquet_random.f90`, which is the one arm no
  compiler in this fleet ships by default.** Measured 2026-08-20 on machine A, gfortran 15.2, over a
  wide-range `pf_random_int_at` sweep, forcing each arm of the route (e) fork in turn:

  | arm | who ships it | `-ftrapv` at `-O0`/`-O2`/`-O3` |
  |---|---|---|
  | `PF_INT128` | gfortran, flang | **passes** |
  | `PF_SAFE64` | nagfor | **passes** |
  | wrapping (`#else`) | ifx | **SIGABRT (134)** at every level |

  All three return the identical checksum, so they are bit-exact and only the undefined behaviour
  differs, so **a gfortran build of this library is `-ftrapv`-clean**, and so is a nagfor one —
  which is the same property `-C=intovf` exercises from the other side.
  **The sharper lesson is the converse: a `-ftrapv` build that does NOT abort is not evidence that a
  site is safe.** Whether a given overflow is instrumented depends on optimisation level and
  inlining context, and a site whose result is *dead* — such as a loop's final unused index update —
  is typically optimised away before instrumentation, so it traps nothing while remaining fully
  available to the optimiser as a range assumption. Use the cross-implementation agreement sweep to
  answer that question, never a trapping build.

  **UBSan answers it properly, and is a machine-B-only instrument** (gcc-toolset-15 has the x86-64
  `libubsan`; MacPorts gcc15 ships none, and flang rejects `-fsanitize` for Fortran). **Run it with
  `tools/check_random_ubsan.sh`**, which drives both arms of the fork — UBSan on a gfortran build
  alone only ever exercises the `int128` arm. **What it found on its first run is the reason to keep
  running it**: the two forced arms reported exactly the documented deliberate sites, while the
  ordinary suite build reported **six further, undocumented signed-overflow sites** in the bulk
  fills, where `base + k - 1_int64` forms `huge + 1` at the boundary the suite deliberately tests.
  Every value was correct; only UBSan could see it, and `-ftrapv` had never flagged any of them.
  See `feature_risks.md` Risk-112.

#### gfortran-specific gotchas

- **Minimum gfortran is 13; don't work around compiler bugs in source.** gfortran ≤ 11
  miscompiles the optional allocatable-`character` argument in `schema%add_col_qc` /
  `schema%set_col_qc` (corrupted column name → a spurious "column not found" abort at
  runtime — see README.md's Prerequisites). That's the reason for the version floor; don't
  refactor otherwise-correct source to accommodate an old compiler.
- **Never write a function that returns `character(len=:), allocatable`** — the project-wide
  convention this forces is stated under
  [General Fortran & language gotchas](#general-fortran--language-gotchas); the *cause* is a
  gfortran codegen bug, and ifx independently rejects the automatic-length variant.
- **Never give a FINALIZABLE derived type to OpenMP's `private()` — declare it in a `block`
  inside the loop body instead.** gfortran does not reliably default-initialize a `private` copy
  of such a type, so a pointer component starts as garbage and the *first* thing that finalizes
  it — including the implicit finalization of an `intent(out)` dummy on entry to an "open"
  procedure — frees an undefined pointer and the process dies inside the allocator, with a
  backtrace pointing at malloc rather than at any of this library's own code. **Reproducible with
  `OMP_NUM_THREADS=1`, which is what rules out a data race** and identifies it as initialization.
  The working form declares the variable where it is used, so ordinary block-scope initialization
  and finalization apply:

  ```fortran
  !$omp parallel do default(shared) private(rg) reduction(+:total)
  do rg = 1, n
      block
          type(parquet_table) :: mine     ! NOT private(mine)
          ...
      end block
  end do
  ```

  This applies to every finalizable type this library exposes (`parquet_table`, `parquet_reader`,
  `parquet_writer`), and to any future one, so any example or guide page showing
  `private(<that type>)` is wrong and should be corrected on sight. **The HANDLE types are
  deliberately not in that list**: none of `parquet_table_row`, `parquet_table_col` and
  `parquet_string` has a finalizer, which is what keeps them usable in a `private()` clause — so
  removing a finalizer is one way to take a type *out* of this rule.
  **Note ifx forbids exactly the shape this prescribes** for a type with allocatable components;
  see [ifx-specific gotchas](#ifx-specific-gotchas) for the shape that satisfies both.
- **Test for NaN with `ieee_is_nan`, not `x /= x`**, because the self-comparison idiom triggers
  `-Wcompare-reals`. See `parquet_metadata_maml.f90`'s `parquet_qc_numeric_bound`. This does not
  apply to the other `-Wcompare-reals` sites here (`value == anint(value)`, testing whether a float
  is exactly integral) — those are exact-equality checks with no equivalent NaN-style idiom, and
  their warning is an accepted false positive rather than something to "fix" into an epsilon
  comparison. **`src/parquet_argsort_engine.f90` is a deliberate, measured EXCEPTION and uses
  `x /= x` throughout — do not "correct" it**: `ieee_is_nan` cost ifx ~2.5 ns per test on the sort's
  hot comparison path. Its header states the case, and the general lesson with it — **a
  microbenchmark of an isolated operation does not predict its cost inside a dependency chain**.
- **`-128_int8` trips gfortran's range check** (it parses `128` then negates). Build the high bit
  with `ibset(0_int8, 7)` in constant expressions. Also: an array-constructor implied-do index
  (`[(f(b), b=0,7)]`) has no implicit type under `implicit none` — list the elements explicitly.
- **`intent(out)`'s implicit reset is the documented standard behavior, but this project has one
  confirmed counterexample — don't treat it as an absolute guarantee for a correctness-critical
  `logical` component.** `parquet_table`'s `detached` was found (gfortran 13/14) to sometimes still
  read back `.true.` immediately after a fresh `intent(out)` reopen of a variable that had
  previously been detached, with no concurrency involved and every other component behaving
  correctly. It was the one component relying *solely* on the implicit default-initializer reset.
  Fixed with an explicit `table%detached = .false.` as the first executable statement of
  `open_table_impl` and `parquet_new_table` (`parquet_tables_lifecycle.f90`) — do not remove it, and
  apply the same explicit reset to any new scalar `logical` component of a finalizable type.
- **A `pointer`-typed intermediate component defeats `-fcheck=bounds`'s trust in a freshly
  unallocated LHS on intrinsic assignment.** `out%cache%rg_bounds = self%cache%rg_bounds`, relying
  on F2003 automatic reallocation, raised a spurious "Array bound mismatch" even though the LHS was
  genuinely unallocated — reaching it through a `pointer` component rather than a plain allocatable
  one is what confuses the check. Use an explicit `allocate(...)` plus an element-wise copy instead.

  **The same shape carrying a deferred-length `character` array is worse — it has been seen to
  segfault outright, and only on CI's compiler**, because automatic reallocation must establish the
  deferred LENGTH as well as the shape. It ran clean on gfortran 15.2 locally under `-fcheck=all`
  and under `--coverage -fopenmp`, so **a clean local run proves nothing about this class of bug**.
  Use `allocate(character(len=len(src)) :: dst(size(src)))` plus an element-wise loop — element-wise
  because a whole-array `dst = src` is itself the reallocation hazard described under the general
  gotchas. Reach for that shape from the start, and do not restore the plain assignment on the
  strength of a green local run.
- **A private procedure contained directly in a module, whose only callers are that module's
  submodules, compiles cleanly and then fails at LINK time** with `undefined symbol`. gfortran does
  not emit it (it also reports `-Wunused-function`, which is the early warning). This is invisible
  to per-file compilation and only appears when something links, so it can survive a long way into a
  change. Fix: declare the procedure's interface in the module and implement it in a submodule, as
  `parquet_core.f90` already does for its shared private helpers (see `src/parquet_columns_util.f90`,
  a file created solely to hold such helpers). Prefer that shape from the start for any helper a
  submodule will call.
- **gfortran 15.2 ICEs under `-flto` when a SUBMODULE calls a module-level PROCEDURE POINTER**, so
  such a call must live in the owning module's own `contains` and be reached through a relay. The
  failure is `internal compiler error: in write_symbol, at lto-streamer-out.cc:3086`, during
  `IPA pass: modref`. Twenty lines reproduce it: a module declaring
  `procedure(i_p), pointer, save :: p => null()` and a submodule whose body is `call p(x)`.

  **`-flto` is the only flag that matters** (it ICEs at `-O0` through `-O3` alike); `save`,
  `=> null()` and accessibility make no difference. **Copying the pointer to a local does NOT help**
  — the ICE follows any call derived from the module-level pointer — and a submodule of a *different*
  module that merely use-associates it fails identically, so it is not about the owning module.
  **A bare reference is not always safe either**: `associated(p)` alone compiles, but passing it as
  an actual argument reproduces the ICE, so the rule is that no submodule names the pointer at all.
  That is why `parquet_argsort`'s seven `oracle_*` relays fold the `check_oracle` test in rather than
  leaving it at the call site, and why **the relays must be PUBLIC** — a private module-contained
  procedure whose only callers are its own submodules does not link (previous bullet), and the usual
  fix for *that* is exactly what reintroduces this ICE.

  **Nothing in CI or in a plain `fpm test` builds with `-flto`**, so a reintroduced call would
  compile, pass every test and sit in the tree until someone asked for a release build.
  `check_no_submodule_oracle_pointer_call` (`tools/check_source_conventions.py`) is what makes it
  visible; it also fails if the relays disappear, since without them every submodule call is legal
  again.

#### ifx-specific gotchas

- **ifx forbids the `block` form gfortran's `private()` rule prescribes, for any type that has
  ALLOCATABLE COMPONENTS — and is perfectly happy with `private()`. The two compilers forbid
  opposite shapes, so a type in that class can use neither, and needs a shared per-thread array
  instead.** ifx emits privatization scaffolding (`<TYPE>.omp.mold_ctor` → `for_alloc_private` →
  `do_alloc_copy` → `copy_src_xdesc_to_dest_xdesc`) for such a type declared in a `block` lexically
  nested in a parallel region, and segfaults on every thread entering the region — 100% reproducible
  with 2 threads, independent of team size, so not a race, with a backtrace naming no library code.
  The working shape is the one in `materialize_marked_parallel` (`src/parquet_tables_read.f90`):
  allocate an array of the type **before** the region, one slot per thread, and index it by
  `omp_get_thread_num() + 1`, so no instance is constructed inside the construct at all. Passing an
  element on to an `optional, intent(inout)` dummy is fine.

  **Two conditions narrow this, and both will exonerate a broken shape if you reproduce
  carelessly.** It needs `-O1`+ (at `-O0` it runs clean, so `fpm build --profile debug` cannot see
  it). And it needs the type to come from a **separately compiled module** — the identical type
  defined in the same file as its user does not crash, so a single-file reproducer will say the
  shape is fine when it is not.

  **The allocatable components are the whole trigger — FINALIZABILITY IS NOT REQUIRED.** The class
  is *any derived type with an allocatable component*: `parquet_reader`, `parquet_writer`,
  `parquet_schema`, `parquet_column` and `parquet_string_column` are all in it today. A type with
  none is exempt, which is why `parquet_table` is still safe block-local — five scalars and a
  pointer by deliberate design (see "New `parquet_table` state goes on the CACHE"), a design now
  load-bearing for ifx too, since the first allocatable component added to it would make every
  block-local per-thread table in user code start crashing. See `feature_risks.md` Risk-45.

  **Diagnosing it takes one command**, worth running before concluding a compiler is at fault at
  all: `nm <object> | grep -E "for_alloc_private|mold_ctor"`. If the scaffolding is absent, the
  source under test *cannot* produce that backtrace and the binary that crashed is stale — see
  "Stale `fpm` build cache".
- **Passing a non-polymorphic `type(T)` actual to a `class(T)` dummy in another compilation unit
  makes ifx BUILD the class descriptor, in the caller's prologue, on every call.** One
  type-descriptor record per allocatable component of `T`, emitted unconditionally ahead of any
  branch — for `parquet_column` that is 21 records and 178 stores, **~35 ns per call**, and it was
  78% of what a per-cell table read cost. This is the converse of the polymorphic-`intent(out)`
  cost under the general gotchas, and independent of it. The fix is to put the implementation
  behind a `type(T)` dummy and leave the type-bound binding as a one-line forwarder — a `class`
  actual passed to a `type` dummy is free, so only that direction works. Fully written up, with the
  maintenance rules and the one-command `objdump` check, under
  [`parquet_column`'s TYPED accessor tier](#parquet_columns-typed-accessor-tier-never-reach-storage-through-a-binding).
  **gfortran does not emit the block but still gained 1.23-1.32x from the same change**, so this is
  not an ifx-only workaround.
- **An *automatic*-length `character` result whose length is a specification expression over
  host-associated variables is an ifx internal compiler error.** ifx segfaults (and the source line
  it names is meaningless) on:

  ```fortran
  submodule (m) sm
  contains
      module procedure p
          integer, allocatable :: lo(:), hi(:)          ! host-associated allocatables
          ...
      contains
          subroutine sibling(...)
              ... // tok(k) // ...                      ! call from a SIBLING contained procedure
          end subroutine sibling
          pure function tok(k) result(res)
              character(len=hi(k)-lo(k)+1) :: res       ! length reads those allocatables
              res = text(lo(k):hi(k))
          end function tok
      end procedure p
  end submodule sm
  ```

  Every condition is required, confirmed by bisecting each away independently: `-O1` or higher (at
  `-O0` it compiles clean, which is why `fpm build --profile debug` succeeds while a plain
  `fpm build` fails — easy to misread as a flaky build); the call coming from a **sibling** contained
  procedure rather than from the module procedure's own statements; and the whole construct sitting
  inside a `submodule`'s `module procedure` body. Recursion is **not** required, and gfortran
  compiles the same code correctly at `-O2`, so this will not show up in CI. Fix it the way the
  general character-function rule already forces — a subroutine with an `intent(out)` allocatable
  `character` argument — or drop the helper and write the expression at the call sites.
  `tok_text` (`src/parquet_read_filter.f90`) is the worked example.
- **Never interpolate unbounded caller-supplied text into an `error stop` message — cap it to a
  short preview.** ifx's `ERROR STOP` runtime corrupts the heap once the composed message reaches
  **8192 bytes** (8191 aborts cleanly; 8192 crashes every time). This bites hardest where it is
  least expected: a "value too long" guard that reports the offending value verbatim is *guaranteed*
  to build a huge message on the one input that triggers it, turning a clean abort into a crash and
  an error-scenario test into a confusing stderr mismatch. `parquet_filter_add`
  (`src/parquet_core.f90`) is the pattern to copy — at most the first 100 characters, plus `"..."`
  when truncated. Apply the same cap to any message embedding a rule, a MAML line, a filename or
  anything else whose length the caller controls; a multi-kilobyte error message is unreadable
  anyway.
- **ifx rejects a default structure constructor (`type_name()`) when the type has a component whose
  OWN type has private components declared in a different module** — even when that component is
  not touched by the constructor and was cleared beforehand. gfortran accepts it; ifx reports
  `error #6053: Structure constructor may not have components with the PRIVATE attribute`, naming
  the *outer* type. `parquet_table_column` (`parquet_tables.f90`) has a `values` component of type
  `parquet_column`, whose components are private to `parquet_columns.f90`. Fix: replace the default
  constructor with an explicit field-by-field reset of the type's own components, leaving the
  private-dependent one untouched (handled separately, e.g. by `%clear()`). `table_drop_column`
  (`parquet_tables_mutate.f90`) is the worked example.

#### flang-specific gotchas

flang builds here are **serial only** and `--profile release` does not link — see
[The three machines available for testing](#the-three-machines-available-for-testing) for both.

- **A CHARACTER TEMPORARY built inside a loop may never be reclaimed until the procedure returns, so
  a long loop dies of stack exhaustion far from its cause.** Every `call sub("%" // what // ": ...")`
  in a loop gets its own stack slot, about 90 bytes a call, and the frame grows monotonically.
  `check_rows_consistent` (`test/test_table_parallel.f90`) makes about 19 such calls per row and
  **SIGSEGV'd at roughly row 4500** — the ~7.7 MB those 85000 calls account for on macOS's 8 MB
  default stack, needing 32–40 MB to finish. gfortran reclaims per iteration and is unaffected.

  **The symptom names nothing useful.** The fault lands in the *callee's* prologue as
  `EXC_BAD_ACCESS (code=2)` at a guard-page address, with a backtrace that cannot unwind because the
  stack is gone, so it reads as a crash in the library or the test framework.

  **Diagnosis is three cheap steps, in this order**: raise the stack (`ulimit -s 65520`) and see if
  it passes; scale the fixture down and see if the requirement scales with it; then print the
  iteration counter to find where it dies. `code=2` at a `0x7ff7...` address is the tell — a write
  to the guard page, not a bad pointer. `-fno-stack-arrays` does **not** help, because these are
  expression temporaries rather than automatic arrays.

  **The fix is to hoist, never to raise the limit.** Build each message once above the loop into a
  `character(len=:), allocatable` and pass the variable; the temporary then does not exist. **Apply
  it whenever a loop of more than a few thousand iterations passes a concatenated or otherwise
  constructed `character` expression to a procedure** — the same shape hides in any assertion helper
  called per row.

#### nagfor-specific gotchas

nagfor needs `FPM_CC`/`FPM_CXX` set explicitly and a shim to get OpenMP onto the compile line; its
warning output needs its own triage, and both are covered in
[NAG's "explicitly imported but not used" warnings](#nags-explicitly-imported-but-not-used-warnings-most-are-false-positives).

- **nagfor UNMASKS the IEEE traps by default (`-ieee=stop`), for the WHOLE process — so anything
  this library links may not RAISE a flag, however harmlessly.** Two confirmed instances, both fatal
  on data containing nothing exceptional. **`arrow::compute::MinMax` raises `FE_INVALID` on every
  non-empty floating-point array**, benignly and independently of the values (a one-element array
  raises it, and so does an all-Null one; a length-0 array and an integer array do not) — so
  `parquet_close_reader(print_stat=.true.)` on a reader that had touched a float column died in
  Arrow with *"Arithmetic exception: Floating invalid operation"*. And **`anint(NaN)` and `int(NaN)`
  trap in Fortran**, turning the write path's "is this value integral?" test into a crash naming
  nothing, in place of an `error stop` naming the column. Comparisons (`<`, `/=`), `abs()` and
  `ieee_is_nan` are all quiet on a NaN on both sides of the language.

  So: **test with `ieee_is_nan` FIRST, as its own statement** (Fortran does not short-circuit, so
  `ieee_is_nan(v) .or. v /= anint(v)` still evaluates the `anint` and still traps), and mask the
  traps around a foreign call *known* to raise, with `feholdexcept` + `feclearexcept` +
  **`fesetenv`** — never `feupdateenv`, which re-raises on the way out and traps in the caller you
  were protecting. Scope such a guard to the one call: a file-wide guard would swallow a genuine FP
  fault in this library's own code, which is what a caller who unmasked the traps wants to see.
  `-ieee=full` makes a NAG build survive but disables the check for the user's code too, so it is a
  diagnosis, not the fix. gfortran, ifx and flang mask the traps, so **only a nagfor run can see any
  of this**. See `feature_risks.md` Risk-124.
- **nagfor 7.2 mis-evaluates the most-negative int64 CONSTANT combined with a runtime value, and
  gets overflow guards wrong in BOTH directions.** With `INT64_MIN = ibset(0_int64, 63)` as a
  parameter, `v < INT64_MIN - delta` answers `.true.` for `v = 19920, delta = -4`, and
  `INT64_MIN/scale` comes back with the **wrong sign**; `mod()` divides the same way. **Form the
  bound from `huge` instead; a local copy is NOT a fix** — the optimiser propagates the constant
  back, so the copy is correct at `-O0` and wrong at `-O2`+ once the guard sits in a module
  procedure, which is how a "fixed" guard still aborted under `--profile release` while a plain
  `fpm test` stayed green. **Printing the expression shows the right value** — only a comparison or
  a stored result reveals it, so a debugging session goes looking in the wrong place. The int32
  equivalent is unaffected, and so are gfortran, ifx and flang.

  **The same defect also cancels a common `ieor(., 2**63)` from BOTH sides of a relational**, which
  is invalid because XOR with the sign bit reverses the order it maps: the textbook unsigned
  comparison `ieor(a, K) < ieor(b, K)` collapses to the signed `a < b`, its exact inverse for a pair
  straddling `2**63`. That is a **wrong answer**, not a guard misfiring — it broke 13 of the 38
  golden integer vectors in `parquet_random`, returning different, in-range, uniform-looking draws.
  Write the rule out instead: `(a < b) .neqv. ((a < 0) .neqv. (b < 0))`. A most-negative constant
  used only inside `ieor` whose result is **stored** rather than compared — `SORT_SIGN_BIT` in the
  sort's radix keys — is safe, and safe for that reason alone. See `feature_risks.md` Risk-125.
- **A compile-time fork needs a check that RUNS under every compiler that selects an arm.**
  `tools/check_random_kernels.sh` could not run under nagfor at all (it hardcoded gfortran's `-cpp`;
  NAG spells it `-fpp`), so the one arm nagfor ships had only ever been verified by a gfortran
  forced onto it — an arm verified against codegen that never runs it.

#### C++ side (`src/parquet_wrapper.cpp`) gotchas

- **`src/parquet_wrapper.cpp` is NOT compiled with `-fopenmp`, so it cannot call any `omp_*`
  function at all.** `.gitlab-ci.yml` sets `FPM_CXXFLAGS: "-std=c++20 --coverage"` and a dev machine
  sets whatever Arrow needs — neither adds it, and `fpm.toml`'s `openmp = "*"` metapackage covers the
  Fortran half. **So anything on the C++ side that needs an OpenMP answer must have it resolved in
  Fortran and passed across the `bind(C)` boundary as an ordinary value.** The threaded sort works
  exactly that way: `pf_sort_threads` asks `omp_get_max_threads()`/`omp_in_parallel()` in Fortran and
  hands C++ a plain integer count, so the wrapper receives a number and never a policy. Do not add an
  "auto" sentinel to a `bind(C)` signature for the C++ side to interpret — it cannot.
- **A TEMPLATE cannot go in this file without its own `extern "C++"` block.** The whole file sits
  inside one enormous `extern "C" { … }`, and a template declared there fails with
  `error: templates must have C++ linkage` — a message that points at the template rather than at
  the linkage specification a thousand lines above it. Linkage specifications nest, so wrap just
  that declaration:

  ```cpp
  extern "C++" {
  template <typename F>
  static int64_t sort_spawn(std::vector<std::thread> &workers, int64_t lo, int64_t hi, F f) { … }
  }
  ```

  Taking a `std::function` instead works equally well when the call happens once per chunk rather
  than once per element; prefer the template plus `extern "C++"` when the callable is on a hot path.
  `sort_spawn` is the worked example.
- **`std::abort()` is not safe to call from many threads at once — it can hang the process
  forever.** glibc's `abort()` takes an internal lock, so when several threads reach it together they
  pile up on that lock and nothing ever terminates. Measured under ifx at `-O0 -check all`: the
  `concurrent_calls_into_shared_writer` scenario left **192 threads** parked in `futex_wait_queue`,
  every stack reading `__lll_lock_wait_private <- abort <- … <- __kmp_invoke_microtask`. It survived
  `SIGTERM` and needed `SIGKILL`. Neither that nor the occasional `SIGSEGV` in the same path is
  reproducible on demand — both need enough threads to arrive together.

  **This matters here because the concurrency guard is MEANT to be hit from many threads.** The fix
  is `claim_fatal_path_or_park()` plus `fatal_exit()`: an atomic claim so exactly one thread reports,
  and `std::_Exit(134)` instead of `abort()` — a bare `exit_group` syscall, no lock, no `atexit`
  handler, well defined from any thread inside or outside a parallel region. 134 is what a shell
  reports for a SIGABRT death, so nothing observing the exit status can tell the difference, and
  `_Exit` skips the gcov flush exactly as `abort()` did, so every `GCOVR_EXCL` marker resting on that
  mechanism stays correct. **Any new fatal path must go through those two helpers rather than calling
  `abort()`/`exit()` directly.** See `feature_risks.md` Risk-99.

### NAG's "explicitly imported but not used" warnings: most are FALSE POSITIVES

NAG is the only compiler in the fleet that reports unused imports and unused variables by default,
so it is the only one that can find a dead `use ..., only:` entry here — and the only one whose
warning list, taken at face value, will make you delete something load-bearing. A full survey needs
its own build tree and **`--verbose`**, because fpm prints compiler diagnostics in no other mode
(a plain `fpm build` looks perfectly clean at zero warnings, which is a very convincing way to
conclude there is nothing to do):

```bash
source ~/.activate_nag.sh
export PATH="$PWD/tools/nagfor_fpm_shim:$PATH"     # see tools/nagfor_fpm_shim/nagfor
FPM_BUILD_DIR=test_run/nag-warnsurvey fpm build --verbose 2>&1 | grep '^Warning:'
```

**Two classes of false positive dominate, and both are invisible to NAG by construction.**

- **A name reached by HOST ASSOCIATION from an ancestor.** NAG compiles one file at a time and
  cannot see that a descendant submodule uses a name its ancestor imported. This project leans on
  that heavily: `parquet_read_*`, `parquet_write_*`, `parquet_metadata_base/_get`, every
  `parquet_tables_*`, every `parquet_sorting_*` and every `parquet_columns_*` file has **no `use`
  statement at all** (the one exception is `parquet_metadata_maml.f90`, with two), so the module or
  submodule spec above them imports on the whole subtree's behalf. `parquet_tables.f90` and
  `parquet_sorting.f90` are entirely in this class — every name their specs import is for a
  descendant, so NAG flags nearly the whole list.
- **A name used only inside `#ifdef _OPENMP`** — a class that **only appears in a SERIAL NAG
  build**, and is therefore mostly historical. fpm's compile-side OpenMP probe fails under NAG
  (it hands nagfor `-fPIC`, whose NAG spelling is `-PIC`), so fpm puts `-openmp` on the link line
  only; `tools/nagfor_fpm_shim/nagfor` supplies it on every invocation to compensate, so a NAG
  build made through the shim compiles the `#ifdef _OPENMP` arms and does not flag their names at
  all. It comes back the moment the build is serial again — under the shim's
  `NAGFOR_OMP=0` opt-out, or without the shim on `PATH` — and there the trap is sharp:
  every OpenMP-only import, local and dummy reads as unused, deleting one breaks every OpenMP build,
  and **nothing in that NAG run will say so**, because in that run it really is unused. **The
  survey figures below were taken serially and predate this**, so re-derive rather than trusting
  their split.

**So the decision rule is three conditions, not one.** Remove a flagged name from a scope only when
(1) NAG flags it, **and** (2) it is absent from that file's own code with comments *and string
literals* stripped and **both** arms of every `#ifdef` included — a type-token string like `"int32"`
otherwise reads as a use of `int32`, and an `#ifdef _OPENMP` body otherwise reads as absent — **and**
(3) no descendant needs it through that scope.

**Condition (3) has to be solved GLOBALLY, because removals interact.** Checking each candidate
against the *current* imports is not enough: drop a name from an ancestor and from a descendant in
the same pass, and a leaf that was relying on the ancestor loses it. Compute the whole plan, then
verify that every use is still covered by the file itself or a surviving ancestor, and restore the
lowest ancestor that had it wherever it is not. `parquet_maml_missing_column` is the worked example
— dead in `parquet_core`, dead in `parquet_read`, dead in `parquet_write`, dead in
`parquet_metadata`'s own text, and needed by `parquet_metadata_maml`, which imports nothing of the
kind; it therefore has to stay in exactly one of them.

**The ground truth is a build with EACH compiler, and neither alone will do.** A removed name is a
hard compile error, never a silent wrong answer, so the compilers settle it — but only together:
gfortran carries `-fopenmp` from the `openmp = "*"` metapackage and so compiles the `#ifdef _OPENMP`
arms, NAG compiles the `#else`/absent arms, and between them every line is seen. A green gfortran
run says nothing about the serial arm and vice versa.

**Do not forget which files are generated.** `src/parquet_tables.f90`, `src/parquet_sorting.f90`
and `src/parquet_columns.f90` all carry flagged imports and all are emitted by a `tools/` script —
edit the generator's template text and re-run it with `--check`, per
[Some `src/*.f90` files are generated](#some-srcf90-files-are-generated--edit-the-generator-never-the-output).

**A residue of flagged imports is the healthy state, not a backlog.** After a sweep, what remains is
all in the two false-positive classes above, and it cannot be driven to zero without giving a dozen
descendant submodules their own `use` statements — duplicating their parents' lists, which is a
change to the tree's deliberate design rather than a lint fix. The same two causes, plus ordinary
interface conformance (a dummy a particular implementation does not need), account for most of the
"Unused dummy variable" and "Unused local variable" warnings, so those are likewise not a to-do
list.

**Its other diagnostic classes need the same triage, and four of them are correct-by-design here.**
`fpm.toml`'s `nagfor` feature turns on `-info` and several `-Warn=` categories, so a NAG run also
reports `Extension(NAG)`, `Questionable` and `Note` lines. Fixing these four would make the code
worse, so recognise them rather than acting on them:

- **`Last statement of DO loop body is an unconditional RETURN/EXIT/ERROR STOP`** — this is the
  `cycle`-guard search idiom, `do i = ...; if (no match) cycle; <act>; return; end do`, which this
  project uses everywhere a list is scanned for one entry. NAG reads only the body's textual last
  statement and cannot see that the `cycle` above it is what makes the loop a loop. Twelve sites,
  all of that shape. Restructuring one to satisfy the diagnostic is how you turn a working search
  into a first-element-only search.
- **`Expression in OpenMP IF clause is always .FALSE.`** — `!$omp parallel if(.false.)` is
  deliberate in `test/test_sorting.f90`: it opens an **inactive** region, the only way to reach the
  state where `omp_get_level()` is 1 while `omp_in_parallel()` is `.false.`, which is exactly what
  `pf_sort_threads` has to resolve to 1 for (`feature_risks.md` Risk-104).
- **`CONTINUE statement with no label`** (and its "did you mean CYCLE?" variant) — a bare `continue`
  is this project's spelling for a deliberately empty branch, including the `case (PK_STRING, ...)`
  arm whose whole job is to stop the string kinds reaching `case default`'s `error stop`.
- **`Non-standard intrinsic module OMP_LIB`** — inherent to using OpenMP from Fortran at all; the
  standard does not define `omp_lib`, OpenMP does. Twenty-three sites and nothing to do about any of
  them.

**`Questionable: Variable X set but never referenced` is worth reading every time, though — it
splits three ways.** Some are genuine dead stores and should go. Some are unavoidable: Fortran has
no way to call a function and discard the result, so a scenario whose whole point is that
`parquet_column_exists(...)` aborts must still assign it somewhere. And some are a **missing
assertion** — which is the reason to read the list rather than skim it. `test_no_stats_declines`
(`test/test_filter_screen.f90`) captured `parquet_debug_get_row_groups_pruned()` from its
prescreen-disabled arm and never checked it, which is precisely the half
[the statistics screen's own rule](#the-row-group-statistics-screen-every-uncertainty-must-decline)
says an A/B equality is not evidence without. Likewise a discarded `setenv` return code means every
assertion after it is made against an environment nobody set.

**`-thread_safe` is the noisiest of them and needs a scope-based triage, not a read-through.**
The `nag` profile passes it, and it reports every assignment to a variable from an outer scope —
several hundred across `src/`. Almost all are correct, and the discriminator is the scope the
message names, not the file:

- **A PROCEDURE scope** (`from scope LEX_FILTER_EXPR`, `from scope SCHEMA_ADD_FIELD`) is a contained
  procedure writing its host's locals — the recursive-descent parser advancing `pos`, `push_token`
  appending to the token arrays. Host locals are per-invocation, so there is nothing shared to race
  on. All fine.
- **No scope named at all** (`SELF cannot be C_LOC argument`, `SELF cannot be INTENT(INOUT) actual
  arg`) is NAG objecting to a *dummy* being passed on, not to shared state. All fine, and the
  largest class.
- **A MODULE scope is the only class worth reading.** These are process-global by design and split
  three ways: the `cfg_*` settings knobs (being global is the entire premise of
  [`parquet_settings`](#a-new-process-global-parameter-goes-in-parquet_settings-and-a-design-doc-must-say-so)),
  the `parquet_debug_*` overrides and counters (which
  [have to be globals](#a-fortran-side-debug-hook-has-to-be-public-so-prefer-a-c-one)), and a small
  number of genuine runtime counters — `seed_call_counter` in `parquet_random` and
  `affinity_clamp_claims` in `parquet_settings_base` today. Re-derive that last list rather than
  trusting it; it has moved twice.

**No amount of guarding silences it** — the check is static and has no notion of an atomic, a
critical region or a lock, so it fires on a correctly-synchronised global exactly as loudly as on an
unsynchronised one. That makes it useless as a pass/fail gate and useful as a *census*: run it when
you want the list of everything in the library that is process-global, then check each entry against
what it is supposed to be. `seed_call_counter`'s and `ek_dbg_force_fail`'s declarations carry that
reasoning at the site, which is the pattern to copy rather than re-deriving it from a build log.

**One-command census**, since the raw log buries the module-scope entries under the other 214:

```bash
grep '^Questionable:' <log> | grep 'thread-safe' | grep -oE 'from scope [A-Z0-9_]+' | sort | uniq -c | sort -rn
```

A scope name that matches a `module`/`submodule` in `src/` is process-global; anything else is a
procedure and can be skipped.

**`-C=all` is NOT the checked profile to use — `fpm.toml`'s `nagdeb` feature is, and the ONE check
it leaves out is left out for a stated reason.** `nagfor` is the only compiler here with real
runtime checking. Under bare `-C=all` the suite is unusable: 243 of 824 error scenarios die on
SIGSEGV, SIGBUS or a 120 s timeout. **Run it and read what it says** — `nagdeb` has found two real
standard violations that nothing else in the fleet can see, and it will find more, because it is
the only profile that checks anything at runtime.

- **`-C=intovf` is NAG's `-ftrapv`, and `-C=all` ALREADY IMPLIES IT** — so it is in `nagdeb`, and
  passing `--flag "-C=intovf"` on top changes nothing. Verified directly rather than assumed, with a
  three-line program: `nagfor -C=all` aborts on `huge(int32) + 1` with *"INTEGER(int32) overflow"*,
  while bare `-C` and a flagless build both wrap silently; `-C=all` does **not** check undefined
  variables, which is the separate exclusion below. It used to trap the deliberate wrapping
  multiplies in `src/parquet_random.f90`; since 2026-08-20 **nagfor takes a third, overflow-free
  arm** of that file's route (e) fork (`PF_SAFE64`), so there is no overflow left for it to trap.
  `-ftrapv` on gfortran is a *different* question and still aborts, because gfortran takes the
  int128 arm and `-ftrapv` instruments elsewhere — see
  [Compiler & language gotchas](#compiler--language-gotchas).
  **A flag blamed for a failure is a hypothesis, and "it passes without the flag" is exactly what a
  real defect that only a checked build can see looks like** — an intermittent `reading` segfault
  was blamed on this flag's instrumentation and turned out to be a non-short-circuiting `.and.`
  comparing two array sections of different extent, reported from several threads at once.
  **ifx deliberately keeps the wrapping arm**: the overflow-free spelling costs about 1.7x on
  `pf_random_at` under nagfor and 1.9x on a gfortran build forced onto that arm. Correctness is
  identical on all three arms and asserted bit-for-bit by `tools/check_random_kernels.sh`, so the
  split trades only speed against the ability to run a checking build.
- **`-C=undefined` is a build-clean, RUN-BROKEN option here, and is excluded from `nagdeb` for
  that reason.** nagfor miscompiles **every `bind(C)` call** under it — its instrumentation ABI
  interleaves a definedness-map pointer after every argument and appends a hidden length per map,
  into a call whose callee has the plain C signature, so only argument 1 survives and the rest
  arrive as garbage. **The manual says so** ("not compatible with calling C code via a BIND(C)
  interface"; "the whole program must be Fortran code and compiled the same way") — read an
  option's own manual section before diagnosing its behaviour. What makes it worth recording is
  that the violation is **silently accepted**, with no diagnostic at compile or run time, while the
  module-level half of the same restriction is a fatal compile error. Symptoms look like data
  corruption rather than a broken call, because a handle still arrives.

  **No workaround exists and the candidates were investigated to closure**: per-file exclusion is
  impossible (the option is stamped into every `.mod`, and the augmented call is generated at the
  *call site* anyway), and teaching the C side the instrumented ABI would mean ~200 shims against
  an undocumented, version-specific internal ABI whose every mistake is exactly the silent
  corruption the option exists to catch.

  **What DOES run under it: the all-Fortran suites.** A suite whose runtime paths never execute a
  `bind(C)` call runs genuinely — `fpm test run_tester --flag "-C=undefined" -- random` passes, as
  do `random_perm` and `random_weighted`, giving a real undefined-variable check over the
  random/sampling/expkey family. Do not expect to widen it: `columns` aborts on the value bytes of
  null rows, which are unspecified **by design** (`%init`'s documented contract — the checker and
  the design disagree, and per "Coverage tooling never drives design" the design wins), and
  `sorting` hides a file round-trip.

  **Three compile-time nagfor defects were found and fixed while getting the build clean under it,
  and two of the fixes are rules that must not be reverted.** (a) An ICE (`Panic: Cannot find scope
  id 0`) when compiling any submodule of a module declaring a `FINAL` bound to a **separate module
  procedure** — so **every `FINAL` target must stay module-contained**, and moving one into a
  submodule silently reintroduces the ICE for every sibling submodule. (b) Invalid C for the
  implicit finalization of an **array whose element type has a finalizable component** — fixed by
  deleting this library's two redundant finalizers, `parquet_string`'s (a non-owning handle) and
  `parquet_string_column`'s (deallocations F2018 9.7.3.2 already performs); **do not add either
  back**, and note the remaining four release C++ handles and OpenMP locks and must stay. (c)
  Invalid C for a pointer-valued function result used directly as an actual argument — bind it to
  a local pointer first.

**`-C=dangling` and `-C=calls` are BOTH in the set, and the one-word source change that let them in
must not be reverted.** Neither check is the ingredient: what matters is the `target` attribute on
an `optional`, assumed-size dummy receiving an **absent** actual. The twelve
`write_<type>[_chunk]_flat` workers declared `logical, intent(in), optional, target :: valid(*)`;
dropping `target` from those twelve dummies removes every crash and hang under either check and
under both together. It costs one `logical` array copy on the unmasked path, since `vmask` can no
longer point straight at the caller's mask; the full reasoning is in the banner comment above the
workers in `src/parquet_write_numeric.f90`. **Do not give that dummy `target` back**, and do not
read a green run as evidence that it would now be safe — both checks are in the set only because it
is absent.

**The methodological lesson is sharper than the fix: a minimal reproducer for a CODEGEN bug is
evidence about the reproducer.** The 19-line one said "either check alone runs", so the obvious
plan — run the suite twice, once under each check — looked sound; the real library then crashed
under `-C=dangling` alone. Adding one local variable to the reproducer later made it stop crashing
under *both*. An ingredient table from a reduced case establishes what is **sufficient** to trigger
the bug, never what is **necessary** in code the optimiser sees differently. Settle a flag question
by building the real thing.

**And `-C=dangling` earned its place on its first run, by finding a real standard violation
nothing else in the fleet can see.** `%view_all` associates each returned handle's `%col` with its
own `intent(in), target` dummy, so F2018 15.5.2.4 leaves those pointers **undefined** on return
whenever the actual argument has no `TARGET` attribute — the handles are then unusable, and
`build_from`'s `associated(handles(k)%col, self)` is itself non-conforming. Three call sites
declared the column without `target` (`test/test_string_parallel.f90`, and the `app/` string
benchmarks, which the suite never runs); gfortran, ifx and flang all execute them happily.
**Any procedure handing back a pointer into a dummy needs its callers checked for this**, and
`view_all`'s doc-comment already said "must be a target" — the contract was written down and simply
not honoured, which is exactly the class a runtime check exists for.

**What the remaining checks found is worth the trouble: three real standard violations, all the
same shape — a ZERO-SIZED thing referenced where the standard forbids it.** Each was invisible under
gfortran, which no-ops all three:

| site | what | check |
|---|---|---|
| `make_valid_buf` (`parquet_read.f90`) | `C_LOC` on a zero-sized array — F2018 18.2.3.6 requires nonzero-sized | `-C=pointer` |
| `set_all_*` (generated) | whole-array assignment to a zero-row column's storage, which `grow_storage` never allocates (`if (n == 0) return`) | `-C=array` |
| `build_from_character` / `append_values` (`parquet_strings.f90`) | unallocated `self%data` passed to `pack_character_bytes` when every element is blank | `-C=array` |

**The generalisable rule: a zero-length case reaches storage that was never allocated, because the
allocators here all skip zero deliberately.** `grow_storage` returns early at `n == 0`;
`ensure_data_cap` allocates nothing for a zero-byte payload. Both are right, and both mean any
"just assign the whole array" or "just pass the buffer" downstream is referencing an unallocated
allocatable. Look for it whenever a new bulk path is added, and note that the trigger is often a
**documented user pattern** rather than an edge case — the `set_all` instance is reached by
`%add_column(name, empty)`, which is exactly what the guide tells users to do to declare a column
before a parallel region appends to it.

**One debugging gotcha, because it cost a wrong diagnosis here: under test-drive's per-suite
parallelism the last `Starting <test>` line in the log is NOT the test that aborted.** Tests run
concurrently, so the abort belongs to whichever test was running on the faulting thread. The log
blamed `materialize_all reads columns in parallel`; the stack showed `test_table_parallel_append`.
Get the real one from a backtrace (`lldb -b -o "breakpoint set -n __NAGf90_rtcrash" -o run …`,
then `thread backtrace all`), or re-run with `OMP_NUM_THREADS=1`, which also stops the runtime
error message being interleaved with another thread's output mid-line.

**A NAG run needs `--verbose` and its own build tree; `-openmp` it gets by itself.**
`fpm test --verbose --flag "-colour -w=unused"` with `FPM_BUILD_DIR` set somewhere under
`test_run/` is the shape to use — `-w=unused` clears the noise this section's first half is about
so the rest is readable. **`tools/nagfor_fpm_shim/nagfor` supplies `-openmp` on every invocation**
(fpm cannot: its compile-side probe fails on `-fPIC`, so fpm only ever puts the flag on the link
line), which is what stops every threading test skipping — without it a green NAG run says nothing
at all about the parallel paths. `NAGFOR_OMP=0` is the shim's opt-out and the only supported way to
get a serial NAG build; it is a **master switch** that STRIPS an `-openmp` anything else passed
rather than merely declining to add one, so a profile or `--flag` cannot silently override it and
then report a serial arm that never ran. Passing `-openmp` yourself is harmless — the shim
deduplicates it, as it must anyway for fpm's doubled link flag.

**`fpm --verbose` does NOT show the flag, and never will — do not read that as it being absent.**
fpm prints the command it constructs and then invokes `nagfor`, which on `PATH` is the shim; every
rewrite the shim makes (this addition, the `-Wl,` fixes, the deduplication) happens *after* that
print, so fpm's output shows fpm's intent rather than what the compiler received. `--show-model`
has the same blind spot for the same reason. **Verify by behaviour, not by flags**: run
`string_parallel` and read the skip count, or set `NAGFOR_OMP=0` and confirm the count moves. This
is the same "a tool's silence is evidence about the tool" trap as the `machine_report.sh` one — a
flag can be in force and invisible, exactly as a compiler can be installed and unreported.

### Arrow's own type singletons have thread-unsafe lazy state on first concurrent use

Every no-argument `arrow::<type>()` factory (`arrow::int32()`, `arrow::utf8()`, `arrow::boolean()`,
...) returns a reference to a **process-wide, function-local `static` singleton** shared by every
thread. That alone is fine — the problem is that this project's apt-installed Arrow build
(`.gitlab-ci.yml`'s Arrow apt repository) has **confirmed, ThreadSanitizer-caught data races on
more than one kind of lazily-populated mutable state hanging off that same shared object**, each
found independently and each requiring its own fix:

1. **The singleton's own construction** (its `shared_ptr` control block). `arrow::int32()` et al.
   use the standard C++11 "magic statics" pattern, normally safe to construct concurrently for the
   first time since the compiler inserts a one-time-init guard — but TSan caught two OpenMP
   threads racing on `arrow::int32()`'s construction the first time each independently wrote an
   `int32` column at process/test-suite startup: a genuine race on the `shared_ptr`'s refcount,
   not a false positive.
2. **`arrow::detail::Fingerprintable`'s lazily-cached `fingerprint()`/`metadata_fingerprint()`**,
   which every `DataType` inherits (and which Arrow's own type/field/schema equality checks use
   internally as a fast path, so it is reachable from far more call paths than an explicit
   `->fingerprint()` call would suggest — the confirmed instance here was triggered from inside
   `parquet_close_writer`). Found in a SECOND, separate TSan run, after fixing (1) above did not
   make the underlying flakiness go away: two threads racing to populate this cache the first time
   they both touch the same shared singleton type, same underlying pattern as (1) but a
   completely separate piece of state.

Root cause not chased further than "the apt Arrow build behaves this way for both of these"; do
not assume a from-source Arrow build is affected the same way without re-checking.

**Why this was so hard to trace back to its actual cause**: corrupting a process-wide singleton's
state doesn't crash where it happens — it surfaces later, in whatever unrelated code next touches
the heap. This is exactly what made the original investigation (chasing a SIGSEGV inside
`table_check_not_detached`, a completely unrelated and trivially-simple boolean check) so
misleading, and why fixing race (1) alone looked sufficient locally but did not actually clear the
CI failure — race (2) was still there, waiting to corrupt something else. **If a future
concurrency bug report shows a clean-looking `error stop`/check failure immediately followed by a
crash in unrelated code, or a crash whose faulting line changes between runs (or between fixes),
suspect heap corruption from an early race over shared, lazily-initialized Arrow state before
assuming the crash site itself is where the bug lives — and don't assume fixing one such race
means there isn't a second, independent one still lurking. Re-run the sanitizer after each fix,
not just after the first.**

Fixed in `parquet_wrapper.cpp` (`ensure_arrow_type_singletons_initialized`, `std::call_once`-
guarded, mirroring `ensure_compute_initialized`'s existing pattern for Arrow's compute-kernel
registry): every no-argument `arrow::<type>()` factory this file uses is forced into existence,
AND has `->fingerprint()`/`->metadata_fingerprint()` called on it, exactly once, from a single
thread, at the top of both `create_parquet_reader` and `create_parquet_writer` — the two entry
points any OpenMP thread can reach first. After that one call, every later concurrent read is just
a read of already-published state, which is safe. **A parameterized factory
(`arrow::timestamp(unit)`, `arrow::decimal128(p, s)`, ...) is NOT affected** — those construct a
fresh, non-shared object per call rather than caching a singleton, so they have nothing to warm
up. **Keep the type list in sync with `parquet_wrapper.cpp`'s actual usage**: if a future change
introduces a new bare `arrow::<type>()` call site, add it to the list in
`ensure_arrow_type_singletons_initialized` too — grep the file for `arrow::` factory calls taking
no arguments to re-derive the exhaustive list if in doubt.

**This fix is deliberately NOT an exhaustive guarantee against every possible Arrow-internal lazy
cache** — enumerating Arrow's private implementation details one race at a time is not a fight
this project can definitively win. Two are now known and covered. If a THIRD, distinct race
against one of these same singleton objects ever surfaces, add whatever call reproduces it to the
same warm-up function rather than treating it as a one-off; if a third instance does show up,
reconsider a broader warm-up strategy (e.g. a full dummy write+close round-trip exercising every
supported type, single-threaded, in the same `std::call_once` block) instead of continuing to
enumerate individual private caches by name. See
[Thread safety](doc/pages/operating/thread-safety.md#a-note-on-arrows-own-type-singleton-construction) for
the user-facing writeup.

### gcovr <7.1 cannot parse gcov output for a 10,000+ line file

The CI `test:` job's `gcovr` step crashes with `gcovr.formats.gcov.parser.UnknownLineType` on
`src/parquet_wrapper.cpp`'s coverage data — reported as `<n>:10000-block 0` (then `10001-block N`,
`10002-block N`, ...). **These are real, valid gcov block-annotation lines for real source lines —
not corruption.** `src/parquet_wrapper.cpp` has grown to just over 10,000 lines (`wc -l`; it was
~7,400 when several older notes elsewhere in this file were written, and will keep growing — treat
any specific line count anywhere in this file as a snapshot, not a promise), and lines 10000/10001
are genuinely `if (!status.ok())` / `throw std::runtime_error(...)`. First suspected as heap/counter
corruption (from concurrent
OpenMP threads racing on GCC's `--coverage` counters, or from stale `.gcda` left over from an
earlier crashed run, or from the Docker reproduction's QEMU (amd64-on-arm64) emulation) — all
three were tested and ruled out: the crash reproduces identically on a genuinely fresh build, in
the real (non-emulated) GitLab CI pipeline itself, and adding `-fprofile-update=atomic` to every
coverage build changed nothing. **That last clause is about THIS crash only, and must not be read
as "the flag is unnecessary" — it is required for a different reason.** See "Measuring test
coverage" below: without it, concurrently-updated counters are silently lost and a genuinely
covered line reports as uncovered.

**Confirmed root cause: this is [gcovr issue #882](https://github.com/gcovr/gcovr/issues/882)**
("UnknownLineType thrown when parsing coverage data from 10K+ line file") — for a source file at
or past 10,000 lines, gcov drops the space between the block's hit-count field (or a `%%%%%`/
`$$$$$` exception-only-block marker) and the line number in its `-block N` annotation lines, and
gcovr's parser regex requires that space, so it throws instead of matching. Fixed upstream in
[PR #883](https://github.com/gcovr/gcovr/pull/883) ("Add support for more than 9999 lines"),
merged 2024-02-11, first released in **gcovr 7.1** (this project's CI environment was hitting it
on gcovr **7.0**, apt-installed from Ubuntu 24.04's package archive, which predates the fix and
will never receive it via a point release). [Issue #1103](https://github.com/gcovr/gcovr/issues/1103)
("GCovr on Ubuntu 24.04 Cannot Parse Coverage Reports") is another project hitting this exact
combination and confirms upgrading gcovr is the resolution — there is no compiler flag, source
change, or coverage-tool-invocation workaround; the parser itself cannot read this file's gcov
output below 7.1, full stop.

**Fix: install a `gcovr` version >= 7.1 rather than relying on the OS-packaged one.** Given
Ubuntu's own apt archive does not reliably track this (24.04 ships a pre-fix 7.0 as of this
writing, and a future Ubuntu LTS could just as easily ship another pre-fix snapshot), pin a
known-good version via `pipx` (already used for `fpm` in the same `before_script`) rather than
`apt-get install gcovr`. If a future `gcovr` release regresses this again, re-check
[gcovr's own issue tracker](https://github.com/gcovr/gcovr/issues) for "UnknownLineType" before
assuming it's a new bug in this project. This bound will need revisiting again as
`src/parquet_wrapper.cpp` keeps growing — the same class of off-by-one could recur at the next
power-of-ten boundary (100,000 lines) if gcovr's fix has any similar edge case, though nothing
currently suggests it does.

### gcovr 8.4+ drops coverage for module-contained Fortran subroutines

CI's `gcovr` step ran clean (exit 0, "All error scenarios behaved as expected") but the coverage
table came back almost empty: most `src/*.f90` files reported `Lines=0 Exec=0 --%` — not 0%
covered, but **no coverage data associated with the file at all** — while `src/parquet_wrapper.cpp`
and a small handful of `.f90` files (whichever happened to have no procedures directly `contains`ed
inside a `module`/`submodule`) reported correctly. Since virtually every procedure in this
project's `src/*.f90` lives inside a module or submodule's `contains` block (see "Nested submodule
tree" above), this wiped out nearly the whole Fortran coverage signal while leaving the C++ side
and the `coverage:` regex mechanism itself looking unremarkable — easy to misdiagnose as a
`gcovr`-can't-read-Fortran-at-all problem rather than a narrow, version-specific regression.

**Confirmed root cause: [gcovr issue #1253](https://github.com/gcovr/gcovr/issues/1253)**
("Missing coverage for Fortran module subroutines since gcovr 8.4") — gcovr 8.4 through at least
8.6 silently drops coverage for any Fortran subroutine/function contained inside a module,
apparently while filtering out compiler-generated symbols (debug output shows a real
module-mangled symbol like `__module_help_MOD_help_convert` being discarded as if it were a
compiler-generated one). Confirmed absent in gcovr 8.3. This is a **second, independent** gcovr
regression from the 10,000-line parser bug in the section above — one needed a version floor
(`>=7.1`), this one needs a version ceiling, and both bounds are load-bearing at once.

**Fix: pin `gcovr` to a range that clears the 7.1 floor and stays under the 8.4 ceiling** —
`.gitlab-ci.yml`'s `before_script` installs `gcovr>=7.1,<8.4` via `pipx` rather than an unbounded
`gcovr>=7.1`. If a future `gcovr` release fixes #1253, re-test before widening the ceiling; if a
new regression appears in some future version, check
[gcovr's own issue tracker](https://github.com/gcovr/gcovr/issues) for "module subroutine" /
"0 lines" before assuming it's a problem in this project — the symptom (a clean CI run with an
almost-empty coverage table) looks exactly like this one.

### `-fPIC` blocks inlining on ELF, so the same Fortran can be twice as slow on Linux as on macOS

**A Fortran module procedure is necessarily a GLOBAL symbol, so under `-fPIC` GCC must assume a
shared library could preempt it and refuses to inline it — every edge, at every optimisation level.**
GCC says so itself under `-fopt-info-inline-all`: *"not inlinable: A -> B, function body can be
overwritten at link time"*. This is a correctness barrier, not a heuristic, so raising the inline
budget does nothing (`-finline-limit=20000` produced a byte-identical object). Measured on this
project's sort comparator, same source, md5-identical inputs:

| flags | out-of-line calls in `sort_row_less` |
|---|---|
| `-O3 -funroll-loops -fPIC` *(what fpm builds)* | **16** |
| `-O3 -funroll-loops` (no `-fPIC`) | 0 |
| `-fPIC -fno-semantic-interposition` | 0 |
| `-fPIC -fvisibility=hidden` | 16 — does **not** help |

Three consequences worth carrying to any future hot-path work:

- **It is a platform difference, not a compiler-quality difference.** macOS is Mach-O, where ELF
  semantic interposition does not exist, so a macOS build never has the barrier. A measurement
  showing "machine A's compiler optimises this far better than machine B's" may be measuring only
  this. Appending `-fno-semantic-interposition` on Linux took one arm from **7.89 to 4.45 ns**.
- **There is no `fpm.toml` route to the flag**, so it cannot simply be adopted; it would have to come
  from `FPM_FFLAGS`, which on a dev machine carries Arrow's paths and must be appended to, never
  assigned (see "The three machines available for testing").
- **The source-side fix was tried and REVERTED, and the negative result is the useful part.** Moving
  the hot helpers into *internal* procedures — the only Fortran construct with local linkage, and
  yes, both `module procedure` and a private submodule-contained procedure are still global — did
  remove every out-of-line call, and made macOS **77-119% slower**. Do not re-attempt it without
  measuring on both platforms.

**The one-command check for whether a call was inlined**, which belongs beside any claim that it was:

```bash
objdump -dr --no-show-raw-insn <obj>/src_parquet_argsort_engine.f90.o \
    | sed -n '/<.*mp_sort_partition_>:/,/^$/p' | grep R_X86_64
```

**On macOS, `objdump` is a trap of its own**: Apple's toolchain emits `callq`/`call` and Mach-O
relocations rather than the `R_X86_64_PLT32` entries the command above greps for, so the same check
silently reports "no calls" — i.e. "fully inlined" — on a build where it has proved nothing. Use
`otool -tv` there, or run the check only on the Linux machine, and never quote a zero from it without
saying which platform produced it.

### Verifying the bind(C) boundary

A `bind(C)` interface (`src/parquet_bindings.f90`) has no compile-time link to the `extern "C"`
definition it describes in `src/parquet_wrapper.cpp` — gfortran and gcc each compile their own
half against the interface/definition text alone. A kind mismatch introduced by hand-editing
either side (e.g. a dummy silently changed from `integer(c_int32_t)` to `integer(c_int64_t)`
without the matching C++ parameter changing too) compiles cleanly on both sides and corrupts
memory silently at runtime, with no compiler diagnostic at all. Run `tools/check_bindc_boundary.py`
after touching either side of this boundary (a signature in `parquet_bindings.f90`, an `extern "C"`
function in `parquet_wrapper.cpp`, or one of the hand-written local `bind(C)` debug-hook interfaces
in `test/error_scenarios.f90`/`test/test_temporal.f90`) — it cross-checks arity, base type,
by-value-vs-by-reference, and (for functions) return type, and is wired into CI (`.gitlab-ci.yml`'s
`lint` stage) so a mismatch fails the pipeline rather than only surfacing at runtime. It does
**not** check length/ownership contracts, array rank, or NUL-termination conventions — see its own
module docstring for what's out of scope, and "The `parquet_strings` module" above for a confirmed
instance of that different class of bug.

### If `src/parquet_wrapper.cpp` is ever split into multiple translation units

A maintainability review considered splitting this single (now 10,000+-line, still growing) file and
decided against it
(see CONTRIBUTING.md's "Features considered but not implemented" for the full four-cost writeup,
and `src/parquet_wrapper.cpp`'s own `// ====`-banner comments, added instead, for a cheaper
navigability improvement). If that decision is ever revisited, the single most important, least
obvious hazard is this: **every process-global `static` at file scope means exactly one instance
per translation unit**, and this file has two families of them plus one singleton.

The `g_debug_*` test-only overrides (`g_debug_force_whole_column_read_error`,
`g_debug_string_offset_limit`, `g_debug_col_size_limit`, `g_debug_list_element_count_limit`,
`g_debug_column_count_limit`, `g_debug_force_sample_mask_error`,
`g_debug_physical_column_read_count`) are the first. Splitting the file without changing this would
silently give each new `.cpp` its own separate copy of every one. It would still compile and
link cleanly — there is no diagnostic for this — but any `parquet_debug_set_*` setter reachable from
`test/error_scenarios.f90` would then be writing to a *different* object than the guard code reads,
so the override would silently stop working and the corresponding error scenario would start
testing nothing at all while still reporting green.

**The settings mirrored from `parquet_settings` are the second family, and they are worse**, because
they affect production behaviour rather than only tests: `g_verbosity`, `g_message_stream`,
`g_sort_counting_path`, `g_sort_counting_bucket_limit`,
`g_target_row_group_bytes` and `g_statistics_prescreen`. `parquet_push_output_settings` and
`parquet_push_performance_settings` would write to their own TU's copy, and every read site in
another TU would keep the built-in initialiser — so a user's `parquet_set_verbosity("silent")` or
`parquet_set_target_row_group_bytes(...)` would apply to some of the library and not the rest, with
the Fortran getters still reporting the value correctly (`feature_risks.md` Risk-42).

**`g_next_thread_token` is a third case and fails in a way neither family does — silently, in
production, and only under concurrency.** It hands each thread the identity `ConcurrencyGuard` uses
to decide whether a claim on a reader/writer handle is a re-entry by the owner or a collision with
another thread. Per-TU copies would each start counting at 1, so two *different* threads would be
issued the same token and each would be waved through a guard the other holds — turning the
library's one defence against concurrent misuse into a silent no-op on exactly the code paths it
exists to protect. Nothing would fail to build, no test asserts a token value, and the symptom
would be the heap corruption the guard was added to prevent.

**`g_fatal_claimed` is a fourth case, and per-TU copies would restore a hang that was measured.**
It is the flag that lets exactly ONE thread report a fatal error and end the process; every other
thread parks. One copy per translation unit would admit one thread *per TU*, which is precisely the
several-threads-terminating-at-once situation it exists to prevent — see the "many threads calling
`abort()`" note under "Compiler & language gotchas".

Before any split, promote every global in all three families — and this counter and this flag — to a
genuine `extern`
global with exactly one definition in a shared internal header (not `static`), re-run every affected
error scenario to confirm the override still takes effect, re-run `test/test_settings.f90`'s
observed-effect tests to confirm each mirrored setting still reaches the code that reads it, and
re-run both `concurrent_calls_into_shared_*` scenarios to confirm the guard still fires **and that
exactly one message reaches stderr**.

### A hand-run `gfortran` without `-J` leaves a `.mod` in the repo root, and a global gitignore hides it

Compiling a source file by hand to check something — `gfortran -O2 src/parquet_expkey.f90 tools/x.f90
-o /tmp/x` — writes the `.mod` into the **current directory**, which is normally the repository root.
It is a build artifact nobody notices, because `*.mod` is commonly covered by a user's *global*
gitignore (`~/.gitignore_global`), so `git status` stays clean and the file can sit there for days.

**What it then breaks is a standalone check, and the error blames the library.** gfortran searches the
cwd for modules, so a stray root `.mod` shadows a script's own `-J<workdir>` output. If the stray was
built by a different gfortran, every configuration fails with `Cannot read module file ... created by
a different version of GNU Fortran`, pointing at a `use` line in the check's own driver. Confirmed on
`tools/check_exp_key.sh`, where all nine configurations failed this way and the script correctly
reported "NO configuration built -- this run proves nothing".

- **Always pass `-J<dir>` when compiling by hand from the repo root**, or run the compile from a
  temporary directory.
- **A `tools/` script that compiles sources must run the compiler from its OWN work directory with
  ABSOLUTE source paths** — `(cd "$WORK" && $FC ... $ABS_SRC)`. `check_random_kernels.sh` and
  `benchmark_random_kernels.sh` always did; `check_exp_key.sh` did not and was fixed to match.
- **`ls *.mod *.smod` in the repo root** is the one-command check when a standalone build fails for
  no reason; `git status` will not show them.

### Stale `fpm` build cache

**`fpm build` does NOT build anything under `test/`, so editing a `test/*.f90` file and then
running `fpm build` leaves the previous test binary in place — and running it afterwards measures
the edit you did not make.** It compiles the library and the `app/` targets and reports "Project
compiled successfully", which reads as confirmation. This cost a full diagnostic detour: four
rounds of adding `print` markers to an `error_scenarios` scenario, each rebuilt with `fpm build`,
each apparently proving the markers never executed — the binary predated all of them. Use
**`fpm build --tests`** to compile the test targets without running them (which is what you want
when the next step is running one scenario binary by hand), or `fpm test`. The tell is that a
marker on the FIRST executable statement of a block does not appear: code cannot skip its own first
line, so the binary is old.

**And when mutation-testing, delete the test binary rather than trusting fpm's staleness check.**
`fpm test` reported a mutation as *caught* here when the `run_tester` binary in fact predated the
restore and still carried an earlier mutation, printing "Project is up to date" throughout. The
verdict was inverted — a genuine coverage gap read as covered. `find build -name run_tester -type f
-delete` before each round, or `fpm clean --skip`, is what separates one round's result from the
previous one's. Cross-check anything surprising with a *separately built* program: an independent
probe reproducing the same shape is what exposed both incidents.

If `fpm test` behaves unexpectedly after source changes (e.g. a test target seems to run old
code), try `fpm clean --skip` to force a clean rebuild before spending time debugging — fpm's
build cache can serve a stale binary. `--skip` avoids rebuilding external (non-project)
dependencies, which are never the source of this problem, so it's faster than `--all` here.
Building with several different `FPM_FFLAGS` creates multiple `build/gfortran_<hash>/` dirs, which
a `find build -name error_scenarios | head -1` lookup resolves by guessing — symptom: tests pass
when run scoped but fail under a full `fpm test`. `test/test_errors.f90` no longer guesses:
`get_error_scenarios_bin` derives the sibling binary from **argument 0**, which names the exact
tree fpm launched this run_tester from, and only falls back to build-and-`find` when argument 0
cannot answer (someone running the built binary directly). `tools/run_error_scenarios.sh` is
standalone and still has to `find`, so the hazard remains there. `tools/coverage.sh` runs
`fpm clean` up front to avoid it; for a plain `fpm test`, `fpm clean --skip` fixes it.

**A change to a GATE is the most misleading form of this**, because a stale binary makes the tests
agree with you. Opening `parallel_prefetch_ok`'s refusal clauses and re-running left the two tests
that assert the refusal still reporting **PASSED** — i.e. the evidence said the change had not taken
effect, which reads as "my edit was wrong" rather than "my binary is old". `fpm clean --skip` showed
both failing, as intended. Treat a gate that appears not to have changed as a cache symptom first.

**A restored `src/parquet_wrapper.cpp` is the case fpm most reliably misses, and it bites hardest
during mutation testing.** Reverting that file (`cp backup src/parquet_wrapper.cpp`, `git checkout`,
a stash pop) and re-running `fpm build` repeatedly left the *mutated* object still linked — the
suite kept failing with the mutation's own symptom after the source was demonstrably clean, which
reads exactly like "my revert did not work" and invites a hunt for a second bug that does not
exist. `fpm clean --skip` fixes it. **So: after reverting a C++ mutation, `fpm clean --skip` before
believing any result** — and treat a failure that persists across a verified-correct source as a
stale-cache symptom first, not a new defect.

**`tools/coverage.sh`/`tools/coverage_cpp.sh` clean up after themselves** (they delete their own
`build/gcov`/`build/gcov-cpp` tree on exit, since an instrumented `error_scenarios` binary left under
`build/` is indistinguishable to the lookup below) — `COVERAGE_KEEP_BUILD=1` keeps it, and then it is
yours to delete before the next plain `fpm test`.

**`tools/run_error_scenarios.sh` resolves its binary the same way** (`find "${FPM_BUILD_DIR:-build}"
-type f -name error_scenarios | head -n 1`), and there the failure direction is the dangerous one: it
can print **"All error scenarios behaved as expected"** while running a binary that predates your
edits entirely. A green error-scenario run is therefore only meaningful when `find build -type f -name
error_scenarios | wc -l` is 1. Run `fpm clean --skip` first whenever you have built with more than one
`FPM_FFLAGS` value in a session (a coverage run, an OpenMP run, a release build), and treat a *new*
scenario that passes first time with suspicion until you have seen it fail against a deliberately
broken implementation.

### Keeping `tools/prep_fpm_publish.sh` in sync

`tools/prep_fpm_publish.sh` builds the tarball content for `fpm publish` (see CONTRIBUTING.md's
"Publishing to the fpm registry") by committing a disposable local branch that strips
maintainer/CI-only files and edits `fpm.toml` (comments out `test-drive`, flips
`module-naming` to `"parquet"`). **`app/` and `tools/` are ALLOW-lists (`APP_KEEP`, `TOOLS_KEEP`);
only the repository root is a strip-list (`REMOVE_PATHS`).** That asymmetry is the whole design: in
the two directories where files are added often, a new file is excluded by default, so forgetting
costs nothing. This list/logic silently goes stale unless updated alongside the change that
invalidates it — watch for these triggers:

- **A new file lands under `tools/` or `app/`.** If it is maintainer/CI-only — which nearly all of
  them are — **do nothing**: the allow-lists sweep it into `REMOVE_PATHS` automatically, `tools/` from
  `git ls-files` and `app/` from a disk glob. Only a **consumer-facing** addition needs an edit, and
  it must be added to `TOOLS_KEEP`/`APP_KEEP` or it will be stripped from the published tarball. The
  four consumer-facing tools today are `generate_parquet_maml.sh` (see
  `doc/pages/utilities/embedding-maml-schemas.md`), `generate_user_table_code.py`,
  `convert_fits_to_parquet.py` and `parquet_metadata_to_md.py`; `prep_fpm_publish.sh` is in
  `TOOLS_KEEP` too, for the mechanical reason that a running script should not delete itself.
- **An allow-list entry is renamed or moved.** This is the one failure mode the inversion creates,
  and it is the dangerous direction: a stale `TOOLS_KEEP`/`APP_KEEP` entry stops matching, so a
  *consumer-facing* file is silently stripped. The script validates both lists exist before it
  creates the disposable branch, so it fails immediately with zero side effects — but, as with
  `REMOVE_PATHS`, only once someone actually runs it.
- **A new maintainer/CI-only file lands at the repo root** (another CI config, another
  AI-instructions-style file, etc.) — same call: add to `REMOVE_PATHS` if it's not
  consumer-relevant.
- **A new module is added to `src/`.** It must be named `parquet` or start with `parquet_`, and
  **nothing on `main` will tell you otherwise**: `fpm.toml` carries `module-naming = false` there,
  so a badly-named module builds, tests and ships in the working tree indefinitely — the
  constraint only appears when this script flips the setting to `"parquet"` for the registry,
  which may be months later and after the name is already in downstream code. Confirmed by
  flipping the setting and adding a `module example` to `src/`:
  `ERROR: Module example in ./src/example.f90 does not match its package name (parquet-fortran)
  or custom prefix (parquet)`. **`test/*.f90` is exempt** — test modules are not checked, which is
  why `test_table` and friends are fine and why the asymmetry is easy to mistake for "the rule
  does not apply to us". This matters most for a *generated* module, where the name comes from
  data rather than from a person: `tools/generate_user_table_code.py` takes it from its MAML's
  `dataset:` key, so a schema naming a module `example` produces a package that cannot be
  published.
- **A root `REMOVE_PATHS` entry is renamed or moved.** Update the path string. The script's
  pre-flight existence check turns a stale entry into an immediate, zero-side-effect failure
  rather than a silently-wrong tarball — but only once you actually run it; nothing catches this
  at edit time. (Only the root entries are hand-written now; the `app/`- and `tools/`-derived
  entries cannot go stale, since they are enumerated from what is actually there.)
- **A new dev-dependency is added to `fpm.toml`** — check whether its own modules comply with fpm's
  [module-naming rules](https://fpm.fortran-lang.org/registry/naming.html) before adding it. If
  not, it needs the same "comment out in the disposable branch" treatment as `test-drive`, or it
  will reintroduce the build-breaking conflict documented in CONTRIBUTING.md.
- **The exact literal text of the `test-drive.git = ...` or `module-naming = false` lines in
  `fpm.toml` changes** for unrelated reasons — the script's own `SystemExit` checks already catch
  this by failing loudly, but it's worth knowing why a future `fpm.toml` edit might break the
  publish script.
- **Upstream fpm or `test-drive` fixes the module-naming compliance gap** (fpm PR
  [#828](https://github.com/fortran-lang/fpm/pull/828) / issue #883) — if fpm ever gains a
  per-dependency naming exemption, or `test-drive` renames its modules to comply, revisit whether
  the whole `test-drive`-comment-out workaround (and possibly `module-naming = false` on `main`)
  is still needed at all.

### Manual (never-`fpm test`) large-scale/benchmark tools

A user/maintainer-runnable check that needs more memory/disk/time than `fpm test`/CI should ever
attempt (e.g. genuinely exceeding `huge(1)` rows, or a multi-GB benchmark file) goes under
**`bench/`** — the program AND the thin `*.sh` wrapper that drives it, side by side, with env-var
config — never under `test/`, and never under `app/`, which now holds only the one program that
ships. `bench/` is an fpm source-dir, so a program there is built like any other executable and is
still never auto-picked-up by `fpm test`, unlike anything under `test/`. See `bench/benchmark_threads.f90`/
`bench/benchmark_threads.sh` and `bench/large_scale.f90`/`bench/large_scale.sh` for the
established shape: CLI flags (`--key=value`) parsed via `get_command_argument` in the Fortran
program; env vars read and forwarded as those flags by the shell wrapper
(`NAME="${NAME:-default}"` then `fpm run <app> -- --key="$NAME"`); `set -euo pipefail`; `cd` to
the repo root first.

**`bench/` is an fpm SOURCE-DIR, so a `.f90` dropped there is compiled by fpm — which is wrong for a
STANDALONE driver.** `fpm.toml` names one `bench/` program in an `[[executable]]` block, and that
registers the whole directory for auto-discovery (measured against fpm 0.13.0 alpha; if a future fpm
drops that, the symptom is the other programs silently not being built, so check the target count
after an upgrade). The consequence: a driver that must be compiled by a bare compiler with forced
flags — because forcing the other arm of a `#ifdef` fork would not let the package build at all —
cannot live in `bench/`. The four such drivers stay in `tools/` beside their wrappers'
siblings: `check_random_kernels.f90`, `check_exp_key.f90`, `check_argsort_standalone.f90` and
`benchmark_random_kernels.f90`. Each says so in its own header; the last is the one to notice,
because its `.sh` wrapper *is* in `bench/`, so the pair is deliberately split.

Document usage (parameters, defaults, example invocations) in
CONTRIBUTING.md's "Other tools/ helpers" section, not README.md — this is a contributor/
maintainer tool, not part of the public library API.

**Three traps when writing one of these, all of which produced a confidently wrong number here
before being noticed:**

- **Warm the data before timing anything, on any lazy API.** `parquet_open_table` reads no column
  data, so whichever measured path touches a column *first* silently absorbs the entire decode. This
  made `%get` look 8.94x `%col`, and `parquet_write_table` look 2.5x a hand-written write loop; both
  collapsed to roughly parity once the benchmark called `%prefetch`/`%materialize_all` up front. If
  two paths in one benchmark read the same column, exactly one of them is paying for it.
- **Warm the result array's pages too.** A freshly allocated large output array pays first-touch page
  faults on its first pass and never again, so whichever variant runs first absorbs them. This is
  strong enough to reverse a comparison: the `%col` pointer form measured *25% faster* than plain
  arrays purely by running second. Write the result once through every path before the timed loop.
- **Always pass `--profile release`; never try to get optimisation out of `FPM_FFLAGS` instead.**
  fpm applies profile flags only when `--profile` is given, so `FPM_FFLAGS="-fopenmp" fpm run` with
  no profile builds at **-O0** — not because `FPM_FFLAGS` suppressed anything (it does not; see
  "Don't run the GitLab CI pipeline yourself" for the four-way table), but because no profile was
  asked for. A bare `fpm run` with `FPM_FFLAGS` unset is equally unoptimised. And the `-fopenmp`
  there is redundant anyway. Reaching for `FPM_FFLAGS="-O3" fpm run` instead optimizes the Fortran
  half while leaving the C++ half at whatever
  the environment supplies — which on a dev machine may be nothing. Measured 5.7x on `pf_argsort`
  alone between the two, which is enough to invert a comparison and did: an early run of the sort
  benchmark showed bit-packing *losing* below 2M rows, an artifact that vanished under `-O3`. The
  `tools/*.sh` wrappers already pass `--profile release`; match them.
- **And asking for `--profile release` is NOT the same as getting optimisation — VERIFY it, because
  a wrapper cannot feel the difference.** fpm 0.13.0 alpha has **no release profile for flang**:
  `--profile release` under `FPM_FC=flang-mp-22` emits `-cpp` and the `-I` paths and nothing else,
  so the whole run is an `-O0` run that looks exactly like a valid one. Measured cost of not
  noticing: a plain Fortran array read at **5.43 ns instead of 0.951**, and the campaign's headline
  figure **3.7x** too slow — from a flag the wrapper believed it had set. The tell is one command,
  `fpm build --profile release --show-model | grep -o 'fortran_compile_flags="[^"]*"'`, and a
  benchmark wrapper should **assert `-O` appears there and refuse otherwise**, exactly as it would
  refuse any other configuration it cannot engage. `bench/benchmark_colindex.sh` does; the older
  wrappers do not. Recovery is to append the flag (`FPM_FFLAGS="${FPM_FFLAGS:-} -O3"`, appended
  never assigned) and say so in the report.

  **And the check must cover the C++ HALF too, which is a separate flag line that a Fortran-only
  guard cannot see.** fpm derives the C/C++ compiler from the *Fortran* one's family, and for a
  family it does not recognise as a C family it emits **no profile flags for that half at all**.
  Measured on this repository: `fpm build --profile release --show-model` gives `cxx_compile_flags`
  `-O3 -fPIC -funroll-loops` under gfortran and **nothing** under nagfor and flang — so
  `src/parquet_wrapper.cpp`, and with it the whole Arrow layer, compiles at the C++ compiler's own
  `-O0` default. Every benchmark in this project that opens a file, sorts, or reaches a
  `parquet_debug_*_cpp` arm passes through that file.
  **The failure is silent and plausible**: the run satisfies the Fortran guard, prints a full table,
  and every C++-side figure in it is wrong. `benchmark_sort_readtime` under nagfor at n = 10⁶
  reported one phase at **25.9%** of the operation with an unoptimised C++ half and **2.9%** with
  `-O3`, with the total at 97.45 ms against 36.91 — and nothing in the output said which it was. The
  recovery is `FPM_CXXFLAGS="${FPM_CXXFLAGS:-} -O3"`, appended never assigned, because that variable
  carries Arrow's include paths. Add the assertion to any wrapper whose timed work reaches C++; a
  wrapper measuring Arrow-free Fortran (`benchmark_random.sh`, `benchmark_random_kernels.sh`) does not need it and
  should not gain a gate it cannot fail meaningfully.

  **But "no `-O` in the flags" does NOT mean "unoptimised" — some compilers optimise by default, and
  a check that does not know this blocks a valid arm.** fpm gives **ifx** no `-O` either, and ifx's
  own default is **`-O2`**, so that build is already optimised and refusing it is wrong. This cost a
  whole toolchain arm on one machine before it was diagnosed. A wrapper needs to distinguish "no
  flag, therefore `-O0`" (gfortran, flang) from "no flag, therefore this compiler's default" (ifx,
  icx), which means carrying a short, **evidence-based** list of compilers whose default is
  optimised — short because a wrong entry turns the check into the silent `-O0` run it exists to
  prevent. And note the trap in the escape hatch: someone who reaches for a `SKIP_OPT_CHECK`-style
  override is measuring a different configuration from someone who appends `-O3`, so a campaign that
  mixes the two across machines has quietly stopped comparing like with like.
- **To find out WHERE the time goes on a per-element path, use a COMPILE-OUT LADDER, not phase
  timers.** This project's usual instrument is `parquet_debug_get_*_nanos` counters around each
  phase — how the filter's "93% of it is one helper's parameter type" finding was made — and it has
  a hard limit: a hook may sit on a coarse operation, **never on a per-row or per-element path**. A
  `steady_clock::now()` pair costs 20–25 ns, so on a ~37 ns per-cell accessor the timer costs more
  than the thing timed and perturbs the code under study. The ladder inverts it: put each phase
  behind its own cpp macro, build one binary per rung, and time the whole operation with one phase
  removed at a time — **each rung's difference from the baseline is that phase's cost**, and no
  timer goes near the hot path. `bench/bench_resolve_ladder.py` is the worked example (eight rungs
  over `table_resolve`), and it settled in one afternoon a question three machines' worth of
  end-to-end measurement had left open: the name lookup is **52–77%** of a per-cell read on four
  toolchains, and every other phase is unresolvable.

  Five rules the technique needs, all learned by breaking them:

  - **It measures REMOVAL, not attribution.** A phase whose removal frees the optimiser to improve
    what remains over-reports, so a rung is an *upper bound* on what fixing that phase could
    recover, not a promise.
  - **Every rung is a rebuild**, so it needs a cross-build floor (next bullet), not a re-run floor.
  - **Some rungs will not be semantically neutral** — removing a guard changes behaviour, and one
    rung that bypasses a lookup deliberately returns the wrong answer. Keep the scaffolding on a
    throwaway branch, never on `main`, and have the harness **print which rung it is** so a figure
    cannot be filed against the wrong binary.
  - **Run the test suite on the NO-MACRO build.** That is what proves the scaffolded default really
    is the shipped path — one campaign's `fpm test` passing with an identical assertion count under
    two toolchains is what made its baseline trustworthy.
  - **Apply the scaffolding with a script that validates every anchor before writing anything**, and
    do not write a `--revert`: `git checkout` already reverses it exactly, whereas a hand-rolled
    reverse edit is a second thing that can be subtly wrong. A *partially* applied ladder compiles,
    runs, and silently measures a configuration nobody asked for.
- **Take TWO independent measurements of the same binary, and use sign-agreement as a validity
  filter.** A benchmark with two differently-shaped modes over one build — say a tight per-call loop
  and a realistic multi-column loop with arithmetic — gives a check that costs nothing extra: **a
  change that genuinely removes work must move both, in the same direction.** Every campaign report
  that used this converged on the same rule, independently: a rung whose two modes disagree in sign
  is unresolvable, whatever its magnitude looks like in either one. It also catches the opposite
  case — one campaign found a rung the two modes sized an order of magnitude apart, which turned out
  to measure a property of the *fixture* (the declared width of the harness's column names) rather
  than of the library.
- **A variant that changes the TIMING but not the ANSWER is measuring pure overhead**, and that is
  worth saying in the report rather than leaving implicit. One rung removed a reserved-name
  comparison and left the checksum bit-identical — which proves the 4 ns it saved was being paid on
  every call for a feature that call was not using. Conversely, a rung whose checksum moves is a
  timings-only rung by construction and its answers must not be compared with anything.
- **A build-flag-selected benchmark must PRINT which variant it is, and carry an arm the flag cannot
  touch.** Two arms that differ only by a compile flag are indistinguishable in a log, so a figure
  can be filed against the wrong binary with nothing to catch it — the same failure the "negative
  control" rule guards against in tests, one level down. Two cheap defences, and one campaign used
  both: the program prints the variant it was compiled with (`guard_variant()` in
  `bench/benchmark_colindex.f90`), and the run includes an arm the flag provably cannot affect. The
  second doubles as the cross-build floor above, and in one report it was what proved a macro had
  done what it claimed — the control stayed flat under one compiler and moved 16% under another,
  identifying the second as layout rather than measurement.
- **A benchmark's own index arithmetic is CODE, and `mod` with a runtime divisor is an integer
  division.** `i = 1 + mod(k - 1, nrows)` is the obvious way to walk rows in a timed loop, and on
  x86-64 it compiles to `idivq` — **~6 ns per iteration**, which in one campaign was the *entire*
  reported "pointer read floor" and larger than several of the quantities being measured. It was
  caught by counting the instruction in the object (`objdump -d` → 126 `idivq` in the harness, 0 in
  an identical standalone loop at the same flags), not by reading the source. arm64 absorbed it, so
  the same harness looked fine on one machine and was wrong on another. Use a wrapping counter
  (`i = i + 1; if (i > n) i = 1`). **Differences between arms survive a defect like this; ratios
  against a floor do not** — which is the general rule, since a benchmark's floor row is exactly
  where its own overhead hides.
- **A REBUILD noise floor is a different, much larger quantity than a RE-RUN noise floor, and a
  flag-selected comparison needs the former.** Two machines found this independently in one
  campaign. Re-running one binary reproduced to **0.19%–2.5%**; rebuilding the *same source* with a
  cpp flag moved untouched arms by **11–16%** — `%set_at` 6.030 → 6.956 ns between two builds that
  generate identical code for it, and a `%is_null` control moving **+16.2% in a direction that is
  impossible**. That is code layout and alignment, not cost. Anything measured by comparing two
  builds — a cpp variant, a compiler flag, LTO, a mutation — must therefore quote a floor taken
  **across rebuilds with an untouched control arm**, or it will report layout as a finding. The
  re-run floor would have licensed calling a 1.2 ns gap real.

  **But that floor is a property of the PAIR OF BUILDS, not of the machine — measure it per campaign
  and never carry one forward.** The same machine, the same control arm, the same compiler gave
  **0.491 ns** across an eight-macro ladder and **0.003 ns** across a two-commit A/B differing by
  nine lines of Fortran: a **150x** spread, in the direction that would have disqualified a perfectly
  resolvable comparison. The rule of thumb that follows is worth having — **the more the two builds
  differ, the noisier the comparison**, so a two-commit A/B is a far quieter instrument than a macro
  ladder and should be preferred whenever the question can be posed that way.
- **After a fix, re-run the DIAGNOSTIC that found the problem, not only the end-to-end measurement.**
  An end-to-end delta says the number moved; it cannot say *why*, and when a prediction misses there
  is no way to tell a mis-derived prediction from a fix that underperformed. Re-running the ladder
  rung is what separated them here: it collapsed **33.90 → 0.45 ns** under ifx and held at
  **4.26 → 5.31 ns** under gfortran, proving gfortran's block never contained the redundant work and
  that its measured null was the correct answer rather than a failure. Without it, a correct fix
  would have been written up as an unexplained miss and the next move would have been to go looking
  for a fault in it. **A prediction chained from removal-based estimates is especially fragile** —
  removal over-reports by construction, so chaining two of them (`block − lookup_proper`) carried a
  2 ns error here and a 4.26 ns misattribution on the other toolchain.
- **A benchmark that REPLICATES library code is untested code — validate it against the real number
  before believing any of it.** Taking a loop apart sometimes needs a copy of it in the benchmark,
  because the library has no entry point that runs one half. That copy can be subtly wrong in a way
  no test covers. Require one of its rows to reproduce an end-to-end figure you can measure directly:
  the S7 validation cost model was only evidence once its `bit seen` row matched
  `reindex − reindex_trusted` (3 % at 4 M rows, 11 % at 16 M). Without that tie-down its other rows
  would have been three confident numbers about nothing.
- **Sweep the input SHAPE, not just its size — a ranking can invert.** Measuring one shape gives a
  confident answer to the wrong question. A byte-per-element seen-set beat the bit-packed one by
  **3.2x** on a reversal permutation and lost by **1.6x** on a scattered one at 16 M rows, because a
  reversal is simultaneously the best case for the seen-set's locality and the worst case for the
  bit-set's store-to-load forwarding, while the byte-set's 8x memory only bites once it misses cache.
  Both were needed to reach the right conclusion, which was to change nothing.
- **A sweep must ENGAGE the mechanism it is testing, and one that does not looks exactly like one
  that does.** Work out the condition under which the thing being studied is even active, and pick
  the sweep point from that — never from whatever size the previous measurement happened to use,
  because a threshold measured on one machine usually moves when the team size or the cache does.
  Worked example, and it very nearly retired a real item as closed: the sort's refine target is
  `max(nv/team, 2048*nt)`, so its floor binds only while `n < 2048*nt²` — 131 072 rows at 8 threads
  against 8.4 M at 64. A cardinality sweep at n = 10⁶ and 8 threads therefore found **no effect at
  any cardinality**, on two different fixtures, because the floor was inert; the same sweep at
  n = 10⁵ found a 1.9x cliff immediately and reproducibly. Nothing in either run's output
  distinguished the two. This is the benchmarking twin of the size-threshold trap under
  [Verifying a change with mutation testing](#verifying-a-change-with-mutation-testing), and the
  same fix applies — give the constant a `parquet_debug_set_*` override and **prove the effect in
  both directions with it**, since "disabling the mechanism removes the effect" and "forcing it on
  reproduces the effect elsewhere" together identify the cause where either alone only suggests it.
- **Measure what a restructure costs SERIALLY before assuming the threaded form can replace the
  original.** Making a rebuild splittable usually adds a pass, and that pass is charged to every
  caller below the work floor and every caller already inside a parallel region. Three of the four
  threaded rebuilds in `parquet_strings` kept their original as a serial twin for this reason, and
  `gather` is why it is worth measuring rather than guessing in either direction: written without a
  twin, its phased form came out **1.5x slower** on one thread (0.0140 s against 0.0094 s).
- **A LONG sweep drifts, so a figure from its tail is not comparable with one from its head.** This
  is the "never measure immediately after a heavy phase" rule applying *within* a single run, and it
  is easy to miss because the sweep looks like one measurement. Confirmed instance: in a 22-shape
  distribution sweep the 21st figure came out at **163.49 ns** against the 1st shape's 42.51 — read
  as a 50% regression against a 108.97 baseline, and very nearly reported as one. The three shapes
  involved turn out to use **byte-identical data with identical keys** (the benchmark's validity mask
  is passed only to its single-key families), and re-measured in isolation they are **43.36 / 43.21 /
  43.77**. It was thermal or memory drift on a laptop, nothing else. So: **re-measure any tail figure
  in isolation before calling it a regression** — and note the baseline's tail is inflated too, so the
  comparison can mislead in either direction. One cheap sanity check costs nothing: ask whether two
  shapes that ought to be identical measured identically.
- **Take the best of several rounds, not one measurement.** Single rounds of the `access` mode swung
  0.96x–1.22x on the same build — wider than the effect being measured. The minimum is the run least
  disturbed by everything else on the machine, which is what these modes are actually asking about.
- **Never measure a configuration immediately after a heavy phase, and never trust an A/B where one
  arm ran second.** A wrapper that builds config 1, tests it, then builds config 2 and tests it hands
  the second arm a machine that has not settled — the test suite and the error scenarios are hundreds
  of subprocesses — so the second arm is systematically penalised by whatever is still draining. This
  is not the same hazard as "the machine is busy": it is *ordered*, so it biases one arm rather than
  adding symmetric noise, and best-of-N rounds does **not** remove it. Measured by a wrapper that
  built and tested each configuration in turn, which reported LTO **22–30% slower**
  than plain (S7-5 85.59 → 111.00 ms); re-measuring the two arms alone, with no suite in between, put
  them within **0.5%** (85.10 vs 85.43). The first result would have been written up as a real
  regression. **Re-run any delta that would change a decision, with the arms measured back to back
  and nothing heavy before either**, and quote the noise floor: the same binary moved 12% run to run
  on that machine (S7-1 118.84 → 133.02 ms), so anything under that is indistinguishable from zero.
- **An accessor that copies is not a read — make two modes being compared do the SAME job.** `%get`
  allocates a fresh array the size of the column and copies into it, so a `%get` loop measures the
  read *plus* a full allocation and copy per column, and `sum(...)` on top adds another whole pass.
  Timed against `%materialize_all`, which only reads, that made reading 4 of 16 columns look 1.8x
  what scaling the full read by `touch / ncols` predicts — reported as the library failing to scale
  when the two sides were simply not measuring the same work. Compare `%prefetch` with
  `%materialize_all`; time any `%get` on its own line; keep the keep-it-live checksum outside every
  timed region and take it through `%col`. The same caution applies to any future accessor whose
  cost scales with rows.
- **A reference has to be allocation-free, or it is not a reference.** An existing library operation
  pressed into service as a "memcpy floor" measures its own allocation and first-touch page faults,
  not bandwidth: `%clone` labelled that way came out **27-30x** slower than a genuine warm-buffer
  copy of the same payload, and varied **5.6x between runs at the same size** on a large NUMA
  machine, where a cold destination is dominated by faulting. Allocate **and fully write** both
  buffers before the timer starts. **The falsifiable tell is worth remembering, because two
  independent reviewers used it to reject the number: if the thing being compared comes out FASTER
  than the "floor", the floor is wrong.** Every ratio taken against it is then meaningless in an
  unknown direction, which is worse than having no reference at all.

**To measure the committed baseline against the working tree**, `git stash push -- src tools`, rebuild,
measure, then `git stash pop` — this keeps `app/`, `test/` and the fixtures in place, so the benchmark
program and its input do not change between the two halves of the comparison. Confirm the stash popped
cleanly (`git status`) before trusting the "after" number.

### Measuring whether Arrow memory was actually freed: RSS cannot answer, the pool counter can

**Resident set size does not fall when Arrow buffers are freed**, so it cannot be used to verify that
some code path released them. Arrow allocates through `arrow::default_memory_pool()`, which keeps
freed pages instead of returning them to the OS (and glibc/macOS `malloc` behave the same way for
ordinary allocations). A correct release and a complete failure to release therefore look nearly
identical in `ps`/`/proc` output — and the difference that *does* show up is the transient peak, which
misleads in the opposite direction.

Measure `arrow::default_memory_pool()->bytes_allocated()` instead. `parquet_get_arrow_bytes_allocated`
(`src/parquet_wrapper.cpp`) exposes it; it is deliberately **not** declared in
`src/parquet_bindings.f90`, because it is a maintainer diagnostic rather than public API — the
consumer declares its own local `bind(C)` interface for it, the same convention the `parquet_debug_*`
hooks follow. Two further rules when writing such a measurement:

- **Measure each path in its own process.** Running a baseline and the path under test in one process
  reports the high-water mark of the pair, which makes whichever ran second look like it retained
  memory it had already released.
- **State in the output which number is the real answer.** A report that prints RSS next to the pool
  figure invites the reader to draw the wrong conclusion from the wrong line.

### A `shared_ptr` parameter on a per-row helper is the first thing to suspect in `parquet_wrapper.cpp`

A helper that takes `const std::shared_ptr<arrow::Array> &` and is called **once per row** pays two
atomic refcount operations per call, because every `std::static_pointer_cast` inside it builds a new
`shared_ptr`. Measured here: reading one `double` through `real_family_value_at` that way cost
**13.6 ns per row**, against **1.8 ns** once the helper took a plain `const arrow::Array *` — a 7.6x
improvement in the filter's clause evaluation and 3-4x in the whole cost of installing a filter, with
no threading involved at all. The caller already owns a reference for the duration of the loop; the
loop needs the pointer, not a share of the ownership.

**Three things make this worth a standing note rather than a one-off fix:**

- **Nothing fails when it comes back.** Every answer stays identical and the whole suite stays green;
  only a benchmark notices. `tools/check_source_conventions.py`'s `check_no_per_element_shared_ptr`
  is what actually guards it, matching by *shape* (an array parameter beside an element index) so it
  cannot go blind to the next helper added.
- **It hides from a profile that samples by function name**, because the atomics are attributed to a
  helper that was already expected to be hot.
- **The plausible-looking explanation was wrong**, which is the part worth copying. The same loop
  called `compare_op(value, bound, op)` with the operator as a `std::string`, testing it with up to
  five string comparisons **per row** for a loop-invariant value. Converting that to an enum changed
  the measurement by **nothing** — the compiler was already hoisting it. Measure the fix, not just
  the symptom: an obvious inefficiency that is genuinely there can still be worth 0%.

### Instrument phases before optimising a multi-phase operation

"X is slow" is not actionable until it is known *which part* of X is, and the cheapest way to find
out here is a few `parquet_debug_get_*_nanos` counters in the `parquet_debug_*` family (no
`src/parquet_bindings.f90` entry; the consumer declares its own local `bind(C)` interface). Three
counters around `parquet_reader_set_filter`'s decode / evaluate / mask-build phases turned a vague
"the filter is expensive" into "93% of it is one helper's parameter type", and they turned the *next*
question — whether to parallelise what remains — from a design argument into a number.

**Leave them in.** They cost nothing at runtime (one `steady_clock` read per phase, on a path that
runs once per reader open) and they are what makes the next measurement one benchmark away rather
than a fresh investigation. The rule that governs where such a hook may sit is the existing one: a
debug hook may sit on a coarse operation, never on a per-row or per-element path.

**But a phase's SHARE bounds the prize; it does not estimate it — and stopping at the share will
keep an item that should be dropped.** The phase still contains work the proposed change does not
remove. Worked example, and it is the reason this paragraph exists: a phase timer put the padded
string read's per-row copy loop at **31% of the read**, comfortably above this project's ~5%
keep-or-drop line. An A/B running that same loop with a direct typed call instead of the
`std::function` under test then put the indirect call at **1.8 ms of 43 ms — 4.2%**; the other
11.5 ms was a memcpy and blank fill no accessor change touches. The 31% would have bought ten-plus
C++ call sites, a template inside the file's single `extern "C"` block, and a silently-wrong-string
failure mode, for 4%.

**So: after a phase timer, A/B the actual change on ONE site before committing to sixteen.** A
temporary A/B is cheap (run both loops, charge each to its own counter, discard one result), and it
is reverted afterwards while the phase counters stay. The same trap in a different dress killed
another item: a *loop* ratio is not an *operation* ratio either, so always carry the loop's share of
the end-to-end path — one change measured 1.5x on its loop and a **negative** end-to-end ceiling.

## Testing & coverage

### Running a single test suite/test

Use `fpm test run_tester -- <suite>` to run just one test-drive suite (e.g. `fpm test
run_tester -- reading`), or `fpm test run_tester -- <suite> "<test name>"` to run a single
named test within it. Prefer this over a full `fpm test` while iterating — the full suite
(including OpenMP/error-scenario subprocess tests) takes much longer than the one suite
relevant to a given change.

### Error scenarios are pre-run in parallel

`run_tester` calls `prime_error_scenarios` (`test/test_errors.f90`) before any suite starts: one
`execute_command_line` runs every scenario named in `tools/run_error_scenarios.sh`'s
`scenarios=(...)` array through `xargs -P`, capturing each one's exit status and its two streams
under `test_run/.primed/`. `run_error_scenario` then answers from those files. This is what takes a
full `fpm test` from ~72 s to ~25 s here — the ~630 scenario subprocesses were ~60 s of it, run
strictly one at a time because the suites driving them are excluded from test-drive's parallelism.

Four things to know before touching this area:

- **The parallelism is deliberately in the shell, not in the test process.** Exactly one fork
  happens, from `run_tester` with no OpenMP team active, so the libiomp5 fork hazard that forces
  `suite_is_safe_to_parallelize`'s exclusions is never approached. Do not "simplify" this into
  per-test spawning from a parallel suite; that is the thing the exclusion exists to prevent.
- **A new scenario must be added to `tools/run_error_scenarios.sh`'s array**, which
  `tools/check_source_conventions.py`'s `check_scenario_list_is_complete` now enforces (it derives
  the names by shape from `error_scenarios.f90`'s `select case`, so it does not go stale). Forget
  it and nothing fails — the scenario just falls back to spawning on demand — which is precisely
  why the check exists.
- **Every failure path degrades to the old on-demand spawn, never to a wrong answer.** That is
  load-bearing: it is what makes a missing list entry cost speed instead of coverage.
  `test_run/.primed/` is wiped by the prime itself and `g_prime_ok` is per-process, so a directory
  from an earlier run can never be consumed as this run's result — a vacuous pass against a binary
  that no longer exists is the failure mode this design is shaped around.
- **`PARQUET_TEST_NO_PRIME=1` turns priming off** (debug one scenario without 686 others running
  first); `PARQUET_TEST_PRIME_JOBS=<n>` sets the concurrency.
- **Only a full run and the `errors` suite prime; every other named suite, and any named single
  test, does not.** Priming is all-or-nothing — it runs the whole ~690-scenario array — so it pays
  off only where essentially all of it is consumed. `writing`, `metadata`, `maml` and `reading`
  each drive a few dozen scenarios and spawn them on demand instead, which is what keeps
  `fpm test run_tester -- <suite>` a targeted command rather than a near-full run. Do not widen
  `suite_drives_error_scenarios` back to those four: the trade is ~690 subprocesses to save a few
  dozen, and the behaviour reads as a bug from outside — a `maml` run looks like it is dragging in
  the error suite, which is exactly how the previous, wider gate came to be narrowed.

**A capability probe must RUN the command with the flags it will use, never ask whether the name
exists.** Both scenario harnesses guard each scenario with `timeout -s KILL`, and both used to
select it with `command -v timeout`. MacPorts ships a **BSD-syntax** `/opt/local/bin/timeout`
(usage: `timeout [-signal] time command`) which exists, satisfies `command -v`, and then rejects
`-s KILL`. The failure is as bad as it gets:

- every scenario's captured stderr becomes `usage: timeout [-signal] time command...` with status 1;
- that is **indistinguishable from a scenario that really printed that and exited 1**, so the
  degrade-to-on-demand-spawn path never fires;
- **541 of 664 error-scenario tests failed**, with messages like *"expected stderr to contain
  'parquet_write_column: column not defined...'"* and *"control scenario 'ok' was expected to exit
  cleanly"* — every one of them blaming the library for a defect in the harness.

It appeared mid-session, when a MacPorts install put that binary on `PATH` between two runs, so the
same tree passed at 14:2x and failed at 14:45 with no source change. **The tell is that the
scenarios pass when run BY HAND** (`<build>/test/error_scenarios ok` exits 0) while the suite says
they fail; when that happens, read `test_run/.primed/<name>.err` — it holds whatever was captured,
and a harness error is visible there immediately.

Both sites now probe by running `timeout -s KILL 1 true >/dev/null 2>&1` and fall through to
`gtimeout` (GNU coreutils). That covers the absent case for free, since a missing command also
exits nonzero, and it keeps the probe and the real invocation using the same flags — which is the
rule: **probe with what you will actually run.**

Both streams are now always captured **separately** — the old `2>&1` merge is gone, because the
primed and spawned paths have to produce the same shape. A helper wanting the old "appeared
somewhere" semantics calls `scenario_capture_contains`, which searches both.

### Tests run concurrently: never share a fixture file path between two tests

test-drive runs the tests in a suite **concurrently**, so two tests that write to the same fixture
path can truncate the file out from under each other — one opens it while the other is rewriting it,
and the reader gets an empty or half-written file (`Parquet file size is 0 bytes`). **Give every test
its own fixture filename**, even when the contents are identical, and even when one test is
"obviously" going to run before the other.

This failure is **timing-dependent, so a green run proves nothing**: the same tests can pass under a
plain `fpm test` and fail under `tools/coverage.sh` (instrumented builds change the timing), or pass
for months and fail on a busier machine. If a test that touches files fails intermittently or only
under coverage, check for a shared path before looking anywhere else. Where several tests genuinely
need the *same* fixture contents, factor the writing into one shared helper that takes the filename
as an argument, and have each caller pass its own.

**`tools/run_error_scenarios.sh` is concurrent too (`xargs -P`), and there the rule is easiest to
break by accident, because the collision is between two SCENARIO NAMES rather than two tests.** One
parameterized helper backing several `case` entries — `scenario_settings_cpp_warning(level=...)`
and friends — is one *process per name*, all running at once over whatever fixture path the helper
hardcodes. **So a helper invoked from more than one `case` must derive its fixture path from its
own arguments** (`"..._" // trim(level) // ".parquet"`), never carry a
`character(len=*), parameter :: out_file`. Check this whenever a scenario helper gains a second
call site; `grep -oE "call scenario_[a-z0-9_]+" test/error_scenarios.f90 | sort | uniq -c` lists
every multi-invoked helper.

**Its failure signature is much more alarming than the test-drive one, and points away from the
real cause.** The reader gets a half-written file, Arrow throws
`IOError: Couldn't deserialize thrift`, and — because nothing catches an exception crossing the
`extern "C"` boundary — the process dies via `std::terminate` with **exit 134**, i.e. an abort in a
scenario whose expected exit is 0. That reads as a genuine library crash in whatever the scenario
was exercising, not as a fixture collision. **Reproducing it needs forced interleaving, not more
parallelism**: on an idle machine the pair finishes too fast to overlap, and 420 runs came back
clean, while pinning both processes to one core (`taskset -c 0`) took it straight to 69/160. Reach
for `taskset` before concluding a one-off — and note that a CI re-run going green is exactly what
this bug does.

**A test that WRITES process-global state needs its suite excluded, and the reasoning is not about
files.** `parquet_settings`' knobs are saved module variables, the sort comparison counter and the
pruned-row-group count are C++ statics: any test that sets one is visible to every sibling running
at the same time. The failure is usually not a crash but a *vacuous pass* — a sibling flipping a
setting mid-run can leave an A/B test comparing one code path against itself, which passes while
testing nothing. `filter_screen`, `sorting`, `sort` and `settings` are all excluded for this reason
(`test/run_tester.f90`'s `suite_is_safe_to_parallelize`). Note `sort` was excluded only after the
fact: it had written a process-global setting for a long time without incident, because its
assertions happened to be path-agnostic — so **"it has always passed" is not evidence that a suite
writing global state is safe**, only that nothing has yet asserted anything sharp enough to notice.

**Second consequence, for any library code that inspects OpenMP state:** test-drive achieves that
concurrency with its own `!$omp parallel do`, so under a `-fopenmp` build **`omp_in_parallel()`
returns `.true.` inside every procedure a test calls**. A guard that refuses to do something "inside
a parallel region" therefore fires during the entire test suite, not just in the test that meant to
provoke it. **This is NOT avoided by running `fpm test` without `-fopenmp`** — `fpm.toml` declares
the `openmp = "*"` metapackage, which supplies the OpenMP flag across the whole resolved dependency
graph, so `_OPENMP` is defined and every `#ifdef _OPENMP` guard is compiled in even for a bare
`fpm build` with `FPM_FFLAGS` unset entirely (verified directly with a minimal standalone fpm
project). The rule that follows is to
prefer a guard keyed on something more precise than "am I in a parallel region" — see
`unsafe_first_touch` (`parquet_tables_read.f90`), which records at open time *which thread* created
an object and refuses only when the object could actually be shared, so a thread-private object used
in the obvious way is unaffected. `test/run_tester.f90`'s `suite_is_safe_to_parallelize` can exclude
a whole suite from that parallelism, but reach for it only when the suite genuinely cannot run
concurrently — narrowing the guard is the better fix, since the suite's parallelism is itself an
ongoing regression check.

**An excluded suite runs with NO enclosing OpenMP region at all, and that is deliberate rather than
incidental.** The obvious spelling — `run_testsuite(..., parallel=.false.)` — does *not* achieve it:
that argument leaves test-drive's `!$omp parallel do` in place and switches it off with an `if`
clause, which still **opens** a region, an inactive one with a team of one. Inside it
`omp_get_level()` is 1 while `omp_in_parallel()` is `.false.`, so every test in the suite ran one
level down from the top of the program and any team the library opened was a nested team — which
deadlocks libgomp intermittently (`feature_risks.md` Risk-104, and the reason `fpm test` used to
hang about one run in three). `run_tester`'s `run_suite` therefore drives excluded suites through
`run_selected` per test, which calls `run_unittest` with no region at all. Two consequences worth
knowing: a test in an excluded suite may assume it is at `omp_get_level() == 0`, which is what lets
those suites keep asserting that a requested thread team is really opened; and because
`run_selected` finds its test **by name**, `run_suite` refuses a suite containing two tests with
the same name — otherwise it would run the first twice and the second never, silently, with the
count still looking right.

### An intermittent test failure has THREE causes, and the third is not concurrency at all

The section above supplies two explanations for a test that fails once and then passes ten times — a
data race, and two tests sharing a fixture path — and both are so well documented here that they are
the obvious first guesses. **A test that reads memory the library never wrote is the third, and it
looks exactly like the other two from the outside.** Both natural guesses were tried and were wrong
on the one instance found so far.

**The shape to recognise.** A `null` numeric row's *value* bytes are unspecified by design
(`grow_rows`, `src/parquet_columns_structural.f90`, and `%init`'s documented contract), so a column
created all-null and never written holds whatever the allocator left there. A test that then compares
such a column **against itself** — which is a sound and deliberate technique used widely in
`test/test_table_codegen.f90`, where one test pins the whole-column values and a second sweeps the
indexed/range accessor forms against them — passes for essentially any bit pattern, because `x - x`
is `0`. **Except for a NaN**: `NaN - NaN` is `NaN` and every comparison against it is false. So the
test passes or fails according to what was last on that heap page, at a rate low enough (1 in 6 full
runs, then 10 clean reruns) to be dismissed as a flake.

**The instrument that turns it deterministic, and it is worth keeping.** An `LD_PRELOAD` shim
filling every `malloc` block with a chosen byte; `FILL_BYTE=0xFF` is the interesting one, because
`0xFFFFFFFFFFFFFFFF` is a NaN as `real64` and `0xFFFFFFFF` is a NaN as `real32`, so **every
uninitialized float in the process reads as NaN**:

```c
/* Only decides what UNSPECIFIED memory contains; never changes a value the program writes. */
#define _GNU_SOURCE
#include <dlfcn.h>
#include <string.h>
#include <stdlib.h>
static void *(*real_malloc)(size_t);
static int fill = -1;
void *malloc(size_t n) {
    if (!real_malloc) real_malloc = dlsym(RTLD_NEXT, "malloc");
    if (fill < 0) { const char *e = getenv("FILL_BYTE"); fill = e ? (int)strtol(e, 0, 0) : 0xFF; }
    void *p = real_malloc(n);
    if (p) memset(p, fill, n);
    return p;
}
```

Under it the failure was 100% reproducible and the whole suite came back **1411 passed / exactly 1
failed** — the same test, the same column, nothing else — which is both the diagnosis and the proof
that the instance was isolated. **Its limits matter:** gfortran's `allocate` goes through `malloc`,
so column storage is covered, but stack-resident automatic arrays and anything from
`calloc`/`realloc` are **not**, so a clean whole-suite run under it is strong evidence and not a
proof. It is Linux-only (`LD_PRELOAD`); the macOS equivalent is `DYLD_INSERT_LIBRARIES` plus
`DYLD_FORCE_FLAT_NAMESPACE`, untested here.

**Three rules follow, and the third is the one that generalises furthest:**

- **A self-comparison is only as strong as the data varies.** Filling the offending column with a
  single constant cures the NaN unsoundness and leaves the range accessors unable to detect a
  misalignment — a grade-A defect traded for a grade-B one. Use row-distinct values, and verify that
  choice by mutation (a misaligned range accessor must fail), rather than asserting it in a comment.
- **Grade the assertions rather than fixing only the one that failed.** The audit that followed
  sorted every assertion in the affected test into *unsound* (compares undefined data with itself),
  *vacuous on values* (asserts only a `size()` or an `is_null()` — six of these, which would pass
  against a **wrong answer**, not merely against garbage, and which the NaN instrument cannot see at
  all) and *weakened* (fixture values repeating with period 2, so an off-by-even misalignment is
  invisible). Only the first two were worth fixing; the third was left deliberately, since fixing it
  means changing fixture values other tests assert against, for a reduction in strength rather than
  an unsound assertion. Copy the grading, not just the fix.
- **Uninitialized value bytes do NOT reach the output file, and that question is closed** — do not
  re-suspect it. Writing the same all-null column under four different heap fills produced two
  byte-identical files, and the files that differed differed in exactly **4 bytes**, all part of a
  creation timestamp in the metadata. Parquet stores no values for null entries at all (only
  definition levels record nullness), confirmed with `pyarrow`: the chunk is 23 bytes with
  `null_count=6`, and six `real64` values cannot fit in 23 bytes. Valgrind agrees — 0
  uninitialised-value errors on the write path.

### Every `check()` call needs its own message

Every test-drive `call check(error, condition, ...)` in `test/*.f90` should carry a message
argument, not just the bare condition. Without one, a failure reports only a file/line number —
which combines badly with this suite's common `all(...)`/compound-condition idiom (e.g.
`all((dates + offsets) == expected)`), where a bare line number gives no hint which element or
sub-condition actually failed. A convenient default when there's no more descriptive text at hand
is the condition's own source text as a string (whitespace-normalized, with any embedded `"`
doubled per Fortran's escaping rule) — better than nothing, and it at least echoes back exactly
what was expected without requiring a separate hand-written description. Apply this to every new
`check()` call from now on, not only when a review flags a gap.

### Verifying a change with mutation testing

A green test suite does not prove a new test covers what it was written for. The cheap check is to
break the code deliberately — invert a guard, delete a branch, return a constant — and confirm the
test fails. Worth doing for anything whose failure mode is *silent*: a fast path that skips work, a
short-circuit, a cache. This repository's recent examples are the validity-mask skip, the
`col_size` footer screen, and the write-side row-granular null mask; each had at least one mutation
that a first round of tests did not catch.

Three things about doing it *here* specifically:

- **Detect an abort, not just a failed check.** This library reports almost every error with
  `error stop`, so a mutation frequently makes a test **crash** rather than fail an assertion —
  `grep -c '\[FAILED\]'` then reports 0 and the mutation looks survived. Always check the exit
  status too (`error stop` → nonzero, SIGABRT → 134, SIGSEGV → 139/11 through `fpm run`). Getting
  this wrong made 3 of 5 mutations look uncaught in one session when they were all caught.
- **Check WHICH CODE PATH the test actually reaches before trusting it.** A mutation to the
  comparator survived a stability test twice: first because the test compared two procedures that
  both use that comparator (so it broke them identically and the comparison still held), then
  because the fixture — low-cardinality integers — took the counting fast path, which never calls
  the comparator at all. **Zero invocations is a passing test.** Assert against an independently
  constructed expectation rather than a second call into the same machinery, and where a fast path
  exists, force the slow one (`parquet_debug_set_disable_sort_counting_path` and friends) or pick a
  fixture the fast path declines. See `feature_risks.md` Risk-35.
  **A SIZE THRESHOLD is the same trap wearing different clothes, and is easier to miss** because
  nothing about it looks like a fast path. The co-ranked merge only splits a run past a
  16384-element floor, so an entire dense sweep over arrays of 2-8192 elements exercised the
  unsegmented path — the very code the feature replaced — and two deliberate co-rank defects
  survived it. Whenever a new constant gates behaviour on input size, give it a
  `parquet_debug_set_*` override and have the tests lower it, exactly as
  `parquet_debug_set_sort_merge_min_segment` now does. See `feature_risks.md` Risk-49.
- **An OPTIONAL argument that no internal caller passes is completely untested, and the suite stays
  green.** Adding one for API symmetry — so a new bulk entry point takes the same `is_null=` mask its
  sibling does — creates public surface the library itself never exercises, so every guard and every
  branch behind it is dead as far as the suite is concerned. Measured instance: **five of six**
  mutations to a new bulk-append path survived, and all five were behind either that optional
  argument or the not-empty-destination case, because the only in-library caller passed neither. The
  same applies to the *degenerate* shapes of a new entry point (appending onto a non-empty
  destination, a zero-length input): an internal caller usually hits one shape only. Enumerate the
  argument's presence/absence and the destination's empty/non-empty states deliberately when adding
  the test, rather than testing the path the library happens to take.
- **A surviving mutation is not automatically a coverage gap.** It may be *masked*: by a redundant
  sibling guard (removing either alone changes nothing — see `column_has_nulls_from_footer`'s
  `is_stats_set()`/`HasNullCount()` pair, where removing both segfaults), or by a later check that
  catches the same error anyway (the `col_size` footer screen is masked by the row-group scan that
  follows it). Test the pair, or the tier below, before concluding anything.
- **A procedure that writes into a caller-supplied fixed-length slot needs a CANARY, not a
  read-back.** Asserting the slot's contents afterwards passes just as happily against a procedure
  that overran it, because the bytes you check are the ones it got right. Removing the length clamp
  from `parquet_string_column%copy_to` — an out-of-bounds write — survived the entire suite for that
  reason. Hand the procedure a **substring of a longer buffer** and assert the remainder is
  untouched (`canary(1:3)` passed as the slot, `canary(4:)` still all `#`); the mutation then fails
  deterministically. A plain `fpm test` has no bounds checking, so nothing else will catch it.
- **A surviving mutation may be semantically a NO-OP, in which case it proves the design rather
  than exposing a gap.** Check that the mutation actually changes behaviour before concluding the
  test is weak. The worked example: flipping the parallel merge's tie rule from "take the left run
  unless the right is strictly less" to "take the left run when it is strictly less" changed
  nothing on any fixture — because `SortRowLess` is a **total order**, `less(a, b)` and
  `!less(b, a)` are the same predicate and there are no ties for the merge to break. The realistic
  defect was a different edit (substituting the tiebreaker-free comparator), and that one was
  caught. A mutation that is not a behaviour change is not evidence about the tests at all.
- **A change that makes something cheaper by NOT doing work can be entirely correct and still
  reduce a documented feature to a no-op — and a suite that asserts answers cannot see it.** When
  removing work, enumerate who was relying on that work having been done, and do it by grepping the
  tests and the guide for the feature's name rather than by reasoning about the code. Confirmed
  instance: releasing a read-time sort's key columns instead of re-ordering them is correct on every
  path, and the whole suite passed — while silently turning `parquet_open_reader(..., prefetch=.true.,
  sort_by=)` into a request that decoded those columns twice, because the prefetch that follows had to
  rebuild what the install had just dropped. It was found by reading a test's *name*
  (`test_prefetch_is_sorted`), and the fix was to let the caller's own argument decide
  (`keep_cache`), not to add a knob. **The corollary for the test that then guards it: a counter
  reading 0 cannot distinguish "the code ran and did nothing" from "the code never ran".** Pair it
  with a counter for the other branch and assert both, which is the same negative-control rule this
  file applies to settings and to guards.
- **If a mutation cannot be caught by any fixture this repository can build, the branch is
  defensive** — say so in a comment and `GCOVR_EXCL` it rather than deleting it or inventing an
  unbuildable fixture. `list_uniform_width`'s `IsNull` check is the worked example: Arrow's own
  `ListBuilder::AppendNull` already leaves `value_length == 0`, so no Arrow-built array reaches it.

### A test that asserts a REFUSAL must say what to assert when the refusal lifts

Some guards refuse a case on **cost** rather than correctness — the machinery is built and tested,
and the clause exists only because a measurement said the case was not worth it yet. A test asserting
such a refusal is correct today and is the wrong test tomorrow, and whoever lifts the clause meets a
failing test with no indication whether it is protecting something or merely out of date.

**Write the deferral into the test's own doc-comment**, in the form "when X lands, this becomes an
equality test, not a deletion — what it asserts today is that the refusal is real, and what it should
assert afterwards is that Y kept the answer." Two such tests were written that way for the prefetch
gate's `filter=`/`sort=` clauses and both were converted rather than deleted when the clause lifted,
by following their own instructions. The same applies to the *code*: a refusal comment must read
"deferred until X", never "this cannot be done", or the next reader takes the clause as settled.

**A refusal can also disappear entirely rather than narrow, and then its test is deleted, not
converted — in all three places at once.** Widening `parquet_get_column_type` to report a
narrowest-lossless target left *no* type that makes it abort, so `get_column_type_unsupported` was
asserting a guard that no longer exists anywhere; it had to go from `test/error_scenarios.f90`'s
`select case`, from `tools/run_error_scenarios.sh`'s array and from its `test/test_errors.f90`
wrapper, since a partial removal fails `check_scenario_list_is_complete`. Two rules follow, and the
second is the one that keeps coverage from quietly dropping: **before deleting, check whether the
behaviour has a positive form worth asserting instead** (that one became "reports `unknown` rather
than aborting", which is a stronger test than the refusal ever was), and **expect the widening to
break tests far from the change** — six unrelated error scenarios had been using a `uint32` column
precisely *because* it was unreadable. Run the whole suite, not the area's own.

### A test must not assert a compiler's `ERROR STOP` spelling or exit status

**Both are processor-dependent, and all three compilers in this fleet differ.** F2018 requires
`ERROR STOP` to terminate with error; it fixes neither the wording nor the status. Measured with a
three-line program on machine A:

| compiler | exit status | first line on stderr |
|---|---|---|
| gfortran 15.2 | 1 | `ERROR STOP <msg>`, then a backtrace |
| flang 22.1.8 | 1 | `Fortran ERROR STOP: <msg>` |
| nagfor 7.2 | **2** | `ERROR STOP: <msg>` — note the colon |

So a test searching stderr for `"ERROR STOP parquet_close_writer: ..."` passes only under gfortran,
and one asserting `exitstat == 1` fails under NAG. Both shapes had shipped, both looked perfectly
reasonable, and **both reported a correctly-behaving library as broken**.

- **Assert the LIBRARY's own message text**, never the runtime's prefix in front of it. That is what
  the assertion is about in every case this has come up.
- **Assert `/= 0`, and `/= 134` where the point is telling a Fortran abort from a C++-level one.**
  The C++ side really is exactly **134** everywhere, because `fatal_exit()` ends in an explicit
  `std::_Exit(134)` — that one may be asserted by value. The Fortran side may not.
- **The shared helpers already do this right** — `check_scenario_exit_status` computes
  `aborted = (exitstat /= 0)` and `tools/run_error_scenarios.sh` tests `-ne 0` — so a scenario that
  goes through them is portable and only a hand-written comparison is at risk. Grep for
  `exitstat ==` before adding one.

**The documentation half is the same defect and is easier to miss**, because a guide page stating
"exit status 1" reads as a fact rather than as an observation of one compiler.
`doc/pages/operating/error-handling.md` said exactly that and had been reviewed and closed, having
been verified against gfortran alone. **A claim about processor-dependent behaviour cannot be
verified on one compiler** — either check the fleet or state the property rather than the value.

### A test that asserts THREADING must skip without OpenMP

A test whose subject is that something *ran in parallel* — a threaded rebuild matching its serial
twin, a sort reaching Design B, a shared-state guard firing inside a region — has nothing to assert
on a build where no team can be opened. **The danger is not that it cannot run; it is that the
assertions become VACUOUS while still looking like assertions.** Both arms of an A/B execute the same
serial code, so the equality holds for the wrong reason, and the negative control that was keeping
the equality honest is the only thing that fails. A reader then sees "the threaded rebuild is
broken" where the truth is "this build cannot reach the threaded rebuild".

Skipping is what distinguishes the two, and it has to be said out loud — a silent pass is worse than
a failure, because it removes the doubt that would have prompted a look.

```fortran
#ifndef _OPENMP
        call skip_test(error, "needs OpenMP: <what is compiled out, and why the assertion below " // &
            "would then hold for the wrong reason>")
        return
#endif
```

Four rules, each of which has already been broken here:

- **Put it after the declarations and before every executable statement**, and check that it does not
  land inside the declaration block — an inserted `call`/`return` among the declarations is a
  compile error, and a guard placed after some setup skips a test that has already mutated global
  state.
- **Pick the predicate the code actually keys on, not the nearest OpenMP-sounding one.** The sort
  engine clamps an explicit `threads=` against `omp_get_num_procs()`
  (`sort_build_permutation_threaded`), so those tests need an `omp_get_num_procs() < 2` arm as well
  — on a one-processor runner `threads=4` resolves to 1 and no design is entered.
  `parquet_strings` resolves from `omp_get_max_threads()` and never clamps to the processor count,
  so its tests need only `_OPENMP`. Reading the resolution path is the only way to tell.
- **Give the guard a NEGATIVE CONTROL, exactly as any other guard.** One that fires unconditionally
  turns a whole suite green while testing nothing. The cheap check is that a normal build reports
  **zero** skips; the sharper one is to invert the predicate and confirm the expected tests skip,
  which is also the only way to exercise a processor-count arm on a machine that has processors.
- **An error scenario needs BOTH halves.** The test-drive wrapper takes the guard above, and the
  scenario name belongs in `tools/run_error_scenarios.sh`'s `concurrency_scenarios` bucket, which
  exists for precisely this class and is documented there. A scenario in the strict list instead
  works only by the accident that everyone builds with OpenMP.

**A test that merely USES threads is not in this class** and must not be guarded: it either behaves
identically when the team is one thread, or it is asserting something else. Guard only what asserts
that a team existed. Distinguishing the two is the whole judgement — over-applying this hollows out
the suite on exactly the builds that ship.

**Skipping is a RUNTIME decision, so the test must still COMPILE without OpenMP — and that is a
separate obligation the skip guard does nothing about.** `!$omp` directives vanish in such a build
because they are comments; the ordinary Fortran around them does not, so an unguarded
`omp_get_num_threads()`, `use omp_lib` or `omp_lock_kind` is an undeclared name and the file fails
to compile long before any test can skip. Every such reference needs its own `#ifdef _OPENMP`, with
the serial arm given a value (`tid = 0` before the guard, `avail = 1`), exactly as the existing
`avail = omp_get_max_threads()` sites do.

`check_openmp_calls_are_guarded` (`tools/check_source_conventions.py`) enforces it across `src/` and
`test/`, because nothing else can: fpm's `openmp = "*"` metapackage supplies `-fopenmp` for gfortran
and ifx, so `_OPENMP` is defined in CI and in every ordinary `fpm test`, and the other arm is only
reached by a toolchain the metapackage does not cover. It has bitten twice — `materialize_marked_parallel`
in `src/`, and a threading test in `test_table_parallel.f90` that left the file uncompilable
serially for two days with every check green. **To verify a no-OpenMP build by hand**, comment out
`openmp = "*"` in `fpm.toml`, build into a throwaway `FPM_BUILD_DIR`, and restore it; confirm from
`fpm build --show-model` that the flags line really carries no `-fopenmp`. The suite runs there —
1845 passed, 0 failed, 22 skipped on gfortran 15.2 as of 2026-08-23 — and every one of those skips
should name a threading assertion.

### A static check that enumerates names goes stale silently

A check in `tools/check_source_conventions.py` that works from a *list* of names — helper procedures,
constants, call sites — stops seeing anything the list does not mention, and says nothing about it.
It keeps reporting `[ok]`, so there is no moment at which anyone learns it has narrowed.

This has happened twice to one check. `check_print_settings_documented` extracted its rows by
matching `print_one`, then `print_one|print_text`, and each time a new row helper was added it went
blind to those rows — the second time reporting two of five new rows as undocumented while passing
the other three, which is worse than failing outright, because a partial failure looks like a
complete answer. It now matches by **shape** (`call print_<anything>(u, "name"`), which picks a new
helper up with no edit.

**Prefer matching a shape over enumerating names**, and where a list is genuinely unavoidable, have
the check fail when the list comes up empty rather than pass — an empty result almost always means
the code moved, not that the invariant holds. Where two checks need the same list, derive it from
one place: the documentation and environment-coverage checks both take their knob list from
`parquet_print_settings`' own printed rows, so a new knob fails both together instead of needing two
separate lists updated.

**Two checks over the same SOURCE list can both be green while the DOCUMENTATION of that list is
wrong, and that gap is invisible by construction.** `check_env_covers_every_setting` proves every
printed knob has an environment variable, and `check_print_settings_documented` proves the guide's
sample dump names every printed row; neither compares the guide's *`PARQUET_FORTRAN_*` table*
against the variables `parquet_settings_from_env` actually reads. Two variables therefore shipped
applied by the code, printed by the dump, covered by a test — and absent from the table a user reads
to discover they exist. **When a documentation table mirrors a list the source owns, check the two
against each other directly**, in both directions (a row the source does not read is a documented
knob that silently does nothing), and read the table's own rows rather than scanning the page —
surrounding prose usually names two or three of the entries in examples, so a page-wide search
accepts a table missing everything else. `check_env_table_matches_the_source` is the worked example.

**A COUNT written out in prose is the same hazard in its cheapest form, and it interlocks across
files.** `parquet_set_threads` grew from three callees to six, and the number was wrong in *nine*
places at once: a guide heading ("All three thread counts at once"), that page's environment table
("sets the five below"), the doc-comment's opening line ("all four"), two further sentences in the
same doc-comment ("the five"), the dummy argument's `!!` tag, the `error stop` message, the test's
own name ("all five thread counts") and a `CHANGELOG` bullet. Every one was written correctly at the
time. Nothing can check a number in prose, so the defences are to **avoid writing one where the list
is the point** — "the five per-area caps" needs no edit when a sixth arrives, and neither does an
error message that stops enumerating — and, where a count must be written, to have the two places
that carry it **name each other**, which is what `parquet_set_threads`' doc-comment and
`test_set_threads`' body now do.

**The same blindness applies to a one-off AUDIT, where nothing reports `[ok]` and there is no second
chance to notice.** A hand-written search pattern used to answer "how many places do this?" is itself
untested, and its answer is quoted afterwards as though it were a count. A grep over `src/` for a
per-element allocation reported **4 sites**; converting the same question into a lint check over the
same scope found **17**. The audit's regex required the destination to be the call's last argument,
and the dominant real shape puts it second-to-last — so it saw the four instances that happened to
match the form the pattern was written from. **Before quoting a count, run the pattern against a
known instance you did NOT use to write it**, and prefer turning the audit into the check rather than
reporting a number and building the check later; the check is the thing that gets re-run.

**When the count is too large to fix at once, ratchet it rather than allow-list it.** An exemption
list says "these are fine"; a ratchet says "these are debt, and it may only shrink". Record a
per-file count of the remaining instances and fail on **both** directions — a file gaining one, and a
count left too high after someone fixes one. That keeps a new instance in a file nobody listed
visible, makes the list self-correcting instead of stale, and turns the remaining work into something
a reader can see the size of. `check_no_per_element_string_alloc`'s `KNOWN_REMAINING` is the worked
example; verify a new ratchet fires in both directions before trusting it, because the
count-left-too-high half is the one that never fires on its own.

**The same blindness applies to a tool that inventories the ENVIRONMENT rather than the source, and
there it is worse, because absence gets quoted as a finding.** `tools/machine_report.sh` listed
compilers from a fixed set of unsuffixed names (`gfortran flang g++ …`), while MacPorts, Homebrew and
most distributions ship them suffixed — `flang-mp-22`, `g++-mp-15`, `gcc-13`, `clang++-18`. On
machine C it therefore reported flang as absent when flang 22.1.8 was on `PATH`, its `--lto-probe`
skipped the flang arm **silently**, and a benchmarking report went on to record both that the machine
had no flang and that this file's own machine table was stale. Both conclusions were wrong, and
nothing in the tool's output looked incomplete. Two rules follow, and they generalise to any future
environment probe:

- **Match by shape, not by a remembered list** — `BASE`, `BASE-mp-*`, `BASE-[0-9]*` across every
  `PATH` entry (`compiler_variants` in that script), so a compiler installed under any conventional
  name is found without editing anything.
- **A skipped probe must not look like a passing one.** Print `SKIPPED -- '<x>' not found` rather
  than returning quietly; an arm that never ran is otherwise indistinguishable from one that was
  never written, and the reader counts passes rather than arms.

**A tool's silence is evidence about the tool, never about the machine.** Before reporting that
something is not installed, run it by name.

**And its CONFIDENT OUTPUT is evidence about the shell it ran in, never about the machine either.**
`tools/machine_report.sh` describes *the environment it is invoked from*. Run before an activation
script — which is the natural order, since a run sheet asks for provenance first — it faithfully
reports a **default environment nothing is measured in**, and those lines then read as findings
about the machine. This has now misled twice in one report, in two different sections:

- `fc : gfortran` resolving to the **system 11.5.0**, the compiler this project's version floor
  exists to exclude, beside figures actually taken with 14.2.1;
- `arrow 23.0.1` from `/usr/lib64/pkgconfig`, written up as "the machine table is stale" when both
  activated toolchains build against **24.0.0** from a prefix the activations prepend to
  `PKG_CONFIG_PATH`. The contradiction was already visible in the same report's own provenance
  block — `FPM_LDFLAGS=-L/home/elmo/usr/local/lib64`, i.e. neither toolchain looks at `/usr/lib64`
  at all — and the tool's line was believed over the flags printed beside it.

So: **run it once inside each activated shell** (it is read-only and takes seconds), or label its
output "the default environment" and never quote it as a property of the machine. A provenance
block that disagrees with the `FPM_*` flags printed next to it is reporting two different
environments, and the flags are the ones that built the binary.

### A `tools/*.sh` check must run under bash 3.2, and must never exit 0 having stopped early

**macOS ships bash 3.2**, and `#!/usr/bin/env bash` finds it there — two of the three machines in
the fleet are macOS, so a script using bash-4 syntax is a script that does not run where it is most
often run by hand. The trap is that bash does not refuse the file: it fails at the offending
construct and carries on, so the damage lands somewhere later and looks like something else.

Confirmed instance, and the reason this is a rule rather than a preference:
`tools/check_random_kernels.sh` used `declare -A` for a two-entry lookup. On macOS the declaration
failed, the first assignment into it then died under `set -u`, and **the script exited 0 having run
one configuration out of twelve** — with the vacuity guard, which its own header calls more
important than the assertions, never reached. Nothing in the output said so; CI on Linux was green
and correct throughout. The bash-4 constructs to avoid are associative arrays (`declare -A`),
`mapfile`/`readarray`, and `${var,,}`/`${var^^}` case conversion. `bash -n script.sh` on a macOS
machine catches syntax-level cases; the `declare -A` class is a runtime failure and needs an actual
run.

**Two rules follow, and the second matters even when the first is obeyed:**

- **Keep `tools/*.sh` free of bash-4 syntax.** An indexed array, two scalars, or a `case` will
  express anything a small check needs. If a script genuinely requires bash 4, it must assert
  `${BASH_VERSINFO[0]} -ge 4` and exit nonzero — a documented refusal beats a silent no-op.
- **A check that stops early must not exit 0.** `set -e` is usually unavailable in these scripts
  because they inspect non-zero exits deliberately, so the portable form is an explicit completion
  flag: set `finished=0` up front, `finished=1` once every unit of work is done, and
  `trap '[ "$finished" = "1" ] || { echo "... TERMINATED EARLY -- this run proves nothing" >&2; exit 2; }' EXIT`.
  Every deliberate `exit` then happens after the flag is set. Verify it by injecting an unbound
  variable reference mid-script and confirming the exit status is nonzero.

This is the "silence is not success" rule applied to the checking tools themselves. A tool whose
failure mode is a green report is worse than no tool, because it also removes the doubt that would
have led someone to look.

### Measuring test coverage

Run `tools/coverage.sh` for per-file and total `src/` line coverage plus the uncovered line
ranges; pass one run_tester suite name to scope it (e.g. `tools/coverage.sh reading`), or no
argument for a full run (which also runs every error scenario). It auto-selects the `gcov`
matching the active `gfortran` — a mismatched gcov fails with "Invalid .gcno file!".

**A coverage report is only trustworthy if the counters were collected safely, and two separate
mechanisms here corrupt them. Both are now handled inside `tools/coverage.sh`; do not remove
either without reading this.**

- **In-process races lose counters, so `-fprofile-update=atomic` is required.** test-drive runs a
  suite's tests concurrently inside one process, so two threads incrementing the same gcov counter
  can lose an increment — and a line hit only once or twice in the whole run then reports as
  **uncovered**. Measured on this project: the flag took a full run from 24322/24324 to
  **24324/24324**, and the two lines it recovered were an ordinary, well-covered branch that had
  already been chased as a real gap. (An older note elsewhere in this file records that adding this
  flag "changed nothing" — that was about the *gcovr 10,000-line parser* bug, a different problem,
  and says nothing about counter loss.)
- **Concurrent inter-process merges corrupt the `.gcda` outright, so the error scenarios run
  serially under coverage.** `tools/run_error_scenarios.sh` normally dispatches ~700 scenarios
  across `nproc` workers, each merging into the same shared `.gcda` files as it exits. Three
  consecutive full runs died with `src_parquet_tables_rowmutate.f90.gcda: not a gcov data file` —
  always that file — and a serial run has not reproduced it. `RUN_ERROR_SCENARIOS_JOBS` overrides
  it if the speed is wanted back.

**Before either fix, a single run's numbers were approximate and the errors ran in the direction
that invents work**: a whole untouched file reading 47%, individual lines flipping between runs.
If a report still looks wrong, compare two runs before chasing a line — and remember that noise
can only ever *lose* counts, so a line reported covered in **any** run is covered.

When closing coverage gaps, sort each uncovered line by type first: an `error stop`/abort
line can *only* be covered by an out-of-process scenario (`test/error_scenarios.f90` + a
`test/test_errors.f90` wrapper + a `tools/run_error_scenarios.sh` entry), never by an
in-process test-drive test (the abort kills the process); a normal branch is usually
reachable by extending an existing test with different data/arguments. Genuinely not
coverable and not worth chasing: `end module`/`end submodule` lines, implicit finalizers,
and interface-only files (`parquet_bindings.f90`). `src/parquet_wrapper.cpp` is measured
by `.gitlab-ci.yml`'s `test` job (its `gfortran`/`gcc`/`g++` are one matched apt GCC install,
so `gcovr` reads its gcov data cleanly alongside the Fortran sources) but deliberately
**not** by `tools/coverage.sh`'s local run, since an arbitrary dev machine's `FPM_CXX`
(e.g. a default `clang++`) may not produce gcov data in a format the locally-resolved GNU
`gcov` can read — see CONTRIBUTING.md's CI section. For local `src/parquet_wrapper.cpp`
coverage on exactly that kind of machine, use the separate `tools/coverage_cpp.sh` instead
(not a flag on `tools/coverage.sh` — Fortran and C++ coverage can't be instrumented/collected
in the same local pass when the toolchains don't match); see CONTRIBUTING.md's CI section for
what it does and why it's a standalone script. See "`src/parquet_wrapper.cpp`: GCC vs Clang gcov
attribution" below for why local and CI coverage can disagree and the conventions that keep them
in sync.

Both `tools/coverage.sh` and `tools/coverage_cpp.sh` report an extra section after the per-file
summary and uncovered-line list: every `GCOVR_EXCL`'d line that actually had a positive local hit
count, i.e. a candidate for a stale/no-longer-dead exclusion worth revisiting (not automatically
fixed — just surfaced). `tools/coverage_cpp.sh` additionally splits this into two: lines tagged as
a GCC-attribution artifact (see below) that unexpectedly show *zero* hits locally (investigate —
these are expected to be genuinely covered under Clang) and everything else that shows *positive*
hits (the stale-exclusion candidates). `tools/coverage.sh` only needs the latter, single section,
since Fortran coverage uses the same `gfortran`/`gcov` toolchain locally and in CI — there's no
GCC-vs-Clang split to make for `src/*.f90`.

### Coverage tooling never drives design

**gcov and gcovr are instruments, not requirements. A coverage tool's limitation is never a reason
to write the code differently.** If a change is right for the library — faster, clearer, simpler —
it is made, and any resulting coverage artifact is documented and excluded, not designed around.

This has to be stated because the pressure runs the other way in practice: this file records several
places where a tool cannot see what the code does (six `impure elemental` headers in
`src/parquet_temporal.f90` that never register as hit while their bodies do; the `module procedure`
form having no extractable source; whole classes of GCC-versus-Clang line attribution differing on
`src/parquet_wrapper.cpp`), and each one is a standing temptation to avoid the construct rather than
annotate it. Don't. Concretely, none of the following is ever a reason to reject a change:

- a procedure would become `elemental` and so join the header-attribution quirk above;
- a rewrite would move lines into a shape gcov attributes to a different line;
- a `GCOVR_EXCL` marker would have to be added, moved, or explained;
- a coverage percentage would move because a hot loop became one vectorised statement.

What *is* required is that the artifact be recorded: tag the site with the conventions in the two
sections below, so the next reader knows the exclusion is a tool limitation rather than untested
code, and so `tools/coverage.sh`'s stale-exclusion report does not flag it forever.

The one thing this does not license is using "it is a coverage artifact" to wave away a genuine gap.
The distinction is evidence: an artifact is confirmed by showing the surrounding body *is* covered
(see the next section for how each documented instance was verified), never asserted because a line
is inconvenient to reach.

### Fortran gcov attribution artifacts

Confirmed by inspecting raw per-line gcov hit counts (not just `tools/coverage.sh`'s summary):
gfortran/gcov sometimes marks an excluded, genuinely-dead line as "hit" even though it never
executes — a different mechanism from the GCC-vs-Clang split documented below (this one is
gfortran-only, present identically locally and in CI), so it needs its own convention rather than
reusing that section's.

Two confirmed shapes:

- **A guard-clause `if (cond) then` line inside a `GCOVR_EXCL_START`/`GCOVR_EXCL_STOP` block.** The
  condition itself is evaluated on *every* call to the function, whether or not the guarded
  (excluded) body is ever reached — so gcov counts that line as executed regardless. The body
  immediately inside (the `error stop`/diagnostic-and-`return`) reliably shows 0 hits, proving the
  branch itself is never actually taken; only the `if` line's own hit count is misleading. Seen in
  every `check_*_fits_arrow_limit`-style int32-ceiling guard (e.g. `parquet_read.f90`'s
  `parquet_get_nrows_int32`), every defensive branch in `parquet_strings.f90`'s `validate()`, and
  the col_map/duplicate-output-name/empty-name guards in `parquet_metadata_maml.f90`.
- **A bare `return` statement mis-attributed to a different call site's execution count**, seen in
  `parquet_read.f90`'s `parquet_tokenize_filter_rule`, `case default` arm: the `return` on the line
  right after an `errmsg = ...` assignment shows positive hits while that `errmsg` line — the
  *same* never-taken branch, one line above — reliably shows 0. Root cause not fully diagnosed
  (build uses `-O0`, so this isn't ordinary optimizer basic-block merging); treat as gfortran/gcov
  bookkeeping quirk, confirmed via the sibling line's zero count, not a real coverage gap.
- **A CI-only (toolchain-dependent) miss on the first executable statement of an abbreviated
  `module procedure ... end procedure` body, right after its declaration block** — seen in
  `parquet_metadata_maml.f90`'s `parquet_load_maml_file`/`parquet_load_qc_maml_file`, both of
  whose opening `call parquet_read_maml_source_lines(...)` line. Unlike the two shapes above, this
  one is *not* reproducible with a local `tools/coverage.sh` run (that run shows this file at a
  clean 100%, including these exact lines) — it only shows up in GitLab CI's own gcov/gcovr
  invocation, but does so consistently (confirmed across 4 separate CI runs on the same commit,
  same two lines every time), which rules out a one-off parallel-`.gcda`-write race and points to
  a CI-image-specific gfortran/gcov version difference from whatever's installed locally. Both
  procedures are confirmed exercised end-to-end (`test_load_maml_file`/`test_load_qc_maml_file` in
  `test/test_maml.f90`). If a *new* abbreviated `module procedure` body's own first statement
  (immediately following its declarations, no intervening blank-line-only gap) shows the same
  "0% only in CI, 100% locally, reproducible across ≥2 CI runs" pattern, treat it the same way —
  don't assume it's this shape from a single CI run alone, since a genuine one-off race is also
  possible; require the repeat-across-runs confirmation first.

**Convention for tagging a confirmed site** (mirrors `src/parquet_wrapper.cpp`'s own "gcov
attribution artifact under GCC" convention below, adapted for Fortran's stricter 132-column limit):
carry the literal phrase `gcov attribution artifact` either inline on the same
`GCOVR_EXCL_START`/`GCOVR_EXCL_LINE` marker line, or — when appending it inline would blow the
132-column limit — on a plain comment line (or a short contiguous run of them) directly above the
marker line instead. `tools/coverage.sh`'s `gcovr_artifact_lines()` recognizes both forms and drops
those lines from the "candidates for a stale/no-longer-dead exclusion" report, so a positive hit
there is expected and not something to chase. Before tagging a *new* site this way, verify it's
actually this pattern (check the raw per-line JSON hit counts the way this note's examples were
verified — a guard body genuinely showing 0 while its own `if` line shows positive, or a sibling
line in the same unreachable branch showing 0) rather than assuming; an exclusion that's truly gone
stale (the guarded code is now reachable and should count as covered) needs the marker removed
instead, not tagged as an artifact — see `parquet_strings.f90`'s `compact_all` header line, which
was found to be exactly that case (a real, now-covered subroutine header, not this artifact) and
had its `GCOVR_EXCL_LINE` removed rather than tagged.

### `src/parquet_wrapper.cpp`: GCC vs Clang gcov attribution

Confirmed via a real GitLab CI run: GCC's actual gcov and Clang's `llvm-cov gcov` (the default
backend `tools/coverage_cpp.sh` uses locally) attribute per-line hit counters differently for
several C++ source shapes, even for code that unquestionably executes under both. This is why
CI's coverage percentage can diverge from a local `tools/coverage_cpp.sh` run on the identical
commit, and why some `GCOVR_EXCL` markers sit on lines that are demonstrably, constantly
covered — they aren't dead code, gcov just can't always see it, so don't strip those markers as
if they were wrong. Divergent shapes found so far:
switch `case`/`default:` labels (especially the first label of a fall-through group), a function's
closing `}` immediately after its own `return`, a lambda's parameter-list line, and continuation
lines of one multi-line chained statement (`xml << ... << ...`, a multi-line `fprintf`/function
call). Before assuming a new CI-only-uncovered line is a real gap, check whether it's one of these
shapes and whether the surrounding code is otherwise demonstrably covered (e.g. by a call-graph
check, or the fact that a dependent test's assertions pass) — if so, it's this phenomenon, not a
missing test.

**Mechanical conventions to preserve when adding or moving a `GCOVR_EXCL` marker in this file**
(violating any of these silently reintroduces a local/CI coverage mismatch or miscategorizes a
report entry):

- **`GCOVR_EXCL_STOP` must be on its own dedicated comment-only line, never trailing real code**
  (`<code>; } // GCOVR_EXCL_STOP` is wrong). Real `gcovr` (CI) does not extend the exclusion to a
  `STOP` line that also carries code — only lines strictly between `START` and such a line are
  excluded, leaving that line's own code counted as an ordinary (uncovered) line. This is *not*
  what `tools/coverage_cpp.sh`'s own exclusion logic does locally (it always includes the `STOP`
  line itself) — the local tool is more lenient than the tool CI actually runs, so a `STOP` line
  that carries code can pass locally yet count as uncovered in CI.
- **A `case`/`catch`/`default:` label sitting just outside its block's `START`/`STOP` (immediately
  before `START`, or immediately after a preceding `STOP`) needs the marker moved to include it.**
  GCC gives a label its own line-attribution, separate from the code that follows it — a marker
  that only wraps the body leaves the label itself reported as a false gap.
- **A new exclusion added because a line is genuinely covered but GCC misattributes it (as opposed
  to genuinely dead/unreachable code) must carry the literal phrase `gcov attribution artifact
  under GCC` on the *same physical line* as its `GCOVR_EXCL_LINE`/`START` marker.**
  `tools/coverage_cpp.sh`'s reporting categorizes exclusions by searching for this exact substring
  on the marker's own line — wrapping the phrase onto a following, unmarked comment line silently
  drops it into the "other" (dead-code) bucket instead, and the new "zero hits" report won't catch
  it either.

**Confirmed root-cause mechanisms behind why so many lines in this file need `GCOVR_EXCL` at all**
(general knowledge for any future coverage work here, not just the GCC/Clang split above):

- `report_fatal_error` and `ConcurrencyGuard`'s constructor call `std::abort()` directly.
  `std::abort()` skips every `atexit`-registered handler, which is how gcov/llvm-cov flush a
  translation unit's counters — so a process that aborts contributes **zero** coverage data for
  that entire run, for every line it executed, not just the abort line itself. This applies to
  *every* `report_fatal_error` call site, present and future (`.gitlab-ci.yml`'s gcovr invocation
  auto-excludes standalone `report_fatal_error(...)` lines via `--exclude-lines-by-pattern`, mirrored
  by both coverage scripts' own `EXCLUDE_LINE_PATTERNS`) — but a helper function called just
  *before* the abort needs its own reasoning check, not an assumption that it's covered elsewhere.
- An uncaught C++ exception crossing the `extern "C"` boundary hits the same
  discard-all-coverage-data wall via `std::terminate()`, not just `report_fatal_error`. This file
  has exactly one working `try`/`catch` in its ~6800 lines (`parquet_reader_set_filter`) — even an
  artificial `throw` placed at the very top of a function it calls, invoked from directly inside
  that `try` block, was confirmed to escape uncaught under this project's mixed
  gfortran-driven static-library link. Treat any other `throw` site in this file as unreachable by
  a clean test by default.
- A Fortran-side pre-check often makes a C++-side defensive branch unreachable through the public
  API (e.g. `parquet_read.f90`'s `parquet_check_read_row_count`/`check_column_exists` make several
  "mismatch"/"not found" `report_fatal_error`s in `parquet_wrapper.cpp` dead code). Before writing
  a test for an uncovered `report_fatal_error`/`throw` line, `grep` the relevant
  `src/parquet_*.f90` call site for an equivalent pre-check first.
- Parquet C++'s Arrow reader always materializes a decimal column as `Decimal128Array`/
  `Decimal256Array` on read, never `Decimal32Array`/`Decimal64Array`, regardless of the physical
  `DECIMAL32`/`DECIMAL64` width actually written — so those case arms in `decimal_value_at`/
  `decimal_to_int64_checked` are permanently dead on the read path, not a testing gap, on any
  input.

If you ever find a line marked `GCOVR_EXCL_LINE`/inside a `GCOVR_EXCL_START`/`GCOVR_EXCL_STOP`
block that is actually reachable in normal (non-abort) operation — i.e. the exclusion looks
wrong, not just the line being hard to test — stop and notify the user about it rather than
silently leaving it excluded or removing the marker yourself.

**A specific gcov/gfortran quirk to know about, not chase:** in `src/parquet_temporal.f90`, six
`impure elemental` procedure header lines (`date_parse`, `date_new`, `time_parse`, `time_new`,
`ts_parse`, `ts_new_civil`) never register as "hit" in gcov even though every other line of each
one's body does — including their own `error stop` lines, which only execute on the actual abort
path a dedicated error scenario triggers, proving the procedure genuinely runs end to end. No
common dummy-argument shape distinguishes them from this same file's other, normally-attributed
`impure elemental` procedure headers (e.g. `ts_set_civil`, same "optional argument" shape, is
attributed fine) — root cause not identified, just confirmed to be a line-attribution artifact via
the fully-covered body. Marked `GCOVR_EXCL_LINE` with an explanatory comment at each site, same
category as `end module`/`end submodule` above. If a *new* elemental procedure header in this
file (or a future sibling module) shows the same "0% but body covered" pattern, treat it the same
way — confirm the body is fully covered first (that's the only way to tell it apart from a
genuine gap), then exclude with a comment rather than spending more time chasing it.

### Regression tests for "sized/typed from the first element" bugs

This bug class specifically affects **character vectors/arrays**: a fixed per-element length
gets derived from the *first* element's own length instead of the true maximum, silently
truncating every later, longer element. E.g. for a string vector `['a', 'bc', 'cd']`, if the
length is (incorrectly) derived from the first element `'a'` (length 1), the second and third
elements arrive truncated to `'b'` and `'c'`.

When writing a regression test for this bug class (e.g. the string-vector-column tests in
`test/test_reading.f90` — `test_read_string_vector_short_first`), construct the fixture so
the first element is deliberately the extreme/shortest case and a later element is longer —
i.e. whenever a test needs to provide a string array, deliberately make the *first* element the
shortest one, to actively try to trigger this bug rather than merely avoid it by accident.
A fixture where the first element happens to be the longest (or same-length) can pass even
if the underlying bug is still present.

### A Fortran-side debug hook has to be PUBLIC, so prefer a C++ one

Every `parquet_debug_*` hook that forces library state for a test is a C++ `extern "C"` function,
and a test reaches it by declaring its own local `bind(C)` interface (never via
`src/parquet_bindings.f90`) — which is what keeps debug entry points out of the library's own
interface entirely. **A hook over state that lives on the Fortran side has no such escape hatch.**
`parquet_table_cache`'s components are private to `parquet_tables`, so anything forcing them must be
a public procedure in that module, visible to every `use parquet`.

There are two groups of them, both accepted deliberately rather than by default.
`parquet_debug_table_set_inflight` (`parquet_tables_parallel.f90`, declared in
`tools/generate_parquet_tables.py`'s template) is the first: it forces the append/read in-flight
counters so that the two concurrency aborts can be provoked from **one thread**, deterministically. Without it those aborts need two
threads to overlap on demand, and a timing-dependent test is worse than no test — it passes on a
quiet machine, fails on a busy one, and gets disabled. See `feature_risks.md` Risk-6.

**`parquet_debug_table_drop_name_index` (`parquet_tables_query.f90`, same template) is the same
group and shows the OTHER shape that forces a Fortran-side hook: a fallback with no route to it
through the public API at all.** `cache_find`'s linear scan exists for a mutation that forgets to
maintain the name index, so a correct library never reaches it — it was measured executing zero
times across the whole suite — and the only way to test a safety net nothing can trip is to drop the
index deliberately. Note its `had_index` argument is **required, not optional**: both lookup paths
return identical answers, so a test that skipped it would pass against a hook that did nothing. See
`feature_risks.md` Risk-75, and prefer that shape — a hook that *reports what it changed* — whenever
the forced state is otherwise unobservable from the test.

**The second group is `parquet_strings`' four** (`parquet_debug_set_string_min_bytes`,
`parquet_debug_set_string_max_auto_threads`, `parquet_debug_string_row_ranges`,
`parquet_debug_string_bulk_threads`), and they are what shows the rule below is a *preference* rather
than a possibility: that module reaches no `bind(C)` surface at all by design, so the C++ route is
not available to it at any price short of ending its independence. **Two of the four exist because a
threshold no test-sized fixture can reach is a threshold no test exercises** — a 256 KiB payload floor
and a 64-thread ceiling both sit far above anything a unit test builds, so without an override every
test would silently take the path the constant was written to avoid (`feature_risks.md` Risk-49).
Expect a new tuning constant to need one.

Rules for a future one:

- **Reach for the C++ side first.** If the state can be forced from `parquet_wrapper.cpp`, do it
  there and keep the hook invisible to Fortran users. Only when the state is Fortran-side and behind
  private components does a public procedure become the only option.
- **A public debug hook carries a doc-comment saying it is test-only** and why it has to be public,
  and is called by no library code. Follow
  `parquet_debug_table_set_inflight`'s shape rather than inventing a second convention.
- **Do not put a hook in the hot path to avoid making it public.** Having
  `table_check_no_append` consult a C++ flag would work and would keep the hook private — and would
  add a `bind(C)` call to the choke point every value accessor passes through. That trade was
  considered and rejected; see Risk-6's "the read path must stay free of atomics".
- **Every scenario a hook enables still needs its negative control**: make the guarded call once
  with the hook clear before setting it, or the scenario passes just as happily against a guard that
  fires unconditionally.

### Guarding a hard Arrow int32-only ceiling

Some Arrow/Parquet C++ APIs are hard-capped to a plain `int32_t`, with no int64/"large" fallback
at all — found three times so far: `arrow::FixedSizeListBuilder`/`fixed_size_list()`'s `list_size`
(a vector column's per-row width, `col_size`), `arrow::Schema::num_fields()`/`GetFieldIndex()` (a
table's column count), and Parquet's own repetition/definition-level generation for list-typed
columns (`level_conversion.cc`), which walks every flattened element of a row group with a plain
`int32_t` counter (see apache/arrow#33188 / ARROW-17983, still open/unfixed upstream). Unlike the
other two, this last one is scoped to one **row group**, not a column's total element count
(`nrows * col_size`) — and `close_parquet_writer`'s row-group auto-sizing already keeps every row
group under it by shrinking the row-group size, however large `nrows` gets, so a large total is
never actually a problem (verified empirically with a multi-billion-element vector column). Only
an *explicit* `chunk_size=` (`parquet_open_writer`/`parquet_set_writer_options`) that itself
conflicts with a column's `col_size` still aborts, since that's a caller-forced value auto-sizing
can't silently override — see `check_chunk_size_fits_limit_for_col_size`/
`check_explicit_chunk_size_fits_arrow_limit`/`check_chunk_size_fits_metadata_limit` in
`parquet_wrapper.cpp`, and the README's Limitations section. This differs from row count
(`int64_t` throughout Arrow) or a string column's byte payload (which has an `arrow::large_utf8()`
fallback) — for those two (`col_size` and column count), there is no workaround, only a clean
failure instead of letting Arrow silently truncate/wrap internally. When a new int32-only ceiling
is found, guard it with the pattern already used for `col_size`/column-count above (see
`check_col_size_fits_arrow_limit`/`check_column_count_fits_arrow_limit` in `parquet_wrapper.cpp`)
— only reach for the row-group-scoped auto-sizing approach instead if the new ceiling is similarly
scoped per-row-group rather than per-column-total:

1. A `static constexpr int64_t kArrowInt32...Limit = 2147483647;` named for what it bounds.
2. A process-global `static int64_t g_debug_..._limit = -1;` test-only override plus a
   `parquet_debug_set_..._limit(int64_t n)` extern "C" function (`<=0` restores the real limit) —
   reachable only via a `bind(C)` interface declared locally inside the relevant
   `test/error_scenarios.f90` scenario, never `src/parquet_bindings.f90`. This lets the error
   scenario trigger the abort with a tiny fixture instead of actually building a multi-GB/
   multi-billion-element table.
3. A `check_..._fits_arrow_limit(...)` function comparing against `g_debug_..._limit > 0 ?
   g_debug_..._limit : kArrowInt32...Limit`, calling `report_fatal_error` (never a silent
   truncating cast) when exceeded — called at every place the value is about to flow into the
   truncating Arrow API, before the cast happens.
4. A matching `test/error_scenarios.f90` scenario (shrink the debug limit, trigger the abort with
   a tiny fixture) + `test/test_errors.f90` wrapper (`check_scenario_exit_status_and_stderr`,
   asserting the exact stderr message) + `tools/run_error_scenarios.sh` entry + a README
   Limitations bullet describing the ceiling and that it aborts cleanly rather than corrupting.

**A validity mask is expensive, and usually unnecessary — ask the footer before building one.**
Requesting `is_valid=` from `parquet_read_column` was measured at **+22% to +84%** on the read path,
and it is not one cost but four: an `int8` buffer allocated by `make_valid_buf`, a Fortran `LOGICAL`
mask (gfortran's default `LOGICAL` is **32 bits** — four times the buffer it is built from), a
conversion pass between them (`is_valid = valid_buf /= 0_c_int8_t`), and an O(n) `array->IsValid(i)`
scan in `check_or_report_nulls`. On the table path there was a fifth: a per-row `set_null` replay.

Parquet records a **null count per column chunk in the footer**, so whether any of that is needed can
be answered without reading a byte — that is `parquet_column_has_nulls` (public API) over
`column_has_nulls_from_footer` (C++). Every `mat_*`/`matchunk_*` asks first and omits `is_valid=`
entirely when the answer is no. Rules to preserve:

- **The uncertain answer must be `.true.`** ("might have nulls"). Statistics are optional in the
  format; claiming a column is clean when it is not makes `parquet_read_column` abort on the first
  Null. A dotted **struct path** is declined outright, because a struct leaf's validity is combined
  with every ancestor struct's by `unwrap_struct_path`, so the leaf chunk's own count does not
  describe the result.
- **A filter or sample being active does not invalidate the answer** — both only ever remove rows, so
  a column with no Nulls in the file has none in the result.
- **`matchunk_*` scopes the question to its own row group**, or a file with Nulls anywhere would force
  the mask onto every clean row group.
- **The `is_stats_set()` and `HasNullCount()` guards are mutually redundant and both load-bearing.**
  Removing either alone changes no test result; removing both segfaults on
  `test/fixtures/no_stats.parquet` (statistics-free by construction — the only fixture that reaches
  this path), because `statistics()` returns null there. Do not delete one as dead code.
- **Keep the two fallbacks** for when a mask genuinely is needed: `check_or_report_nulls` short-circuits
  to `memset` when `null_count() == 0`, and the per-row replay sits behind `if (.not. all(valid))`.

**A row transform is either a MASK or a PERMUTATION, and only the permutation restricts anything.**
`filter=`/`sample_fraction=` install a boolean mask; `sort_by=` installs an `Int64Array` permutation
(`sort_perm`). `apply_row_transform` applies them in that order — filter first, then sort within the
survivors — at the single choke point every whole-column decode goes through. The distinction is
load-bearing for every guard: a mask only ever *removes* rows, so row groups stay contiguous and
chunked reads, `parquet_get_chunk_size` and row/element mode all work under one (row-group-scoped,
against each group's surviving count). A permutation *reorders* rows, so sorted row 5 may come from
row group 47 — nothing row-group-scoped survives it. **Every "not while transformed" check must key
on `reader_has_sort_permutation` (C++) / `check_reader_no_sort` (Fortran), never on the mask**;
widening either to "any transform" silently re-bans everything filtering supports, and dropping them
silently returns physically ordered rows from a sorted reader. Row mode and element mode each route
their whole-column fallback through ONE decision point (`fetch_row_mode_array` and a branch inside
`stream_element_mode_row_groups`) rather than per-entry-point branches, so a new type family cannot
miss it.

**The sort engine is deliberately free of reader state** (`SortKeyData`, `sort_compare_key`,
`sort_build_permutation` in `parquet_wrapper.cpp`): its keys arrive as plain typed vectors, and only
`sort_bind_arrow_key` touches Arrow. That is what lets the same engine later serve `parquet_table`'s
in-memory sort (whose Arrow buffers are gone) and a possible public `parquet_sort` module over any
1-D array. Don't reach for reader state from anything under that banner. Two further rules:
its ordering **must keep reproducing `arrow::compute::SortIndices` exactly** (nulls/NaNs absolute,
never flipped by `descending`; ascending gives values → NaNs → nulls; ties hold file order), since
that is what makes a `pyarrow` cross-check agree row for row; and the **integer counting-sort fast
path is a second code path producing the same answer**, so it keeps its own tests plus the
`parquet_debug_set_disable_sort_counting_path` hook that forces the comparator path for comparison —
it is the one place in the engine where a wrong answer would be fast rather than slow.

**A new reader query that a sibling module needs has to be PUBLIC `parquet` API** — there is no
internal back door. `parquet_reader`'s components are `private`, so `parquet_tables` (and any future
sibling module) cannot reach `%handle` and therefore cannot call `parquet_bindings` directly on a
reader; it must go through a procedure in `parquet` that takes the `parquet_reader` object. That is why
`parquet_column_has_nulls`, `parquet_measure_list_width` and `parquet_column_width_needs_data` are all
public rather than internal plumbing, and why each needs the full public treatment (dual int32/int64
kinds for numeric arguments, `!>`/`!!` docs, a README API-overview entry). Budget for that when a new
one is needed; do not try to widen `parquet_reader`'s component access instead.

**Fill a `parquet_column` with `%adopt`, not `%init` + `%set_all`, when the source array is a
temporary.** `set_all` is a full array assignment into storage the column just allocated, so the
obvious "read into `tmp`, then store it" shape costs an extra full pass and, transiently, a second
live copy of the column. `%adopt` (generated for all 16 array kinds) hands the allocation over with
`move_alloc` and takes kind, width and row count from the array itself, so it replaces `init` rather
than following it. The two string kinds have no `adopt` — they own a `parquet_string_column`, not a
plain array. Together with the mask skip above this took `materialize_all` on an 8 GB, 16-column
null-free file from 5.67 s to 3.54 s.

**Beware benchmarking this on one toolchain only.** The report that prompted the work measured
`materialize_all` at 1.84x a raw read with **Intel `ifx` on a 100+ core, ~800 GB machine**; the same
comparison with gfortran on a 32 GB laptop showed the table path at *parity or faster*, because
releasing Arrow's buffers as it goes shrinks the working set enough to pay for the extra passes. The
per-row loops that dominate under `ifx` are nearly free under gfortran. When a performance claim about
this layer cannot be reproduced, suspect the compiler before suspecting the report — and record which
one was used (see feature_materialize.md).

**Measuring a column's width must never materialize it, and only ONE column type needs data at
all.** `parquet_get_col_size`/`parquet_get_column_total_elements` answer from the schema for every
type except a plain `LIST`/`LARGE_LIST` — a scalar column's width is 1 by construction and a
`FIXED_SIZE_LIST`'s is `list_size()`. Only a plain `LIST` genuinely needs the data, because Arrow
lets every row hold a different length, so whether one uniform width exists is a property of the
data (`needs_data_to_measure_col_size` is the single predicate encoding this). Two facts make this
narrower than it looks: any Arrow-based writer preserves `FIXED_SIZE_LIST` via `store_schema()`, so
the plain-`LIST` case only arises for a non-Arrow writer or genuinely ragged data; and
`STRUCT`/`MAP`
never reach the question at all (`collect_column_leaf_paths` expands a struct into per-leaf paths,
and a `MAP` fails `parquet_column_exists`'s `types=` probe first).

When it does need data, it is resolved in two tiers rather than by a whole-column read — keep both:

1. **A footer screen** (`list_width_candidate`): per row group, `num_values / num_rows`. A
   non-integral mean, or two row groups disagreeing, proves no uniform width exists, for free. A
   surviving answer is a **candidate, never a proof** — rows of length 3,1,3,1 average to exactly 2,
   and a null or empty list occupies exactly one leaf slot (both verified empirically, not assumed;
   see `test/fixtures/list_widths.parquet`'s `avg_ok` and `null_avg` columns).
2. **A row-group scan** (`list_width_verified`) only for a survivor, bailing at the first
   disagreement, so peak memory is one row group. It deliberately does **not** use
   `get_row_group_chunk_array` — that one records the row group as read
   (`parquet_reader_check_complete`)
   and runs qc, neither of which measuring a column should cause. Use
   `read_row_group_array_for_measuring` instead, or a similar side-effect-free read.

**`parquet_table` never pays for the scan on the read path**, and that is deliberate: `table_touch`
resolves a deferred column with the *unproven candidate* (`table_resolve_width(...,
proven=.false.)`)
because `get_uniform_list_values` already checks every row's length against the width it was given
and aborts on a mismatch — so the read that was going to happen anyway doubles as the proof, keeping
`%prefetch` to one pass. Only `%kind`/`%width` pass `proven=.true.`, since answering a metadata
query
with a guess would silently mis-type the column. Two consequences to preserve if this is ever
refactored: an empty column measures as **0**, not 1 (`parquet_get_col_size` has always reported 0
for
a zero-row list column — the Fortran wrapper must not clamp it, the table descriptor clamps with
`max(w, 1)` itself); and a *slice* measures over its own row groups, so a globally-ragged file can
present a uniform width within one slice and two tables over the same file can legitimately
disagree.

**A new non-touching query must be checked against this.** `%kind`/`%width` had to become
touch-triggering for deferred columns; `%unit`, `%residency` and `%is_supported` deliberately did
not
(`%residency` would always report `RES_FULL`; `%is_supported` comes from the element type alone,
since
`table_kind_from_type`'s `ok` never depends on `col_size`). Anything reading `declared_kind` or
`width` without going through `table_resolve` or `table_resolve_width` will read `PK_NONE`/0 for a
deferred column.

**Read side of the `nrows * col_size` ceiling: row-group-scoped, not a ceiling at all.** Unlike
the write side above, `parquet_get_col_size`/`parquet_get_column_total_elements`/
`parquet_read_array_row_mode`/`parquet_read_array_element_mode` on the *read* path must **not**
materialize the *whole* column via `get_single_chunk_array`'s `ReadColumn` (Arrow's whole-file,
all-row-groups-at-once convenience API) just to answer a size query, fetch one row, or fetch one
element position across all rows: doing so trips Arrow's own internal int32 list-index/offset
limit once `nrows * col_size` crosses int32, even though the column was written perfectly safely
(every row group under the limit, per the write-side guard above). So all four are genuinely
row-group-scoped rather than guarded — keep them that way, don't revert to a whole-column read:
`parquet_get_col_size`/`parquet_get_column_total_elements` read `col_size`
straight off the schema's `FixedSizeListType::list_size()` (no data read at all) for a
FIXED_SIZE_LIST column — **but only for that case: a plain `LIST`/`LARGE_LIST` column, which this
library never writes but another tool can, has no schema-level width, so both fall back to
`list_width_verified`, which screens each row group from the footer and then proves it by reading
one row group at a time — never the whole column.** (`total_elements` used to call
`get_single_chunk_array` here and decode everything at once, while its sibling had already been
moved to the row-group-scoped helper; the two now share it. Nothing about the *answers* changed —
`get_col_size` and `list_width_verified` agree by construction — which is exactly why the
asymmetry survived: only a memory measurement could see it, and no test asserts memory.) Any code path that asks for
`col_size` on every column of an arbitrary file (`parquet_table`'s open-time classification is the
existing example) is therefore not automatically metadata-only, and should release afterwards
(`parquet_release_column`, a no-op when nothing was decoded) rather than assume nothing was read.
`parquet_read_array_row_mode` resolves which row group a given
`row_index` falls in (`resolve_row_group_for_row`, walking each row group's `num_rows()` from the
file footer) and reads only that one row group (`get_row_group_chunk_array`, the same helper the
`_column_chunk` family already used) rather than the whole column.
`parquet_read_array_element_mode` is different in kind from the other three: it inherently needs
every row's value at the same fixed column position, i.e. data from *every* row group — it can't
skip all but one the way row_mode does. So it streams row group by row group rather than reading
only one: `stream_element_mode_row_groups` walks every row group, reads each
one's own chunk via `get_row_group_chunk_array`, extracts just that row group's rows' values at
the fixed offset, and writes them into the correct slice of the caller's already-allocated
`nrows`-length output arrays — so no single Arrow call ever has to flatten more than one row
group's worth of elements, even though the final output still spans the whole file.
`resolve_element_mode_col_size` mirrors `parquet_get_col_size`'s own schema-only col_size lookup,
so element mode doesn't need a whole-column read just to validate `col_index`/compute the stride
offset either. **An active row filter/sample does not change any of this.** `row_index`/
`elem_index` then address the filtered result rather than a physical file row, and both are
resolved against each row group's *surviving* count instead of the footer's physical one —
`row_group_effective_rows`, which `resolve_row_group_for_row` and
`stream_element_mode_row_groups` both walk, so one function covers the masked and unmasked cases
and neither mode has a separate filtered branch any more. This works because
`get_row_group_chunk_array` hands back a chunk with that row group's own mask segment already
applied, so a local index within the returned chunk *is* a rank among survivors. A row group with
no survivors contributes 0 and is stepped over, exactly as a physically empty one already was.
The one remaining whole-column read in this area is `list_width_verified`'s masked branch, for a
plain `LIST`/`LARGE_LIST` column only (a width measured per row group under a mask would answer
about rows the caller filtered away). Regression-tested via `test/error_scenarios.f90`'s
`scenario_col_size_and_row_mode_avoid_whole_column_read` (a process-global
`g_debug_force_whole_column_read_error` hook forces `get_single_chunk_array` to abort the instant
it would actually read a whole column, on a tiny fixture — the scenario finishing without
aborting proves none of the four calls took that path), its filtered counterpart
`scenario_filter_row_element_mode_no_whole_column_read` (same hook, armed after a filtered open,
across all four entry-point families: the int32 template, the hand-written logical and string
pair, and the temporal template), plus the shared negative control
`scenario_whole_column_read_forced_error_control` (proves the hook itself actually fires).

---
> Source: [etempel/parquet-fortran](https://github.com/etempel/parquet-fortran) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
