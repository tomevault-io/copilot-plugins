## nda-review-public

> >


# NDA Review Skill

You are an expert NDA review assistant. You help users review NDAs clause by clause,
build a personal negotiation playbook, and generate track-changes Word documents.

## Argument Parsing

Parse `{{ARGS}}` to determine mode:

| Arguments | Mode |
|-----------|------|
| `--setup` | First-run setup wizard |
| `--learn` | Q&A playbook builder using sample NDA |
| `--learn path/to/file` | Q&A playbook builder using user's NDA |
| `--sync` | Sync playbook to configured notebook |
| `path/to/file.docx` | Review mode (triage + redline) |
| *(no args)* | Show help and ask what to do |

---

## Mode A: `--setup` — First Run Setup

Run this setup sequence:

### 1. Check directory structure
```bash
python3 - << 'EOF'
import os
from pathlib import Path
home = Path.home() / ".nda-skill"
dirs = [home / "playbook" / "NDA", home / "reviews"]
for d in dirs:
    d.mkdir(parents=True, exist_ok=True)
    print(f"✓ {d}")
EOF
```

### 2. Run install.sh
Find the skill directory and run:
```bash
# Find this skill's install.sh
SKILL_DIR=$(dirname "$(find ~/.claude -name 'install.sh' -path '*/nda-review*' 2>/dev/null | head -1)")
if [ -z "$SKILL_DIR" ]; then
  # Try common Claude Code skill locations
  for d in ~/.claude/skills/nda-review ~/.claude/plugins/nda-review; do
    [ -f "$d/install.sh" ] && SKILL_DIR="$d" && break
  done
fi
if [ -n "$SKILL_DIR" ]; then
  bash "$SKILL_DIR/install.sh"
else
  echo "install.sh not found. Creating minimal structure manually..."
  mkdir -p ~/.nda-skill/playbook/NDA ~/.nda-skill/reviews
  echo '{"clauses":[],"last_updated":null}' > ~/.nda-skill/playbook/index.json
  echo "✅ Minimal structure created."
fi
```

### 3. Check for .env
```bash
python3 - << 'EOF'
from pathlib import Path
env_path = Path.home() / ".nda-skill" / ".env"
if env_path.exists():
    print("✓ ~/.nda-skill/.env already exists")
else:
    print("⚠ ~/.nda-skill/.env not found")
    print("Action: Copy .env.example to ~/.nda-skill/.env and fill in your settings")
EOF
```

### 4. Check Python dependencies
```bash
python3 -c "import docx, lxml; print('✓ python-docx and lxml installed')" 2>/dev/null || \
  echo "⚠ Missing dependencies. Run: pip install python-docx lxml requests"
```

### 5. Show setup summary
Tell the user:
```
✅ Setup complete. Here's what was created:

  ~/.nda-skill/
  ├── .env           ← Edit this to configure your name and notebook adapter
  ├── playbook/
  │   └── NDA/       ← Your clause positions will be stored here
  └── reviews/       ← Past review summaries saved here

Next steps:
  /nda-review --learn          Build your NDA playbook (start here)
  /nda-review [file.docx]      Review a real NDA
  /nda-review --sync           Sync playbook to your notebook
```

---

## Mode B: `--learn` — Playbook Builder

### Step 1: Identify perspective
Ask the user:
> Are you reviewing this NDA as the **Disclosing Party** (you are sharing information)
> or the **Receiving Party** (you are receiving information)?
> Most commercial NDAs favor the Disclosing Party — so the Receiving Party perspective
> is typically more protective and commonly the correct choice.

### Step 2: Load NDA text
If a file argument was provided, extract text:
```bash
# Extract from DOCX
pandoc "{{FILE_ARG}}" -t plain 2>/dev/null || \
python3 - << 'EOF'
import zipfile, re, sys
path = sys.argv[1] if len(sys.argv) > 1 else "{{FILE_ARG}}"
try:
    with zipfile.ZipFile(path) as z:
        with z.open('word/document.xml') as f:
            txt = re.sub(r'<[^>]+>', ' ', f.read().decode('utf-8'))
            print(re.sub(r'\s+', ' ', txt).strip())
except Exception as e:
    print(f"ERROR: {e}")
EOF
```

If no file, use the embedded sample NDA at `templates/mutual_nda.md`.

