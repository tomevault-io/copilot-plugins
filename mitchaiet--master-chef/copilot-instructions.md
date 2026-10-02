## master-chef

> For user setup requests, read [docs/AGENT_SETUP.md](docs/AGENT_SETUP.md) and

# Working on Halo Vision

For user setup requests, read [docs/AGENT_SETUP.md](docs/AGENT_SETUP.md) and
[docs/SETUP.md](docs/SETUP.md), then use `setup.sh` rather than reconstructing
the installation commands. Run `--stage check --json` first.

- The public repository is `https://github.com/mitchaiet/master-chef`.
  Verify the remote before any explicitly authorized publication.
- The user supplies their own retail Halo PC ISO/installation, valid product
  key, and Apple signing. Keys are entered in the original installer, never
  chat or command-line flags. Registry exports are private local inputs.
- Keep original media, installations, saves, signing choices and unrelated
  files intact. Setup artifacts belong in ignored local directories.
- Do not remove executable verification, fabricate installation values, or
  publish generated engine source, private registrations, saved profiles or
  signing material. Complete release assets require explicit authorization,
  an exact file manifest and privacy review; never upload a raw installation
  directory or the owner's signed app. See docs/RELEASING.md.
- For setup changes, run `python3 tools/test_setup_halo.py`. Source checks are
  in `tools/run_source_checks.py`; use checks relevant to the change.
- Distinguish preparation, generation, compilation, signing, installation,
  launch and actual gameplay validation in your report.

---
> Source: [mitchaiet/master-chef](https://github.com/mitchaiet/master-chef) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
