## fabric

> This repository is the **Microbus framework** itself - the foundation that downstream application projects import and build on. You are working on framework internals, not on an application built with the framework.

This repository is the **Microbus framework** itself - the foundation that downstream application projects import and build on. You are working on framework internals, not on an application built with the framework.

## Two kinds of code in this repo

1. **Framework packages** - `connector/`, `service/`, `application/`, `transport/`, `frame/`, `httpx/`, `pub/`, `sub/`, `cfg/`, `env/`, `lru/`, `dlru/`, `mem/`, `openapi/`, `trc/`, `utils/`, `setup/`, `workflow/`. These are library code consumed by every Microbus application. Changes here affect all downstream projects.

2. **Microservices built with the framework** - `coreservices/` and `exampleservices/` contain microservices that follow the same conventions as any downstream application. When working in these directories, follow the patterns and skills described in `.claude/rules/microbus.md`.

## Working on framework packages

- **Public API surface matters.** Exported types, functions, and interfaces in framework packages are consumed by downstream projects. Avoid breaking changes to exported signatures.
- **`connector/`** is the backbone - it implements the messaging bus, subscriptions, lifecycle, and configuration machinery that every microservice relies on.
- **`service/`** builds on `connector/` to provide the higher-level `Service` base type with convenience methods (`LogInfo`, `DistribCache`, `Now`, etc.).
- **`application/`** handles microservice orchestration, startup/shutdown sequencing, and the test harness (`RunInTest`).
- **Tests** for framework packages are standard Go unit tests (`go test ./connector/...`), not the microservice integration test pattern used in `coreservices/` and `exampleservices/`.

## Working on coreservices/ and exampleservices/

These directories contain microservices built using the framework. Treat them the same way you would treat microservices in a downstream application project:

- Follow all conventions in `.claude/rules/microbus.md`
- Use the skills in `.claude/skills/` for scaffolding and adding features
- Respect `MARKER` and `HINT` comments
- **`myserviceapi/definition.go` is the source of truth; `manifest.yaml` and the boilerplate are derived from it.** `cmd/genservice` regenerates `manifest.yaml`, `myserviceapi/client.go`, `intermediate.go`, `mock.go`, and `mock_test.go` from `definition.go` (the housekeeping skill runs it); never hand-edit those generated files, hand edits are overwritten. The manifest exists as a fast navigational map for coding agents - what a microservice exposes - so an agent can model the system without reading every file. After changing `definition.go`, regenerate rather than editing the derived files.
- **The generated `ToDo` interface in `intermediate.go` must stay an interface.** `NewIntermediate(impl ToDo)` takes an interface, not a concrete `*Service`, so one constructor serves both `*Service` in production and `*Mock` in tests; it is also the compile-time proof that both implement every handler. genservice derives it from the feature set, so there is nothing to hand-edit - but do not redesign the generator to take a concrete type, which would break mocking.

## The token minters are trust roots

The `Mint` endpoints on `access.token.core` and `bearer.token.core` sit on port `:666` and sign whatever claims the
caller supplies (only `iss`, `idp`, `iat`, `exp`, `jti` are stamped by the service). This is deliberate: it is what
makes claims enrichment and actor impersonation work. It also means any caller that can publish to `:666/mint` can
mint any claims, including administrative roles.

Do not try to secure a `Mint` endpoint by adding `requiredClaims` or an in-handler check that the caller already
holds some claim. That is circular: obtaining any verified claim requires a token, and issuing a token is exactly
what `Mint` does. A trust root cannot be gated by the credential system it roots. The only technical control is the
NATS `:666` `PUB` ACL, which `cmd/gencreds` grants a microservice solely because its source code calls a `:666`
endpoint; the operational controls (isolating trust-root microservices, and a CI allow-list that fails the build on
an unlisted `:666` grant) live in the production deployment guide's "Hardening the Trust-Root Tier" section, not in
framework code. The ingress token exchange is not a safer mint either: it verifies and issuer-pins the bearer token
and then passes its claims into `Mint` verbatim, so the access token is exactly as trustworthy as the bearer token,
and what each token carries is application policy rather than a framework-enforced invariant.

When reviewing or changing framework code, treat any new `:666` call site as a trust-root capability expansion, and
never rework a minter to gate itself on a claim.

## Authoring framework upgrade skills

Every fabric release ships exactly one upgrade skill, [upgrade-microbus](.claude/skills/microbus/upgrade-microbus),
and it is self-propagating: its Phase 1 migrates a project by the single increment `SOURCE -> DEST`, and its Phase 2
pins `go.mod` to the next release, installs that release's `.claude` from git, and runs that release's copy of the
same skill - chaining one increment at a time until it reaches the target. There is no separate orchestrator and no
numbered per-version skills.

**Cutting a release edits only two things in the skill:**

1. The **Release Constants** - set `SOURCE` to the immediately-preceding published release and `DEST` to the
   release being cut. The chain fires a hop's Phase 1 only when the project's version equals that hop's `SOURCE`,
   and it steps through every published version in order, so `SOURCE` must be the exact release before `DEST` (the
   latest patch included) or that hop's migration is silently skipped.
