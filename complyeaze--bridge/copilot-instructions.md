## bridge

> This document defines agent-level expectations and review responsibilities for this repository.

# AGENTS.md

This document defines agent-level expectations and review responsibilities for this repository.

## Agents and responsibilities

- **Core implementation agent**: owns Rust/Tauri and React implementation and module-level code health.
- **Security agent**: owns credential handling, endpoint validation, and data-leak prevention checks.
- **Release agent**: owns CI, packaging, changelog/release prep, branch policy,
  dependency-license inventory, and proof that license/NOTICE resources ship
  in supported installers.
- **Docs and governance agent**: owns onboarding docs, PR templates, issue
  lifecycle, contribution licensing, provenance checks, and NOTICE updates.

## Review flow

- All code changes go through a pull request.
- Every PR must include:
  - Functional summary
  - Test or reproduction command
  - Migration impact notes if changing sync behavior
  - Security impact notes for Tally/credential changes
- Each PR must link to one line in [review-checklist.md](./review-checklist.md) as completed before merge.

## Rectification expectations

- **When non-security defects are found**: open a follow-up `Bug` issue and
  include a `Rectify` PR with root-cause and regression check.
- **When a vulnerability, credential leak, or sensitive-data exposure is
  suspected**: follow [SECURITY.md](./SECURITY.md) privately. This requirement
  supersedes public issue/PR creation until coordinated disclosure is safe.
- PRs that touch existing workflows must include rollback notes and migration compatibility.
- Keep issue triage actionable:
  - assign exactly one area label (`area:tally`,
    `area:documents`, `area:infra`, or `area:security`)
  - set one bug severity label (`severity:p1` urgent / `severity:p2`
    production impact / `severity:p3` medium / `severity:p4` cleanup)
  - avoid open "wip" tasks without acceptance evidence.
- If regression was introduced by a specific PR, link it explicitly in the rectify issue and include it in the fix PR summary.
- For non-security production regressions, use a dedicated fix branch and label (`type:rectify`).

## Private knowledge hub

A **private** cross-repo knowledge repository holds material that must **not** live in this
public repo: vulnerabilities, crash triggers, competitor teardowns, pricing, market research,
and **sensitive** protocol findings. Its name, URL and local path are supplied out-of-band to
people and agents who have access, and are deliberately **not** recorded here — see the last
bullet.

**The test for "sensitive":** would publishing it let someone **harm a user or a Tally
instance**, or hand a competitor something they **could not measure themselves in an
afternoon**? If yes, it goes in the hub and only a de-fanged rule stays here. Ordinary
request/response behaviour, field semantics and protocol invariants belong in the appropriate
topic part declared by `docs/tally/TALLY_PROTOCOL_REFERENCE.md`, which remains the canonical index.

This distinction matters because source comments cite that reference by section. **A citation
into a private document is worse than no citation** — it looks auditable and is not.

- **Consult before you build.** Before implementing any flow touching Tally, GST, portal auth,
  MCA, or a competitor feature, search the hub for the topic first. Its contribution contract
  lives in the hub itself.
- **Write sensitive findings there, not here.** A vulnerability, crash trigger, sensitive
  protocol behaviour, or market/pricing fact goes in the hub; leave only a de-fanged rule here.
- **Never reference the private repo by name, URL, or filesystem path in a committed public
  file**, and never paste an entry's sensitive body into this tree. The boundary is the point,
  and it erodes one convenient pointer at a time: the repo name, the sibling path and the
  directory taxonomy are each small disclosures that compose into a map. This paragraph
  previously carried all three and contradicted the rule directly beneath it.

If the hub is not present (fresh clone, CI, or no access), skip the consult step — it is an
enhancement, never a build blocker.

## Engineering principles

These are derived from defects actually found in this repository, not from general advice.
Each principle names the failure it prevents. Where a principle and a deadline conflict, the
principle wins — every item below is here because ignoring it already cost this project weeks.

### P1. Nothing is built past the point where it has been proven against reality

The read pipeline reached ~15,500 lines, with 445 passing tests and 15 ADRs, on a code path
that had **never returned a single row from a real Tally**. The tests passed because they ran
against a simulator this repository wrote for itself — it verified that Bridge agreed with
Bridge.

- A component may not grow beyond a working end-to-end slice without live evidence.
- Fixtures must be **captured from a real instance**, never authored by hand. A hand-written
  fixture encodes an assumption and then defends it.
- A simulator is a regression tool for behaviour already observed live. It may never be the
  first or only evidence that something works.

### P2. Make illegal states unrepresentable, rather than writing rules people must remember

`docs/tally/IMPLEMENTATION_GUIDE.md` lists sixteen traps. Every one currently relies on a
developer recalling a rule. Prefer types that make the trap impossible:

| Instead of a rule | Build a type |
| --- | --- |
| "Always pin the company" | A `PinnedCompany` that can only be constructed from a response whose GUID matched |
| "Boundary dates must be day 1, 2 or 31" | A `DateWindow` whose constructor rejects other days |
| "Success needs all four counter conditions" | An `ImportOutcome` with no `Success` variant constructible unless all four hold |
| "Never tombstone from an unverified scan" | Distinct `CompleteScan` / `PartialScan` types, where the tombstone function accepts only the former |

That last row turns the most dangerous defect in the system — mass false deletion — into a
compile error. Prefer that over any amount of documentation.

Use the newtype and typestate patterns for this. Avoid primitive obsession: a `String` that is
sometimes a verified company name and sometimes user input is a latent bug.

