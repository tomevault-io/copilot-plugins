## visa-application

> Use when the user wants help applying for any visa from any country — Schengen tourist (Italy, France, Germany, Spain, etc.), US B1/B2, UK visitor, Japan, Canada, China, Australia, or any other. Triggers on phrases like "I need to apply for a visa", "help me with my Schengen visa", "Italy visa application", "US B1/B2", "UK visitor visa", "Japan tourist visa", "Canada eTA", "Australian visa", mentions of VFS / TLScontact / BLS visa centres, consulate appointments, biometrics, or visa renewals. Walks the user from a one-line ask to a fully assembled, officer-ready Print Pack: brief Q&A → research current official requirements online (cross-validated against ≥2 sources, government sites preferred) → reuse or build a portable user profile → set up or reuse the application folder → generate cover letter, employment letter, filled application form, and printable checklist PDF → assemble numbered Print Pack. Maintains a profile so the next application is a one-day job, not a one-week job.


# Visa Application

A general-purpose workflow for assembling a complete, officer-ready visa application from any country to any country. Designed to be invoked once per application and reused across them — the user's reusable data is captured once and replayed forever.

## Why this skill exists

Visa applications are high-stakes, high-paperwork, and rules drift. Most applicants either over-search and waste days, or under-search and get rejected on a single missing line. This skill standardises the workflow: capture the few things only the user knows, look up the rest from official sources, cross-validate, generate every document the consulate actually wants in the order they want it, persist what's reusable, and pick up the thread on subsequent invocations all the way through to granted-visa capture.

Accuracy is the principal currency. If a fact disagrees between two sources, surface the disagreement to the user — don't paper over it.

## When to invoke

Whenever the user mentions applying for a visa to any country, even casually ("can you help me sort out my visa for Japan?"). Don't wait for them to ask for "the visa skill" — that's not a phrase they'll use.

### Activation ritual (same every time, no variation)

The skill's first turn must look identical on every invocation. Three things happen in this exact order, in the same response:

1. **Print the banner** (visible — five-line pixel-style header in a fenced code block):

   ```
     ██████
     █▒▒  █   visa-application  v1.0  ·  MIT
     █▒   █   any visa · any country · officer-ready in one session
     █  ●▒█   by @Shadowhusky · github.com/Shadowhusky/visa-application
     ██████
   ```

2. **Run Phase 0 searches** (silent — tool calls, not narrated):

   ```bash
   bash scripts/find-existing.sh profile
   bash scripts/find-existing.sh folder "<destination from user msg, or 'visa'>"
   ls -la ~/.claude/visa-profile.json ~/.claude/visa-history.json 2>/dev/null
   ```

3. **Call the AskUserQuestion tool** with either the cold-start 4-question set (if Phase 0 found no profile) or the warm-start single question (if Phase 0 found one).

No prose between steps. No "I'd be happy to help" preamble. No "Let me check…" narration. Banner → silent searches → interactive question.

## The workflow

The skill runs in nine phases in **strict order**. Do not skip Phase 0. Do not list options in prose when the AskUserQuestion tool can be used. The same invocation should produce the same workflow every time — that's the whole point of having a skill.

### Phase 0 — Searches (silent, part of the activation ritual)

This is the second step of the activation ritual above. Already specified there. The result determines whether Phase 1 is the cold-start 4-question kickoff or the warm-start single question.

### Phase 1 — Ask via interactive UI (MANDATORY)

**Always use the `AskUserQuestion` tool when asking the user anything in this skill.** Never list options in prose like "1. Tourist 2. Business …" — that's what the structured tool is for, and the user's experience must be consistent across invocations.

The tool caps at 4 options per question with an auto-"Other" fallback. Choose options that cover ~80% of likely answers; the long tail goes to "Other".

#### Cold start (no profile found in Phase 0)

Send **one** `AskUserQuestion` call with the questions below. **Important:** before deciding which to include, *parse the user's initial message* for facts already given. If they said *"I need a Schengen visa to Italy from the UK, 5 days in late June"*, that's destination + origin + visa type + duration *already answered* — do NOT re-ask. Only include the questions whose answers are missing.

| # | Question | header | Options |
|---|---|---|---|
| 1 | Which country are you applying for a visa to? | `Destination` | `Schengen area (Italy, France, Germany, Spain, NL…)` · `United States` · `United Kingdom` · `Canada / Australia / Japan / Other` |
| 2 | Which country are you applying *from* (where you legally reside)? | `Applying from` | `United Kingdom` · `United States` · `India` · `China` |
| 3 | What type of visa? | `Visa type` | `Tourist / Visitor` · `Business / Conference` · `Study / Work` · `Visit family / Transit / Other` |
| 4 | Trip duration? | `Duration` | `Under 1 week` · `1–2 weeks` · `2–4 weeks` · `Over 1 month` |

If the user picks an "Other" or umbrella option, follow up with a single free-text question to clarify (e.g., "Which Schengen country specifically?" or "Which Asian country?"). Don't try to enumerate 27 Schengen members in the tool — let them type the country name.

