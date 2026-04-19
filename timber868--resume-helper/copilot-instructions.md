## resume-helper

> Tailor cover letters using the master cover.tex form (letter class, header, body, closer)


# Cover letter tailoring

When the user asks for a **tailored cover letter** for a job (or to create/update cover letters), always use the **form and structure** of the master template:

**Master template:** `data/master/cover_letter/cover.tex`

## Required form (match cover.tex)

1. **Document class and preamble**
   - `\documentclass[12pt]{letter}`
   - Same packages as in `cover.tex`: `inputenc`, `fullpage`, `hyperref`, `graphicx`, `fontawesome5`, `eso-pic`, `charter`
   - Same layout (topmargin, textheight, `gr` color for header)

2. **Personal and recipient info**
   - `\input{../../master/cover_letter/info}` from the job folder (or equivalent path from `data/jobs/<job-id>/`), then override job-specific macros:
   - `\recipient` (e.g. “Canadian Space Agency Hiring Team”)
   - `\company` if needed
   - Do not change `\myname`, `\myemail`, `\myphone`, `\mylinkedin`, `\mytitle`, `\greeting`, `\closer` unless the user asks.

3. **Header**
   - Shaded banner: `\AddToShipoutPictureBG` with `\color{gr}` and the rule as in `cover.tex`
   - Centered block: `\myname` (large, small caps), then the contact line (see below).
   - **Contact line: use icons, not plain text (mandatory).** Email, LinkedIn, and phone must be displayed **with Font Awesome icons** as in the master template, not as plain text labels like "Email:", "LinkedIn:", "Phone:". Always include the `fontawesome5` package and use:
     - `\href{mailto:\myemail}{\faEnvelope\enspace \myemail}\hfill`
     - `\href{https://linkedin.com/in/\mylinkedin}{\faLinkedinIn\enspace linkedin.com/in/\mylinkedin}\hfill`
     - `\href{tel:\myphone}{\faPhone\enspace \myphone}\hfill`
     Do **not** replace these with plain-text labels; the header must show the envelope icon, LinkedIn icon, and phone icon next to the respective contact info so the PDF matches the intended professional look.

4. **Opening block**
   - Date (e.g. current application date)
   - `\greeting\ \recipient,` in the same font size block (e.g. 14pt)

5. **Body**
   - `\setlength\parindent{24pt}` and `\fontsize{11pt}{18pt}\selectfont`
   - Tailored paragraphs for the specific job (role, company, fit, experience, closing sentence)
   - Mirror tone and structure of `data/master/cover_letter/body.txt`: opening line (application + role + dates), fit with your background, experience evidence (concrete examples), what you bring, languages/bilingual if relevant
   - Cross-check the job description (or `data/jobs/<job-id>/job_posting_extracted.json`): use wording that matches the posting’s requirements and assets without over-claiming (see “Accuracy and fit to the job posting” below).

## Output location (mandatory)

**All job-specific cover letter output must live in the same folder as the job description.**

- Job description lives in: `data/jobs/<job-id>/` (e.g. `description.txt` or `job_posting.pdf`).
- **Write only there:** Put the tailored cover letter LaTeX, body text, and compiled PDF in that same `data/jobs/<job-id>/` folder. Do **not** write job-specific cover letter content or overwrite `data/master/cover_letter/body.tex` or `data/master/cover_letter/info.tex` for a given job.
- **Output naming (mandatory) — same format as tailored resume:** Use the **same naming convention** as in `resume-helper.mdc` for the tailored resume. The only difference is the document type: **Cover** instead of **Resume**.
  - **Short posting name:** Derive it **identically** from the job folder: take the folder name (e.g. `Multi-Static Radar`, `AI and Human Factors`), replace spaces with hyphens, keep it concise (~30 chars max). Examples: `Multi-Static Radar` → `Multi-Static-Radar`; `AI and Human Factors` → `AI-and-Human-Factors`; `CSA/Space Weather` → `CSA-Space-Weather`.
  - **Filenames:** `Timothe-Roma-Cover-{Short posting name}.tex` and `Timothe-Roma-Cover-{Short posting name}.pdf`. So for the same job, the resume is `Timothe-Roma-Resume-Multi-Static-Radar.pdf` and the cover letter is `Timothe-Roma-Cover-Multi-Static-Radar.pdf`.
  - **Do not use** generic names like `cover_letter.tex` or `cover_letter.pdf`; always use the `Timothe-Roma-Cover-{Short posting name}` form.
