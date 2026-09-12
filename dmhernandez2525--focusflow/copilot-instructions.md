## focusflow

> - NEVER run `npm install`, `yarn add`, `pnpm add`, `bun add`, or any package

## DEPENDENCY SECURITY RULES (MANDATORY)

### Package Installation Policy
- NEVER run `npm install`, `yarn add`, `pnpm add`, `bun add`, or any package
  installation command without explicit approval from the developer.
- NEVER run `npm update`, `yarn upgrade`, `pnpm update`, `bun update`, or any
  package update command. All updates are manual and reviewed.
- NEVER modify package.json dependency versions (dependencies, devDependencies,
  peerDependencies, optionalDependencies) without explicit approval.
- NEVER add new dependencies to the project without explicit approval.
- NEVER run remote-execution commands that download and run a package on the
  fly: `npx`, `pnpm dlx`, `yarn dlx`, `bunx`. Use only packages already
  installed locally in the project's lockfile.

### When a New Package Is Needed
If a task requires a package that is not currently installed:
1. STOP and inform the developer which package and version is needed.
2. Explain WHY the package is needed and whether alternatives exist in the
   current dependency tree.
3. WAIT for the developer to manually install and verify the package.
4. Do NOT suggest running install commands. The developer will handle installation
   through the safe install process.

### When Fixing Dependency Issues
- If `npm audit` or similar shows vulnerabilities, present the findings but
  do NOT auto-fix. The developer will evaluate each update manually.
- If a lockfile conflict occurs, present the conflict but do NOT resolve it
  by running install commands.
- If node_modules is missing, inform the developer. Do NOT run install.

### Lockfile Integrity
- NEVER delete or regenerate package-lock.json, yarn.lock, or pnpm-lock.yaml
  without explicit approval.
- `npm ci --ignore-scripts` may be run ONLY when ALL of the following are true:
  (1) the developer has explicitly approved this command for this session,
  (2) the lockfile on disk is unmodified from the committed version,
  (3) the goal is to restore node_modules so existing tests can run.
  Any other install / restore command requires fresh approval.
- If the lockfile appears corrupted, flag it for manual review.

### Script Execution
- NEVER run lifecycle scripts (preinstall, postinstall, prepare) as part of
  any workflow.
- When running npm commands, always append `--ignore-scripts` unless the
  developer has explicitly approved script execution for a specific package.
- NEVER run remote-execution wrappers (`npx`, `pnpm dlx`, `yarn dlx`, `bunx`)
  for packages that are not already in the project's lockfile.

### Version Pinning
- All new dependencies MUST use exact versions (no ^ or ~ prefixes).
- If you see caret (^) or tilde (~) ranges in package.json, flag them for
  the developer to review but do NOT change them without approval.

### Suspicious Package Detection
- If you encounter any of the following in a project's dependencies, immediately
  alert the developer:
  - plain-crypto-js (any version)
  - @tanstack/setup (not a real package, IOC for Shai-Hulud)
  - Any package with a preinstall/postinstall script that downloads binaries
  - Any package that references sfrclak.com, checkmarx.cx, git-tanstack.com,
    azurestaticprovider.net, getsession.org, litter.catbox.moe, or api.svix.com
  - node-ipc versions 9.1.6, 9.2.3, or 12.0.1
  - axios versions 1.14.1 or 0.30.4
  - @bitwarden/cli@2026.4.0
  - @cap-js/sqlite@2.2.2, @cap-js/postgres@2.2.2, @cap-js/db-service@2.10.1,
    mbt@1.2.48
  - intercom-client@7.0.4 or 7.0.5
  - Unscoped `tanstack@2.0.4` through `tanstack@2.0.7` (brand-squat, distinct
    from the legitimate scoped `@tanstack/*` packages)

### Reference
Security research documents live under the developer's local
`command-center/Security research/` directory. See `npm-mitigation-plan.md`
in that directory for the full plan and incident-response runbook.

---
> Source: [dmhernandez2525/FocusFlow](https://github.com/dmhernandez2525/FocusFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-12 -->
