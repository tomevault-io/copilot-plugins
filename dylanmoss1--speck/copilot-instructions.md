## speck

> Speck is a specification workflow for AI-assisted programming — inserting a new design step between 'Plan Mode' and 'Code Generation'.


# Speck

## Skill Motivation

**Speck is a specification workflow for AI-assisted programming — inserting a new design step between 'Plan Mode' and 'Code Generation'.**

**Plan Mode** generates large blocks of text, which are hard to debug and reason about precisely.

**Code Generation** is costly, we want to _fail fast_ rather than fix design flaws at this stage.

What we need is a stage in between: a flexible specification language which improves human-AI communication while keeping you in control of code generation.

Benefits:
- Faster, more precise communication with LLMs.
- Fail fast: questions and design decisions are front-loaded.
- Hands-off implementation, with confidence that LLMs understand your design.
- Visualise and guide the final structure of the program.

Models are getting smarter, LLM-coding is getting better, but human-AI communication remains the same — speck aims to bridge this gap with a specification tool which can integrate into any existing workflow.

The purpose is communication: speck exists so the human and the AI converge on a precise, reviewable description of a change *before* any code is written, catching misunderstandings while they are still cheap to fix. The speck diff is the medium for that agreement. When rules conflict, the three overriding rules (see [The three rules that override everything](#the-three-rules-that-override-everything)) decide first; when a situation arises that no rule directly covers, fall back on this purpose — do whatever best keeps the human and AI working from the same precise, agreed description.

## The speck diff

The **speck diff** is the `git diff` between the 'before' speck files (committed as the Base Commit in step 5) and the 'after' speck files (in the working directory). It is the canonical specification of the feature: everything in it gets implemented, nothing outside it does.

The speck diff can be obtained with:

```sh
git diff
```

The diff is expected to contain only `.speck` files (the gate-check in rule 9 enforces this), so no path filter is needed. If `git diff` shows a non-`.speck` file, source changes have leaked and you must stop and investigate. Adapt the command to the user's VCS if it is not `git` (see step 2).

## Workflow at a glance

1. **Get planned feature** — plan mode ends and the plan is approved (step 1).
2. **Get the VCS** — detect git/jj/etc. and adapt the commands accordingly (step 2).
3. **Check the git state** — commit any outstanding changes to a temporary commit, kept separate from the speck baseline (step 3).
4. **Generate 'before' speck files** — capture the current state of relevant files, then format (steps 4–4.1).
5. **Commit the changes** — commit the 'before' speck files as the Base Commit (step 5).
6. **Generate 'after' speck files** — edit the specks to the target state, then format. The `git diff` is the feature (steps 6–6.1).
7. **Iterate on the design** — refine the speck with the user, then run a consistency check across files (steps 7–7.1).
8. **Tests** — ask whether they want test specks (default yes); if so, repeat the before/after flow for tests (step 8).
9. **Sign-off and implementation** — get explicit sign-off, then generate code by following the speck diff line by line (step 9).
10. **Further iterations** — keep speck files, code, and feature requirements in sync (step 10).
11. **Cleanup** — once the user verifies the code, delete the speck files and clean up the git history (step 11).

## Rules

### The three rules that override everything

If the rest of this document — the other rules, the workflow steps, the plan, or the conversation — ever appears to conflict with the following, these three win:

1. **No source code before sign-off.** Never edit a non-`.speck` file until the user has explicitly signed off on the speck.
2. **The speck diff is the contract.** Only what is in the signed-off speck diff gets implemented. Everything in it must be implemented. Nothing else.
3. **Never skip the workflow.** Every code change goes through the speck workflow, however small or mechanical it seems.

Everything below elaborates these three. When a lower rule or a workflow step seems to contradict one of them, these win. For any conflict these three do not resolve, fall back on the purpose (see [Skill Motivation](#skill-motivation)).

Never skip the workflow or any workflow steps:
- If the speck skill is invoked, it must start the workflow.
- Once the speck workflow has started, all the steps must be completed in order without skipping any steps.
- Once the workflow has started, it cannot end or exit until all steps are complete.

The only exception to the above rules are explicit instructions from the user to skip steps or end the workflow. For example, the user sends an explicit "skip the remaining speck workflow steps" instruction.

Common rationalizations that do NOT justify skipping:
- "The change is trivial / just a rename / only touches one file"
- "The user said 'implement' so they want code directly"
- "The plan is specific enough that a speck is redundant"
- "This is a mechanical change with no design decisions"
- "These are code review fixes, not new design work"
- "The plan already contains exact code snippets"

Workflow rules:
1. **No code before sign-off.** Until the user signs off on the speck, do NOT edit non-`.speck` files, and never edit a `.speck` file and its corresponding source file at the same time. (After sign-off, during the code phase, you do keep the two in sync — see steps 9 and 10 below.)
2. **No mention in Plan Mode.** Do not reference speck files during planning — they are irrelevant until the plan is approved.
3. **The speck diff is the contract.** The speck diff is the sole, canonical specification of what changes get made. If a change is not in the speck diff, it does not get implemented — no matter what the plan, conversation, or prior discussion says. If a change is in the speck diff, it must be implemented — no exceptions. During implementation, treat the speck diff the way a builder treats blueprints: follow it exactly.
4. **No overriding signed-off changes.** Once the user signs off on a speck, every change in it is final. Do not skip, weaken, or reinterpret any signed-off change based on your own judgment. If you have concerns (e.g. a comment contradicts the new code), implement the speck as-is and raise the concern with the user afterward.
5. **Annotations are directives.** Comments added to speck files (e.g. `# Change this to X`, `# Instead do Y`) are implementation instructions, not discussion notes. They carry the same weight as structural changes like renamed functions or modified signatures. Act on every annotation in the signed-off speck diff.
6. **No project config changes.** Do not modify project configuration files (e.g. `pyproject.toml`, `.gitignore`, `Makefile`, `tsconfig.json`) to accommodate speck files unless the user explicitly asks. Speck is a workflow tool used alongside other developers and tools that should not be aware of its existence. Speck files are temporary artifacts — they must not leave traces in the project's configuration.
7. **If the source has diverged, stop.** If the working-directory source differs from the base commit in ways that are not reflected in the speck files (e.g. a hotfix, a teammate's commit, an unfinished edit), do not generate a speck diff that silently ignores this. Surface the divergence to the user and agree on how to handle it before proceeding. A speck diff is only trustworthy when the "before" speck genuinely matches the committed source it was derived from.
8. **Confirm ambiguous sign-offs.** A user message like "looks good" or "yes" may mean "I approve this one modification" or "I approve the whole speck and you may proceed." If it is unclear which they mean, and you believe they may be signing off the entire speck (i.e. authorising you to move to code), check before proceeding. Do not treat an ambiguous approval of a single change as sign-off on the whole speck.
9. **Gate-check the diff.** Ensure the speck diff only contains `.speck` files with `git status --porcelain | grep -qv '\.speck' && exit 1 || exit 0` after every change to the diff. If any non-`.speck` file appears, you have leaked source changes: stop, stage or revert it, and investigate before continuing. The diff must contain speck files and nothing else.

## Workflow Steps

Follow these steps in a strict order. NEVER skip steps. See Rules for a list of rationalizations that do NOT justify skipping.

Use these step numbers internally to keep track of where you are in the workflow and to avoid skipping steps. But avoid referring to these step numbers when communicating with the user as they are not aware of the skill's internals.

### 1. Get planned feature

If the user has already planned a feature, immediately continue to step 2.

Else, work with the user to plan their feature and produce a 'Plan'.

### 2. Get the VCS

Check what VCS the user is using. The following instructions assume `git` by default, if the user is not using `git` as their main VCS then adapt the instructions accordingly.

Check for common files such as `.git`, `.jj`, etc.

If there is more than one, pick the most appropriate one as their main VCS (for example, if `.git` and `.jj` are present, use `jj` as the VCS not `git`).

### 3. Check the git state

Check that there are no unstaged or outstanding changes in the git repository.

If there are, then commit these files under a temporary commit (kept separate from the speck Base Commit, and undone at cleanup in step 11). If there are files you think you should not commit (e.g. large files, `.env`) then check these with the user first.

Confirm with:

```sh
git diff --quiet && git diff --cached --quiet
```

If this command fails then retry this step from the beginning.

### 4. Generating 'before' speck files

Create speck files representing the state of the codebase 'before' the proposed feature is implemented.

Only create speck files for files that are relevant to the change.

Leave speck files for tests until step 8 unless these are an explicit part of the proposed feature.

If the 'before' state does not contain a speck file but the 'after' state does, then create a speck file for the 'before' state but leave it EMPTY.

#### Speck Files

Speck files live next to their corresponding original files in the file structure.

`repo/my_program.py` gets the additional speck file `repo/my_program.speck.py`.

This workflow is language-agnostic. A speck file mirrors its source with the same extension (`main.speck.py`, `lib.speck.ts`, `mod.speck.rs`, and so on) and the same transformation: keep structure and signatures, drop implementation. The examples in this document use Python, but the workflow applies unchanged to any language.

To generate a speck file from a source file, copy the source but remove all concrete implementation. Preserve:
- Class and function definitions (signatures only)
- Type signatures, import statements, decorators, constants
- All docstrings

Copy all relevant parts — do not leave sections omitted. Replace method bodies with an empty / incomplete implementation marker (e.g. `...` in Python and `todo()` in Rust).

Capture every kind of change the feature requires in the speck: new files, modified signatures, removed definitions, added imports, and changed constants. The speck diff is the contract (see [The speck diff](#the-speck-diff)) — if a change is not visible in it, it will not be implemented; if it is visible in it, it must be implemented.

The diff should also contain a list of all references / calls to user-defined constants and functions in relevant functions (listed in the docstring under headings `CONSTANTS` and `CALLS`). Include these for any function that is new, modified, or whose call graph changes. For modified functions, include both the "before" and "after" states (ensure the full list is provided for both so the diff is clear as to what has changed); for new functions, include the "after" state only.

If the function's behaviour is not clear from a docstring alone, if the docstring would become long and hard to read, or if there are implementation-specific changes (rather than behavioural changes), you can include basic pseudocode in the docstring to describe the function's behavior / implementation.

For example this might look like:

```python
# URL to the postgres DB
DB_URL: str

def add_row_to_db(row: Row) -> None:
    """Insert a row into the database, retrying on failure.

    PSEUDOCODE:  # Only include if needed
    1) parse and validate the row
    2) send row to db
        2.1) attempt to update db with row
        2.2) if this fails, backoff and retry at most 3 times

    CONSTANTS:
      - DB_URL: str

    CALLS:
      - parse_row(...)                           # Use '...' for args if this
                                                 #   function is not important
                                                 #   to this speck change
      - send_to_db(db_url: str, row: Row)        # Otherwise include argument
                                                 #   names and types
      - send_to_db_with_backoff(                 # If the function is too long,
            db_url: str,                         #   split it across multiple lines
            row: Row,  # Parsed + validated row  # Optionally comment
                                                 #   constants / functions / args
                                                 #   if they are important to this
                                                 #   speck (only do this if relevant)
            backoff_time: int
        )
    """
    ...
```

### 4.1. Formatting

Once all 'before' state speck files have been created, check if the codebase uses a formatter. If it does, run this formatter on the speck files (ignore any speck-specific errors).

### 5. Commit the changes

Commit only the speck files as the Base Commit, with the message `speck [before state]: <very short description of changes>`. Stage the `.speck` files explicitly (do not use `git add -A`) so the Base Commit contains speck files and nothing else — the user's own outstanding work stays in the separate temporary commit from step 3.

### 6. Generating 'after' speck files

Edit all speck files to represent the program specification 'after' the proposed feature is implemented.

Make liberal use of comments and docstrings to highlight changes that cannot be easily represented with program structures and types.

**IMPORTANT:** Every part of the change must be captured in the speck diff (the contract — see [The speck diff](#the-speck-diff)). Assume conversation context might be cleared before implementation begins, so nothing can be left implicit.

Always check that the only files in the diff are `speck` files. Confirm with the rule 9 gate-check:

```sh
git status --porcelain | grep -qv '\.speck' && exit 1 || exit 0
```

Common rationalizations that do NOT justify omitting this speck diff status check:
- "The diff is small / one file / obviously only speck files"
- "I'll eyeball it instead of running the command"
- "I just ran it a moment ago" (state may have changed — run it again)

**IMPORTANT:** Run `git diff` to ensure the speck diff represents the comprehensive and complete set of changes required by the feature.

#### Speck diff example

Given `main.py` with functions `fun1` and `fun2`, to implement a feature that removes `fun1` and adds `fun3`:

- Base Commit: `main.speck.py` contains `fun1` and `fun2` (run formatter if it exists).
- Working directory: `main.speck.py` modified to remove `fun1` and add `fun3` (run formatter if it exists).

`git diff` shows: `fun1` removed, `fun3` added, `fun2` preserved.

### 6.1. Formatting

Once all 'after' state speck files have been created, check if the codebase uses a formatter. If it does, run this formatter on the speck files (ignore any speck-specific errors).

Take care that empty / incomplete implementation markers should not move between the 'before' and 'after' state unless the function is modified.

### 7. Iterating on the design

Now stop and check the speck with the user. Do not continue until the user has given approval.

**IMPORTANT:** Before every conversation, check if any of the speck files have been modified by the user.

Incorporate user directives (e.g. `# Change this to X`, `# Instead do Y`) as implementation instructions into the speck file definition, updating comments and docstrings if necessary.

It is your responsibility to keep the speck diff up-to-date with the requirements of the feature change. If the feature requirements change you must ALWAYS change the 'before' and 'after' speck files to reflect this.

When making changes to the 'before' and 'after' speck files, ALWAYS run `git diff` to ensure the speck diff represents the comprehensive and complete set of changes required by the feature.

Speck files are a shared language between you and the user. Make constant use of them to communicate technical details with the user.

To modify the Base Commit (e.g. to add new 'before' state files):

```sh
git stash -u
# Edit before-state speck files
# Run formatter if it exists
git add <files>
git commit --amend
git stash pop
```

### 7.1. Consistency check

Before moving onto the next step, first stop and review speck files for errors, inconsistencies, and contradictions. Work with the user to fix any issues found before proceeding.

### 8. Tests

Check with the user to see if they want to create speck files for program tests before implementation.

If they say yes: repeat steps 4 -> 7.1 for all test files related to the feature change.

If they say no: proceed straight to the next step. But still implement the tests and create speck files for the tests after they have been generated.

### 9. Sign-off and implementation

Sign-off steps:
1) To proceed, the user must explicitly say "I sign off on the speck changes"
2) Before generating code, review all speck files for errors, inconsistencies, or contradictions between files. Fix any issues found before proceeding.

**IMPORTANT:** Never generate any non-`.speck` files until explicit implementation signoff has been given. If it is ever ambiguous whether signoff has been given, check with the user first.

Implementation steps:
1) Check with the user for their preferred implementation strategy. For example, RED-GREEN-REFACTOR TDD, multi-agent setups, etc.
2) Do not create a new commit for the implementation changes. These **MUST** be in the same place as the 'after' speck changes, so the code and speck changes can be viewed together as a diff.
3) Generate code by following the speck diff line by line. 
4) The speck diff is the implementation checklist — every addition, removal, and modification it contains must be reflected in the generated code, and no other changes should be made. In long sessions especially, re-read the speck diff immediately before generating and again after, to confirm nothing was missed: the diff, not the conversation, is canonical.
5) After generating code, re-read the speck diff and confirm nothing was missed.
6) If the code generation edits files / functions not included in the speck diff, make sure to edit and add `.speck` files accordingly to ensure the speck diff directly represents both the feature change and the code representation.
7) Keep `.speck` files during and after code generation. These are not removed until step 11.

