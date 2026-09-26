## whallm

> Use the ubiquitous language in `CONTEXT.md`.

# Project instructions

Use the ubiquitous language in `CONTEXT.md`.

## App packaging behaviors

Treat `build local` and `release new version` as workflow requests.

### `build local`

When the user requests `build local`:

1. Run the applicable tests.
2. Run `make package` to create `dist/Whallm.app` and
   `dist/Whallm-macOS-arm64.zip`.
3. Verify the App with `codesign --verify --deep --strict`.
4. Extract the ZIP to a temporary directory. Verify the extracted App again.
5. Start the packaged App in an isolated check that denies access to the project
   `.build` directory. Confirm that the App stays open and does not trap during
   `L10n` initialization.
6. Confirm that the packaged App loads the English, Simplified Chinese, and
   Traditional Chinese localization files from `Contents/Resources` before it
   accesses `Bundle.module`.

The packaged App must not require an absolute path from the build machine.
Keep local packaging on the local machine. Do not create a tag, notarize an
artifact, or upload an artifact.

`build local` is complete only when the App and the extracted ZIP pass all
checks.

### `release new version`

When the user requests `release new version`:

1. Complete `build local` first.
2. If the user does not give a version, read the latest stable `vX.Y.Z` Git tag
   and increment `Z` by one.
3. Confirm that the release changes, tests, public documentation, and available
   local documentation in `.vscode/docs/` are current.
4. Require `CODE_SIGN_IDENTITY`, `NOTARY_PROFILE`, authenticated `gh`, and the
   Sparkle signing tools.
   On this Mac, use `NOTARY_PROFILE=deepseek_ssd` (verified 2026-09-07).
   Check it with `xcrun notarytool history --keychain-profile deepseek_ssd`
   before asking the user for a profile name; credentials remain in Keychain.
5. Before `gh release create`, require the release script to verify the final
   signed and notarized App, its ZIP, its localization files, and App startup
   without access to the project `.build` directory. Stop the release if a check
   fails.
6. Run `make release VERSION=X.Y.Z`.
7. After publication, download the GitHub Release ZIP to a temporary directory.
   Verify its signature, notarization ticket, localization files, and isolated
   startup again.
8. Confirm that the GitHub Release contains the ZIP and `appcast.xml`. Report the
   tag, release URL, and SHA-256 of both files.

`release new version` is complete only when the downloaded release artifacts
pass all checks.

## Documentation

Public documentation must be readable from a fresh checkout. Keep `README*.md`,
`BENCHMARK.md`, and `CONTEXT.md` free of links to local documentation or archives.
`.vscode/` contains local files excluded from version control; keep them local.

### `.vscode/docs/`

When maintaining runtime behavior or recording validation, start with
`.vscode/docs/README.md` if it is available. It is the local documentation index
for the current project. If local documents are absent, use the code, tests,
and public documentation to establish current behavior.

Keep `.vscode/docs/` consistent with the current code, tests, installed model
contract, and reproducible measurements.
Update the applicable document when an API, default value, data path, limit, or
measured result changes.

Existing `.vscode/docs/` archives and manifests preserve historical benchmark
artifacts. Keep raw runs in `scratch/` and validated machine-readable artifacts
in `.vscode/docs/benchmarks/`; preserve historical archives as snapshots.
Record the commit, environment, workload, configuration, cache state, and output
token hash for each formal performance result.

Separate current validation, historical measurements, external facts, and
research hypotheses.
Do not write an estimate or an external benchmark as a project result.

### `.vscode/research/`

When continuing research, read `.vscode/research/README.md` and `.vscode/TODO.md`
if available. They contain research status and optional next steps.
`.vscode/research/` contains active investigation, experiment plans, source audits,
and unconfirmed technical directions.
Research files are not the source of truth for current runtime behavior.

Use primary sources for external technical claims.
State assumptions, test conditions, stop criteria, and evidence limits.
When the project adopts a research conclusion, update the applicable file in
`.vscode/docs/`.

The archives in `.vscode/research/` contain superseded plans and historical research.
Keep archived files for traceability.
Add a warning when a file no longer describes the current runtime.
Do not use an archived value as a current default or current performance result.

---
> Source: [yanun0323/Whallm](https://github.com/yanun0323/Whallm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
