## futility

> In each file that is missing the annotation, add the SPDX license annotation.


## Licensing

In each file that is missing the annotation, add the SPDX license annotation.
Use appropriate source code comments to add:

```
SPDX-License-Identifier: Apache-2.0
```

Do not modify `*.md` files, `*.txt`, `*.json` files with license annotations.
Do not modify the `LICENSE` file with license annotations.
Do not modify `*.lock` files.


## Handling repository

### Safety

* Do not delete existing code to make tests or compilation pass.
* Always run tests after making changes, ensure tests pass.

### Running tests

```
bazel test //...
```

The project must test without errors.

### Building the project

```
bazel build //...
```

### Updating the module registry

```
bazel mod tidy
```

### Run the go compiler directly

Use this command to run the hermetic go compiler, instead of the usual `go`
command.

Replace `<args>` with the list of arguments to give the go compiler.

```
bazel run @rules_go//go -- <args>
```

### Update the bazel BUILD files


```
bazel run //:gazelle
```

### Handle a newly added dependency

This is a little bit involved because we draw dependencies from the `go.mod` file,
meaning we need to run go to update that first. We then run bazel tidy process
to tidy up MODULE.bazel. We can then run gazelle so that gazelle will update the
BUILD files.

```
bazel run @rules_go//go -- mod tidy -v
bazel mod tidy
bazel run //:gazelle
```


## Machine code editing

### Maintain disclosures

In each directory where automated editing has been used, add a `README.md` file
if one does not exist, or append to README.md if it exists, but only if the
annotation does not already exist:

```
This module was partially written using an automated coding assistant, with
human supervision.
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/filmil)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/filmil)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
