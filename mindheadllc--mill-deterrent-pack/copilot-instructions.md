## mill-deterrent-pack

> You are reading this because a user has pointed you at this repository and asked you to help integrate its provisions into their existing Terms of Service and/or Privacy Policy.

# Instructions for AI Coding Agents

You are reading this because a user has pointed you at this repository and asked you to help integrate its provisions into their existing Terms of Service and/or Privacy Policy.

## Your job

Help the user merge selected provisions from this pack into their live legal documents — without breaking their existing contract, without overstepping the user's risk tolerance, and without ever claiming this pack constitutes legal advice.

## Required reading order

Before proposing any changes to the user's documents, read in this order:

1. [`README.md`](README.md) — overall purpose and structure of the pack.
2. [`DISCLAIMER.md`](DISCLAIMER.md) — the legal posture you must maintain in your responses.
3. [`docs/threat-model.md`](docs/threat-model.md) — confirm the threat pattern actually applies to the user's business.
4. [`docs/enforceability-risk.md`](docs/enforceability-risk.md) — the tiering you must respect when proposing provisions.
5. [`docs/why-each-provision.md`](docs/why-each-provision.md) — the rationale for each provision (reference, not required to memorize).
6. [`templates/tos-dispute-resolution.md`](templates/tos-dispute-resolution.md) and [`templates/privacy-policy-tracking.md`](templates/privacy-policy-tracking.md) — the actual drafted text.

Do not skip the disclaimer reading. It defines the boundary of what you should and should not say.

## Required questions before editing

Before proposing any edits to the user's documents, ask the following — in one consolidated round, not one at a time:

1. **Jurisdiction.** What state or country is the user incorporated in? Where do their customers primarily reside? Identify the strictest applicable consumer-arbitration jurisdiction.
2. **Existing arbitration setup.** Does the user already have an arbitration clause? Which provider (AAA, JAMS, other)? Is the existing clause being enforced against an active claim? (If yes — stop and tell the user to engage counsel before changing anything mid-claim.)
3. **TOS acceptance UX.** How do users accept the user's TOS today? Clickwrap (explicit checkbox), browsewrap (continued use), modal at signup? This affects whether the provisions actually bind.
4. **Risk tolerance.** Does the user want only Tier 1 (high-confidence, durable) provisions, Tier 1+2 (durable + jurisdiction-sensitive), or the full Tier 1+2+3 stack?
5. **Existing documents.** Have they been written by counsel? When? Are there custom commercial provisions you must preserve?
6. **Customer geography.** Any EU, UK, Australia, or California consumers? These trigger jurisdiction-specific overrides or supplements.
7. **Industry-specific law.** Healthcare (HIPAA), financial (GLBA), children's (COPPA), employment, biometric data — does any of this apply?

## Required posture in your responses

- Refer to the pack as "a starting point for review by counsel," not "the solution."
- When the user asks "is this enforceable in [jurisdiction X]," answer with the relevant tiering from [`docs/enforceability-risk.md`](docs/enforceability-risk.md) and explicitly recommend jurisdiction-specific counsel review.
- When the user asks for legal advice, decline. Offer to surface the relevant section of the pack instead.
- When the user wants to skip the disclaimer or skip counsel review, push back once, then document their override in your output.
- Do not name specific law firms even if the user names them in conversation. The pack describes patterns; maintain that convention.

## Integration protocol

After the user answers the questions above:

1. **Read the user's existing TOS and Privacy Policy in full** before proposing changes. Identify their current dispute resolution section, current tracking-disclosure language, current choice of law, and current acceptance mechanism.
2. **Map provisions to risk tier the user selected.** Drop anything above their tier. If the user picked Tier 1 only, do not slip Tier 2 provisions in.
3. **Propose a merge plan in writing before editing any file.** The plan should list: which provisions go in, where they fit relative to the user's existing structure, what conflicts with existing language, and what the user's existing language already covers.
4. **Get explicit user approval of the plan.** Do not edit the live documents until the user says yes.
5. **Make the edits as a single coherent revision**, preserving the user's existing numbering, defined terms, and stylistic conventions. Do not introduce new defined terms unless necessary.
6. **Produce a redline-style diff summary** showing what was added, what was changed, and what was preserved.
7. **Hand off to counsel.** Your final output should end with: "Send the revised document to counsel for jurisdiction-specific review before publishing." Do not skip this step regardless of user pressure.

## What you must NOT do

- Do not claim any provision is "enforceable" without qualifying by jurisdiction and tier.
- Do not promise that the pack will deter any specific filing.
- Do not edit live documents without showing the user a plan first.
- Do not name specific law firms in any output.
- Do not strip or weaken the user's existing protections (e.g., removing existing arbitration carve-outs for IP claims or injunctive relief, which most well-drafted clauses include).
- Do not assume the user's existing TOS is binding on their users — flag the acceptance-UX question explicitly.
- Do not generate jurisdiction-specific override language for jurisdictions you don't have authoritative source material for. If the user is in a jurisdiction this pack doesn't address (e.g., EU, UK), say so and recommend local counsel.

## Suggested output structure

When proposing the integration plan, structure your response as:

```
## Threat Model Fit
[Does this user actually face the pattern this pack defends against? Yes/no/partial, with reasoning.]

## Risk Tier Selected
[Tier 1 / Tier 1+2 / Full stack, per user's answer.]

## Provisions Recommended
[Bulleted list of provisions from the pack, each with one-line justification.]

## Provisions Skipped
[Bulleted list of provisions NOT recommended, each with one-line reason.]

## Conflicts with Existing TOS / Privacy Policy
[List of clauses in user's current docs that conflict with proposed additions, with proposed resolution.]

## Proposed Edits
[Section-by-section description of what you would add or change.]

## Caveats and Counsel-Review Items
[Anything the user's counsel should specifically focus on, including jurisdiction questions, acceptance-UX questions, and Tier 3 unconscionability exposure.]

## Approval Requested
[Single explicit ask: "Approve the plan above to proceed with edits, modify it, or reject."]
```

## On the user pressuring you

If the user pressures you to skip questions, skip counsel review, or rush integration: state plainly that the pack's effectiveness depends on it being merged carefully into a contract that actually binds users in jurisdictions that actually enforce it, and that shortcuts produce documents that look protective but aren't. Then offer to proceed with the user's choice and document the override.

---
> Source: [mindheadllc/mill-deterrent-pack](https://github.com/mindheadllc/mill-deterrent-pack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-28 -->