- **Concrete outputs in the job folder:**
  - `data/jobs/<job-id>/Timothe-Roma-Cover-{Short posting name}.tex` (main letter)
  - Optionally: `data/jobs/<job-id>/cover_body.tex` or `cover_body.txt` for body-only
  - Compiled PDF: `data/jobs/<job-id>/Timothe-Roma-Cover-{Short posting name}.pdf`
- Compile from the job folder: `pdflatex "Timothe-Roma-Cover-{Short posting name}.tex"` run from `data/jobs/<job-id>/` so paths like `\input{../../master/cover_letter/info}` (or `../../../` for nested folders) resolve. The PDF is produced in that same job folder.

## Voice and style (human-written)

Cover letter body text must **sound human-written** and match the writing style of `data/master/cover_letter/body.txt`:

- **Match body.txt style:** Direct first person (“I am submitting”, “I have”, “I would bring”). Short to medium sentences. Concrete details (dates, tools, numbers) without sounding like a list. Natural transitions (“Thanks to these experiences”, “Finally,”).
- **Forbidden (enforce strictly):**
  - **No em dashes (—).** Before finalizing, search the entire body for the character `—` (Unicode U+2014) and replace every occurrence with a comma and space, the word “and”, or a new sentence. Em dashes read as AI-generated and are not allowed.
  - Also avoid: “leveraged”, “utilized”, “seamless”, “robust”, “delve”, “passionate about”, “excited to”, long chains of adjectives, and stiff or overly formal phrasing. Prefer plain, clear wording.
- **Human pass:** Before finalizing, read the body aloud (or in your head) and fix any phrase that sounds generic, padded, or like typical AI prose. Prefer the same level of warmth and specificity as in body.txt.

## Accuracy and fit to the job posting

Tailor claims and wording so the letter matches the posting **without over-claiming**. Use the job description (or `job_posting_extracted.json` in the job folder) as the source of truth.

- **Align with core requirements, but do not overstate:** If the posting asks for experience with AI/ML models *trained* on images/videos (e.g. for artifact detection), and the candidate has **not** trained or fine-tuned such a model, **do not** say their experience “aligns with models trained on images and videos.” Use safer wording instead, e.g. “building image and video pipelines in Python and integrating computer-vision inference into applications aligns closely with the role’s focus on deploying ML for image- and video-based detection.” Only use “trained” or “fine-tuned” if the resume or user confirms it.
- **Only claim what is backed by the resume:** Every concrete skill, library, or outcome (e.g. TensorFlow, PyTorch, OpenCV, precision/recall, labeled data) must appear in the master resume or be confirmed by the user. If the posting cites specific libraries (e.g. TensorFlow, PyTorch, OpenCV), **name only those the candidate has actually used**; do not add libraries that are not on the resume.
- **If the candidate did train or evaluate a model:** Add at most one clear sentence with a measurable detail (e.g. evaluated on labeled clips, tuned thresholds to reduce false positives). Include this **only if** the resume or user confirms it.
- **Map to “assets” when helpful:** When the posting lists assets (e.g. Linux, satellite/space imagery, quality control, specific location or labs), add a short forward-looking sentence that expresses interest in applying the candidate’s skills to those areas or working in that environment, **without inventing experience** (e.g. “I’m particularly interested in applying these techniques to space imagery and quality-control style detection, and I look forward to the opportunity to work in [specific labs/location].”). Prefer “look forward to” over “excited by” to stay within voice rules.
- **Keep sentences readable:** Avoid one long sentence with multiple semicolons or many clauses. Split into two shorter sentences when it improves clarity (e.g. separate “During my co-op at X I did A; in an earlier role I did B” into two sentences).

6. **Closing**
   - Separate line: “Thank you for your time and consideration.” in the same 11pt font
   - `\vfill` then right-aligned: `\closer,` then `\myname` and `\mytitle`

## Where to put job-specific files