If all four facts are already in the user's initial message, skip the cold-start questions entirely and confirm in one short sentence (*"Got it — Italy Schengen tourist, ~5 days, applying from the UK. Moving on."*) before proceeding to Phase 2.

After the four kickoff answers, ask one free-text follow-up for the *specific* dates (e.g., "What's the exact departure → return date range?"). Dates have unbounded answer space and aren't a good fit for the option tool.

#### Warm start (profile found in Phase 0)

Skip the 4-question kickoff entirely. The profile already has identity, residence, employment, banking, and prior-visa history. Send **one** `AskUserQuestion`:

| Question | header | Options |
|---|---|---|
| Profile found ({last_updated}). What would you like to do? | `Next step` | `Continue existing application` (only if a relevant folder was found) · `Start new application to a different country` · `Update my profile (employer, address, etc.)` · `Something else` |

Branch the rest of the workflow based on the answer:

- **Continue existing** — read the application folder's `application_status.html` to figure out what's already done and what's outstanding. Jump to the *first* incomplete phase (could be appointment booking, document upload, form filling, or just the cross-check). Don't restart from Phase 1.
- **Start new application** — do the 4-question kickoff to get destination/dates for a fresh application.
- **Update profile** — ask what specifically changed (new job, new address, renewed passport, etc.) and save to `~/.claude/visa-profile.json` without starting any application workflow.
- **Something else** — free-text follow-up to understand intent.

#### When the user declines / cancels the structured question

If the AskUserQuestion call returns "User declined to answer", fall back to a single short free-text message: *"No worries — just tell me in your own words: where are you going, where are you applying from, what kind of visa, and roughly when?"* — and continue from their reply.

### Phase 2 — Profile setup

Phase 0 already told you whether a profile exists. Now handle each case:

**Profile found**: read the file, summarise the headline identity to the user in one sentence (via plain prose, not AskUserQuestion — this is a confirmation, not a question), and proceed to Phase 3. The "warm start" question in Phase 1 has already determined intent.

**No profile found**: capture data using the **document intake** flow below — never type-by-type Q&A unless the user prefers it. Write to `~/.claude/visa-profile.json` after each successful extraction so progress isn't lost.

### Document intake — when the user uploads files

Most users have IDs, payslips, bank statements, hotel bookings, and flight confirmations as PDFs or images already. Treat every uploaded file as an opportunity to **(a)** extract structured data, **(b)** cross-check against the profile, and **(c)** file it into the application folder.

When a user pastes or attaches a file:

1. **Read it** with the Read tool (PDF, JPG, PNG all supported). Extract every field that maps to the profile schema or to a per-application document.
2. **Classify the document first.** Before extracting fields, identify *what kind of document this is* — passport, payslip, bank statement, share code, visa vignette, BRP, employment letter, hotel booking, insurance policy, etc. State the classification to the user. This prevents misattributing fields (e.g., a share code expiry date is not a visa expiry date).
3. **Confirm** the extracted values to the user in one short message — "I read your passport: name SHARMA PRIYA, passport Z7621984, expires 2030-08-13. Saving to profile." — and **wait for the user to acknowledge** before writing to the profile. Don't save and move on silently.
4. **Flag ambiguous fields.** If a date or value extracted from a document could reasonably map to more than one profile field, **do not guess — ask**. Use `AskUserQuestion` to let the user clarify. Common traps:
   - A share code expiry date ≠ visa expiry date (share codes expire in 30 days; the visa they prove can be valid for years)
   - A BRP expiry date ≠ leave-to-remain expiry date (BRP cards were phased out; the underlying immigration permission may outlast the card)
   - An insurance policy issue date ≠ coverage start date
   - A hotel confirmation date ≠ check-in date
   - A payslip "period end" date ≠ payment date
5. **File it** into the application folder with a clean name. Don't dump it next to a dozen other files; rename to something like `passport-bio.pdf`, `payslip-{YYYY-MM}.pdf`, `hotel-{city}.pdf`.
6. **Cross-check** against the profile. If the payslip says salary £80k but the profile says £65k, flag it — the user may have had a raise, or one of them is wrong. If a newly extracted date conflicts with an existing profile date for the *same field*, surface both values and ask which is correct.
7. **Update the profile** with anything new (NI number, employer registered office, bank IBAN, etc.) — only after the user has confirmed the extracted values in step 3.

The user shouldn't have to retype data that's already on a document they have. The flow should feel like *"drop the file, get a short summary back, confirm, move on"*.

### Questionnaire form — when you need 4+ answers

