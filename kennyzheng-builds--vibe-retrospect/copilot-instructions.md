## vibe-retrospect

> >-


# Vibe Retrospect

A dual-purpose skill for vibe coding knowledge transfer.

- **Retrospect mode**: Analyze a completed project → generate an HTML article (for humans) + a knowledge doc (for agents)
- **Learn mode**: Import someone else's knowledge doc → configure CLAUDE.md with teaching rules + project references → agent guides the user through building

## Mode Detection

At the start, determine which mode the user needs:

**Retrospect mode** triggers:
- "复盘", "retrospect", "总结开发经验", "打包我的开发经验", "review my project"
- User is in a project directory with code/git history
- User wants to generate output FROM their project

**Learn mode** triggers:
- "我想学做这个项目", "帮我学习这个", "参考这个来做", "coach me", "learn from this"
- User provides a `-knowledge.md` file or mentions someone else's project
- User wants to BUILD something based on a reference

If ambiguous, ask: "你是想复盘自己的项目，还是想参考别人的项目来学习开发？"

---

## Mode 1: Retrospect (复盘)

### Goal

Analyze a completed vibe coding project and generate THREE outputs:
1. **`{project-name}-story.html`** — A self-contained HTML article for humans to read
2. **`{project-name}-knowledge.md`** — A structured knowledge document for agents to reference
3. **`outputs/cards/card-*.png`** — Social media image cards (1080x1350 @2x, 4:5 ratio) for sharing on Xiaohongshu/Instagram

### Step 1: Locate All Data Sources

Silently gather everything before talking to the user.

**1A. Codebase**
```bash
# Project identity
basename $(pwd)
git remote get-url origin 2>/dev/null
```

Read (if they exist):
- `CLAUDE.md` — project rules, collaboration norms, workflow
- `README.md` — project description
- `docs/` — specs, decisions, status, tasks
- Main source files, config files (`package.json`, `wrangler.toml`, etc.)
- Deployment scripts

**1B. Git history**
```bash
git log --format='%ai' --all | tail -1    # first commit date
git log --format='%ai' --all | head -1    # last commit date
git log --all --oneline | wc -l           # total commits
git log --all --format='%ai | %s' --reverse  # full timeline
git log --all --format='%ad' --date=short | sort | uniq -c | sort -rn | head -5  # peak days
git log --all --name-only --format= | sort | uniq -c | sort -rn | head -10  # most changed files
```

**1C. Session logs (the real gold)**

Claude Code stores conversation history at `~/.claude/projects/{encoded-path}/`.
The path encoding replaces `/` with `-` and prepends `-`.

```bash
CWD=$(pwd)
ENCODED=$(echo "$CWD" | sed 's|/|-|g')
CLAUDE_DIR="$HOME/.claude/projects/$ENCODED"
ls "$CLAUDE_DIR"/*.jsonl 2>/dev/null
```

Each `.jsonl` file = one conversation session. Extract from them:
- **User messages**: every decision, feedback, question the human made
- **Decision points**: where user said "不要"/"改成"/"可以"/"停"/"no"/"change"/"ok"
- **Collaboration feedback**: "以后"/"下次"/"记住"/"always"/"never" — rules born from experience
- **First message**: often reveals the origin story
- **Debugging sessions**: long back-and-forth exchanges = pitfalls

Use the parse script: `bash ~/.claude/skills/vibe-retrospect/scripts/parse-sessions.sh "$CLAUDE_DIR" ./tmp/session-analysis`

**1D. Memory files**
```bash
ls "$CLAUDE_DIR/memory/" 2>/dev/null   # MEMORY.md, daily notes
```

**1E. GitHub (if remote exists)**
```bash
gh pr list --state all --limit 50 2>/dev/null
gh issue list --state all 2>/dev/null
```

### Step 2: Present Summary, Ask for Gaps

Show the user what you found. Ask them to supplement what can't be extracted automatically:

1. **Origin story** — "I see your first message was '{first_msg}'. What triggered this idea?"
2. **Key turning points** — "On {date} you made {N} commits. What happened that day?"
3. **Advice for others** — "If someone wants to build something similar, top 3 things to tell them?"
4. **Cost** — "What's the monthly cost to run this?"
5. **Anything sensitive to redact?**

