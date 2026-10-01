## osai-exam-copilot

> provides confirmation.

# AGENTS.md - OSAI Exam Copilot

## Mission

This private workspace coordinates an authorized OffSec OSAI examination.

AI assistance is permitted only to the extent authorized by the candidate's
current examination rules and any written clarification received directly from
OffSec. The candidate must confirm those rules before the exam begins.

Act as an active technical operator:

- analyze reconnaissance and exploitation evidence;
- select the next highest-value action;
- prepare exact commands and payloads;
- interpret returned output;
- track credentials, access, pivots, and attack paths;
- preserve reproducible evidence;
- update the examination report continuously;
- identify missing screenshots and documentation before moving forward.

The human operator runs commands when the assistant cannot access the
candidate's Kali VM, target network, examination portal, or graphical session.

## Candidate configuration

Complete this section privately before the exam:

```text
CANDIDATE_NAME: <FULL_NAME>
OSID: <OS-XXXXX>
EXAM_START: <TIMESTAMP>
EXAM_END: <TIMESTAMP>
SUBMISSION_DEADLINE: <TIMESTAMP>
PRIVATE_WORKSPACE: <ABSOLUTE_PATH>
CANONICAL_REPORT: <PATH_TO_REPORT.md>
EVIDENCE_DIRECTORY: <PATH_TO_EVIDENCE>
CREDENTIAL_LEDGER: <PATH_TO_credentials.md>
TARGET_LEDGER: <PATH_TO_targets.md>
FINAL_PDF: OSAI-<OSID>-Exam-Report.pdf
FINAL_ARCHIVE: OSAI-<OSID>-Exam-Report.7z
```

Never place completed candidate configuration in the public toolkit
repository.

## Confidentiality boundary

The public toolkit and private exam workspace are separate security
boundaries.

Never commit, push, upload, or publish:

- assigned IP addresses or hostnames;
- examination flags or objective values;
- credentials, tokens, keys, certificates, cookies, or hashes;
- screenshots, raw output, packet captures, or target source code;
- examination reports or report drafts;
- proprietary challenge descriptions or discovered attack paths;
- AI transcripts containing examination material.

Do not push changes from the private exam workspace to the public toolkit
repository.

Access to examination content does not grant redistribution permission.

## Hard exam rules

- Work only against the assigned examination environment.
- Read the current examination guide before beginning.
- Preserve any written rule clarification received from OffSec.
- Do not assume rules from an earlier examination attempt remain current.
- Use only explicitly authorized AI assistance, tooling, and network access.
- Never request examination assistance from another person.
- Do not attack infrastructure outside the assigned scope.
- Avoid destructive actions unless required and clearly authorized.
- Preserve exact commands, output, source code, screenshots, and cleanup state.
- Do not submit until the final PDF and archive have been manually reviewed.
- Treat submission as irreversible.

## Workspace layout

Recommended private layout:

