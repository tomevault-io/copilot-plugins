## aclif

> Instructions for coding agents changing aclif. If you are an agent that *uses* the `aclif` binary to reach SaaS platforms, read the skill in [skills/aclif/](skills/aclif/) instead; [docs/USING_WITH_AGENTS.md](docs/USING_WITH_AGENTS.md) says how to install it.

# Working in this repository

Instructions for coding agents changing aclif. If you are an agent that *uses* the `aclif` binary to reach SaaS platforms, read the skill in [skills/aclif/](skills/aclif/) instead; [docs/USING_WITH_AGENTS.md](docs/USING_WITH_AGENTS.md) says how to install it.

## What this is

aclif builds command-line tools for AI agents. An agent gets a single tool that provides a unified abstraction across every SaaS provider: one grammar, and canonical names that reach the same record by the same name on any platform. Every command describes itself on request, declares its safety metadata, and runs unchanged from a shell or inside the embedded runtime. It is built on oclif as an ordinary dependency. The package is `@aclif/core`; the binary is `aclif`. Node 22 or later.

## Commands

```bash
npm install            # also generates src/providers/index.generated.ts
npm run build          # tsc, data files, topic table in package.json
npm run typecheck      # source and tests
npm test               # vitest, then lint and the circular-dependency check
npx vitest run test/conformance          # the rules every provider must satisfy
npx vitest run test/providers/native/<name>   # one provider's fixture suite
npm run golden:capture                   # refresh introspection goldens after a contract change
```

Definition of done for any change: `npm run build && npm run typecheck && npm test` passes. CI runs the same on Linux and Windows, Node 22 and 24, and refuses pull requests that touch the private tier.

## Layout

- `src/core/` the contract, runtime, credentials, registry, manifests, aliases. Never imports a provider.
- `src/cli/` the oclif surface: base command, core commands, hooks, config. Never imports a provider except through `src/providers/index.ts`.
- `src/providers/<tier>/<name>/` one directory per provider. Tiers: `native` (project-maintained), `contributed` (maintainer-named), `private` (fork-owned; upstream holds only a README). A provider imports its own directory, `src/core`, `src/util`, and the base command, and nothing else.
- `src/testing/` the conformance suite and test harness, exported as `aclif/testing`.
- `test/conformance/` calls that suite over the built-in registry. `test/providers/` fixture suites over msw. `test/contract/golden/` introspection output, one file per command and flag.
- `docs/` the contract, configuration, embedding, forking, and provider-authoring guides; `docs/providers/<tier>/<name>/SETUP.md` per provider.
- `scripts/` the provider and CLI scaffolds, the golden capture, the boundary check.

## Adding a provider

The mechanical work is yours; the conformance suite is the acceptance test. Follow [docs/PROVIDER_AUTHORING.md](docs/PROVIDER_AUTHORING.md), in this order:

1. Read `docs/CONTRACT.md`, `docs/PROVIDER_AUTHORING.md`, and `src/providers/native/servicenow/` as the provider to imitate.
2. Start from the platform's machine-readable API specification, and cover the whole API. Scope is enforced where a command is bound to an agent, never inside the provider.
3. `npm run scaffold-provider -- --tier <tier> --name <name> --display "<Display Name>"`. It writes only under the tier's four prefixes.
4. Fill in `credentials.ts`, `client.ts` (fetch, no vendor SDK unless the dependency allowlist names it), `errors.ts`, `metadata.ts`, `base.ts`, `commands/**`, fixtures, the fixture suite, and `SETUP.md`.
5. Run `npm run build`, then the conformance suite, then the provider's fixture suite, then `npm run golden:capture`, then `npm test`. Iterate until green. Report each conformance failure you fixed: those are where the spec and the contract disagreed.
6. Mark any `aciMetadata` row you were unsure about with a `TODO` for human review. Mislabelled mutability or blast radius is a governance hole the suite cannot detect from code.

If the spec is ambiguous about pagination, auth precedence, or which field is the record id, stop and ask.

## Rules that get broken

- Every example command string starts with the literal `$BIN` token; the base command renders the binary name at emit time.
- Examples name nothing internal: `example.com`, `alice`, `acme`. No real tenant, hostname, email, or path.
- No `process.env` reads in provider code; credentials arrive through the credential schema.
- No writes to the filesystem in a provider unless a flag names the target.
- `private/` under `src/providers`, `test/providers`, `test/fixtures`, and `docs/providers` is fork-owned. Upstream changes never touch it, and `node scripts/check-sync-boundary.mjs upstream/main` fails a branch that does.
- Commits follow Conventional Commits (`feat`, `fix`, `docs`, `test`, `refactor`, `chore`, `ci`); `feat` and `fix` release a version on push to `main`. A commit hook checks the message.
- A change to the contract (envelope, errors, exit codes, metadata, introspection output, schemas) bumps `CONTRACT_VERSION`, regenerates the goldens, and gets a CHANGELOG line.
- Prose in docs and README: plain declarative sentences, no em-dashes, no similes, no "X, not Y" constructions, no closing aphorisms.

## Where to go deeper

- [docs/CONTRACT.md](docs/CONTRACT.md): envelope, errors, exit codes, metadata, introspection, catalogue, manifests, alias sets.
- [docs/PROVIDER_AUTHORING.md](docs/PROVIDER_AUTHORING.md): every rule id the conformance suite enforces, with the file checklist and a sample generation prompt.
- [docs/EMBEDDING.md](docs/EMBEDDING.md): the runtime a host process drives.
- [docs/FORKING.md](docs/FORKING.md): building a CLI package on the framework, forking with a private tier, contributing back.
- [CONTRIBUTING.md](CONTRIBUTING.md): pull request requirements and what automation decides.

---
> Source: [agent-cli-framework/aclif](https://github.com/agent-cli-framework/aclif) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
