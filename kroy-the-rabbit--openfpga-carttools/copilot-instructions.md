## openfpga-carttools

> **Before every commit, check that the documentation still describes what the

# Working rules for this repository

## Docs must align before any commit

**Before every commit, check that the documentation still describes what the
code does.** Not after, not in a later pass. If a change makes a sentence in
`README.md`, `docs/STATUS.md`, `docs/HANDOFF.md` or any other document false,
that document is part of the change.

This is a rule because it has been broken repeatedly, always in the same
direction: the docs understate what works, because they were written before it
did. Examples, all real:

- `README.md` said *"Pre-release. Nothing here has touched real hardware…
  Today it identifies a GBA cartridge and shows the raw header"* — after
  nineteen images had been written to a real card across three systems. It was
  the front page of a repository that had just been published.
- It also headed a section *"To dump a Game Boy cartridge today, use MROM"*
  long after this core dumped GB, GBC and GBA.
- `docs/FILE-FORMATS.md` read as a contract for a companion app while
  specifying a directory tree and a sidecar file that nothing writes.
- `docs/STATUS.md` recorded two cartridges as shipping bad checksums when the
  dumper had been at fault.
- Counts drift constantly: "sixteen cartridges", "two cartridges", "seventeen
  files" all outlived the sessions that made them true.

### What to check

Sweep for the **class**, not the specific sentence. The failure mode is a
claim written in the future tense about something now finished, or a number
that was right once.

    grep -rn "not started\|not built\|does not\|never\|nothing has\|planned" README.md docs/
    grep -rn "one\|two\|three\|four\|.*teen\|twenty\|thirty" README.md docs/STATUS.md docs/HANDOFF.md

Then ask, for the change in hand:

1. Does `README.md`'s capability table still hold? It is the first thing under
   the title and the first thing anyone reads.
2. Does `docs/STATUS.md` still separate **verified** from **assumed**? Nothing
   moves to verified without evidence someone could go and check, and for
   anything touching a cartridge that evidence comes from a cartridge.
3. Does `docs/HANDOFF.md`'s numbered list still describe work that is open?
   Items get done; renumbering has already produced a duplicate once.
4. If a plan document (`docs/*-PLAN.md`) now disagrees with the code, say
   which is right. Twice the code has been right and the plan behind it, and
   recording that was more useful than quietly conforming.

### When the docs cannot be fixed in the same commit

Say so in the commit message, name the file, and add it to
`docs/HANDOFF.md`. An untracked mismatch becomes a claim someone trusts.

## Other standing rules

- **Never run `make cart` or copy to the SD card without being asked.** A
  Quartus build is several minutes and a card write is outward facing. State
  the intent, then wait.
- **`make test` must pass before a commit.** It is 60-90 seconds.
- **Keep a bad dump.** When a dump fails its checksum, copy the image off
  before re-dumping. There has still never been a corrupt-and-clean pair of
  the same cartridge to diff, because the one bad image was overwritten by its
  own re-dump.
- **Mutation-check a new test.** A test for a rare condition is worth nothing
  until it has been watched to fail. Two monitors in this tree passed with the
  fix removed before anyone checked.
- **Nothing new in `ui_screen`'s per-column path.** `col -> tb_char` has failed
  setup three times and is the critical path again.

---
> Source: [kroy-the-rabbit/openfpga-carttools](https://github.com/kroy-the-rabbit/openfpga-carttools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
