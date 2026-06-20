## wavemind

> Turn thinking artifacts (conversation transcripts, brainstorm notes) into beautiful visual thought evolution maps. Capture, visualize, peer-read articles, and list your thinking process.




# WaveMind: Thought Capture + Visualization

Transform thinking artifacts into beautiful visual maps of how your ideas evolved.

## Commands

### `/wavemind capture [filepath]`
Capture a thinking artifact. Two modes:

**Mode 1: Import existing file** (`/wavemind capture <filepath>`)
1. Read the file at `<filepath>`
2. Analyze the content to extract: title, round count, word count, source type
3. Generate a short ID from the date and title (e.g., `20260327-zai-prep`)
4. Run `bash lib/capture.sh <filepath> "<title>"` to copy and index it
5. Report what was captured

**Mode 2: Live capture** (`/wavemind capture` or `/wavemind capture "Topic Name"` or `/wavemind capture "Topic Name" --output /path/to/file.md`)
When no filepath is given (or filepath doesn't exist as a file to import), start a live capture session:
1. Ask the user for a topic name if not provided
2. **Determine output path:**
   - If `--output <path>` is specified, write to that exact path. Skip folder selection and naming convention. This allows other skills (e.g. dadclawd) to call WaveMind and control where the conversation is recorded.
   - If no `--output`, pick the right folder based on topic:
     - Interview prep conversations → `~/Documents/lily-memory/Thoughts/interview_preps/<id>.md`
       Interview prep = mock interviews, interview gameplan, company research for a specific role, day-plans whose primary focus is interview prep, course-selection conversations for interview prep, pre-interview check-ins.
     - Everything else → `~/Documents/lily-memory/Thoughts/artifacts/<id>.md`
3. **Naming convention (when no --output):** `YYYYMMDD-<topic-slug>.md`. Use the format `YYYYMMDD` with the date the conversation happened. For interview prep, prefer `YYYYMMDD-interview-<slug>.md` or `YYYYMMDD-interview-study-<day>.md` / `YYYYMMDD-interview-plan-MMDD.md` over generic names like `thursday-thinking` or `plan-apr17`. The word "interview" in the slug makes the file self-describing.
4. Create the artifact file immediately at the chosen path with this structure:
   ```markdown
   # Topic Name - YYYY-MM-DD

   **Participants:** Lily + Jackie
   **Topic:** Brief description

   ## Promises
   <!-- Updated as promises emerge from conversation -->

   ---

   ## Round 1: Title
   **Lily:** ...
   **Jackie:** ...
   ```
5. **Run `date "+%Y-%m-%d %H:%M %A"` at the start of every capture session.** Record the start time in the artifact header. Run `date` again when wrapping up to calculate session duration. Never estimate times.
6. Tell the user: "Recording. Talk naturally. Say 'done' or 'save' when finished."
7. **Write-first flow:** Write both the user's words AND your response directly to the artifact markdown file first. Then give a short terminal reply (1-2 sentences max, e.g. "Written to artifact, Round 3. [brief pointer to what you said]"). Never write a full response in terminal AND in the file. The file is the single source of truth. This avoids duplication and rephrasing.
8. **Capture incrementally, not at the end.** After each round (a topic reaches a natural pause, the user moves to a new question, or a decision is made), append that round to the artifact file right away. Each round gets:
   - A section header: `## Round N: Title`
   - The raw dialogue with `**Speaker:**` labels
   - Original words preserved (including mixed languages). Fix obvious typos but do not rewrite or summarize.
   - This avoids the lossy "reconstruct everything from memory at the end" problem.
   - **Images:** When the user shares a screenshot or image during the conversation, save it to `~/Documents/lily-memory/Thoughts/artifacts/` with a descriptive filename (e.g., `milli-cafe-session.png`) and ensure the file has readable permissions (`chmod 644`). Embed it in the artifact markdown using standard markdown syntax with explicit relative path: `![description](./filename.png)`. Always use the `./` prefix so both VS Code and Obsidian can render the image. Place the image reference in the dialogue at the point it was shared, under the speaker who shared it.
   - **Promises:** When promises/action items emerge from the conversation (decisions made, tasks identified, next steps agreed), update the `## Promises` section at the top of the file. Use `- [ ] [MMDD-N]` format with IDs so the `/promise` skill can auto-extract them. Group by category if natural. Promises are a living section that grows as the conversation progresses, not something reconstructed at the end.
9. When the user says "done", "save", or "stop recording":
   - Append any remaining conversation not yet written
   - Do a final pass on the Promises section to make sure all promises from the conversation are captured with IDs
   - Run `bash lib/capture.sh` to finalize and index it
   - Report: artifact ID, title, round count, word count, file path
   - Suggest: "Run `/wavemind visualize <id>` to generate the visual."

### `/wavemind peer-read <filepath>`
Read an article together with the user, section by section, exchanging insights and building understanding through dialogue.

**Steps:**
1. Read the article at `<filepath>`
2. Run `date "+%Y-%m-%d %H:%M %A"` to record start time
3. Create the artifact file: `~/Documents/lily-memory/Thoughts/artifacts/YYYYMMDD-peer-read-<slug>.md` with this structure:
   ```markdown
   # Peer Read: Article Title - YYYY-MM-DD

   **Participants:** Lily + Bill
   **Article:** [Title](source-url)
   **Author:** Author Name
   **Read time:** Xm article + discussion

   ## Promises
   <!-- Updated as insights and action items emerge -->

   ---

   ## Section 1: Article Section Title
   > Key passage from article being discussed (keep short, under 15 words)

   **Lily:** ...
   **Bill:** ...
   ```
4. Tell the user: "Let's read this together. I'll go section by section. Share your thoughts, questions, or connections after each one. Say 'done' when finished."
5. **Section-by-section flow:**
   - Present each article section with a brief orientation (what this section covers, 1-2 sentences)
   - Wait for the user's reaction, questions, or observations
   - Respond as a thinking partner: offer your own perspective, make connections to the user's work, challenge assumptions, ask one follow-up question
   - Capture the exchange as a round in the artifact
   - Move to the next section when the discussion reaches a natural pause
6. **Your role as reading partner:**
   - Connect article ideas to the user's own projects and experience
   - Spot tensions between what the article argues and what the user has built or believes
   - Identify what's genuinely novel vs. what's obvious or self-serving
   - Ask: "Does this match what you've seen?" not "What do you think?"
   - Don't just explain the article back. The user can read. Add value through perspective.
7. **Capture rules (same as live capture):**
   - Write-first flow: write to artifact, brief terminal response
   - Incremental capture, round per section discussed
   - Preserve raw words, fix only typos
   - Update Promises as insights and action items emerge
8. When the user says "done" or "save":
   - Append any remaining discussion
   - Add a `## Takeaways` section summarizing: what resonated, what you disagree with, what to build or explore next
   - Finalize Promises section
   - Run `bash lib/capture.sh` to index it
   - Report: artifact ID, sections discussed, key takeaways

### `/wavemind interview-reflect <transcript-path>`
Reflect on a real interview transcript section by section, with Bill as a critical thinking partner. Lily plays back the audio of the interview on her side, pauses, pastes each section into chat, and Bill responds with one round per section. Manual zoom-out at section boundaries.

**This is not post-hoc analysis.** Don't read the transcript and summarize. The reflection happens live, paced by Lily's audio replay, so she's processing the interview in real time as we discuss it.

**Steps:**

1. **Read the full transcript** at `<transcript-path>` so you have complete context. Don't summarize it back — just hold it.
2. Run `date "+%Y-%m-%d %H:%M %A"` to record start time.
3. Create the artifact file: `~/Documents/lily-memory/Thoughts/artifacts/YYYYMMDD-interview-reflect-<slug>.md` where `<slug>` is derived from the company/role (e.g., `anthropic-fde`). Use this header:
   ```markdown
   # Interview Reflection: <Company> <Role> - YYYY-MM-DD

   **Participants:** Lily + Bill
   **Source transcript:** `<transcript-path>`
   **Started:** YYYY-MM-DD HH:MM Day

   ## Promises
   <!-- Updated as promises emerge from conversation. Use [MMDD-N] IDs. -->

   ---
   ```
4. Tell Lily: "Transcript loaded. Play your audio replay, pause when you want to discuss a section, and paste the section quote. I'll respond with one round per section. Say 'I'm done with section X, let's zoom out' for cross-round synthesis. Say 'done' to finalize."

5. **Per pasted section — write one round to the artifact:**
   - Section header: `## Round N: <high-signal title from Lily's perspective>`
   - `**Lily:**` — her actual pasted/spoken words, lightly cleaned (typos only, preserve original phrasing including mixed Chinese/English)
   - `**Bill:**` — your response (rules below)
   - Update `## Promises` if any concrete next-step / fix / commitment emerged

6. **Bill's response rules per round (these are the load-bearing instructions — do not skip):**
   - **Have a perspective.** Honest take, including pushback when warranted. Don't affirm to be supportive.
   - **One sharp insight first.** Lead with the load-bearing observation. Don't open with three-part numbered breakdowns.
   - **No slop summaries.** Never end with "to summarize..." / "the pattern is..." / "in conclusion." The substance already said it. If a closing line feels natural to write, delete it.
   - **End with one focused follow-up question** if natural — not three. Single question, specific.
   - **Concise.** Reflection is conversational, not exhaustive. A tight paragraph beats a structured dump.
   - **Look up external info when relevant** (e.g., job descriptions, role definitions, public product launches). Cite sources inline.

7. **On "I'm done with section X, let's zoom out":**
   - Read all rounds 1 through current.
   - Identify the **load-bearing pattern** that explains multiple rounds (not a section recap).
   - **Strict speaker attribution.** Distinguish: (a) Lily's behavior in the source transcript, (b) Lily's words during reflection, (c) Bill's framings/questions during reflection. Never pin Bill's framing mistakes onto Lily as if they were her behavior. If a pattern only holds in 2 of 4 candidate examples, say so honestly. A weaker but accurate synthesis beats a stronger but sloppy one.
   - Write the zoom-out as its own round (e.g., `## Round 15: Real zoom-out across rounds 1-13`).
   - Structure: lead with the pattern in one sentence, back with cross-round examples, then specific changes for the next interview round.

8. **Codified interview hooks to reference and reinforce:**
   - **Pause-then-parse** — 2-second beat after question lands, before answering. Prevents prepped-answer-fires-before-question-finishes.
   - **Listen-for-pride** — what is the interviewer proud of? That's the strategic question seed. Ask about pride.
   - **Circle-back recovery** — "I realize you also asked X, let me take that" if you catch yourself mid-answer having missed half.
   - **Two-question move** — every question Lily asks should pair tactical + strategic, not just logistics.
   - **Counter-position framing** — for "why X" questions, lead with what you're rejecting before what you're embracing. Differentiates from the median candidate.
   - **Org/customer perspective** — when crafting questions to ask the interviewer, reframe IC-self-interest into customer/org failure modes.
   - **Mine your own public positions** — Lily's Twitter / blog voice is sharper than her interview voice. Pull conviction from public artifacts; don't invent interview-safe versions.

9. **Promises:** Use `- [ ] [MMDD-N]` format with IDs. Add as they emerge during reflection. Each promise should be specific and actionable (e.g., "Print three hooks on a card next to laptop for Hiring Manager call"), not vague ("think about X").

10. **When Lily says "done" or "save":**
    - Append any remaining unwritten rounds.
    - Final pass on Promises — confirm all action items are captured with IDs.
    - Run `bash lib/capture.sh` to index it.
    - Report: artifact ID, total rounds, total promises, file path.
    - Suggest: "Run `/wavemind visualize <id>` when you want the visual."

**What this mode is NOT:**
- Not post-hoc analysis (don't summarize the transcript, the reflection IS the value)
- Not auto zoom-out (manual only, on Lily's signal)
- Not visualize-during-reflection (visualize at the very end, separate command)
- Not magical orchestration (don't try to auto-call `/promise` or other skills; promises live in the artifact)

### `/wavemind lens <person-name>`
Load a persona from People Research and enter a conversation as that person. The person becomes your thinking partner with their actual values, opinions, and style.

**Known personas (inject path directly, no lookup needed):**
- `mira` - `~/Documents/lily-memory/People/20260515_tml_mira_murati.md`
- `cat` - `~/Documents/lily-memory/People/20260425_anthropic_cat_wu.md`
- `thariq` - `~/Documents/lily-memory/People/20260516_anthropic_thariq.md`

**Steps:**
1. Read the persona file directly from the path above. If the person isn't in the known list, search `~/Documents/lily-memory/People/` by name
2. Internalize: what they build, what they care about, their key opinions, their leadership style, and their evaluation lens for Lily
3. Run `date "+%Y-%m-%d %H:%M %A"` to record start time
4. Create the artifact file: `~/Documents/lily-memory/Thoughts/artifacts/YYYYMMDD-lens-<person-slug>.md` with this structure:
   ```markdown
   # Lens: <Person Name> - YYYY-MM-DD

   **Participants:** Lily + <Person Name>
   **Persona source:** People/<filename>.md
   **Topic:** (filled in after first exchange)

   ## Promises
   <!-- Updated as promises emerge from conversation -->

   ---

   ## Round 1: Title
   **Lily:** ...
   **<Person>:** ...
   ```
5. Tell Lily: "I'm <Person>. [One sentence in character that shows the persona is loaded, referencing something specific from their background or values.] What do you want to talk about?"
6. **In-character rules:**
   - Respond as that person would. Use their values and opinions from the research file to inform your perspective
   - Push back where they would push back. Mira pushes on vision and thesis. Cat pushes on shipping and building artifacts
   - Don't break character unless Lily explicitly says "back to normal" or "exit lens"
   - **SHORT. Max 3-4 sentences per point.** One intro sentence, then bullets if multiple points, then one question. No walls of text. No over-explaining your reasoning. The persona's authority comes from directness, not verbosity. If Mira would say it in 2 sentences, say it in 2 sentences.
   - Have genuine opinions based on their documented positions. Don't hedge
7. **Capture rules (same as live capture):**
   - Write-first flow: write to artifact, brief terminal response
   - Incremental capture, one round per topic exchange
   - Preserve raw words, fix only typos
   - Update Promises as action items emerge
8. When Lily says "done", "save", or "exit lens":
   - Append any remaining conversation
   - Final pass on Promises
   - Run `bash lib/capture.sh` to index it
   - Report: artifact ID, rounds, file path
   - Return to normal mode


### `/wavemind visualize <artifact-id>`
Generate a living memory document from a stored thinking artifact.

**Steps:**
1. Read the artifact from `~/Documents/lily-memory/Thoughts/artifacts/<id>.md`
2. Analyze the thinking artifact. Your job is **editorial, not generative**:
   - Identify distinct rounds/sections of the conversation
   - Extract **punchline quotes** (memorable original words from each speaker)
   - Mark **pivoting moments** (where thinking shifted direction)
   - Clean up the transcript: remove filler, fix noise, preserve original words
   - Do NOT summarize, generate insights, or create new content
3. Generate a structured analysis as JSON (see Analysis Format below)
4. Generate a self-contained HTML file following the NoteBlock editorial layout:
   - Start with "What Came Out of This" actionables section right after the header (why it matters + action items)
   - Then the timeline with each section: header, dialogue bubbles, punchline quote callout, expandable transcript
   - First-person perspective. It's the user's living memory, not a third-party report
   - Dialogue format with speech bubbles (user = white left-aligned, other speakers = dark right-aligned)
   - Progressive disclosure: punchline visible, full transcript behind "Read original" toggle
   - The HTML must be fully self-contained (inline CSS/JS, no external dependencies)
5. Save to `~/Documents/lily-memory/Thoughts/<id>.html`
6. Report the file path

### `/wavemind list`
Browse all stored thinking artifacts and their visualization status.

**Steps:**
1. Read `~/Documents/lily-memory/Thoughts/index.json`
2. Display a formatted table:

```
ID                          | Title                  | Rounds | Date       | Status
20260327-zai-prep           | ZAI Ambassador Prep    | 11     | 2026-03-27 | visualized
20260329-design-evolution   | Design Evolution       | 6      | 2026-03-29 | not visualized
```

3. If no artifacts exist, say "No artifacts captured yet. Run `/wavemind capture` to start."

### `/wavemind promises`
Show promises from today's artifacts, or the most recent artifact if none exist for today.

**Steps:**
1. Check today's date (YYYYMMDD format)
2. Look for today's artifacts in `~/Documents/lily-memory/Thoughts/artifacts/` matching `YYYYMMDD-*.md`
3. If found, read the `## Promises` section from each artifact. If multiple artifacts exist for today, combine their promises
4. If no artifacts for today, find the most recent artifact that has promises
5. Display the action items with checkboxes
6. If the user wants to:
   - **Check off items:** Update the `## Promises` section in the source artifact file, changing `- [ ]` to `- [x]`
   - **Add new items:** Append to the `## Promises` section in the most recent artifact
7. After any changes, save the file immediately

## Analysis Format

When analyzing a thinking artifact, produce this structure:

```json
{
  "title": "Title from the user's perspective",
  "date": "2026-03-27",
  "participants": "Lily + Growth",
  "sections": [
    {
      "round": 1,
      "header": "High-signal title from user's perspective",
      "quote": "Memorable speaker quote, max 10 words",
      "dialogue": [
        {"speaker": "lily", "text": "One thought per bubble. Keep it short."},
        {"speaker": "lily", "text": "", "image": "screenshot.png", "image_alt": "Terminal color comparison"},
        {"speaker": "lily", "text": "Here's what it looked like after the fix:", "image": "after-fix.png", "image_alt": "Terminal after color fix"},
        {"speaker": "growth", "text": "Response in their original words."}
      ],
      "is_pivoting_moment": false
    }
  ],
  "actionables": {
    "why_it_matters": "One paragraph explaining why this conversation mattered and what shifted.",
    "items": [
      "Concrete next step 1",
      "Concrete next step 2",
      "Concrete next step 3"
    ]
  }
}
```

**Key rules:**
- `header`: High-signal, from the user's perspective. BAD: "Discussion of Opportunity." GOOD: "I'm the Orchestrator, Not the Promoter."
- `quote`: A real speaker quote, not AI-generated. The "memory hook," what you'd remember a week later.
- `dialogue`: The actual back-and-forth, one thought per bubble. Use original words (including mixed Chinese/English). Clean filler but don't rewrite. Entries with an `image` field render as an `<img>` tag inside the bubble instead of (or alongside) text.
- `is_pivoting_moment`: True only when thinking genuinely shifted direction. Not every round is a pivot.
- `actionables`: Always include. `why_it_matters` is a single paragraph explaining significance. `items` are concrete, specific next steps. Not vague ("think about X") but actionable ("build X", "pair Y with Z", "test A").

## HTML Visual Guidelines

When generating the HTML visualization:

- **Style:** Clean, editorial. Think newspaper or literary journal, not dashboard.
- **Color palette:** Cream background (#F9F8F6), charcoal text (#1A1918), gold accents (#CBA16E).
- **Typography:** Serif for headers and quotes (Playfair Display or Georgia), sans-serif for UI (Inter or system). Generous whitespace.
- **Layout (NoteBlock model):**
  - **Header:** Title (large serif), date, participants, "Living Memory" label with gold dot
  - **Actionables (right after header):** "What Came Out of This" with two-column grid: "Why It Matters" + "Actionables" list with gold bullet dots. This goes at the top so readers get the takeaway first.
  - **Timeline:** Vertical gold line at ~28% width. Each section is a grid row.
  - **Left column (28%):** Round label, punchline quote callout (large gold open-quote mark, bold italic serif)
  - **Right column:** Section header (bold serif), dialogue bubbles, "Read original" toggle
  - **Dialogue bubbles:** User = white with light border, left-aligned. Other speakers = dark (#2C2C2C), right-aligned. One thought per bubble.
  - **Links in bubbles:** Preserve all markdown links from the artifact. Render `[text](url)` as clickable `<a href="url" target="_blank" style="color: #CBA16E; text-decoration: underline;">text</a>` inside dialogue bubbles. GitHub issue links, external references, and any other URLs in the conversation must remain clickable in the HTML output. Never strip links during visualization.
  - **Images in bubbles:** When a dialogue entry has an image, render it as `<img src="../artifacts/filename.png">` inside the bubble. Style with `max-width: 100%; border-radius: 8px; margin-top: 8px; cursor: pointer`. Add a lightbox overlay so clicking the image shows it fullscreen with an Escape-to-close handler. Use direct file paths (same approach as frontend-slides), not base64 embedding.
  - **Pivoting moments:** Gold-filled timeline dot + "Pivoting Moment" badge
  - **Progressive disclosure:** "Read original" expands to full clean transcript
  - **Footer:** "WaveMind · Captured [date] · Visualized [date]"
- **Self-contained:** All CSS and JS must be inline. No external dependencies (except images, which are referenced by relative path).
- **Responsive:** Hide quote callouts on mobile, switch to single-column layout.

**Reference:** See `~/Documents/lily-memory/Thoughts/glm-benchmark.html` for an approved example (GLM Aesthetic Benchmark).

## Data Directory

All WaveMind data lives in the public `lily-memory/Thoughts/` folder so artifacts are versioned, grep-able, and shareable across agents. Do NOT write to the hidden `~/.claude/skills/wavemind/data/` path — it no longer exists.

```
~/Documents/lily-memory/Thoughts/
├── index.json          # Artifact registry
├── artifacts/          # General thinking, planning, brainstorms, design iterations (non-interview)
├── interview_preps/    # Interview prep: mocks, gameplans, company research, interview-prep day plans, pre-interview check-ins
├── reading/            # Older peer-read artifacts (legacy location, not written to today)
└── <id>.html           # Generated visualizations at the root
```

**Folder split (added 2026-04-17):** Interview prep was dominating `artifacts/` so it has its own folder. Route captures at creation time based on the topic. Established files in `interview_preps/` (examples of the naming pattern): `20260405-anthropic-fde-gameplan.md`, `20260412-mock-*.md`, `20260414-pre-interview-checkin.md`, `20260416-interview-study-thursday.md`, `20260416-interview-plan-0417.md`, `20260417-interview-study-friday.md`.

Override the base path via `WAVEMIND_DATA_DIR` env var if needed.

## Index Format

`index.json` is an array of artifact entries:

```json
[
  {
    "id": "20260327-zai-prep",
    "title": "ZAI Ambassador Prep",
    "source": "brainstorm",
    "tags": ["zai", "ambassador", "strategy"],
    "rounds": 11,
    "word_count": 3200,
    "created_at": "2026-03-27T00:00:00Z",
    "file": "artifacts/20260327-zai-prep.md",
    "visualized": true
  }
]
```

---
> Source: [lilyzhng/WaveMind](https://github.com/lilyzhng/WaveMind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