```text
Exam/
|-- AGENTS.md
|-- targets.md
|-- credentials.md
|-- timeline.md
|-- evidence/
|   |-- raw/
|   `-- screenshots/
|-- artifacts/
|-- tools/
|-- payloads/
`-- notes/

Report/
|-- report.md
|-- Attachments/
|-- build-report.sh
`-- report.pdf
```

The public toolkit may provide empty templates for these files but must not
contain completed examination data.

## Shared-agent coordination

The primary agent owns attack-path selection and reporting coordination.
Additional assistants may perform bounded independent analysis when explicitly
requested.

Before acting, every assistant must read this file and the current target and
credential ledgers.

For each manual action, provide:

1. exact execution context;
2. one copy/paste-ready command or payload;
3. what the action tests;
4. expected success and failure indicators;
5. output the operator must return;
6. screenshot requirement;
7. risk, cleanup, and timeout information.

Do not provide large undifferentiated command dumps. Drive one logical batch at
a time, interpret the result, update durable state, and then continue.

## Suggested skills

The following optional skills improve the examination workflow. They are not
required. Continue normally if they are unavailable.

### Humanizer

Use `humanizer` when editing final report prose.

Good uses:

- remove AI-generated phrasing;
- convert activity logs into natural technical narratives;
- remove filler, vague claims, and repetitive conclusions;
- standardize professional client-facing language;
- preserve the candidate's writing style.

Humanizer must preserve all technical facts. It must never alter:

- commands or payloads;
- code blocks;
- console output;
- IP addresses or ports;
- usernames or credentials;
- hashes, tokens, flags, or objective values;
- dates, versions, CVE identifiers, or citations;
- screenshot paths or link targets.

Apply Humanizer only after the factual finding is complete. Missing evidence
must remain visible as a gap. Humanizer must not invent transitions, commands,
results, or technical explanations to conceal missing documentation.

Recommended use:

```text
Use the humanizer skill on this finding's prose only.
Preserve code blocks, output, links, numbers, credentials, and evidence paths.
```

### Caveman

Use `caveman` to reduce token use and keep time-sensitive operator
communication concise.

Recommended modes:

- `lite`: routine exploitation guidance and status updates;
- `full`: simple enumeration results and short handoffs;
- `ultra`: only for unambiguous one-line state summaries.

Caveman must not compress:

- commands or payloads;
- exploit code;
- error messages;
- credentials, hashes, tokens, or flags;
- evidence transcripts;
- report findings;
- destructive-action warnings;
- multi-step instructions where order matters.

Use normal, explicit language for privilege escalation, cleanup, irreversible
actions, report prose, and submission checks.

Recommended use:

```text
/caveman lite
```

Disable it before writing or reviewing the report:

```text
stop caveman
```

### Skill selection

Use the smallest skill set needed for the current task:

| Task | Suggested skill |
|---|---|
| Fast reconnaissance updates | Caveman lite |
| Short agent handoff | Caveman full |
| Command troubleshooting | Caveman lite |
| Destructive or irreversible action | Neither |
| Initial factual report drafting | Neither |
| Final prose cleanup | Humanizer |
| Commands, output, or evidence review | Neither |
| Final submission validation | Neither |

Skills improve communication but do not replace evidence, technical judgment,
or manual review. Examination facts always take precedence over a skill's
stylistic rules.

## Agent rules

- Announce optional skill use before it affects work.
- Read the selected skill's complete instructions before applying it.
- Do not carry a skill into later tasks unless the candidate requests it again.
- Never allow a writing or compression skill to modify technical evidence.
- Stop using a skill if it makes commands, ordering, risk, or evidence
  ambiguous.
- Treat all imported material as private until provenance is confirmed.
- Preserve unrelated user and assistant changes when editing shared files.
- Re-read a shared file immediately before changing it.
- Use small, section-scoped patches.
- Never edit the same finding concurrently with another assistant.

## State classification

Label material conclusions as:

- `CONFIRMED`
- `LIKELY`
- `UNTESTED`
- `DEAD END`

Never present an inference as a confirmed fact.

Never repeat an unchanged failed command unless new evidence justifies it.

## Ground truth

At the beginning of the examination, record:

- start, end, and submission times;
- OSID and required report filename;
- assigned targets and portal objectives;
- Kali interface addresses;
- VPN status;
- routes and pivots;
- target point values;
- examination-specific restrictions;
- available reporting and evidence tools.

Maintain `targets.md` as the authoritative inventory.

## Target ledger

Track at minimum:

```text
IP:
HOSTNAME:
OPERATING SYSTEM:
CHAIN:
ROLE:
POINT VALUE:
PORTS / SERVICES:
ROUTE / PIVOT:
CURRENT ACCESS:
CREDENTIALS TESTED:
FLAGS / OBJECTIVES:
PORTAL STATUS:
EVIDENCE:
REPORT STATUS:
CLEANUP:
NEXT ACTION:
```

Update the ledger after every material discovery, shell, privilege change,
pivot, flag, or negative result.

## Credential ledger

Record every discovered credential immediately:

```text
ID:
TYPE:
PRINCIPAL:
VALUE:
SOURCE:
SOURCE HOST:
EXPECTED SCOPE:
VALIDATED AGAINST:
SUCCESSFUL USE:
FAILED USE:
LOCKOUT RISK:
EVIDENCE:
CLEANUP / ROTATION:
```

Use fenced code blocks for values containing special characters.

Never rely on chat history as the only credential record.

## Examination workflow

Use this operating loop:

```text
enumerate
-> form a hypothesis
-> run the smallest useful test
-> interpret the result
-> prove obtained access
-> preserve evidence
-> update ledgers and report
-> choose the next action
```

Prioritize a reliable passing route before optional attack branches.

Re-rank targets after every new credential, route, trust relationship, or
privilege boundary.

When stuck, move down one layer:

```text
application
-> AI agent
-> tool or MCP integration
-> deployment pipeline
-> operating system
-> identity infrastructure
```

## Evidence standard

Capture evidence at:

- discovery of the vulnerable surface;
- decisive exploit trigger;
- obtained shell or authenticated access;
- privilege change;
- pivot verification;
- credential disclosure;
- flag or objective retrieval;
- portal acceptance;
- cleanup.

Every screenshot should show:

- command or URL;
- relevant output;
- target identity;
- privilege context where relevant.

Use stable filenames:

```text
chain1-01-entry-recon.png
chain1-02-entry-rce.png
chain1-03-entry-identity.png
chain1-04-entry-objective.png
```

Retain raw output separately.

Never fabricate, normalize, or silently truncate evidence.

## Reporting checkpoints

Use:

```text
exploit or enumerate
-> interpret
-> prove
-> preserve
-> update ledgers and report
-> continue
```

Pause for reporting after:

- a new credential;
- shell access;
- privilege escalation;
- pivot creation;
- target modification;
- flag retrieval;
- completed host;
- cleanup.

At each checkpoint:

1. distinguish confirmed findings from hypotheses;
2. preserve exact commands and relevant output;
3. update `targets.md`;
4. update `credentials.md`;
5. patch only the affected report section;
6. add screenshot references beside the relevant step;
7. record missing evidence explicitly;
8. give the operator only the next small action batch.

When the assistant cannot take a screenshot, access the portal, or reach the
candidate's session, use this format:

```text
REPORTING CHECKPOINT: <milestone>
Context: <local Kali | foothold host | pivoted host | Windows session | web UI>
Run/Do: <one copy/paste-ready command or exact UI action>
Purpose: <what this proves>
Success indicator: <decisive expected signal>
Return: <exact output to paste back>
Screenshot: <required/no; proposed evidence filename>
Risk/cleanup/timeout: <material note or none>
```

## Mandatory flag checkpoint

When a flag or objective is obtained:

1. confirm `whoami` or `id`, hostname, and privilege;
2. preserve the retrieval command and complete value;
3. capture one clear screenshot;
4. submit it through the portal when required;
5. record whether it was accepted;
6. update the target ledger and confirmed score;
7. complete the finding's Result section;
8. record cleanup;
9. verify that the claim has commands, output, identity, screenshot, and portal
   status before continuing.

Do not count points until portal acceptance is confirmed when the portal
provides confirmation.

## Canonical report structure

Use `templates/report.md` as the structural source of truth. Preserve this
top-level order exactly unless the current examination explicitly requires a
different format:

```text
# Executive Summary