User can skip any question. Fill gaps from session logs and git history.

### Step 3: Generate Three Outputs

Place all in `./outputs/` directory.

#### Output 1: `{project-name}-story.html`

A self-contained HTML article with embedded CSS. Warm, editorial style. For humans to read and share.

Structure:
- Project origin story (with direct quotes from session logs)
- Key decisions and why they were made
- Architecture overview
- Development timeline with milestones
- Pitfalls encountered and how they were solved
- Collaboration methodology (how human + AI worked together)
- Lessons learned

Design guidelines:
- Self-contained (all CSS inline/embedded, no external dependencies)
- Readable typography (serif for body, sans for headings)
- Warm color palette (cream/off-white backgrounds)
- Responsive (works on mobile and desktop)
- Include direct quotes from session logs throughout
- Must include in `<head>`: SVG emoji favicon (`<link rel="icon" href="data:image/svg+xml,...">`) and Open Graph meta tags (`og:title`, `og:description`, `og:type`, `og:url`) so browser tabs show an icon and social sharing shows preview

#### Output 2: `{project-name}-knowledge.md`

A structured knowledge document following the **Knowledge Document Schema** below. This is NOT a skill — it is reference material that another user's agent can read.

---

## Knowledge Document Schema

Every knowledge output MUST follow this structure. This is the standard — it ensures any agent that reads it can find information in predictable locations.

```markdown
# {Project Name} — Project Knowledge

> One-sentence description of what this project is and does.

## Quick Facts

| Metric | Value |
|---|---|
| What | {one-line description} |
| Built by | {author} with {AI tool} |
| Timeline | {start} → {end} ({N} days) |
| Commits | {N} |
| Tech stack | {stack} |
| Monthly cost | {cost} |
| Repo | {url or "private"} |

## The Problem

{2-3 paragraphs: what pain point does this solve? What was the "aha" moment?
 Include direct quotes from session logs if available.}

## Key Decisions

{For each major decision:}

### Decision: {title}
- **Context**: {why this decision came up}
- **Options considered**: {what alternatives were discussed}
- **Chosen**: {what was picked}
- **Why**: {reasoning — from session logs if possible}
- **Quote**: "{direct quote from conversation if available}"

## Architecture

### Tech Stack
| Layer | Choice | Why |
|---|---|---|
| {layer} | {tech} | {reason} |

### System Design
{ASCII diagram or description of how components connect}

### Data Model
{Key data structures, storage design, API routes}

## Development Timeline

| Day/Date | What happened | Commits |
|---|---|---|
| {date} | {milestone description} | {N} |

{Highlight peak days and turning points}

## How Builder & AI Collaborated

### Workflow Rules (from CLAUDE.md)
{Extract and list the actual rules from the project's CLAUDE.md}

### Collaboration Patterns That Worked
{What methodology did they use? Multi-window? Context recovery? Review cycles?}

### Feedback That Shaped the Process
{Direct quotes of collaboration feedback from session logs:
 "以后不要直接写代码，先讨论方案" → became a rule in CLAUDE.md}

## Pitfalls & Solutions

{For each pitfall:}

### Pitfall: {title}
- **Symptom**: {what went wrong}
- **Root cause**: {why}
- **Fix**: {how it was solved}
- **Prevention**: {how to avoid it}

## Build Guide (For Someone Starting Fresh)

### Prerequisites
{Accounts to register, tools to install, things to buy}

### Recommended Build Order
1. {Module 1}: {what to build first and why}
2. {Module 2}: {what to build next}
3. ...

### CLAUDE.md Template
{A ready-to-use CLAUDE.md based on what worked in this project.
 Not the original — adapted as a starting template for someone new.}

## Lessons Learned

### On Product
{Bullet points}

### On Vibe Coding
{Bullet points}

### On Technical Choices
{Bullet points}
```

#### Output 3: Social Media Cards (`outputs/cards/card-*.png`)

After generating story.html, automatically generate a set of social media image cards for sharing.

**Approach**: Do NOT screenshot/split the long story.html. Instead, create a NEW `outputs/story-cards.html` with content re-laid-out as independent card divs, then use Puppeteer to screenshot each card element.

