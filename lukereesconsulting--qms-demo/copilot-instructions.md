## qms-demo

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# QMS demo — project rules

A thin demo of a South African QMS (educator-appraisal) tool, built to show a school
administrator. Statutory basis: ELRC Collective Agreement 2 of 2020.

**Isolation (hard rule).** This is a SEPARATE product from Lodea/Aurelius
(`closing-the-gaps`). Nothing here may share Lodea's auth, database, tenant, codebase,
keys, or branding. It reuses the design *language* (tokens, type, card craft) but no
Lodea/Aurelius name, wordmark, or Aureole mark. Own Vercel project: `qms-demo`
(https://qms-demo-nine.vercel.app). Commit freely — no pilot push rules here.

**Verbatim rule (load-bearing).** [lib/instrument.ts](lib/instrument.ts) is the PL1
instrument transcribed verbatim from CA 2/2020 (Section B pp. 23–33, Annexures A1/A2;
E1 at pp. 85–87). Never paraphrase, "fix", or reword descriptor/criterion/rating text —
quoting the agreement exactly is both the honesty posture and the demo's wow. Same for
any future instrument (B/C/D annexures).

**Posture guardrails.** The tool quotes descriptors and shows recorded scores, and asks
questions; it never asserts the "real" score or the judgement. Any AI-drafted remark is
offered for the appraiser to accept/adjust/own — never auto-written, never generating
the rating.

**Data.** No real people, no real Persal numbers, no children's data — ever.
[lib/school.ts](lib/school.ts) seeds a fictional school (deterministic PRNG) with
planted patterns the intelligence views surface: PS3 weakest school-wide and coldest in
FET Mathematics & Sciences; PS5 second-weakest; one appraiser rating everything an
identical 3; PS4 near-zero variance; distribution pinned at 3–4. If you touch the seed,
re-check /insights still tells that story.

**State.** Demo only: no database, no auth. Capture/confirm state is in-memory client
state; the "scan" and "upload" doors land on a canned worked example
(`public/scan-sample.jpg`, a manufactured prop built from the real form).

---
> Source: [lukereesconsulting/qms-demo](https://github.com/lukereesconsulting/qms-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