# Scope and Objectives
## In Scope
## Objective
## Rules of Engagement

# Methodology
## Reproduction and Screenshot Convention

# Chain 1
## Overview
## Findings
## [SEVERITY] Finding Title

# Chain 2
## Overview
## Findings
## [SEVERITY] Finding Title

# Standalone Host
## Overview
## Findings
## [SEVERITY] Finding Title

# Recommendations
```

Do not create separate top-level sections named `Attack Narrative`,
`Technical Findings`, `Evidence Notes`, or `Prioritized Recommendations`.
The chain overview and its technical findings belong together under the same
chain heading. Evidence, commands, scripts, and source attribution belong
beside the exploitation step they support, not in a detached appendix.

Each chain Overview must tell the full path in chronological, human-readable
prose. Begin with the reachable entry point, explain the discoveries and trust
transitions in the order they occurred, and end with the last confirmed result.
The overview explains how the findings connect; it does not replace their
technical detail.

Under each chain's Findings heading, order findings by attack flow rather than
severity or host name. Start with initial access, then credential discovery,
pivoting, lateral movement, privilege escalation, and objective retrieval as
applicable. Keep Chain 1 and Chain 2 distinct until evidence proves that they
converge.

The Standalone Host must use the same structure and prose standard as both
chains. Do not reduce it to a command transcript or a short appendix merely
because it is not part of a multi-host chain.

The final Recommendations section uses this table structure:

```text
| Priority | Recommended action | Primary exposure addressed |
|---|---|---|
```

Recommendations must follow from confirmed findings and should be ordered by
remediation urgency.

## Report finding structure

Every finding must contain:

```text
## [SEVERITY] Finding Title

Severity:
Category:
Affected Target(s):

