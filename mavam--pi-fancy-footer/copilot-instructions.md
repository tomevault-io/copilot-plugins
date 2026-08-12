## pi-fancy-footer

> `pi-fancy-footer` is a pi extension that replaces the default footer with a

# pi-fancy-footer

`pi-fancy-footer` is a pi extension that replaces the default footer with a
compact two-line status footer.

## Setup

Install Lefthook once per clone:

```bash
uvx lefthook install
```

Pushing runs the quality gates automatically. You don't need to run checks
manually.

## Development

When you change user-facing behavior, configuration, defaults, or dependencies,
update `README.md` in the same change.

## Release engineering

- Use `tenzir-ship` for changelog management and releasing.
- Add changelog entries for user-facing changes.
- Before releasing, ensure `main` is in sync with `origin/main`.
- To release, dispatch `.github/workflows/release.yaml` with a title and
  introduction.

---
> Source: [mavam/pi-fancy-footer](https://github.com/mavam/pi-fancy-footer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
