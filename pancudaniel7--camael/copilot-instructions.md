## camael

> - **Never run `git commit`** — the user handles all commits.

# Git Workflow

- **Never run `git commit`** — the user handles all commits.
- **Never add `Co-Authored-By`** trailers to commit messages.
- You may stage files with `git add` only when explicitly asked.
- When changes are ready to commit, tell the user the suggested commit message and let them run it.

# License Compliance

This repository is distributed under two licenses: **MIT** (`LICENSE-MIT`) and **AGPL-3.0** (`LICENSE-AGPL`). Every code modification must comply with both licenses.

## Mandatory rules for every modification

1. **Do not introduce license-incompatible dependencies.** Before adding any external package, verify that its license is compatible with both MIT and AGPL-3.0. Packages under proprietary licenses or more restrictive copyleft terms (e.g. GPL-2.0-only) are not allowed.

2. **Preserve copyright headers.** Do not remove or alter the `Copyright (C) 2020-2026 Denver Technologies, Inc.` line in existing files. When creating a new file that contains substantial logic, add the same header at the top.

3. **Source code must remain available (AGPL-3.0).** Any modification to code that runs as a network service (server, API) must be reflected in source code accessible to users of that service. Do not encapsulate server logic in opaque binaries or source-less dependencies.

4. **Derivative works must remain under the same licenses.** Do not re-license individual files or code sections under terms incompatible with MIT or AGPL-3.0.

5. **Do not remove license notices from distribution artifacts.** The `LICENSE-MIT` and `LICENSE-AGPL` files must be present in any distribution or fork of the project.

6. **Document the origin of copied third-party code.** If you include code snippets from external sources, note the origin and its license in a comment directly above the snippet.

---
> Source: [pancudaniel7/camael](https://github.com/pancudaniel7/camael) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
