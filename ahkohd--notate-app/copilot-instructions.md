## notate-app

> Constraints that are cheap to honour and expensive to undo. An explicit

# Working in this repository

Constraints that are cheap to honour and expensive to undo. An explicit
instruction from the maintainer overrides anything here.

## Licence

The project is **AGPL-3.0**. Every new file inherits it — do not add MIT or
Apache headers to individual files, and do not copy in code under a licence that
cannot be combined with AGPL.

The name is **not** covered by the licence. "Notate", the logo, and
`usenotate.app` are reserved separately; see [TRADEMARK.md](TRADEMARK.md). Never
suggest that a fork may ship under the Notate name.

Opening a pull request means agreeing to [CLA.md](CLA.md). If you are an agent
preparing a contribution on someone's behalf, say so in the PR description.

## Paid collaboration features are coming — keep the seam

Hosted collaboration will be sold as a subscription. Two rules follow, and both
are much cheaper now than later:

**Collaboration code goes in its own SPM target**, not in `Sources/Notate`. That
target does not exist yet; create it when the first file needs it, rather than
leaving an empty one. Today `Sources/Notate` is a single executable target of two
dozen mutually-dependent files, so anything added there is effectively welded in —
extracting it afterwards means untangling all of it.

**The sync server is a separate codebase and is never published.** It needs no
licence decision, because code that is never distributed cannot be taken. Do not
add a server to this repository.

Whether the collaboration client is eventually carved out of AGPL is deliberately
undecided. The module boundary keeps that option open; making the licence decision
early does not help and costs goodwill.

## Build

```sh
./Scripts/run.sh      # builds with SPM, assembles build/Notate.app, signs, launches
```

Signing matters: the Screen Recording permission grant is keyed to the code
signature, so an unsigned build asks for permission again. First launch prompts for
Screen Recording.

There is no UI test suite. UI changes need a manual smoke test — run the app and
verify the change end to end.

## The website

`Web/site` is a separate npm project (Astro and Tailwind) with its own
[README](Web/site/README.md) covering the house style, and
[Web/docs/VIDEO.md](Web/docs/VIDEO.md) covering the demo-clip pipeline. Read the
relevant one before changing either — both record decisions that look arbitrary
until you know the reason, and several are traps that cost a re-encode or a burnt
version prefix to rediscover.

## Style

Match the surrounding code. No formatter is enforced.

Comments carry technical facts, not product rationale: why the code is shaped this
way, what breaks otherwise, what a value was measured against. Not what the feature
is for.

---
> Source: [ahkohd/Notate.app](https://github.com/ahkohd/Notate.app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
