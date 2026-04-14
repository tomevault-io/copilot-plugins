## ai-coding-sop

> Firstly, read README.md to understand how to build and what the purpose of

# Instructions

Firstly, read README.md to understand how to build and what the purpose of
things is.

Never auto-modify the following files:
* //tools/bazel
* Any dotfile.
* Any `*.lock` files.
* Any `*.nix` files.

Unless otherwise instructed, only apply maintenance tasks to files in the git
index, or uncommitted files, to avoid redoing work on files that are already
committed to git.

# Documentation

Use Doxygen rules for documenting.

Whenever you add Doxygen documentation, also add the source filegroup targets ,
or source files where filegroups are unavailable, to the `srcs` attribute of
the `doxygen` target named "//:docs", so that doxygen docs could be updated
too. Include all VHDL files, but also C headers, and any other program source
files which contain documentation.

Do not run buildifier, as it will mess up the VHDL file ordering.

When updating documentation run `bazel build //:docs` to verify that it is
correct.

# License maintenance

When maintaining the license files do not modify the following:

* Files matching `*.gtkw`.
* Files under the directory `//third_party`.
* Any files with filenames beginning with a dot.

For all source files and all BUILD files, verify that they have a license
reference at the beginning of the file.

If a file does not have a license reference, add a comment with the SPDX
label of the applicable license.

# `//third_party` maintenance

Every subdir under `//third_party` must have a LICENSE file with the appropriate
license copied from its source distribution.

# Public API documentation maintenance

Ensure that the repository is clean before starting this procedure.

For all source files, we want to maintain an up-to-date documentation of their
respective public API.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/filmil)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/filmil)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