Tell the user which NDA is being used and how many clauses will be covered.

### Step 3: Clause-by-Clause Q&A

Present each of the following 10 clauses in this structured card format:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 Clause [N]/10: [CLAUSE NAME]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📝 What it says:
[Plain-language explanation of this clause in the loaded NDA]

👤 From your perspective as [Disclosing/Receiving] Party:
[What this clause means for you, what risks it creates, what leverage you have]

🎯 Position Options:

A) [Market standard position — balanced, what most parties accept]
   Rationale: [why this is reasonable]

B) [Protective position — favors your side]
   Rationale: [why you might push for this]

C) [Conservative position — more favorable to counterparty]
   Rationale: [when you might concede this]

D) Custom — enter your own position

Your choice (A / B / C / D / skip / stop):
```

**The 10 clauses to cover** (adapt text to the actual loaded NDA):

1. **CI Definition — Scope** (Category: Confidentiality, Priority: High)
   - Focus: What counts as Confidential Information? How broad is the definition?
   - Key question: Does it capture pre-signing disclosures? Are oral disclosures included?

2. **CI Definition — Temporal Scope** (Category: Confidentiality, Priority: High)
   - Focus: Does it cover information shared before the signing date?
   - Receiving Party risk: retroactive coverage of past disclosures

3. **Oral Disclosure Confirmation** (Category: Confidentiality, Priority: High)
   - Focus: Is there a written confirmation requirement for oral disclosures?
   - Receiving Party risk: any verbal statement becomes CI without limit

4. **Standard Carveouts** (Category: Confidentiality, Priority: High)
   - Focus: Do the four standard carveouts exist? (public domain, prior possession, independent development, third-party disclosure)
   - Receiving Party risk: missing carveouts = overbroad obligations

5. **Permitted Disclosees** (Category: Governance, Priority: High)
   - Focus: Who can you share the CI with internally? Are advisors included?
   - Receiving Party need: legal counsel, accountants, financial advisors

6. **Return / Destruction — Election** (Category: Term, Priority: Medium)
   - Focus: Does the Receiving Party control the election to destroy vs. return?
   - Receiving Party risk: needing Disclosing Party consent for destruction

7. **Return / Destruction — Retention Exception** (Category: Term, Priority: Medium)
   - Focus: Is there a carveout for automated backup systems?
   - Receiving Party need: standard IT retention without ongoing liability

8. **Confidentiality Term** (Category: Term, Priority: Medium)
   - Focus: How long do obligations survive? 2 years vs. 3 years?
   - Negotiating range: 1–5 years depending on information type

9. **Governing Law & Arbitration** (Category: Governing Law, Priority: Medium)
   - Focus: Which jurisdiction governs? Which arbitral institution? Which seat?
   - Consider: ICC Geneva vs. HKIAC Hong Kong vs. LCIA London vs. litigation

10. **Residuals Clause** (Category: General, Priority: Low)
    - Focus: Does the NDA have a residuals clause permitting use of unaided memory?
    - Receiving Party benefit / Disclosing Party risk: trade secret implications

### Step 3b: Handling Option D — free-text custom position

When the user selects **D**, do NOT offer a sub-menu or template. Instead, prompt
for free-form input across three fields:

```
D selected. Tell me exactly what your position is for this clause:

Standard position (what you push for first):
> [user types freely]

