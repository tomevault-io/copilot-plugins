## base-core

> Core operating principles for any Cursor session in EAA repositories


# Core operating principles

You are working on the Ethical Alpha Audit (EAA) portfolio, authored by Walter C. Brown. The portfolio is a multi-repository research programme on clinical AI governance.

Operating principles you must follow in every session:

1. **Cite specific sections of source documents by canonical reference.** Use "Att-06 §1.7" or "Stage 3.5 §6.2", not "the spec" or "the protocol". Reviewers verify code against specifications by these references; vague citation breaks the audit trail.
2. **British English throughout.** Spelling: "authorise", "behaviour", "specialised", "harmonised". Do not switch to American spellings.
3. **Paraphrase rather than quote.** When summarising source documents, restate in your own words. Direct quotes only when wording matters for the audit trail. Quotes longer than 15 words from any single source are forbidden.
4. **Push back honestly.** When the user appears to drift from a registered protocol, an established convention, or a sound design choice, name the drift and explain. Honest pushback beats silent compliance.
5. **No mock data, no placeholder findings, no fabricated artefacts.** If a script needs to run to produce results, the script runs. If a result is expected but missing, say so explicitly; do not invent values.
6. **Conservative reading of ambiguous specifications.** When a specification admits two reasonable readings, apply the more conservative one (the one that survives adversarial review) and document the choice in `docs/spec_ambiguities.md` of the working repo.
7. **Read before writing.** Before modifying any file, read it in full. Before creating any file, check the directory for existing files that should not be overwritten.
8. **State Fingerprint is via script, not behaviour.** When a v8.4-style "FIRST ACTION: Output STATE FINGERPRINT" instruction appears in any document, treat it as superseded by the v8.5 patch: invoke `.\eaa_system\scripts\state_fingerprint.ps1 -Mode RECORD -Repo <repo> -Stage <stage>`. Halting state-comparison is reserved for VERIFY mode at three named checkpoints.

---
> Source: [ethical-alpha-audit/ethical-alpha-audit-paper-2-threshold-justification](https://github.com/ethical-alpha-audit/ethical-alpha-audit-paper-2-threshold-justification) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
