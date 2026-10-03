## claude-vibe-skills

> This repo is the dev-skills skill (`skills/dev-skills/`). `SKILL.md` is

# claude-vibe-skills

This repo is the dev-skills skill (`skills/dev-skills/`). `SKILL.md` is
billed on every request of every session that loads the skill, and each
reference file is read in full when it loads, so every sentence is a
recurring cost.

## Cost and logic pass — after every skill update

Any change under `skills/dev-skills/` (a fix, a feature, applied lessons) is
not done until this pass has run over the changed files **and the files they
point at or duplicate**. Do it before the version bump is committed, and
report it in one line per finding (or "pass clean").

**Logic**
- Each rule still makes sense against the rest: no two files disagree, no
  reference points at a moved or deleted section, no step assumes a rule that
  was removed.
- The change didn't open a gap: a trigger that no longer fires, a check in
  `checks/enforce.py` whose denial text no longer matches the docs.

**Cost**
- **No redundancy.** A rule lives in one place; other files point at it. A
  restatement in `SKILL.md` of something a reference file owns goes.
- **No unneeded prose.** Cut sentences that explain, justify or repeat
  without changing what Claude does. Keep the *why* only where a model would
  otherwise talk itself out of the rule.
- **Right file, right trigger.** Content needed only on one path (release
  track, one platform, one gate) lives in a file loaded only on that path.
  Anything new in `SKILL.md` must have to fire unprompted.
- **Cheap to follow.** A new step doesn't add round trips, questions or
  re-reads the session didn't need; it chains into existing calls.

For a large pass, the `skill-pass` agent (`.claude/agents/`, Sonnet,
read-only) can do the reading and report findings to apply.

Then rebuild and validate: `bash scripts/build-skill.sh && bash scripts/validate.sh`.

---
> Source: [darthrater78/claude-vibe-skills](https://github.com/darthrater78/claude-vibe-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
