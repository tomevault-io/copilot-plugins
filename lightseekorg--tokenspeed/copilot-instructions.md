## tokenspeed

> > If a `AGENTS.local.md` file exists alongside this file, read and respect it--

# General Agent Guidelines

> If a `AGENTS.local.md` file exists alongside this file, read and respect it--
> it contains developer-specific overrides that supplement this shared guidance.

## Development environment

* Before any work, check local Python venv and activate if one exists.
* Don't install pip packages outside the local Python venv if one exists.

## Code changes

* Add tests and update docs for the changed code.
* Before creating commits, run `pre-commit run --all-files` to format.
* When creating commits, perform sign off on behalf of the author.

## tokenspeed-kernel

Inside the root tokenspeed-kernel/ directory:

* All direct tokenspeed-triton imports should happen in `_triton.py` and then
  re-import to other places.
* All direct third-party code should be placed in `thirdparty/` and imported
  into `ops/` then registered via `register_kernel`.
* Files under `ops/` should follow `<family>/<solution>` structure, like
  `gemm/trtllm.py` or `attention/triton/`.

---
> Source: [lightseekorg/tokenspeed](https://github.com/lightseekorg/tokenspeed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