Implementation rules — Workflow rules 3 and 4 apply in full here:
- **The speck diff is the contract** (rule 3): implement every change in it and nothing outside it. Treat it the way a builder treats blueprints — follow it exactly.
- **No overriding signed-off changes** (rule 4): do not skip, weaken, or reinterpret any signed-off change based on your own judgment. If you have concerns, implement the speck as-is and raise the concern with the user afterward.

Common rationalizations that do NOT justify omitting code changes in a signed-off speck:
- "This comment is stale / misleading after the other changes"
- "This was just a discussion annotation, not a real change"
- "This is an annotation/comment, not a structural change"
- "A code reviewer flagged this as unnecessary"
- "This rename doesn't affect behavior"
- "I'll mention it to the user instead of implementing it"

### 10. Further iterations

**IMPORTANT:** Before every conversation, check if any of the speck files have been modified by the user.

It is your responsibility to keep the speck files up-to-date with the state of the codebase, and to keep the speck diff up-to-date with the requirements of the feature change.

If either the speck files, codebase, or feature requirements change you must ALWAYS update the files accordingly.

### 11. Cleanup

Once the user has given their final verification then cleanup the files and git history. This must be an explicit "I have finished the speck workflow and want to cleanup" from the user.

Cleanup:
- Delete all `.speck` files.
- Clean up the git history to remove intermediary speck git commits.
- If any unstaged or outstanding changes were committed to a temporary commit under step 3, make sure to undo these changes (restoring the user's original outstanding work to the working tree).

---
> Source: [DylanMoss1/speck](https://github.com/DylanMoss1/speck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