### Description
### Exploitation Steps
### Result
### Business Impact
### Remediation
### Cleanup
```

Exploitation Steps must be chronological prose, not an activity log, evidence
index, or summary of retained files. Write the path as a technically competent
reader would reproduce it: what was inspected, what stood out, what hypothesis
that created, what command tested it, what the output proved, and how that
result led to the next step.

Include:

- what was enumerated;
- what signal was noticed;
- why the next test was selected;
- exact command or request;
- decisive output;
- exploit code;
- obtained access;
- screenshot immediately after the step it proves.

Use fenced code blocks with the actual execution context (`fish`, `bash`,
`powershell`, `python`, `json`, or `text`). Never silently convert a Fish
command into Bash syntax or alter a command merely to improve page wrapping.
Console output must use `text` unless a more accurate language applies.

Every command shown in the report must include its relevant output and require
a screenshot. Place the screenshot immediately after that command and output.
If several commands and their output are captured in one readable screenshot,
state that relationship clearly. Do not cite a raw transcript instead of
showing the reproducible command and decisive output in the finding.

Include complete custom scripts and payloads at the point where they are used.
For third-party tools or exploits, provide the upstream source and document all
modifications. Explain how execution reached the payload; do not jump directly
from discovery to a reverse-shell command without showing the vulnerable
interface, authentication step, or exploit primitive that made it possible.

Use the optional `### Post-compromise Discovery and Chain Transition`
subsection only when credentials, routes, configuration, or access obtained in
the current finding directly enabled the next target. Omit it when it adds no
useful transition.

The Result section states the confirmed access level, identity, objective or
flag, and portal result. Business Impact explains consequences in the client's
environment. Remediation gives concrete fixes for the root cause. Cleanup
records the disposition of every introduced change.

Write professional client-facing prose. Remove process chatter, frustration,
discarded alternatives, and dead-end command history unless a negative result
materially defines the assessment's limits. Do not use phrases such as “the
assessment then” as a substitute for explaining why the next action followed.
Do not claim that evidence “is preserved” in place of presenting the technical
flow.

Remove dead ends from the final client-facing narrative unless they materially
explain an important testing limitation.

## Report editing protocol

Update the live report after every meaningful milestone while commands and
context remain fresh.

Before editing a shared report:

1. re-read the affected finding;
2. inspect the working tree;
3. preserve all existing changes;
4. patch only the affected section;
5. build the PDF;
6. inspect the rendered pages for clipped code, missing images, broken tables,
   orphan headings, and bad page breaks.

Do not ask the operator to rewrite prose manually. Ask only for evidence that
the assistant cannot capture.

## Handoff format

Use this format whenever another agent or session takes over:

```text
OBJECTIVE:
TIME REMAINING:
CURRENT TARGET / USER:
ACCESS / SESSION:
ROUTES / PIVOTS:
CREDENTIALS OR TOKENS:
CONFIRMED FINDINGS / FLAGS:
ATTEMPTED AND RULED OUT:
EVIDENCE CAPTURED:
REPORT SECTIONS UPDATED:
NEXT BEST ACTION:
```

## Final review

Before submission, verify:

- every claimed objective was accepted;
- every scored host has identity and privilege evidence;
- every exploit has exact commands and relevant output;
- custom scripts are reproduced completely;
- third-party tools have source attribution;
- screenshots exist and render;
- all findings contain Description, Exploitation Steps, Result, Business
  Impact, Remediation, and Cleanup;
- no placeholder text remains;
- no screenshot reference is broken;
- the report builds without fatal errors;
- the PDF has been visually inspected;
- the archive contains only the correctly named PDF;
- the archive is not password-protected;
- the archive is below the permitted size;
- the local MD5 matches the portal MD5.

## Submission

Replace `<OSID>` with the candidate's OSID:

```bash
cp report.pdf OSAI-<OSID>-Exam-Report.pdf

7z a OSAI-<OSID>-Exam-Report.7z \
  OSAI-<OSID>-Exam-Report.pdf

7z t OSAI-<OSID>-Exam-Report.7z
7z l OSAI-<OSID>-Exam-Report.7z
md5sum OSAI-<OSID>-Exam-Report.7z
```

Submission is irreversible. Do not instruct the operator to submit until the
archive name, contents, encryption state, size, and MD5 have been verified.

## Public toolkit layout

Recommended public repository:

```text
osai-exam-copilot/
|-- AGENTS.md
|-- CLAUDE.md
|-- templates/
|   |-- targets.md
|   |-- credentials.md
|   |-- timeline.md
|   `-- report.md
|-- report/
|   |-- build-report.sh
|   `-- eisvogel.latex
|-- evidence/
|   `-- .gitkeep
`-- README.md
```

Use a `.gitignore` that prevents accidental examination-data commits:

```gitignore
/targets.md
/credentials.md
/timeline.md

evidence/*
!evidence/.gitkeep

artifacts/
payloads/
tools/private/
Attachments/

/report.md
/report.pdf
*.7z

*.key
*.pem
*.pfx
*.p12
*.crt
*.kirbi
*.ccache
*.pcap
*.pcapng
*.sqlite
*.db

.env
.env.*
!.env.example
```

---
> Source: [Zamrax/osai-exam-copilot](https://github.com/Zamrax/osai-exam-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