**Card splitting logic** — map story.html sections to 5-8 cards:

| Card | Content | Design |
|---|---|---|
| 1 (Cover) | Title + subtitle + key stats | Centered, large serif text, stats row at bottom |
| 2 (Origin) | The problem / origin story | Narrative + blockquote + insight highlight box |
| 3 (Decisions) | Core creative decisions | Pass/Chosen comparison grid (2-3 rows) |
| 4 (Timeline) | Development timeline | Vertical timeline with dots, dates, milestones |
| 5 (Method) | Collaboration methodology | Quote + numbered rule cards with left border |
| 6 (Lessons) | Lessons / advice | Numbered items (01-05) with title + description |
| 7 (Closing) | Epilogue | Centered quote, divider, emotional closure |

**Splitting principles**:
- Each card = one topic, readable in 3 seconds
- Cover and closing cards have more whitespace; middle cards can be denser
- Scatter developer quotes (blockquotes) across different cards for authenticity
- 5-8 cards total — adapt count to project content volume

**Card HTML structure**:
```html
<body>
  <div class="card" id="card-1">
    <div class="card-inner"><!-- content --></div>
    <div class="card-footer">
      <span>PROJECT NAME</span>
      <span>1 / N</span>
    </div>
  </div>
  <div class="card" id="card-2">...</div>
</body>
```

**Card CSS requirements**:
```css
.card {
  width: 1080px;
  height: 1350px;           /* fixed 4:5 ratio for Xiaohongshu/Instagram */
  position: relative;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  font-family: var(--sans);
}
.card-inner {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;  /* vertically center content — prevents top-heavy layout */
  padding: 60px 80px 40px;
}
.card-footer {
  display: flex;
  justify-content: space-between;
  padding: 0 80px 48px;
  font-family: var(--mono);
  font-size: 22px;
  color: var(--text-3);
}
```

**Visual style**: Inherit the color palette from story.html. Extract CSS variables from the story page (background, accent colors, text colors, fonts) and reuse them in the cards. Keep cards visually consistent with the story page. Use warm, readable color schemes — avoid harsh dark backgrounds.

**Font sizes**: Cards are 1080px wide, so body text should be 22-28px, titles 44-50px, heading numbers 48-58px. Content should fill roughly 70-85% of the card height — if there's too much whitespace top/bottom, increase font sizes, line-heights, and gaps.

**Puppeteer capture script** (`outputs/capture-cards.mjs`):
```javascript
import puppeteer from 'puppeteer-core';
import path from 'path';
import fs from 'fs';
import { fileURLToPath } from 'url';

const __dirname = path.dirname(fileURLToPath(import.meta.url));
const htmlPath = `file://${path.join(__dirname, 'story-cards.html')}`;
const outDir = path.join(__dirname, 'cards');
fs.mkdirSync(outDir, { recursive: true });

// Find Chrome executable
const chromePaths = [
  '/usr/bin/google-chrome',
  '/usr/bin/chromium-browser',
  '/usr/bin/chromium',
];
// Also check puppeteer cache
const cacheDir = path.join(process.env.HOME, '.cache/puppeteer/chrome');
if (fs.existsSync(cacheDir)) {
  for (const ver of fs.readdirSync(cacheDir)) {
    chromePaths.unshift(path.join(cacheDir, ver, 'chrome-linux64/chrome'));
  }
}
const executablePath = chromePaths.find(p => fs.existsSync(p));
if (!executablePath) { console.error('Chrome not found'); process.exit(1); }

const browser = await puppeteer.launch({
  headless: true,
  executablePath,
  args: ['--no-sandbox', '--disable-setuid-sandbox', '--disable-gpu']
});
const page = await browser.newPage();
await page.setViewport({ width: 1080, height: 1350, deviceScaleFactor: 2 });
await page.goto(htmlPath, { waitUntil: 'networkidle0', timeout: 30000 });
await page.evaluate(() => document.fonts.ready);
await new Promise(r => setTimeout(r, 1500));

