## ghostai

> GhostAI is a suite of Claude Code skills for book authors on Leanpub. It operates

# GhostAI

GhostAI is a suite of Claude Code skills for book authors on Leanpub. It operates
at the filesystem level, reading your full manuscript structure, maintaining context
across chapters, and editing in place.

## Skills

| Skill | Purpose | When to use |
|-------|---------|-------------|
| `/ghost-start` | Book onboarding wizard | Starting a new book from scratch |
| `/ghost-outline` | Structured outline builder | Planning or restructuring chapters |
| `/ghost-review` | Development editor | After writing 3+ chapters, for structural feedback |
| `/ghost-edit` | Copy editor + pre-flight | Before publishing any chapter |
| `/ghost-draft` | First draft generator | Turning an outline into prose |
| `/ghost-expand` | Bullet-to-prose expander | Turning notes into polished text |
| `/ghost-interview` | Interview-driven writing | When you know the material but can't get it on the page in your voice |
| `/ghost-rewrite` | Section-level structural revision | When a passage exists but the staging is wrong (POV, order, framing) |
| `/ghost-status` | Manuscript dashboard | Checking progress anytime |
| `/ghost-train` | Author self-assessment | Teaching Ghost your strengths, weaknesses, and stylistic dislikes |
| `/ghost-voice` | Voice profile manager | Updating, promoting, or sharing your voice profile across tiers |
| `/ghost-update` | Update GhostAI | Updating to the latest version |

## Manuscript Format

GhostAI expects Leanpub-style manuscripts in Markua format, following the
canonical Leanpub directory layout (see the default book template at
https://github.com/leanpub/default-new-book-content):

```
manuscript/
  Book.txt        — chapter manifest (one filename per line)
  *.md            — chapters in Markua (Markdown for books)
  resources/      — images and code samples (referenced by bare filename)
```

All manuscript files live inside the `manuscript/` directory. `/ghost-start`
scaffolds this structure automatically. The preamble also detects `Book.txt`
at the repo root for backwards compatibility with older projects.

Free sample chapters are marked with `{sample: true}` above the chapter
heading — there is no separate Sample.txt file. Resources (images, code
files) are referenced by bare filename (e.g., `![](photo.png)`) and Leanpub
resolves them from `manuscript/resources/`.

## Persistence

GhostAI looks for each config in three tiers (highest existing wins, no merging):

1. **Global** — `~/.ghostai/<file>` (machine-wide default)
2. **Project-local** — `~/.ghostai/projects/{slug}/<file>` (per-book on this machine)
3. **In-repo** — `{repo}/.ghostai/<file>` (committable alongside the manuscript)

The configs are:
- `voice-profile.json` — extracted writing style characteristics
- `style-guide.md` — documented style preferences
- `learnings.jsonl` — terminology and style decisions from past sessions
- `reviews/` — saved review reports

When creating a new voice profile or style guide, GhostAI asks which tier to
save to. Subsequent silent writes (learnings, reviews) follow the same tier
via `~/.ghostai/projects/{slug}/.tier`. See `shared/config-hierarchy.md` for
the full model and the recommended `.gitignore` for in-repo configs.

## Skill Routing

When the user's request matches an available skill, ALWAYS invoke it using the Skill
tool as your FIRST action. Do NOT answer directly, do NOT use other tools first.

Key routing rules:
- "Start a book", "new book", "set up my manuscript" → invoke ghost-start
- "Outline", "plan chapters", "restructure" → invoke ghost-outline
- "Review my book", "structural feedback", "cross-chapter" → invoke ghost-review
- "Edit", "proofread", "check my chapter", "before I publish" → invoke ghost-edit
- "Write a draft", "generate a chapter", "draft from outline" → invoke ghost-draft
- "Expand", "flesh out", "turn bullets into prose" → invoke ghost-expand
- "Interview me", "talk through a chapter", "let me explain this out loud", "this doesn't sound like me" → invoke ghost-interview
- "Rewrite", "rejig", "this section feels awkward", "the structure is off", "I can't tell who's speaking", "the framing is wrong", "doesn't land", "buried lead" → invoke ghost-rewrite
- "Status", "progress", "how's my book", "word count" → invoke ghost-status
- "Train ghost", "teach ghost about my writing", "what am I good at", "what are my weaknesses", "things I dislike in writing" → invoke ghost-train
- "Update voice", "change voice profile", "promote voice", "share voice" → invoke ghost-voice
- "Update ghostai", "update ghost", "upgrade ghostai", "get latest ghost" → invoke ghost-update

Disambiguating the writing skills (draft vs. expand vs. interview vs.
rewrite vs. edit): they overlap on verbs ("write", "fix", "redo") but
differ on the author's *state* and what's already on the page. Pick by
what's blocking them, not what they typed:
- Blank page, no notes → `/ghost-draft` (AI proposes, author reacts)
- Notes/bullets, just need prose → `/ghost-expand`
- Knows the material cold but the prose won't sound right → `/ghost-interview`
- Passage exists, content roughly right, but staging is wrong (POV, order,
  framing, who-does-what) → `/ghost-rewrite`
- Passage exists, structure is fine, sentences need polish → `/ghost-edit`
- Wants Ghost to learn about *them*, not write a chapter → `/ghost-train`
- "I'm stuck" with no other signal → ask which kind of stuck, then route

---
> Source: [leanpub/ghostai](https://github.com/leanpub/ghostai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
