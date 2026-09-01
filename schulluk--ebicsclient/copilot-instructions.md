## ebicsclient

> This repo is a **from-scratch, pure-Python EBICS 3.0 (H005) client library**. Source-available,

# EBICS client library — project instructions

This repo is a **from-scratch, pure-Python EBICS 3.0 (H005) client library**. Source-available,
dual-licensed (free personal / paid commercial). The library is bank- and app-agnostic; ZKB is the
first bank targeted for protocol validation.

## The bar: this software moves money

This library talks to banks. It reads statements **and initiates payments** — it moves real money — and
it handles live credentials and account data throughout. **Correctness and security are the product.** A banking library people trust (and pay for) is the entire value proposition;
the moment it feels sloppy, that value is gone.

So **hold every line to a hardened standard from day one** — including the read-only MVP, because shortcuts
compound and are rarely revisited. Sloppy shortcuts that erode trust are **not acceptable**: magic strings
(`output_format='html'` instead of a `StrEnum`), missing type hints, absent or filler docstrings, unvalidated
input, swallowed errors/exceptions, abbreviated or vague names, secrets in logs. None of these are "fine for
now." Every function is fully typed, documented, validated, and fails closed. The detailed standard lives in
[docs/06-engineering-conventions.md](docs/06-engineering-conventions.md) — follow it without exception.

## Before writing code

Read `docs/` in order — especially [docs/04-implementation-plan.md](docs/04-implementation-plan.md)
(scope + build order) and [docs/01-protocol-and-formats.md](docs/01-protocol-and-formats.md)
(what H005 actually requires). Real ZKB credentials live in `../local/` — the workspace `local/` directory
that sits **outside** this repo (so it can't be committed). Load them from there via the `EBICS_*` env vars,
never hardcode or commit them.

## Hard rules

- **Pure Python only.** Crypto via `cryptography`, XML via `lxml`. No PHP/Java sidecar, no `fintech`
  dependency (it's proprietary and gates statement downloads).
- **Dependency licenses must stay permissive** (BSD/Apache/MIT). Never add a GPL/AGPL dependency —
  it would poison the source-available/commercial license model. See doc 02.
- **Implement from the EBICS spec.** You may read `ebics-client-php` (MIT) as a *behavioral reference*
  to sanity-check crypto/canonicalization, but do **not** copy/translate its code — keep the
  implementation independent so licensing stays unencumbered (doc 02 explains why).
- **Never commit secrets**: private keys, the keyring/passphrase, or anything from the workspace `../local/`
  (kept outside the repo precisely so it cannot be committed — don't copy its contents in).

## The two parts that are easy to get wrong (budget your care here)

1. **The authentication signature** — every `ebicsRequest` carries an XML-DSig (`AuthSignature`) over
   the nodes marked `authenticate="true"`, using **inclusive Canonical XML 1.0**
   (`http://www.w3.org/TR/2001/REC-xml-c14n-20010315`, *not* exclusive c14n — verified against the H005
   XSD and an independent canonicaliser; see docs/08), SHA-256 digest, RSA-SHA256 with the X002 key.
   Byte-exact canonicalization is the classic failure point, and lxml's inclusive c14n needs the
   standalone-node workaround in `crypto.canonicalize` to be spec-correct.
2. **Order-data encryption/decryption** — order data is deflate-compressed then AES-128-CBC encrypted
   with a random transaction key; that transaction key is RSA-encrypted to the E002 key. For download
   you reverse this with your E002 private key. Exact mode/padding matters.

Validate both against **ZKB's test platform** (testplattform.zkb.ch) before going live.

## Verification discipline (protocol correctness is a *spec* question, not just a code one)

A wrong-but-self-consistent implementation passes every round-trip test — that is exactly how
we shipped exclusive c14n instead of the mandated inclusive C14N 1.0 (see docs/08). To avoid
repeating that class of bug:

- **Cite the normative source.** Every protocol constant/structure — namespaces, algorithm URIs,
  element names, order types, encodings — must trace to the H005 XSD or an EBICS spec section,
  *not* to "how XML/crypto usually works" and *not* to another client. An uncited protocol claim
  is a TODO, not a fact; flag it as unverified until checked.
- **Check against an external oracle before calling a wire-format layer "done".** Round-trip
  (sign→verify, encrypt→decrypt) proves consistency, never interoperability. Validate against at
  least one *independent* authority: the XSD (`python tools/fetch-schemas.py`, then
  `tests/test_schema_validation.py`), published spec vectors, or a second implementation
  (`tools/php-parity/`). **Golden vectors must come from that authority — never hand-derived from
  our own output.**
- **Verify foundations earliest.** The more foundational the choice (algorithm, namespace, message
  shape), the higher the confidence bar and the sooner it must be schema-checked — those errors
  propagate furthest. Keep spec-uncertainty distinct from execution-uncertainty; don't hedge the
  hard part while silently trusting the foundation.
- **Version-boundary rule.** Knowledge from EBICS 2.x is *contamination* until re-cited against the
  3.0 spec — the spec's own amendment-history table (pages 2–3) lists everything that changed and is
  the checklist. This is how the initialisation letter shipped with the 2.x public-key hash instead
  of the 3.0 certificate-DER hash and failed a real ZKB activation (see docs/12).
- **"Protocol surface" includes paper.** Anything the bank parses, compares, or verifies falls under
  the citation discipline — the initialisation letters are machine-compared by the bank's back
  office and are wire format, not "rendering".
- **Maintain the verification ledger** ([docs/12-verification-ledger.md](docs/12-verification-ledger.md)):
  every protocol-surface claim gets a row (citation + oracle + status); no oracle ⇒ labelled
  UNVERIFIED there *and* in the README. New protocol code lands with a ledger row, or not at all.
  Paths the test platform cannot exercise (it auto-activates, so letters; EDS) never silently count
  as validated.

## Scope discipline

MVP = read **and** write, validated live against ZKB: key init (INI/HIA/HPB) → download
`EOP/camt.053.001.08` and parse balances → upload `MCT/pain.001.001.09` with the A006 electronic
signature. All of this is built and bank-accepted.

The pain.002/camt.052/camt.054 parsers are built (M5, validated on genuine ZKB messages).
Still deliberately **out of scope** — do not build until there is a concrete need and a way to
validate it: distributed signatures (EDS — the ZKB test subscriber has ``NumSigRequired=0``, so
EDS cannot be validated there) and multi-bank quirk handling. **H004 and earlier EBICS versions
will never be supported** (owner decision; see docs/04) — the `protocol/` seam exists for a
future EBICS version, not for the past. Scope creep is what turns a stable standard into a
maintenance burden.

## Workflow

- **Repo layout:** this git repo is the `repo/` folder *inside* a workspace. Only publishable content lives
  here. The workspace parent holds `../local/` (credentials) and other non-published material, deliberately
  outside the repo. Open `repo/` as the project root; never `git add` anything from `../`.
- Commits on `main` (repo just created, no commits yet).
- After a meaningful change, keep the README "Status" section current.

---
> Source: [schulluk/ebicsclient](https://github.com/schulluk/ebicsclient) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