- **Per-job folder:** `data/jobs/<job-id>/` (the folder that contains the job description). All cover letter outputs for that job go here only.
- **Tailored cover letter:** `data/jobs/<job-id>/Timothe-Roma-Cover-{Short posting name}.tex`
  - Self-contained or `\input{../../master/cover_letter/info}` (adjust path depth for nested job folders) and optional `\input{cover_body}` for body text.
- **Optional body-only file:** `data/jobs/<job-id>/cover_body.tex` (or `.txt`) if you prefer to keep body text in a separate file; then in the main cover letter use `\input{cover_body}` instead of inlining the body.
- **Compiled PDF:** `data/jobs/<job-id>/Timothe-Roma-Cover-{Short posting name}.pdf`. Do not rely on or write the final PDF into `data/master/cover_letter/` for a job-specific letter.

## Compilation

- Run `pdflatex "Timothe-Roma-Cover-{Short posting name}.tex"` from `data/jobs/<job-id>/` so that `\input{...}` paths to master resolve (e.g. `../../master/cover_letter/info` or `../../../` for nested folders).
- Output: `Timothe-Roma-Cover-{Short posting name}.pdf` in the same job folder (nowhere else).
- **fontawesome5 required:** The template uses `\faEnvelope`, `\faLinkedinIn`, and `\faPhone` for the header. Keep `\usepackage{fontawesome5}` in the preamble. If compilation fails with "Font fa5free2solid not found", run `initexmf --mkmaps` and `initexmf --update-fndb`, then recompile; do not remove the package or switch the contact line to plain text.

## Page length (one page, mandatory)

Cover letters must be **exactly one page**. No content may spill to a second page.

- **Target:** The body, opening, closing, and signature must fit on a single page with the template’s header and layout. Aim for body length that leaves a small margin at the bottom; do not under-fill (e.g. half a page) or over-fill (second page).
- **When creating or editing a cover letter:** After writing or updating the body (in `Timothe-Roma-Cover-{Short posting name}.tex` or `cover_body.tex`), **compile** the PDF, then **check page count**.
- **If the PDF is more than one page:** Shorten the body by:
  - Trimming or merging sentences within paragraphs (keep one strong example instead of two).
  - Removing the least essential paragraph or folding its key point into another paragraph.
  - Shortening clauses and removing filler; keep concrete details (dates, tools, numbers) and direct first-person voice.
  Do **not** reduce font size or change the template’s layout to fit; trim content only. Recompile and re-check until the PDF is exactly one page.
- **After compilation:** Confirm the output is exactly one page. If a second page appears, apply the shortening steps above, then recompile and verify again. Do not deliver or finalize a cover letter that is over one page.
- **Checklist:** Exactly one page; no content on page 2; body fits with the standard template (header, date, greeting, body, thank-you line, signature).

## Summary

- **Output location:** All cover letter files and the compiled PDF for a job go in `data/jobs/<job-id>/` (the folder where the job description is). Do not write job-specific cover letter content into `data/master/cover_letter/`.
- **Structure:** Always use the same structure as `data/master/cover_letter/cover.tex`: letter class, shaded header, date + greeting + recipient, body, thank-you line, signature block.
- **Header contact line:** Email, LinkedIn, and phone must be displayed **with icons** (`\faEnvelope`, `\faLinkedinIn`, `\faPhone`) and the `fontawesome5` package, not as plain text labels. Preserve this in the master template and in any job-specific cover letter.
- **One page:** The cover letter must fit on exactly one page. After compiling, check the PDF; if more than one page, shorten the body (trim sentences, merge or remove a paragraph) and recompile until exactly one page.
- **Files:** Produce the main letter as `data/jobs/<job-id>/Timothe-Roma-Cover-{Short posting name}.tex` (same naming format as the tailored resume in resume-helper.mdc; same short posting name, only Cover vs Resume differs). Body may be inlined or in `cover_body.tex`/`cover_body.txt` in that same folder.
- **No em dashes:** Before delivering, search the body for `—` and replace with comma, “and”, or a new sentence. Do not use em dashes.
- **Accuracy:** Match the posting’s requirements and assets without over-claiming; claims must be resume-backed and wording must align with the JD.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Timber868) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