const totalCards = await page.$$eval('.card', cards => cards.length);
console.log(`Found ${totalCards} cards, capturing...`);
for (let i = 1; i <= totalCards; i++) {
  const el = await page.$(`#card-${i}`);
  if (!el) continue;
  const outPath = path.join(outDir, `card-${i}.png`);
  await el.screenshot({ path: outPath, type: 'png' });
  const stat = fs.statSync(outPath);
  console.log(`card-${i}.png — ${Math.round(stat.size / 1024)}KB`);
}
await browser.close();
console.log('Done!');
```

**Key technical notes**:
- Use `puppeteer-core` (not `puppeteer`) with explicit `executablePath` — more reliable in diverse environments
- `deviceScaleFactor: 2` → output 2160x2700 actual pixels, sharp on retina displays
- `element.screenshot()` not `page.screenshot()` → precise cropping per card
- `justify-content: center` on `.card-inner` → content vertically centered, no top-heavy whitespace
- `waitUntil: 'networkidle0'` + `document.fonts.ready` + 1.5s delay → ensure fonts render

**Execution steps**:
1. Install puppeteer-core if needed: `npm install puppeteer-core` (in project or outputs dir)
2. Generate `outputs/story-cards.html` (re-laid-out card version inheriting story.html palette)
3. Generate `outputs/capture-cards.mjs` (screenshot script with Chrome auto-detection)
4. Run: `node outputs/capture-cards.mjs`
5. Verify output images exist and visually check a few cards for readability and whitespace balance

**Fallback**: If Puppeteer/Chrome is not available, generate only `story-cards.html` and tell the user to open it in a browser and manually screenshot each card.

### Step 4: Publish to AgentsLink (Optional)

After generating all three outputs, offer to publish to AgentsLink for easy sharing via URL.

**How it works**: One URL serves different content based on who opens it:
- **Browser** → story HTML (the beautiful article)
- **Agent/curl** → knowledge markdown (the structured doc)

**Publish script**:
```bash
# Read story HTML and knowledge markdown, publish to AgentsLink
STORY=$(cat outputs/{project-name}-story.html | python3 -c "import sys,json; print(json.dumps(sys.stdin.read()))")
KNOWLEDGE=$(cat outputs/{project-name}-knowledge.md | python3 -c "import sys,json; print(json.dumps(sys.stdin.read()))")

curl -s -X POST "https://agentslink.link/publish" \
  -H "Content-Type: application/json" \
  -d "{\"story\":${STORY},\"knowledge\":${KNOWLEDGE},\"title\":\"{Project Title}\",\"author\":\"{Author}\"}"
```

**Response**:
```json
{
  "id": "abc123",
  "url": "https://agentslink.link/p/abc123"
}
```

Tell the user:
- "已发布到 {url}"
- "浏览器打开是 HTML 文章，Agent 访问是知识文档"
- "这个链接永久有效，可以直接分享给别人"

If the user declines publishing, skip this step — local files are always the primary output.

---

## Mode 2: Learn (学习)

### Goal

Help a user learn from someone else's project experience. Configure their development environment so their agent can guide them through building a similar project step by step.

### Step 1: Get the Reference Material

Ask the user: "请把参考项目的知识文档（xxx-knowledge.md）发给我，或者告诉我文件路径。"

The user provides:
- A `{project-name}-knowledge.md` file (generated by Mode 1)
- Or pastes the content directly
- Or provides a URL/path to the file

Read and parse the knowledge document. Extract:
- Project description and core problem
- Tech stack and architecture
- Build order and prerequisites
- CLAUDE.md template
- Key decisions and pitfalls
- Lessons learned

### Step 2: Understand the User's Goal

Ask the user:
1. "你想做一个跟参考项目一样的东西，还是有自己的想法？"
2. "你的编程经验如何？完全零基础 / 有一些基础 / 比较熟练？"
3. "你有什么已经准备好的东西？（比如域名、账号等）"

### Step 3: Configure the User's Project

Based on the reference material and user's answers, do the following:

**3A. Create or modify CLAUDE.md**

Write a CLAUDE.md that combines:
- **Teaching rules** (adapted to user's experience level):
  - 先讨论再动手（discuss before coding）
  - 每步都解释（explain every step）
  - 主动阶段检查（proactive stage checks）
  - 防跳步拦截（prevent skipping steps）
  - 教用户做判断（teach user to make decisions）
- **Project reference** (from the knowledge document):
  - Reference project description
  - Recommended tech stack and why
  - Build order with module breakdown
  - Known pitfalls to avoid
  - Cost expectations
- **Context recovery mechanism**:
  - docs/status.md for progress tracking
  - docs/tasks/ for task details
  - Read status on every new conversation

**3B. Create project scaffolding**

Create the basic project structure:
```
docs/
  status.md          # Progress tracker (initialized)
  tasks/             # Task files directory
  reference/         # Store the knowledge doc here
    {project}-knowledge.md