### P3. Parse at the boundary; never validate downstream

Convert untrusted input into a type that cannot be wrong **once, at the edge**, and let the
rest of the system rely on the type. Do not re-check the same property at three layers — that
is how `is_clean_success()` ended up with a three-of-four success path while its callers each
compensated differently.

Corollary: parsing must **fail closed with a typed error**. An empty `TYPE="Amount"` is not
zero; an absent narration is not `""`.

### P4. Argue every added line; prefer deletion and reuse

Before adding code, answer in the PR body:

1. What existing component could do this, possibly with a small change?
2. What is deleted as a result of this addition?
3. What breaks if this is simply not built?

Evidence that this matters here: ~7,000 lines of compatibility and qualification machinery has
produced zero rows of evidence; ~1,100 lines of canary ceremony guarded a write that never
dispatched; nine copies of one formula that terminates the Tally process. **A code-negative PR
is a good PR.** Net LOC is a reported metric, not an accident.

### P5. Verification reads bytes, not greps

Every wrong conclusion in the 2026-07 live-probe programme came from a measurement error, not
a reasoning error:

- `curl … | head` truncated a capture via SIGPIPE; the short file looked complete
- `grep -c '<VOUCHER'` also matched `<VOUCHERNUMBER>` and `<VOUCHERTYPENAME>`
- a narrow `grep -A1` window missed a difference that reversed a finding
- a harness recorded failed connections as "0 rows" instead of erroring
- a framed-protocol test double truncated a request and measured itself instead of the code
- a diagnostic log outgrew every retrieval tool, leaving four hypotheses untested for days

Therefore: write responses to a file and inspect the file; count structure, do not substring
match; **any tool that can report "nothing found" must distinguish that from "the request
failed"**; and repeat anything anomalous before believing it. A test double speaking a framed
protocol must read to the terminator, parse the declared length, and read exactly that many
bytes; never assume one read suffices, because a silently truncating double measures itself.
The same distinction applies to error paths: assert the typed error variant so a transport
failure cannot pass as proof of a protocol failure; never match only an error-message substring.
A harness, probe, or gate must bound diagnostic output at its source: aggregate or cap per-item
emissions so the resulting evidence remains retrievable.

### P6. Change one variable at a time, and record confidence

A multi-attribute change established that a *combination* worked while leaving the actual cause
unknown. Findings must be recorded with an explicit confidence marker — verified / partial /
unverified — and an unverified assumption may not be built upon. See
`docs/tally/TALLY_PROTOCOL_REFERENCE.md` for the convention.

### P7. Prefer failures that are loud and in-band

Given a choice between two request shapes, choose the one that fails with an error in the
response over the one that fails inside Tally's UI — the latter blocks the gateway until a
human intervenes. Applied more generally: design so that the failure mode is detectable by the
program, not only by a person watching a screen.

Manual voucher numbering is preferred over automatic for the same reason: identical failure,
but one silently duplicates a client's voucher and the other cleanly rejects.

### P8. Architecture: layered, with dependencies pointing inward

Keep the existing crate layering intent, and enforce it:

- protocol and canonical layers must not depend on transport, storage, or Tauri
- the Tauri command layer contains no business logic and holds no transport handles
- one dispatch path to Tally; nothing bypasses it, including tests and harnesses
- a new dependency requires justification in the PR body — Tauri's value is a small binary and
  a small attack surface, and both erode by default

### P9. Documentation lives with the decision, not in a wiki

A behavioural discovery goes into the appropriate declared topic part linked from
`docs/tally/TALLY_PROTOCOL_REFERENCE.md`. Keep that canonical file as the navigation and
legacy-link index; retain its section redirects and update them when adding or moving a
heading. Use unique ATX section headings in the declared parts. The route gate supports ASCII
text, `§`/`—` punctuation, `**bold**` and single-backtick inline code; unsupported heading or redirect
markup is refused rather than approximated. Raw HTML blocks and inline HTML IDs are unsupported
in the split index and declared parts, except for the canonical index’s complete standalone
legacy-anchor lines. Each redirect uses one anchor line, one blank line and one standalone link
line; orphan or duplicate anchor IDs are refused. Setext section headings are not supported in
the split reference. Literal HTML examples must use fenced blocks or same-line paired inline
code; indented and multiline inline-code HTML examples are outside this gate’s admitted format.
The index and every declared part are compatibility-pinned sources, so follow
`docs/release-process.md` after edits (a pull request that changes a pinned file adds an
acknowledgement file; there is no reseal step). A build rule goes into `IMPLEMENTATION_GUIDE.md`;
a plan change goes into the plan with a dated deviation note. A
finding that exists only in a conversation is lost. If code encodes a non-obvious external
behaviour, the comment cites the reference section rather than restating it.

## Safety expectations

- Never commit hardcoded secrets, tokens, API keys, or raw certificate output.
- Never commit personal or customer data, local usernames, home directories, or
  developer-specific absolute paths; use synthetic examples and repository-relative paths.
- Any credential path changes require a security-focused reviewer comment.
- Any platform-sensitive change must be validated on affected Windows and macOS hosts,
  or the missing platform evidence must be called out explicitly in the PR.
- Never merge a PR that introduces destructive DB migrations without rollback notes.
- Never relicense or add third-party code or assets without documented authority
  and preservation of applicable copyright, license, and attribution notices.
- If you discover policy drift from this file, open an explicit PR to rectify before feature work.

---
> Source: [ComplyEaze/bridge](https://github.com/ComplyEaze/bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
