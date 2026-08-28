## payment-name-startos

> This is a StartOS service-package repository — it builds a `.s9pk` for StartOS.

# AGENTS.md

This is a StartOS service-package repository — it builds a `.s9pk` for StartOS.

Develop it inside a StartOS packaging workspace created by `start-cli s9pk init-workspace`,
which provides the packaging guide and agent context one level up. If you're reading this in a
bare clone with no workspace, the full guide is at <https://docs.start9.com/packaging>.

**Start every task at the recipe index** — `../start-technologies/projects/start-sdk/docs/src/recipes.md`
(or <https://docs.start9.com/packaging/recipes.html>). It maps an intent ("prompt the user to create
admin credentials", "expose a web UI") to the constructs, the reference pages, and a named production
package to copy. Find the recipe before you read this package's neighbours: a package you reach by
grepping may be non-conformant, and the recipe outranks it.

Freshly scaffolded? Work the
[New Package Checklist](../start-technologies/projects/start-sdk/docs/src/new-package-checklist.md)
(or <https://docs.start9.com/packaging/new-package-checklist.html>) from top to bottom. It is a
guide page, not a file in this repo — read it, don't copy it in.

Keep `README.md` (technical reference for an AI support or administering agent) and
`instructions.md` (end-user docs) in sync with your changes.

**Bugs and feature requests are GitHub issues on this repo** — file them as you find them.
Don't record work in the repo instead: no `TODO.md`, no `NOTES.md`, no `PLAN.md`. What you
verified, tried, and decided belongs in the commit message and the PR body.

## This repo

- **`packageRepo` is this fork** (`Start9-Community/payment-name-startos`); `upstreamRepo` is the
  developer's own repo, which is where the software itself lives. Packaging changes land here.
- **The default branch is `main`.** `build.yml`'s PR target and `tagAndRelease.yml`'s push trigger
  both name it; a template that still says `master` gives this repo CI that silently never runs.
- **Resolve the published record over DNS-over-HTTPS, never the container resolver.** StartOS
  forwards DNS without `RRSIG` or the `AD` flag, so over port 53 a DNSSEC-signed answer is
  indistinguishable from an unsigned one — and BIP-353 turns entirely on that distinction. Node's
  `dns.resolveTxt` looks like a simplification and silently destroys the check.
- **Report a problem only when every resolver that answered agrees.** Turning DNSSEC on flips a zone
  from unsigned to signed and resolvers holding the old state hard-fail until their caches expire;
  a single-resolver check calls a healthy name compromised, and a check that cries wolf stops being
  believed.
- **The container executes nothing.** The daemon is a `sleep infinity` whose only job is to give the
  health check something to hang off; adding a process to the image will not run it.
- **Write the hosted settings only after the claim succeeds.** Saving first leaves the health check
  watching a name that was never claimed, and the pre-filled form offering it back as the user's own.

---
> Source: [bitsagarob/payment-name-startos](https://github.com/bitsagarob/payment-name-startos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
