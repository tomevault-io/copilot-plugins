## technocore-sonnet-challenge

> The rules and canonical validator block live in `sonnet-game.md`.

# Working on technocore-sonnet

The rules and canonical validator block live in `sonnet-game.md`.
Edit that block, then run `python3 scripts/build.py` to regenerate its
standalone file and `manifest.json`. Do not edit generated files directly.

The dictionary and its original license are frozen to the revision and hashes
in `upstream.json`. Do not silently replace either file. Any approved dictionary
change requires updating the provenance, the rule document's hash, and the
manifest together, and creates a new contest package version.

Validate changes with:

```sh
python3 scripts/build.py --check
python3 scripts/verify.py
python3 -m unittest discover -s tests -v
```

The package is a draft. Publishing repository changes, posting messages, creating
live rooms, posting to X, and sending payments require explicit user authorization.
Keep credentials, signing seeds, ballots, and participant data out of this repo.

This repository is intended for public distribution. Keep operator implementation,
monitoring, referee SQL and their tests in the separate archive repository.
The cycle launcher may invoke an explicitly supplied external runner; do not
vendor its implementation or internal configuration here.

Public helpers may validate caller-supplied words and poems. Keep automated word
search, signer-assignment solvers, composition helpers and their tests in the
private archive repository. Do not publish those files in public Git history,
branches or release artifacts, including as an intermediate commit later deleted.

---
> Source: [flop-labs/technocore-sonnet-challenge](https://github.com/flop-labs/technocore-sonnet-challenge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-11 -->
