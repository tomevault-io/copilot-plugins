## yolo-particle-segmentation

> Run speckit specify workflow when user types /speckit.specify


# /speckit.specify command

When the user types **`/speckit.specify`** (with optional feature description after it), treat it as the **speckit specify** workflow. Cursor does not load `.specify/commands/`; this rule makes the same behavior available in Cursor.

## What to do

1. **User text after the command** is the feature description (e.g. `/speckit.specify Add user login` → "Add user login"). If they only type `/specify` with no description, ask for the feature description.

2. **Generate a short name** (2–4 words) from the description (e.g. "user-auth", "oauth2-api").

3. **Run the create-new-feature script** from repo root:
   - Run `git fetch --all --prune` first.
   - Find the next branch number for that short-name (check remote branches, local branches, and `specs/N-short-name` dirs).
   - Run: `.specify/scripts/bash/create-new-feature.sh --json --number <N> --short-name "<short-name>" "<feature description>"`
   - Use the script's JSON output for `BRANCH_NAME`, `SPEC_FILE`, `FEATURE_DIR`, etc.

4. **Load** `.specify/templates/spec-template.md` and write the specification to `SPEC_FILE` following the template (User Scenarios, Requirements, Success Criteria, etc.). Follow the same content rules as in `.specify/commands/speckit.specify.toml` (no implementation details, testable requirements, max 3 [NEEDS CLARIFICATION] markers).

5. **Optionally** create the quality checklist at `FEATURE_DIR/checklists/requirements.md` and validate the spec as described in speckit.specify.toml.

6. **Reply** with branch name, spec path, and that they can run `/speckit.clarify` or `/speckit.plan` next (or the Cursor equivalent workflows if you add rules for those).

## Paths

- Repo root: directory containing `.specify/` (and usually `.git/`).
- Script: `.specify/scripts/bash/create-new-feature.sh`
- Spec template: `.specify/templates/spec-template.md`
- Full workflow reference: `.specify/commands/speckit.specify.toml`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PRIYE)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/PRIYE)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
