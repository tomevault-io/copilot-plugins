## pokeyforwindows

> This is a hobby project. Direct commits to `main` are acceptable for routine

# Repository operations

## Contribution workflow

This is a hobby project. Direct commits to `main` are acceptable for routine
changes after local validation; pull requests are optional.

## Validation

Use the repository wrapper for local validation:

```powershell
.\build.ps1
```

To publish self-contained x64 and ARM64 executables locally:

```powershell
.\build.ps1 -Publish -Version 1.0.0
```

The interop harness must pass before release artifacts are built.

## Versioning and releases

GitVersion reads `GitVersion.yml`. Release tags must match `vX.Y.Z`.

Required GitHub Actions secrets:

- `AZURE_CLIENT_ID`
- `AZURE_CLIENT_SECRET`
- `AZURE_SUBSCRIPTION_ID`
- `AZURE_TENANT_ID`

Optional GitHub Actions variables:

- `AZURE_TRUSTED_SIGNING_ENDPOINT`
- `AZURE_TRUSTED_SIGNING_ACCOUNT`
- `AZURE_CERTIFICATE_PROFILE_NAME`

When variables are absent, the workflow uses the shared Hanselman signing
endpoint and account with the `WindowsEdgeLight` certificate profile.

Run the `Build Release` workflow manually to validate signing without creating
a GitHub Release. Tagged runs require signing and publish immutable x64 and
ARM64 ZIP assets:

```powershell
git tag v1.0.0
git push origin v1.0.0
```

Never replace an existing release tag or published asset; increment the
version instead. Do not add secrets or credential values to tracked files.

## Windows Package Manager

Winget submission is intentionally deferred. Add package documentation and an
automated submission workflow only after the first package manifest has been
accepted and the final package identifier is known.

---
> Source: [shanselman/PokeyForWindows](https://github.com/shanselman/PokeyForWindows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-13 -->