CLAUDE.md            # Teaching-mode rules + project reference
```

**3C. Initialize status.md**

Create an initial status file based on the reference project's build order:
```markdown
# Project Status

## Current Phase: Phase 1 — Product Planning

## Build Plan (from reference: {Project Name})

### Phase 1: Product Planning
- [ ] Define one-sentence description
- [ ] Identify target user and core problem
- [ ] Discuss MVP scope

### Phase 2: Technical Setup
- [ ] Confirm tech stack
- [ ] Set up project structure
- [ ] {prerequisites from reference}

### Phase 3-N: {modules from reference build order}
- [ ] {tasks derived from reference}
```

### Step 4: Brief the User

Tell the user what was set up and how to proceed:

1. "我已经配置好了你的项目环境"
2. Explain what CLAUDE.md does (the "employee handbook" analogy)
3. "接下来你只需要跟 Claude Code 说你想做什么，它会一步步引导你"
4. Give them the first prompt to send to Claude Code:
   ```
   我想做一个 [description]，参考 docs/reference/ 里的项目经验。
   请先帮我讨论产品方案，确认后再开始写代码。
   ```

---

## Rules

### Abstraction
- This skill file contains ZERO project-specific content
- All project-specific content goes into the GENERATED output files
- The knowledge document schema is the standard — every generated file follows it

### Three Distinct Outputs (Retrospect Mode)
- **HTML** (`story.html`): For humans to read. Editorial, storytelling style. Self-contained.
- **Knowledge doc** (`knowledge.md`): For agents to reference. Structured, predictable format. Parseable.
- **Social cards** (`cards/card-*.png`): For social media sharing. Re-laid-out from story.html as independent visual cards, captured via Puppeteer.
- Never combine these into one file. They serve different audiences.
- Cards are NOT screenshots of the story page — they are a separate layout optimized for 4:5 ratio social media images.

### Knowledge Doc is NOT a Skill
- The knowledge document is reference material, not behavior definition
- It does not have skill frontmatter (no `---` yaml block)
- It is placed in `docs/reference/` when consumed by Learn mode, not in `~/.claude/skills/`

### Data Priority
When information conflicts, trust in this order:
1. User's direct input (this session)
2. Session logs (historical conversations)
3. CLAUDE.md / docs/
4. Memory files
5. Git history
6. Codebase

### Session Logs > Git History
Git shows WHAT changed. Session logs show WHY. Always prefer session log data for decisions, reasoning, and pitfalls.

### Direct Quotes
Include direct quotes from session logs whenever possible. They make the experience authentic. Format: `> "{quote}" — from session on {date}`

### Sensitive Information
Before outputting, scan for and redact:
- API keys, tokens, passwords, secrets
- Internal URLs, private IPs
- Personal emails, phone numbers
- Private repo URLs (unless user says it's ok)

Ask the user if unsure.

### Large Session Logs
Session logs can be 40MB+. Strategy:
- Use the parse script to extract user messages first
- Prioritize first session (origin story) and largest sessions (most work)
- Focus on user messages — assistant responses are mostly tool calls
- Use subagents to analyze sessions in parallel if there are many

### Language
Match the user's language throughout all generated files. If user speaks Chinese, the entire output is in Chinese. If English, English.

### Learn Mode: Respect User Autonomy
In Learn mode, the goal is to SET UP the teaching environment, not to do the teaching yourself. After configuration:
- The user's own agent (in future sessions) will do the actual teaching
- The CLAUDE.md rules ensure the agent behaves as a teacher
- The reference material in docs/reference/ gives the agent knowledge to draw on
- This skill's job is done once the environment is configured

---
> Source: [kennyzheng-builds/vibe-retrospect](https://github.com/kennyzheng-builds/vibe-retrospect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