2. **Step 3, the migration** - the source edits that take a project from `SOURCE` to `DEST`, chosen by the nature of
   the change:
   - a pure rename is `sed`/`perl` in the skill;
   - a rewrite that needs judgment is a grep-guided manual step: grep the old shape, rewrite each site, and ask the
     user when the correct new shape is a design choice rather than mechanical;
   - a transform that must parse or synthesize Go ships as a small Go program **in the skill's own directory**, run
     with `go run ./<tool>.go`. It is versioned with the skill and deleted by the next release; migration tooling is
     per-skill, never central.

   Confine Step 3 to source edits. If the release has no breaking change, Step 3 is empty - the skill's whole job is
   then the verification the machinery already runs: Phase 1's `go get @DEST` + `genservice` regeneration +
   `go vet`, and the chain-terminating `go test`.

Leave the Phase 1 and Phase 2 machinery alone; it is identical in every release. In particular, **target
propagation** (a user-supplied target is carried across every hop, and only an unspecified target defaults to
"latest") and **self-propagation** (Phase 2 overwrites the skill on disk with the next release's copy and re-reads
it) are machinery, not per-release concerns. Phase 1 runs `genservice` + `go mod tidy` + `go vet` at every increment
- `go vet` is the deterministic compile gate the whole chain relies on - and runs no tests between versions. Phase 2's
terminal branch runs `go test` once, at `TARGET`, as a best-effort behavior check: a project whose tests are flaky or
cannot run locally still upgrades on the `go vet` gate. A Step 3 migration never runs a generator or a verify step
itself.

Two invariants keep the chain intact:

- **Every release ships the skill**, even a no-op one - the chain advances to "the next published version", so a
  release missing the skill would break it.
- **A skill runs only against its own release's world.** Phase 2 pins `go.mod` to `DEST` and installs `DEST`'s rules
  and skills before running it, so every tool and skill a migration names (`genservice`, `add-microservice`,
  `regenerate-boilerplate`, ...) is that release's. A migration therefore needs no future-proofing.

## Working on code

- **Comments explain the API, not the rationale.** A comment on a function, type, or field should describe what it is and how to use it. Design rationale (the *why*) belongs in the package's `CLAUDE.md`, not in godoc or inline comments.
- **Keep comments brief.** Default to a single short sentence. Do not restate what well-named identifiers already convey, and do not reference the current task, caller, or recent fix.
- **Use `//` for short, single-paragraph comments.** Switch to `/* */` block comments only when the comment spans multiple paragraphs or contains an example.
- **No trailing period on short one-liner inline comments.** A code-block comment that is a label or fragment (e.g. `// Locality-aware routing`) ends without a period. Godoc comments on declarations, and any comment that is a complete sentence, keep their terminal punctuation.

## Writing prose

User-facing documentation lives in the separate [`microbus-io/cloudflarepages`](https://github.com/microbus-io/cloudflarepages) repo, served at [docs.microbus.io](https://docs.microbus.io). When writing prose in this repo - READMEs, `CLAUDE.md` files, code comments - the following conventions apply:

- **Link text is a description, not a path.** A markdown link's display text must describe what the reader will find at the destination. `[../../cmd/gencreds](../../cmd/gencreds)` is wrong because the display text is just the file path. Write `[gencreds](../../cmd/gencreds)` instead, using the tool's name or a short noun phrase. Wrapping a canonical name in code formatting (e.g. `` [`gencreds`](../../cmd/gencreds) ``) is fine when the name is the most natural descriptor.
- **No link-stuffing.** Cross-references should be woven into sentences that genuinely advance the reader's understanding, not appear as subjects of weak announcement verbs. "For X, see [Y](z)." is the most obvious form, but "[Y](z) covers X" and "[Y](z) walks through X" and "[Y](z) has the details" are the same anti-pattern in disguise: the link title is the subject and the verb is just a filler signal that another doc exists. Either integrate the link as a noun in a substantive sentence ("...which is what makes [interservice ACL](z) work end-to-end"), or remove the cross-reference and trust the reader to follow the load-bearing link in surrounding prose. If a sentence's only function is to announce another doc, delete the sentence.
- **Prefer "microservice" over "service".** When referring to a Microbus microservice (a unit with its own hostname and code), always use "microservice", not the bare "service". Compounds like `service-to-service`, `cross-service`, `per-service`, `single-service`, `service mesh` are established terms and stay as-is. Standalone references ("this service", "every service", "the service's hostname") should be rewritten to use "microservice".
- **Use "source code" or "code", not bare "source".** The bare word "source" is ambiguous (it collides with the `<src>` segment in NATS subjects, with "source" as a verb meaning to fetch, and with `git source`). When referring to a microservice's program source, write "source code" or "code". The verb form ("to source claims from a backend") stays as-is.
- **Quote `PUB` and `SUB` in backticks.** When referring to the NATS ACL verbs, format `PUB` and `SUB` in backticks. They are wire-protocol tokens, not English words. Inside a larger backticked subject pattern (e.g. `` `PUB <plane>.safe...` ``) the surrounding backticks already cover them, so no extra formatting is needed.
- **Wrap lines at 120 characters, not 80.** Applies to `.go`, `.md`, and other text files in the repo. Long lines are fine when they aid readability (a single godoc sentence, a table row, a URL), but soft-wrap prose and multi-clause expressions at the 120-column boundary rather than the older 80-column convention.

---
> Source: [microbus-io/fabric](https://github.com/microbus-io/fabric) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