**Never dump a multi-question text wall.** If you need 4 or more free-text answers from the user (remaining profile fields after document intake, visa-form-specific questions like parents' names / education history / travel history, etc.), generate an interactive HTML questionnaire instead.

**When to use:**

- **≤ 3 questions** → ask in chat (one message, or `AskUserQuestion` if multiple-choice).
- **4+ questions** → generate `questionnaire.html` in the application folder.

**How to generate:**

1. Identify every missing field. Group them into logical sections (e.g., "Family", "Education", "Travel history").
2. Build the form by substituting placeholders in `templates/questionnaire.html`:
   - `QUESTIONNAIRE_TITLE` — e.g., "US B1/B2 Visa — Additional Information"
   - `QUESTIONNAIRE_SUBTITLE` — e.g., "Fill in the fields below. Green fields are pre-filled from your profile."
   - `OUTPUT_FILENAME` — e.g., "ds160-answers.json"
   - `PREFILLED_JSON_PLACEHOLDER` — JSON object of values already known from the profile
   - `FORM_SECTIONS_PLACEHOLDER` — the dynamic fieldset blocks (see format below)
3. Write the filled HTML to the application folder.
4. Tell the user: *"Open {path}/questionnaire.html in your browser, fill in the fields, click Save. Come back here and say 'done' when finished."*
5. When the user says "done", find and read the downloaded JSON file:
   ```bash
   find ~/Downloads -name "OUTPUT_FILENAME" -newer "{app-folder}/questionnaire.html" 2>/dev/null
   ```
6. Parse the JSON and write every answer into `visa-profile.json` and/or the application-specific state.

**Fieldset format** for `FORM_SECTIONS_PLACEHOLDER`:

```html
<fieldset data-section="family">
  <legend>Family</legend>
  <div class="field">
    <label for="father_name">Father — full name (pinyin)</label>
    <input type="text" id="father_name" name="father_name" placeholder="e.g., ZHANG WEI">
  </div>
  <div class="field">
    <label for="father_dob">Father — date of birth</label>
    <input type="date" id="father_dob" name="father_dob">
  </div>
</fieldset>
```

Use the right input type for each field: `type="date"` for dates, `type="tel"` for phone numbers, `type="email"` for email, `<textarea>` for multi-line answers (travel history, prior employment), `<select>` for fixed-choice fields. Use the `class="field-row"` wrapper to pair two short fields side-by-side (e.g., first name + last name, from-date + to-date).

**Pre-population:** every field the profile already has gets pre-filled (green tint in the UI). The user only types what's missing. If the profile already covers 60% of the fields, the form is 60% done before the user even opens it.

### Phase 3 — Locate or create the application folder

Each application gets its own folder. **Always search first** for an existing one the user may have started:

```bash
bash scripts/find-existing.sh folder "{destination}"
```

This searches the common iCloud Drive `Travel/` and Documents tree, and prints any folder whose name matches `*{destination}*`, `*visa*{destination}*`, or `*{destination}*visa*`.

**If a candidate folder is found:** ask via `AskUserQuestion` (not prose):

| Question | header | Options |
|---|---|---|
| I found a folder that looks related to your {destination} application — use it? | `Folder` | `Use this folder` · `Create a new one` · `Show me where it is first` |

**If no candidate is found OR the user chose "Create a new one":** ask via `AskUserQuestion`:

| Question | header | Options |
|---|---|---|
| Where would you like the new application folder? | `Location` | `~/Documents/Visa Applications/` (default) · `iCloud Drive` · `Desktop` · `Custom path` |

Then create the folder with destination + year in the name. Don't create silently.

Inside the application folder, you'll later create:

```
{destination}-{year}/
├── Cover Letter.html                 ← opens in browser
├── Cover Letter.pdf                  ← PDF fallback / print
├── Employment Letter.html            (if employed)
├── Employment Letter.pdf
├── Visa-application-form-FILLED.pdf  (if portal exists)
│   or application_form_data.html + .pdf  (Tier 4 fallback)
├── Checklist.html
├── Checklist.pdf
├── application_status.html           ← running state (schema below)
├── Application Status.pdf
├── (user-provided documents the user drops in)
└── Print Pack/
    ├── 00 - CHECKLIST.pdf
    ├── 01 - …
    └── …
```

Every skill-generated document has both `.html` (for browser viewing) and `.pdf` (for mobile / print). User-provided uploads (passport scans, bank statements, etc.) stay in their original format.

#### `application_status.html` schema

This file is the source of truth for multi-session continuity. Phase 0 reads it via the folder search; Phase 1's "Continue" branch resumes from it; Phase 7 writes to it; Phase 8 reads it to decide which post-submission branch to enter. Use the `templates/application-status.html` template — substitute ALL_CAPS placeholders with real values. The result is a styled HTML page that opens in any browser and is still easy for the agent to parse via the Read tool.

The status badge uses one of: `in_progress`, `submitted`, `awaiting_decision`, `granted`, `refused` — set both the badge text and the CSS class (`status-in_progress`, etc.) to match.

Progress rows use the `PROGRESS_ROWS_PLACEHOLDER` marker. Each row is an `<li>` with class `done` or `todo`:

```html
<li class="done">Research complete</li>
<li class="done">Profile loaded</li>
<li class="todo">Submitted at centre</li>
```

After substituting placeholders, also render the HTML to PDF via `render-pdf.sh` so the user has both formats:

```bash
bash scripts/render-pdf.sh "{application-folder}/application_status.html" "{application-folder}/Application Status.pdf"
```

Keep this file updated at the end of every phase that changes state. Re-write the whole file (not just append) so it remains a clean snapshot.

### Phase 4 — Research current official requirements

This is the part where accuracy matters most. Read `references/research-protocol.md` for the full protocol — the short version:

1. **Hit the destination consulate page in the user's origin country first.** That's the most authoritative source. E.g., for Italy from UK: `conslondra.esteri.it`. Search "{destination} consulate {origin city} visa {type}".
2. **Hit the visa-centre operator second.** VFS Global for most Schengen + UK + many others; TLScontact for France, China, Saudi; BLS for some others. Search "VFS {destination} {origin}" or "TLScontact {destination} {origin}".
3. **Hit one third-party 2026 guide third** as a sanity check (Wise, Visard, Schengen Visa Support, etc.) — but treat as a tertiary signal, not a source of truth.
4. **Cross-validate at least two sources for every material claim** (fees, photo specs, insurance minimum, fund minimum, required documents, biometric rules, processing time, online vs paper form). If they disagree, surface to user.
5. **Check the published / updated date** of every source. Visa rules change often (EES launched Oct 2025; Italy goes fully digital 1 Jun 2026; UK BRPs phased out for eVisa late 2024 onward). Old guidance is dangerous.

See `references/known-portals.md` for known online application portals (Italy `e-applicationvisa.esteri.it`, US `ceac.state.gov` DS-160, UK `gov.uk/standard-visitor`, etc.) and `references/document-checklist.md` for typical document lists by visa type — both as starting points, not gospel; always re-verify online.

### Phase 5 — Book or capture the appointment

The visa appointment is the timeline anchor. Every document has to be ready before this date, and most countries require the appointment to be booked *before* generating the application form (the form often references the appointment number). It's also a common user blind spot — they assume "the visa centre will fit me in next week" and discover slots are 4–6 weeks out. So the skill asks early.

**Send one `AskUserQuestion`:**

| Question | header | Options |
|---|---|---|
| Have you already booked your visa appointment? | `Appointment` | `Yes, already booked` · `No, need to book now` · `Not sure — let me check my email` · `Not required for this visa type` |

#### Branch A — "Yes, already booked"

Capture the appointment details (free-text follow-up): date, time, visa centre / consulate, address, reference number. Read any confirmation PDF the user drops in. Save into the application folder and into the profile's `current_application.appointment` block.

These details are used in Phase 6 (Cover Letter mentions the appointment date) and Phase 7 (the Checklist PDF prints them as the cover sheet).

#### Branch B — "No, need to book now"

Walk the user through booking, end-to-end, via the browser MCP. Generic flow:

1. Open the official booking URL for `{destination}` from `{origin}` (see `references/known-portals.md`).
2. Select **service category** (Tourist / Business / Study / Other — match the visa type from Phase 1).
3. Select **applicant centre / city** — closest to the user's address from the profile.
4. Look at the available slots. **Recommend a date that gives at least:**
   - The consulate's documented **processing time** + a 5–7 day buffer, before the trip departure date.
   - At least 7–10 days from today (so the user has time to print and sign documents).
5. Fill the personal details from the profile (passport number, contact details, etc.).
6. **Stop at the payment step.** The user enters card details themselves — the skill *never* enters payment info, full stop.
7. After the user confirms, capture: date/time, reference number, address, and any confirmation PDF the portal generates.

**Country-specific nuances** (see `references/known-portals.md` for full list):

- **US (B1/B2):** DS-160 form → MRV fee payment → schedule interview at `ustraveldocs.com`. Three sequential steps, not one — the skill walks the user through each.
- **Schengen via VFS:** select destination → category → centre → slot → optional priority/premium add-ons → pay. CAPTCHA is common at slot-selection — user solves it.
- **France via TLScontact:** the France-visas portal *creates the TLS appointment slot* as part of the application — no separate booking step.
- **UK Visitor:** book biometrics appointment via UKVCAS / VFS after submitting the gov.uk application.

#### Branch C — "Not sure"

Help the user check. Common places appointments hide:
- Email search for "{destination}", "VFS", "TLScontact", "BLS", or "appointment confirmation"
- Visa-centre account on the operator's website (login + dashboard)
- Calendar entries near the trip date

If found, treat as Branch A. If not found, treat as Branch B.

#### Branch D — "Not required"

Some categories don't need an in-person appointment (Australia visas done entirely online, some Caribbean nations, ETIAS/ETA-style authorisations). Verify against the Phase 4 research — if the research already confirmed appointment-free, proceed straight to Phase 6 without further questions.

#### Timing cross-check (automatic, every branch)

After Phase 5 settles an appointment date, automatically check:

- **Appointment ≥ trip-departure − consulate processing time − 7-day buffer.** If not, flag clearly.
- **Documents won't be too stale:** payslips and bank statements should be < 30 days old at submission. If the appointment is unusually far in the future, warn that documents may need to be re-issued.
- **Peak-season risk:** summer Schengen (June–August), Chinese New Year, Hajj season, US Thanksgiving/Christmas all create backlogs. Surface if relevant.

### Phase 6 — Generate documents + assemble Print Pack

**Pre-flight: document freshness check.** Before generating anything, verify the supporting documents in the folder aren't stale relative to the submission (appointment) date:

- **Payslips:** the most recent must be < 30 days old at appointment date. If not, ask the user to upload the latest payslip via `AskUserQuestion` ("Latest payslip date?" / `Past 30 days` · `30–60 days old` · `Older` · `Don't have one`). Don't generate the document pack with stale evidence.
- **Bank statement:** must end within 30 days of appointment date. Same check.
- **Hotel / flight bookings:** must be active reservations (not cancelled). Spot check by reading the latest copy in the folder.
- **Insurance:** dates must cover the trip ±1 day buffer.

If any check fails, surface to user *before* generating documents — there's no point producing a Print Pack against stale evidence.

#### Language adaptation

Generated documents fall into two categories with different language rules:

**User-facing documents** — use the user's input language:
- Checklist (from `templates/checklist.html`)
- Application status tracker (from `templates/application-status.html`)
- Questionnaire (from `templates/questionnaire.html`)
- Form-data sheet (from `templates/form-data.html`)

When generating these, translate **all** visible UI text — headings, labels, button text, instructions, checklist items — to the user's language. The template's English text is a structural reference, not the final output. If the user writes in Chinese, the checklist heading should read "签证预约清单" not "Visa Appointment — Checklist". If they write in Spanish, "Lista de verificación — Cita de visado". Detect the user's language from their messages, not from the destination country.

**Application-facing documents** — use the language the consulate expects:
- Cover letter (from `templates/cover-letter.html`)
- Employment letter (from `templates/employment-letter.html`)
- Filled visa application form

These are read by visa officers. Use the language required by the specific consulate (e.g., English for the Italian consulate in London, French for the French consulate in Beijing, English for the US consulate everywhere). When in doubt, default to English — it's accepted by virtually all consulates.

#### Dual output: HTML + PDF for every generated document

Every document the skill generates is produced as **both HTML and PDF**:

- **HTML** — the primary format. Opens styled in any desktop browser. Keeps the application folder browsable and inspectable without special tools.
- **PDF** — the automatic fallback. Rendered from the HTML via `render-pdf.sh`. Opens on mobile, prints cleanly, goes into the Print Pack.

The workflow for each document:

1. Substitute placeholders in the relevant `templates/*.html` file. For user-facing documents, also translate all static UI text to the user's language (see "Language adaptation" above).
2. Write the filled HTML to the application folder (e.g., `Cover Letter.html`).
3. Render the HTML to PDF:

```bash
bash scripts/render-pdf.sh "{application-folder}/Cover Letter.html" "{application-folder}/Cover Letter.pdf"
```

(See `scripts/render-pdf.sh` for the canonical invocation. It uses Chrome headless on macOS; for Linux/Windows the script self-adjusts.)

If `render-pdf.sh` fails (no Chrome, no fallback renderer), keep the HTML — it's still usable — and warn the user that PDF generation failed.

#### Progress updates during generation

Don't generate all documents silently. After each document is written, tell the user in one line:

```
Wrote: Cover Letter.html + .pdf
Wrote: Employment Letter.html + .pdf
Wrote: Checklist.html + .pdf
Assembling Print Pack…
Print Pack ready (7 files).
```

This keeps the user informed without flooding them. The full manifest and review instructions come in Phase 7.

#### Documents to produce, in order

1. **Cover letter** — addressed to "Visa Section, Consulate General of {destination}, {origin city}". One page. State purpose, dates, employment, self-funding, and reference to enclosed evidence. Avoid promising things the documents don't back. See `templates/cover-letter.html`.
2. **Employment letter** — if employed. Should be signed by manager or HR on company letterhead. Render a draft, give to user, they get it signed. See `templates/employment-letter.html`.
3. **Filled visa application form** (note: this is the *full visa application form*, distinct from the short booking form filled in Phase 5 — they may live on the same portal or different ones). The skill fills this itself. Asking the user to fill it by hand is a last resort, not the default. Follow the four-tier strategy in `references/form-filling-strategy.md`:

   - **Tier 1 — Online portal.** If the destination has an official portal (Italy `e-applicationvisa.esteri.it`, France `france-visas.gouv.fr`, US DS-160 `ceac.state.gov`, etc.), load the Chrome browser MCP via ToolSearch and fill the portal page-by-page using profile data. The portal generates a PDF with a 2D barcode the officer scans. This is the cleanest output.
   - **Tier 2 — Interactive PDF.** If no portal but the PDF has AcroForm fields (`pymupdf` detects this via `doc.is_form_pdf`), fill the named fields directly. Perfect alignment guaranteed.
   - **Tier 3 — Vision-driven coordinate overlay.** If the PDF is flat, render the blank to PNG, use vision to read field positions, overlay text via pymupdf, **re-render and verify visually**, refine coordinates until clean. Don't stop at first attempt — inspect the result and adjust.
   - **Tier 4 — Data sheet for manual transcription.** Only if 1–3 all fail (genuinely rare). Generate `application_form_data.html` + PDF from `templates/form-data.html` — a styled table of field names and values. Hand the user the blank printed form plus this sheet, and they transcribe.

   **Quality gate:** every tier must end with the agent visually inspecting the rendered output. Offset, overlapping, or otherwise messy PDFs are not "done" — drop down to the next tier rather than ship a poor result.
4. **Checklist (00 - CHECKLIST)** — a one-page summary the user prints as the cover sheet of the Print Pack. See `templates/checklist.html`. Include: appointment time/place, before-leaving-home tick list, the numbered stack order, likely officer questions and how to answer them, and the cross-checks you've already verified.
5. **Application status** — the running state tracker. See `templates/application-status.html`. Updated at the end of every phase that changes state.

After documents are generated, assemble the Print Pack:

```bash
bash scripts/build-print-pack.sh "{application-folder}"
```

This copies (not moves — keep originals) each PDF into a `Print Pack/` subfolder with a numbered, human-readable prefix in the order an officer flips through them. The Print Pack uses PDFs only; the HTML copies stay in the main folder for easy browsing.

### Phase 7 — Final cross-check report

This is the user's single summary of everything the skill has done. It must be concrete and actionable — file paths the user can click, things to check, things to print. Use this exact three-part template:

#### Part 1 — Document manifest

List every file in the application folder, grouped by purpose. The user should be able to open Finder / Explorer to the folder and match what they see to this list.

```
📂 {application-folder-path}

   Documents to review (double-click the .html to preview in your browser):
   ├── Cover Letter.html / .pdf         — review wording, sign the PDF
   ├── Employment Letter.html / .pdf    — forward to HR for signature on letterhead
   ├── Checklist.html / .pdf            — your appointment-day cheat sheet
   ├── application_status.html / .pdf   — tracks what's done and what's pending
   └── (any other generated documents)

   Documents you provided:
   ├── passport-bio.pdf
   ├── payslip-2026-04.pdf
   └── … (list each)

   Ready to print:
   └── Print Pack/
       ├── 00 - CHECKLIST.pdf
       ├── 01 - …
       └── … (numbered in officer flip-order — print all, single-sided, A4)
```

#### Part 2 — Cross-checks

```
✅ Compliant: <list of items already verified>
❌ Still needed before {appointment date}: <specific action items>
⚠️ Verify yourself: <items the skill can't confirm, e.g., fund balance, immigration status>
📦 Bring to appointment: <visa fee amount + currency, biometric photos, original passport>
```

Surface any inconsistency found — date mismatches, salary discrepancies, expired documents. Better to flag now than have the officer find it.

#### Part 3 — Next steps

A short numbered list of exactly what the user still has to do, in order:

```
1. Open Cover Letter.html — read it, make sure dates and employer are correct
2. Forward Employment Letter.pdf to your HR — they sign on letterhead and return it
3. Print everything in Print Pack/ — single-sided, A4, keep in numbered order
4. Sign the visa form (page X) and the cover letter (bottom of page 1)
5. Paperclip 2 biometric photos to file 04
6. Bring: printed stack + original passport + payment card + phone
```

Tailor the list to what's actually outstanding for this application. Don't include steps that are already done.

---

After showing this to the user, write the same state to `{application-folder}/application_status.html` (and re-render its PDF) so the next invocation (Phase 0 + warm-start "Continue") can read where things stand without re-prompting the user.

### Phase 8 — Post-submission: tracking, collection, outcome

The workflow doesn't end when the user walks out of the visa centre. Applications take days to weeks to decide, and the skill needs to be able to pick up the thread on a later invocation.

**Trigger detection:** during Phase 0, after `find-existing.sh folder` returns a path, read that folder's `application_status.html`. If the status badge reads `submitted` or `awaiting_decision` or `granted` or `refused`, Phase 1's warm-start question shifts from the standard "Continue/New/Update" to the post-submission variant below.

**Warm-start question (post-submission variant):**

| Question | header | Options |
|---|---|---|
| Your {destination} application is in processing — what's happened? | `Outcome` | `Still waiting / want to check status` · `Approved — passport collected` · `Refused — need to understand next steps` · `Other (resubmission request, courier issue, etc.)` |

#### Branch A — Still waiting

Help the user check status:

1. Read the tracking URL from `application_status.html` (stored at the end of Phase 5).
2. Open the centre's tracking dashboard via browser MCP. Most centres need just the application reference + passport number.
3. Report status to user. If status is "decision made — ready for collection", switch to Branch B.
4. If status has been "submitted" for longer than the documented processing time + 5 days, flag it as unusually slow and surface the centre's escalation contact.

#### Branch B — Approved

Capture from the user (or from the granted visa sticker PDF if courier-delivered with a scan):

- Visa sticker / reference number
- Valid from / Valid until
- Number of entries (single / two / multiple)
- Days of stay allowed
- Type code (e.g., Schengen C, US B1/B2)

Update:

- `~/.claude/visa-profile.json` → append to `visa_history[]` with the full granted entry. The next application can answer "previous visas in last 3 years" precisely.
- `application_status.html` (+ re-render PDF) → status `granted` with full visa details.

Briefly remind the user of:

- Trip-day documents to carry (passport with visa, return ticket, insurance, hotel print-outs).
- EES biometric capture at the first Schengen border crossing if relevant.
- Any country-specific "things you cannot do" on this visa (no professional sportsperson, no public funds, etc.) — important if the visa allows residence as well as travel.

#### Branch C — Refused

Capture the refusal reason from the user (or read the official refusal letter PDF if they upload it). The Schengen refusal letter is highly standardised — each reason has a code (1–9) — extract the code(s).

Assess and advise (don't promise outcomes):

- **Documentary refusal** (missing document, support not credible) → file an appeal within the consulate's deadline (usually 30 days for Schengen). Draft the appeal letter if requested.
- **Fundamental refusal** (insufficient ties to home country, intent doubts) → re-applying is sometimes possible but usually needs a substantively stronger file, not just a re-submission.
- Some Schengen refusals say "no appeal possible" but re-application is allowed; others permit appeal but not re-application. Country-specific.

Update `visa_history[]` with `outcome: "Refused"` and the reason code. Most future applications require disclosure of prior refusals — this entry will be picked up automatically.

If the user wants to appeal, generate an appeal letter PDF (use a variant of the cover-letter template) addressing each refusal reason point by point. The user signs and submits within the deadline.

#### Branch D — Other

Free text follow-up. Common cases:

- Passport collection logistics — in-person at the centre vs. courier delivery (paid extra fee).
- Mid-processing documentation request from the consulate — research what they're asking, help respond.
- Withdrawal of application — capture and update history.

## The "do, don't ask" principle

Throughout the workflow, default to acting. The user has limited time and wants the application *done*, not narrated.

- **Don't ask for data you can extract.** If the user uploads a passport scan, read the bio page yourself — don't ask "what's your passport number?"
- **Don't ask the user to fill the form.** Use the 4-tier strategy above.
- **Don't ask the user to search for hotels / flights** — they already have these as PDFs in their email. Ask them to drop the confirmations in, then extract the relevant fields yourself.
- **Don't ask the user to "verify" things you can verify yourself.** Cross-check passport DOB against profile DOB, flight dates against hotel dates, employer address against payslip header — automatically, in the background. Only surface the *discrepancies*, not the matches.
- **Don't ask the user where to put files.** Default to `~/Documents/Visa Applications/{Country}-{Year}/` and only ask if the user previously indicated a different location.
- **Don't pause to confirm trivial decisions.** Don't ask "should I generate the cover letter now?" — just generate it. Surface a summary at the end, not a confirmation at every step.
- **Never dump a multi-question text wall.** A numbered list of 4+ free-text questions in chat is bad UX — the user has to scroll, retype, track what they've answered. If you need 4+ answers, generate `questionnaire.html` (see "Questionnaire form" section above). The user fills it at their own pace in a browser with proper form fields, pre-populated values, and a progress bar. This is a hard rule, not a suggestion.

**But: unsure means ask.** "Do, don't ask" applies to *unambiguous* data. When a value could map to more than one field, or when you're interpreting what a document *means* rather than reading what it *says*, you **must** confirm with the user before writing it to the profile or any generated document. A wrong value silently committed to the profile will propagate to every document the skill generates — cover letter, application form, checklist — and may not be caught until the officer rejects the application. One confirmation question is cheap; a rejected visa is not.

Concrete examples of when to ask:

- A date on a document could be an expiry date for *different things* (share code expiry ≠ visa expiry ≠ BRP expiry)
- A document contains a status or category you haven't seen before
- The extracted value contradicts something already in the profile
- You're not certain what type of document the user uploaded

Where you *do* need to involve the user:

- Payments (card details — never enter on user's behalf)
- Wet signatures (printed forms only)
- CAPTCHAs and biometric prompts
- Ambiguous document extractions — when a value could map to multiple profile fields (see document intake rules above)
- Genuinely ambiguous declarations (e.g., "are you in possession of any weapons?" — ask once, save to profile, reuse forever)
- Data that can't be extracted from any document and has 4+ fields — use the questionnaire form, not chat

## Failure modes — how to fail gracefully

Real applications hit edges. When they do, be honest and useful:

- **Research finds nothing reliable.** If neither the consulate page nor the visa-centre operator lists a particular field (fee, photo spec, etc.), say so clearly: *"I couldn't find an authoritative figure for X — third-party guides say Y, but verify by emailing {consulate-email} before relying on it."* Don't invent.
- **Online portal is down or unreachable.** Tier 1 of form-filling becomes Tier 2 or 3. Tell the user the portal isn't responding right now; offer to retry or fall back. Don't loop indefinitely.
- **Profile file is corrupt or partially-populated.** Read what you can, ignore garbled fields, ask for missing essentials only. Don't refuse to proceed because one optional field is malformed.
- **User's appointment is too soon.** If processing time + buffer exceeds days-until-appointment, surface this prominently and suggest either rescheduling the appointment or arranging a courier-back-to-applicant decision delivery if available.
- **Country / route the skill has no specific knowledge of.** Fall back to the harmonised Schengen baseline (passport, photos, insurance, funds, bookings, cover letter) and tell the user "I'm working from the standard short-stay tourist visa template — verify country-specific requirements on the consulate page before submitting."
- **User uploads something the skill can't read** (encrypted PDF, password-protected file, unreadable image). Ask the user to re-export as a standard PDF or screenshot the relevant pages. Don't fail silently.

## Things to be paranoid about

- **Stale data.** Today's date is whatever the harness says. Use it. Visa rules change quarterly.
- **The "current address" trap.** The user may have moved. Confirm address against payslips/bank statements before generating the cover letter.
- **The "passport about to expire" trap.** Many countries require 3 *or 6* months past return. Spell it out.
- **The "BRP vs eVisa" trap.** Several countries (UK, Australia, EU member states) have migrated from physical residence cards to digital eVisas. Don't reference outdated physical cards.
- **The "share code ≠ visa" trap.** A UK share code expires in 30 days (it's a temporary proof link), but the underlying visa it proves may be valid for years. Never use a share code expiry date as the visa expiry date. Same principle applies to any temporary proof document — its expiry is not the permission's expiry.
- **The "wrong date from the wrong document" trap.** This is the most dangerous class of error the skill can make. Every document has multiple dates (issue date, expiry date, coverage dates, payment dates). Before writing any date to the profile, be certain you know *which date it is* and *which profile field it maps to*. If there's any ambiguity, ask. A silently wrong date propagates to every generated document and may not be caught until the officer rejects the application.
- **The "two adults on a single hotel booking" trap.** If the user is travelling solo but their hotel booking is for two, the officer will notice. Flag it.
- **Insurance dates one day short.** The travel insurance must cover ≥1 day past return, every single day of the trip.
- **EES (Entry/Exit System).** Live in the Schengen area since Oct 2025. First crossing captures biometrics at the border. Mention it so the user isn't surprised.
- **Multiple-entry visa signing.** For multiple-entry Schengen visas some consulates expect a second signature acknowledging the insurance clause. Verify per consulate.

## Reuse mindset

Every time you finish a visa application, **update `~/.claude/visa-profile.json`** with anything new learned (new employer, new address, new prior visa, new passport). The next application — for the same person, possibly to a different country — should be substantially faster because the profile is already 80% filled.

Also update `~/.claude/visa-history.json` with a summary of what was applied for (country, type, dates, outcome if known). Past prior-visa entries matter: many Schengen forms have a "previous visas in last 3 years" question, and the past entries are exactly what's needed.

## File map for this skill

| File | Purpose |
|---|---|
| `SKILL.md` | This file — the workflow |
| `references/research-protocol.md` | How to research and cross-validate sources |
| `references/document-checklist.md` | Standard document lists by visa type |
| `references/known-portals.md` | Online application portals, appointment-booking URLs/flows, country quirks |
| `references/profile-schema.md` | The `visa-profile.json` schema |
| `references/form-filling-strategy.md` | 4-tier strategy for filling application forms without bothering the user |
| `references/visa-scenarios.md` | Less-common scenarios (renewal, family, business, transit, student, refusal/appeal, multi-country) |
| `templates/cover-letter.html` | Cover letter template, A4 single page |
| `templates/employment-letter.html` | Employment letter template, A4 single page |
| `templates/checklist.html` | Print Pack cover-sheet checklist |
| `templates/application-status.html` | Application status tracker template |
| `templates/form-data.html` | Tier 4 form-data sheet template (manual-transcription fallback) |
| `templates/questionnaire.html` | Interactive HTML form for bulk data collection (replaces text-wall Q&A) |
| `scripts/render-pdf.sh` | HTML → PDF via Chrome headless |
| `scripts/find-existing.sh` | Search the user's machine for existing profile / folder |
| `scripts/build-print-pack.sh` | Assemble the numbered Print Pack |
| `tests/smoke-test.sh` | Structural sanity check (file presence, executable scripts, no leaked personal data, render pipeline) |
| `tests/scenarios.md` | Eleven end-to-end behavioural scenarios with expected agent responses |

---
> Source: [Shadowhusky/visa-application](https://github.com/Shadowhusky/visa-application) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