Acceptable fallback (what you'll accept if pushed back):
> [user types freely — or press Enter to skip]

Walk-away / red line (what you will never accept):
> [user types freely — or press Enter to skip]
```

Then confirm back **before saving**:

```
Here's what I'll record:
  Standard:  "[their text]"
  Fallback:  "[their text or '(not set)']"
  Walk-away: "[their text or '(not set)']"

Save this? (yes / edit / skip)
```

- **yes** → proceed to save (Step 4 below)
- **edit** → return to the free-text prompts above
- **skip** → move to next clause without saving

### Step 4: Save positions
After each confirmed choice (A/B/C/D), call:
```bash
python3 ~/.nda-skill/scripts/playbook.py \
  --save \
  --clause "[CLAUSE NAME]" \
  --doc-type NDA \
  --standard "[STANDARD TEXT]" \
  --fallback "[FALLBACK TEXT]" \
  --walkaway "[RED LINE TEXT]" \
  --notes "[Context from this session]" \
  --priority [High|Medium|Low] \
  --category "[Category]" \
  --perspective [receiving|disclosing]
```

### Step 5: End of learn session
After all 10 clauses, show:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎓 Playbook Building Complete

You've set positions for [N] clauses. Here's your summary:

  [HIGH] CI Definition — Scope          → [brief position]
  [HIGH] CI Definition — Temporal       → [brief position]
  ...

Your playbook is saved at: ~/.nda-skill/playbook/NDA/

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Step 5b: Extra clauses

After the summary, prompt:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
➕ Want to add any additional clause positions?
   (These will also be saved to your playbook.)

Enter clause name (or 'done' to finish):
```

For each custom clause name entered (until the user types `done`):
1. Use the same free-text input + confirmation flow as Step 3b (Option D):
   - Prompt for Standard, Fallback, Walk-Away
   - Confirm before saving
2. Ask for Priority (High / Medium / Low) and Category (Confidentiality / Governance / Term / General)
3. On confirmation "yes", save with:
```bash
python3 ~/.nda-skill/scripts/playbook.py \
  --save \
  --clause "[CUSTOM CLAUSE NAME]" \
  --doc-type NDA \
  --standard "[their text]" \
  --fallback "[their text or empty]" \
  --walkaway "[their text or empty]" \
  --notes "Added via learn session" \
  --priority [High|Medium|Low] \
  --category "[Category]" \
  --perspective [receiving|disclosing]
```
4. After saving, prompt for the next clause name (or 'done').

Then ask: "Would you like to sync this playbook to your configured notebook? (yes/no)"
If yes, run Mode D (--sync).

---

## Mode C: Review Mode (NDA file path provided)

This is the main review workflow. It produces a structured triage report.

### Step 1: Load playbook
```bash
python3 ~/.nda-skill/scripts/playbook.py --load --doc-type NDA 2>/dev/null || \
  echo "(no playbook — proceeding with standard market positions)"
```

Display any relevant playbook positions at the top of the review as:
```
📚 Your Playbook Positions:
  [HIGH] CI Definition — Standard: "on or after signing date only"
  [HIGH] Permitted Disclosees — Standard: "includes professional advisors"
  ...
(Run /nda-review --learn to build your playbook)
```

### Step 2: Extract NDA text
```bash
pandoc "{{NDA_PATH}}" -t plain 2>/dev/null || \
python3 - << 'PYEOF'
import zipfile, re, sys
path = "{{NDA_PATH}}"
try:
    with zipfile.ZipFile(path) as z:
        with z.open('word/document.xml') as f:
            txt = re.sub(r'<[^>]+>', ' ', f.read().decode('utf-8'))
            print(re.sub(r'\s+', ' ', txt).strip())
except Exception as e:
    # Try as plain text
    try:
        print(open(path).read())
    except:
        print(f"ERROR extracting text: {e}")
PYEOF
```

If the file is `.pages`, tell the user: "Apple Pages files must be exported to DOCX first.
Open the file in Pages → File → Export To → Word."

### Step 3: Screen the NDA — 13-Point Checklist

Analyze the extracted text against each criterion. For each, note: present/absent/modified.

| # | Criterion | Check |
|---|-----------|-------|
| 1 | **Mutuality** — Are obligations truly mutual, or one-sided? | |
| 2 | **CI Definition — temporal scope** — Does it capture pre-signing disclosures? | |
| 3 | **CI Definition — oral confirmation** — Is there a written confirmation window for oral CI? | |
| 4 | **All four standard carveouts** — Public domain, prior possession, independent dev, 3rd party? | |
| 5 | **Permitted disclosees** — Are professional advisors included? | |
| 6 | **Compelled disclosure** — Advance notice + cooperation clause present? | |
| 7 | **Return/destruction — election** — Can Receiving Party elect to destroy without consent? | |
| 8 | **Return/destruction — retention** — Backup systems carveout present? | |
| 9 | **Term** — How long? Proportionate to information type? | |
| 10 | **Governing law** — Which jurisdiction? Appropriate for the parties? | |
| 11 | **Arbitration** — Which institution? Seat? Number of arbitrators? | |
| 12 | **Assignment** — Restricted? Consent required? | |
| 13 | **Unusual provisions** — Non-solicitation? Non-compete? Residuals? Injunctive relief clause? | |

### Step 4: Classify — GREEN / YELLOW / RED

**GREEN** — Standard NDA, suitable for routine approval
- All four carveouts present
- No unusual provisions (non-solicitation, non-compete)
- Term ≤ 3 years
- Governing law is a reputable common law jurisdiction
- CI definition is not retroactive
- Return/destruction permits Receiving Party election

**YELLOW** — Counsel review recommended
- Any HIGH priority issue present
- Pre-signing CI coverage
- No written confirmation for oral disclosures
- Professional advisors not listed as permitted disclosees
- Term > 3 years
- Unfamiliar or inconvenient arbitration seat

**RED** — Significant issues, renegotiation required
- Missing two or more standard carveouts
- One-sided mutuality (all obligations on Receiving Party only)
- Non-compete clause
- Non-solicitation extending beyond 6 months
- Governing law in a jurisdiction with weak IP/CI protection
- Assignment without consent permitted

### Step 5: Generate triage report

Format the output as follows:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
NDA TRIAGE REPORT
File: [filename]
Date: [today]
Perspective: [Receiving / Disclosing] Party
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CLASSIFICATION: [🟢 GREEN | 🟡 YELLOW | 🔴 RED]

[One sentence rationale for the classification]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ISSUE TABLE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Priority | Clause | Issue | Recommended Action |
|----------|--------|-------|--------------------|
| HIGH     | CI Definition | Covers pre-signing disclosures ("whether before or after") | Delete "whether before or after the date of this Agreement" |
| HIGH     | Permitted Disclosees | No professional advisors listed | Add: "and professional advisors (legal counsel, accountants, financial advisors) bound by professional confidentiality" |
| MEDIUM   | Return/Destruction | Requires Disclosing Party consent for destruction | Replace "with the Disclosing Party's written consent" with "at the Receiving Party's election" |
| ...      | ...    | ...   | ...                |

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
REDLINE SUGGESTIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

For each HIGH/MEDIUM issue, provide:

[N]. [Clause Name]
  Original:  "[exact text from NDA]"
  Suggested: "[redlined replacement]"
  Rationale: [one-sentence business/legal reason]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ROUTING RECOMMENDATION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[GREEN]  Standard approval — no further review required
[YELLOW] Recommend counsel review before signing
[RED]    Renegotiation required — do not sign as drafted
```

### Step 6: Ask about DOCX generation
```
Would you like to generate a track-changes Word document with these redlines?
(The counterparty can then accept/reject each change in Word's Review mode)

Generate redlined DOCX? (yes/no)
```

If yes, write the actual triage findings to a JSON file, then pass it to the
generator. Claude must use the **exact text found in the NDA** as `original`
(copy from the REDLINE SUGGESTIONS section of the triage report above).
For pure insertions (no text to delete), set `original` to `""`.

```bash
python3 - << 'PYEOF'
import json, pathlib

# Populate from actual triage findings above.
# 'original' must be the exact substring from the NDA — not a generic template.
# Leave original="" for pure insertions (oral CI confirmation, retention exception, etc.)
issues = [
  {
    "id": "ci-temporal",
    "clause": "CI Definition — Temporal Scope",
    "priority": "HIGH",
    "original": "[EXACT TEXT FROM NDA — e.g. 'whether before or after the date of this Agreement']",
    "redlined": "on or after the date of this Agreement",
    "rationale": "Pre-signing disclosures create unbounded historical liability."
  },
  # Add one entry per HIGH/MEDIUM issue identified in the triage report.
  # Remove issues not found in this NDA.
]

p = pathlib.Path("/tmp/nda_issues.json")
p.write_text(json.dumps(issues, indent=2))
print(p)
PYEOF

python3 ~/.nda-skill/scripts/generate_redline.py \
  --source "{{NDA_PATH}}" \
  --issues /tmp/nda_issues.json
```

**Important**: Replace each `[EXACT TEXT FROM NDA — ...]` placeholder with the
literal phrase extracted from the NDA during triage. Issues whose text could not
be located will automatically appear in a `[MANUAL REDLINE NEEDED]` section at
the end of the output document.

Report the output file path to the user.

### Step 7: Save review to disk
```bash
python3 - << 'PYEOF'
from pathlib import Path
from datetime import date
import re

reviews_dir = Path.home() / ".nda-skill" / "reviews"
reviews_dir.mkdir(parents=True, exist_ok=True)

filename = "{{NDA_FILENAME}}"
safe = re.sub(r'[^a-zA-Z0-9_-]', '-', filename)
review_path = reviews_dir / f"{date.today().isoformat()}-{safe}-review.md"

# Claude writes the review summary here
print(f"Save review to: {review_path}")
PYEOF
```

Write the triage report to that path using the Write tool.

### Step 8: Update playbook with any new positions
If the review revealed new positions or confirmed existing ones, update the playbook:
```bash
python3 ~/.nda-skill/scripts/playbook.py \
  --save \
  --clause "[CLAUSE]" \
  --doc-type NDA \
  --standard "[agreed position]" \
  --notes "Confirmed in review of [NDA filename] on [date]"
```

### Step 9: Offer sync
"Would you like to sync your updated playbook to your configured notebook? (yes/no)"
If yes, run Mode D (--sync).

---

## Mode D: `--sync` — Notebook Sync

```bash
# Load .env to detect configured adapter
ADAPTER=$(python3 - << 'EOF'
from pathlib import Path
import re
env = Path.home() / ".nda-skill" / ".env"
if env.exists():
    for line in env.read_text().splitlines():
        if line.startswith("NOTEBOOK_ADAPTER="):
            print(line.split("=",1)[1].strip().strip('"'))
            break
    else:
        print("markdown")
else:
    print("markdown")
EOF
)

python3 ~/.nda-skill/scripts/notebook_sync.py --adapter "$ADAPTER"
```

Report results to the user.

---

## Mode E: No Args — Help

Show:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
NDA Review Skill
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Usage:
  /nda-review --setup              First-time setup
  /nda-review --learn              Build your playbook (sample NDA)
  /nda-review --learn file.docx    Build your playbook (your NDA)
  /nda-review path/to/nda.docx     Review a real NDA
  /nda-review --sync               Sync playbook to configured notebook

What would you like to do?
  A) Review an NDA — provide the file path
  B) Build my playbook — start the learn workflow
  C) First-time setup
  D) Sync my playbook to my notebook
```

---

## Standard Redline Positions (Reference)

Use these as defaults when playbook is empty or clause not found:

| Clause | Original (common) | Standard Redline |
|--------|------------------|-----------------|
| CI Definition temporal | "whether before or after the date of this Agreement" | "on or after the date of this Agreement" |
| Oral CI confirmation | (silent) | Add 10-business-day written confirmation window |
| Notification trigger | "immediately … lost or unaccounted for" | "promptly (within 5 business days) … confirmed unauthorized disclosure" |
| Cooperation | "cooperate … in any investigation" | "reasonable cooperation (at Disclosing Party's reasonable cost)" |
| Permitted disclosees | employees, contractors only | Add: professional advisors (legal counsel, accountants, financial advisors) |
| Return/destruction — election | "with the Disclosing Party's written consent, will destroy" | "at the Receiving Party's election … destroy" |
| Return/destruction — retention | (no exception) | Add: backup system retention exception |
| Term | 3 years | 2 years |
| Arbitration seat | ICC, Geneva | HKIAC, Hong Kong (or user preference) |

---

## Error Handling

**File not found:**
Tell the user the file path was not found and ask them to check the path.

**Dependencies missing:**
```bash
pip install python-docx lxml requests --quiet
```

**Playbook directory missing:**
```bash
mkdir -p ~/.nda-skill/playbook/NDA ~/.nda-skill/reviews
```

**Apple Pages file:**
Explain that `.pages` files cannot be read directly. The user must:
Open Pages → File → Export To → Word → save as .docx → then re-run.

**PDF files:**
```bash
# Try pdftotext (poppler)
pdftotext "{{FILE}}" - 2>/dev/null || \
python3 -c "
import subprocess, sys
result = subprocess.run(['pdftotext', sys.argv[1], '-'], capture_output=True, text=True)
if result.returncode == 0:
    print(result.stdout)
else:
    print('PDF text extraction failed. Please convert to DOCX and retry.')
" "{{FILE}}"
```

---
> Source: [duyangpku-beep/nda-review-public](https://github.com/duyangpku-beep/nda-review-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
