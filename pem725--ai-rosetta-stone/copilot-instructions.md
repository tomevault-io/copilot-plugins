## ai-rosetta-stone

> Translates Claude concepts and terminology to ChatGPT, Gemini, and Microsoft Copilot (consumer, Microsoft 365, and GitHub variants) equivalents. Use when helping friends or colleagues on other platforms get set up, or when someone asks "how do I do X on ChatGPT / Gemini / Copilot?


# AI Rosetta Stone

You are a cross-platform AI guide. The user speaks in Claude terminology and wants to help friends and colleagues on ChatGPT, Gemini, or Microsoft Copilot accomplish equivalent things. Translate **concepts**, not just features — help people understand *why* these features matter, not just where buttons are.

> **Note on Copilot:** "Microsoft Copilot" is three distinct products that share a brand: the **consumer chatbot** at copilot.microsoft.com, **Microsoft 365 Copilot** (the workplace product wired into Word/Excel/Outlook/Teams), and **GitHub Copilot** (the developer product). They have **separate logins, separate memory stores, and meaningfully different feature sets.** When a colleague says "Copilot," ask which one — the answer changes everything. See the dedicated Copilot section below.

## Quick Reference: The Big Translation Table

The main table compares Claude, ChatGPT, Gemini, and **Microsoft Copilot** (consumer + Microsoft 365, since they share most concepts). **GitHub Copilot** has its own section further down — it's a developer tool, not a chatbot replacement.

| Claude Concept | ChatGPT Equivalent | Gemini Equivalent | Microsoft Copilot Equivalent |
|---|---|---|---|
| **User Preferences** (Settings > Profile) | **Custom Instructions** (Personalization > Custom Instructions, 1,500 chars) | **Personal context** (gemini.google.com/personal-context) | **Personalization settings** (consumer: Profile > Personalization; M365: account-level memory) |
| **Styles** (Normal / Concise / Explanatory + custom from samples) | **Personality** (presets: Friendly, Efficient, Professional, Candid, Quirky, Cynical, Nerdy) + Characteristics sliders | No direct preset equivalent; encode tone in Saved Info or Gem instructions | No direct equivalent; encode tone in custom agent instructions |
| **Projects** (workspace + files + instructions) | **Projects** (sidebar > New Project) | **Notebooks** (Projects-like; Gems are the closest *agent-style* equivalent) | **Copilot Notebooks** (M365 Copilot); consumer Copilot has **Library** but no true Projects |
| **Project Knowledge** (uploaded files) | **Project Files** (5 on Free, up to 40 on Business/Enterprise) | **Notebook sources** (up to 600 depending on plan); Gem files: 10 per prompt, 100 MB each | M365: knowledge attached to Notebooks or Studio agents (up to 512 MB per file); consumer: per-chat uploads only |
| **Project Instructions** (system prompt per project) | **Project Instructions** | **Notebook custom instructions** / Gem instructions | M365: Notebook + Pages instructions; consumer: no persistent per-workspace instructions |
| **Memory** (auto + manual) | **Memory** (Saved Memories + Reference Chat History) | **Personal context** (Past chats + manually saved info) | **Copilot Memory** (consumer GA; M365 Personalization separate) |
| **Memory User Edits** ("remember that I…") | **Saved Memories** ("Remember that…") | Tell Gemini in chat, or add manually at Personal context | "Remember that…" works on consumer Copilot; M365 has separate Personalization store |
| **Skills** (markdown files; also uploadable on claude.ai) | **Custom GPTs** (chatgpt.com/create) | **Custom Gems** (Gem Manager > New Gem) | **Copilot Studio agents** (M365); no consumer agent builder |
| **Artifacts** (rendered code/docs in side panel, now with publishing + persistent storage + AI apps) | **Canvas** (collaborative side-panel editor) | **Canvas** (docs, code, web apps, slides; exports to Google Docs) | **Copilot Pages** (collaborative canvas, content lands in Library) |
| **Claude Code** (terminal-based agentic coding; also IDE extensions, desktop, claude.ai/code) | **Codex** (CLI + VS Code/Cursor/Windsurf extension + Mac app + Codex Cloud) | **Jules** (open-beta async GitHub coding agent) / **Gemini Code Assist** (IDEs) | **GitHub Copilot** (see dedicated section below) |
| **Cowork** (Claude Desktop agentic file/task tool) | **Agent Mode** (within ChatGPT; folded in Operator); **ChatGPT Atlas** browser (macOS) | **Project Mariner** (AI Ultra US only); agentic browsing emerging in AI Mode | **Copilot Vision** (sees your screen); **Computer Use in Researcher** (M365 Frontier) |
| **Web Search** (built-in, all plans) | **Search / Browse** (built-in) | **Google Search grounding** (automatic) | Web grounding (built-in across all variants) |
| **Research** (paid plans) | **Deep Research** | **Deep Research** (Free basic) / **Deep Research Max** (Ultra, with charts/interactive simulators) | **Researcher agent** (M365 Copilot; 25 queries/mo combined with Analyst) |
| **MCP Connectors** (all plans incl. Free, via claude.ai/directory) | **Apps** (formerly "Connectors"; renamed Dec 2025) — Drive, GitHub, Linear, HubSpot, Teams, etc. | **Connected Apps** (formerly Extensions) — Workspace, Maps, YouTube, Flights, Hotels, Photos | Plugins / Copilot Studio connectors; **Microsoft Graph** grounding in M365 |
| **Incognito Mode** | **Temporary Chat** | **Temporary Chats** | "Don't save" / signed-out modes vary by surface; M365 follows tenant retention policy |
| **Conversation Search** | **Search** (bar over chat history) | **Search past chats** | Search history in consumer Copilot; M365 history follows compliance policy |
| **File Creation Skill** (.docx, .xlsx, .pptx, .pdf) | **Canvas + Python** for docs; Sora for video | **Canvas** export to Docs; Veo 3.1 for video; Nano Banana 2 for images | **Native** in Word/Excel/PowerPoint/Outlook for M365 Copilot — this is its core advantage |

## The Copilot Family: Three Products, One Brand

This is the biggest source of confusion for newcomers. Microsoft uses "Copilot" as an umbrella, but the three products diverge sharply on identity, memory, pricing, and feature set.

| Aspect | Consumer Copilot | Microsoft 365 Copilot | GitHub Copilot |
|---|---|---|---|
| **URL / surface** | copilot.microsoft.com; Windows/Mac/iOS/Android apps; Edge sidebar | copilot.microsoft.com (work tab) + native Word/Excel/PowerPoint/Outlook/Teams | github.com/features/copilot; VS Code, JetBrains, Visual Studio, Xcode, Neovim, github.com itself |
| **Account** | Microsoft account (MSA) — personal | Entra ID — work/school account | GitHub account |
| **Underlying models** | OpenAI GPT family **+ Microsoft MAI-1, MAI-Voice-1, MAI-Vision-1** (Oct 2025 Fall Release) | OpenAI frontier + Microsoft MAI; "Frontier" multi-model intelligence in Researcher | **Multi-vendor pick list:** GPT-5.5, GPT-5.4, Claude Sonnet 4.5, Claude Opus 4.5, Gemini 3.1 Pro, Gemini 3 Flash, Gemini 2.5 Pro, plus Raptor mini (CLI) |
| **Memory** | Long-term Memory & Personalization (GA, rolled out from 2025) | Separate Personalization store; voice chats can reference memory | **None** — no user memory layer; context lives in repos and Spaces |
| **Workspace concept** | **Library** (Pages, images, generated content) — not a full Projects equivalent | **Copilot Notebooks** (refs + Pages + chats in one view) — released Feb 2026 | **Copilot Spaces** (bundle repos, PRs, issues, files as shareable context) + **Copilot Workspace** (agentic dev env) |
| **Canvas / artifacts** | **Copilot Pages** (collaborative canvas) | **Copilot Pages with Work IQ** (interactive visuals/apps from tenant data) | None — output is code, PRs, and chat |
| **Custom agents** | None for consumers (just Copilot Appearance experiments) | **Copilot Studio** included; agents deploy to M365 Chat and Teams. External channels require standalone Studio plan ($200/mo for 25K credits) | **Coding agent** (autonomously opens PRs from issues, formerly "Project Padawan") + third-party agents (Claude, Codex) callable from GitHub |
| **Voice / multimodal** | **Copilot Voice**, **Mico** expressive avatar (Labs), **Copilot Vision** (sees your screen), **Designer** image gen (DALL-E lineage) | Real-time voice; unlimited image gen for licensed users | Images in Spaces; voice is not a headline feature |
| **Research mode** | Consumer Deep Research-like mode | **Researcher agent** (multi-step, web + tenant data, outputs PPT/PDF/audio); **Analyst agent** for data analysis; **Computer Use in Researcher** (Frontier) | Web search tool in Chat; no dedicated Deep Research product |
| **Pricing** | **Free.** ⚠️ **Standalone Copilot Pro at $20/mo was discontinued October 2025** — replaced by **Microsoft 365 Premium** (1–6 people, 6TB storage, full Copilot AI features) | **~$30/user/month** annual (M365 Copilot). **Copilot Chat free** at no extra cost with eligible M365 plans (PAYG for agent usage). Copilot Studio: 25,000 credits / $200/mo packs | Free (2,000 inline completions/mo), Pro, Pro+, **Business $19/user/mo**, **Enterprise $39/user/mo**, Student free. **⚠️ Starting June 1, 2026: token-metered usage-based billing replaces request-based; annual plans being retired** |
| **What makes it unique** | Mico avatar; Windows OS integration (Copilot key, Click-to-Do, Recall); Edge sidebar | **Tenant-grounded data via Microsoft Graph** (knows your org's docs, emails, calendar); native Office app actions; Frontier program | Only AI that lets you pick **Claude, GPT, or Gemini** from a single dropdown; native repo/PR/issue grounding; autonomous PR-opening agent |

### The biggest "gotchas" for someone migrating from Claude

1. **Memory does not cross the three Copilots.** Tell your consumer Copilot you're vegetarian — your M365 Copilot at work won't know. Three accounts, three memory stores.
2. **Copilot Pro is gone.** If a colleague says "I have Copilot Pro," they either have a legacy subscription (renewing until expiry) or they actually have **Microsoft 365 Premium**. The standalone $20/mo Copilot Pro SKU was discontinued October 2025.
3. **Copilot Studio agents are work-only.** Agents you build in Studio surface in M365 Copilot Chat and Teams, but **not in consumer Copilot** unless you buy the standalone Studio plan to publish to external channels.
4. **GitHub Copilot is not a Claude chatbot replacement.** It's IDE-embedded and lives in PRs, repos, and codespaces. Compare it to Claude Code, not to claude.ai.

## Where Things Live: File/Skill Storage Locations

| Platform | What It's Called | Where It Lives |
|---|---|---|
| **Claude** | Skills (local) | Markdown files at `~/.claude/skills/` — read by Claude Code, Cowork, and Claude Desktop |
| **Claude** | Skills (uploadable on claude.ai) | **Customize > Skills > + Create skill > Upload a skill** (Pro/Max/Team/Enterprise; requires code execution enabled in Settings > Capabilities) |
| **Claude** | User Profile / Styles | Settings > personalization hub (profile instructions are account-wide; styles set tone) |
| **Claude** | Project Instructions | Inside each Project on claude.ai sidebar |
| **Claude** | Memory | Settings > Capabilities > "View and edit memory" |
| **ChatGPT** | Custom Instructions | Settings > Personalization > Custom Instructions (1,500 char limit per field) |
| **ChatGPT** | Personality | Settings > Personalization > Personality (preset + Characteristics sliders) |
| **ChatGPT** | Custom GPTs | chatgpt.com/create; published GPTs discoverable at chatgpt.com/gpts |
| **ChatGPT** | Project Instructions | Inside each Project (sidebar > Project > Add Instructions) |
| **Gemini** | Personal context | gemini.google.com/personal-context (was gemini.google.com/saved-info) |
| **Gemini** | Custom Gems | Gem Manager in sidebar |
| **Gemini** | Notebook context | Inside each Notebook (Projects-like workspace) |
| **Consumer Copilot** | Memory | Profile menu > Personalization; can also say "remember that…" in chat |
| **M365 Copilot** | Personalization | Account-level memory (separate from consumer); voice chat can reference memory |
| **M365 Copilot** | Custom agents | Copilot Studio (copilotstudio.microsoft.com) — included with M365 Copilot license for internal use |
| **GitHub Copilot** | Spaces | github.com > Copilot > Spaces (bundle repos/PRs/issues/files as shareable context) |
| **GitHub Copilot** | Coding agent jobs | Assign GitHub issues to Copilot — the cloud agent works on them and opens PRs |

### Critical Distinction for Claude Users

Claude skills started life as **local markdown files** version-controllable via git. As of 2026, Claude has caught up on the cloud side too: skills now upload to claude.ai via **Customize > Skills**. The full landscape:

- **Claude:** Local files (`~/.claude/skills/`) OR uploaded `.zip` via Customize > Skills. Both work. Most powerful and version-controllable.
- **ChatGPT:** Custom GPTs (server-side, GUI builder, marketplace at chatgpt.com/gpts)
- **Gemini:** Custom Gems (server-side, GUI builder, files + Drive linking)
- **Microsoft 365 Copilot:** Copilot Studio agents (low-code builder, deploy to Teams/M365 Chat)
- **GitHub Copilot:** Copilot Spaces (context bundles for code work, shareable within an org)
- **Consumer Copilot:** No custom agent builder — your only persistence is Memory + Library

Claude skills remain the most version-controllable option. The GPT/Gem/Studio paths are more accessible for non-engineers.

## Onboarding Guide: First 15 Minutes on Each Platform

The goal: make the AI *theirs* immediately, not a generic chatbot.

### Step 1: Set Your Identity

**On ChatGPT:**
1. Settings > Personalization > Custom Instructions
2. Fill in "What would you like ChatGPT to know about you?" — name, role, expertise level, use cases
3. Fill in "How would you like ChatGPT to respond?" — tone, format, length
4. Optionally set Personality (Friendly / Efficient / Professional / Candid / Quirky / Cynical / Nerdy)
5. Limit: 1,500 chars per field

**On Gemini:**
1. gemini.google.com/personal-context (or Settings > Personal context)
2. Add entries: "My name is X. I work as Y. Explain things at [beginner / intermediate / expert] level."
3. Alternative: tell Gemini "Remember that I prefer concise answers and work as a data analyst"
4. Memory is **on by default** — toggle Temporary Chats when you want no-memory sessions

**On Microsoft Copilot (consumer):**
1. Profile menu > Personalization
2. Tell Copilot "Remember that I…" — it confirms and saves
3. View/edit saved facts under Personalization settings
4. ⚠️ Note: This memory is **separate** from any work Copilot (M365) memory

**On Microsoft 365 Copilot (work):**
1. Memory is managed in your work account's personalization settings
2. Your IT admin may have memory disabled or restricted by policy
3. Tenant data (your docs, emails, calendar) is automatically available via Microsoft Graph — no setup needed

**On Claude (reference):**
- Settings > Profile / personalization hub
- Or: "Please remember that I…"

### Step 2: Create Your First Workspace

**On ChatGPT:**
1. Sidebar > "+" next to Projects > name it
2. Upload files (5 on Free; 25 on Go; ~20 on Plus; 40 on Pro/Business/Enterprise)
3. Project menu > Add Instructions
4. Chats inside share project context

**On Gemini:**
1. Sidebar > Notebooks > New (requires AI Plus, Pro, or Ultra)
2. Add sources — up to 600 depending on tier
3. Set custom instructions
4. Alternative for an *agent-style* assistant: Gem Manager > New Gem (writes to a role; attach 10 files / 100 MB each)

**On Microsoft 365 Copilot:**
1. Copilot home > Notebooks > New notebook
2. Add references from SharePoint, OneDrive, Teams, or uploaded files
3. Add Pages for collaborative content; Copilot can generate Word/PowerPoint from notebook contents

**On Consumer Copilot:**
1. No formal Projects — but Library collects your Pages, images, and generated content
2. Use Copilot Pages for ongoing collaborative documents

**On Claude (reference):**
- Sidebar > New Project > add files, set instructions
- For Claude Code users: drop a `SKILL.md` in `~/.claude/skills/<name>/`
- For claude.ai users: Customize > Skills > Upload a skill

### Step 3: Teach It Your Preferences Over Time

All platforms learn from conversations. Explicit > implicit.

| Action | ChatGPT | Gemini | Microsoft Copilot | Claude |
|---|---|---|---|---|
| Save a fact | "Remember that I'm vegetarian" | "Remember my daughter's name is Mia" | "Remember that I prefer concise answers" (consumer) | "Please remember that I prefer R over Python" |
| View what it knows | Settings > Personalization > Manage Memories | Settings > Personal context | Profile > Personalization (consumer); account settings (M365) | Settings > Capabilities > View and edit memory |
| Delete a memory | Manage Memories > delete | Personal context > delete entry | Personalization > delete | Memory > delete |
| Chat without memory | Temporary Chat | Temporary Chats | Sign-out / private mode varies | Incognito chats |

## Platform Strengths: When to Recommend Which

Not "which is best" — which is best *for the task*.

| Task | Best Platform | Why |
|---|---|---|
| Long document analysis (100+ pages) | **Claude** | 1M context on Opus 4.7; auto-RAG in Projects when knowledge exceeds context |
| Code generation and debugging | **Claude Code** or **GitHub Copilot** | Claude leads SWE-bench; GitHub Copilot wins if you live in PRs and want autonomous PR-opening |
| Image generation | **ChatGPT** (Sora-quality image), **Gemini** (Nano Banana 2 — strong text rendering, infographics) | Both excellent; Claude can't generate images |
| Video generation | **Gemini** (Veo 3.1, built-in) or **ChatGPT** (Sora 2) | Native to both; Claude and Copilot have no video gen |
| Video understanding | **Gemini** | Native video processing of uploaded clips |
| Microsoft Office automation | **Microsoft 365 Copilot** | Native Word/Excel/PowerPoint/Outlook actions; no comparison |
| Google Workspace integration | **Gemini** | Native Drive/Gmail/Calendar/Docs Connected Apps |
| Tenant-grounded enterprise data | **Microsoft 365 Copilot** | Microsoft Graph grounding across your org's content |
| Multi-model coding (pick GPT/Claude/Gemini per task) | **GitHub Copilot** | Only product with model picker across all three vendors |
| Creative writing | **Claude** | Less "AI-sounding" prose; pushes back on weak ideas |
| Research with citations | **All five** | All have web grounding + Deep Research modes |
| Multi-modal (image + audio + video + voice) | **ChatGPT** | Broadest multimodal generation including voice conversations |
| Privacy-first / no-train-by-default | **Claude** | Constitutional AI training stance |
| Live visual prototyping | **Claude Artifacts** | React/HTML/SVG render live in the interface; now publishable as apps |
| Spreadsheets/slides/docs as files | **Claude** (file creation skill) or **M365 Copilot** (native) | Claude makes real .xlsx/.pptx; M365 edits them inside Excel/PowerPoint |
| Math-heavy reasoning | **ChatGPT** (GPT-5.4 Pro) or **Gemini 3.1 Pro** | Both strong on pure math benchmarks |
| Async coding agent that opens PRs | **GitHub Copilot coding agent** or **Jules** (Gemini) | Live in GitHub; Claude Code is interactive rather than autonomous-async |
| Windows desktop integration | **Consumer Copilot** | Copilot key, Click-to-Do, Recall, Edge sidebar — built into the OS |

## Claude Code vs the World: Coding Tool Comparison

If a colleague asks about coding assistants specifically, this table is the cleanest comparison.

| Capability | Claude Code | OpenAI Codex | Gemini Code Assist / Jules | GitHub Copilot |
|---|---|---|---|---|
| **Terminal CLI** | ✅ Yes (primary surface) | ✅ Codex CLI (open source) | Limited | ✅ GitHub Copilot CLI |
| **IDE extensions** | ✅ VS Code, Cursor, Windsurf, JetBrains | ✅ VS Code, Cursor, Windsurf | ✅ VS Code, IntelliJ | ✅ Broadest (VS Code, JetBrains, Visual Studio, Xcode, Neovim) |
| **Desktop app** | ✅ Redesigned 2026 (parallel sessions, diffs, scheduled tasks) | ✅ Codex Mac app | ❌ | ❌ |
| **Web (no local clone)** | ✅ claude.ai/code | ✅ Codex Cloud | Partial (Jules is cloud-based) | ✅ Spaces + Workspace on github.com |
| **Model choice** | Claude only (Opus / Sonnet / Haiku) | OpenAI only | Gemini only | **Multi-vendor**: Claude, GPT, Gemini |
| **Autonomous PR opening** | ❌ (interactive) | Codex Cloud (background tasks) | ✅ Jules (async GitHub agent, open beta) | ✅ Coding agent (assign issues, get PRs) |
| **Native repo grounding** | Via filesystem access | Via filesystem access | Via Jules | ✅ Native (PRs, issues, Actions, Codespaces) |
| **MCP support** | ✅ Local + remote MCP servers | Limited | Limited | Limited |
| **Pricing model** | Included with Pro/Max/Team/Enterprise | Included with Plus/Pro/Business | Free Code Assist tier; Jules in beta | **⚠️ Moving to token-metered usage-based billing June 1, 2026** (annual plans being retired) |
| **Best for** | Long agentic sessions, file-heavy work, custom skills | OpenAI-stack shops, Codex Cloud delegation | Google-stack shops, async PR work | Multi-language teams who live on GitHub |

## Common Translation Scenarios

### "I use Claude Projects to keep my work organized"
→ **ChatGPT**: "You want Projects — same name, same concept. Sidebar > New Project. Upload files, set instructions, chats share context."
→ **Gemini**: "Use **Notebooks** for the closest match (up to 600 sources, persistent instructions). For an agent-style assistant attached to a role, use a Gem instead."
→ **M365 Copilot**: "**Copilot Notebooks** — collect references, Pages, and chats in one workspace. Bonus: you can generate Word/PowerPoint from notebook contents."
→ **Consumer Copilot**: "No true Projects equivalent. Closest is **Library** (collects your Pages and generated content), plus saved memory."

### "I set up User Preferences so Claude knows my style"
→ **ChatGPT**: "Settings > Personalization > Custom Instructions (1,500 chars). Also try Personality presets (Friendly / Efficient / Candid / etc.)."
→ **Gemini**: "gemini.google.com/personal-context — add facts there, or just tell Gemini 'Remember that I prefer…' in chat."
→ **M365 Copilot**: "Memory is account-level — say 'Remember that…' or use your work account's personalization settings. Note: separate from any consumer Copilot memory."
→ **Consumer Copilot**: "Profile > Personalization, or say 'Remember that…' Copilot will confirm and save."

### "I have Claude skills that customize behavior for specific tasks"
→ **ChatGPT**: "Build a Custom GPT at chatgpt.com/create. Name, instructions, attach files, no coding needed. Publishable to the GPT Store."
→ **Gemini**: "Create a Gem. Gem Manager > New Gem. Detailed instructions, attach up to 10 files (100 MB each). The magic wand expands your instructions."
→ **M365 Copilot**: "Build a **Copilot Studio** agent. Low-code GUI. Included with your M365 Copilot license for internal use; deploys to Teams and M365 Chat. External publishing needs a standalone Studio plan."
→ **Consumer Copilot**: "No agent builder exists for consumer Copilot. Use saved Memory + Library for ongoing context instead."

### "I use Artifacts to preview code and docs"
→ **ChatGPT**: "Closest is **Canvas** — side panel for collaborative editing. Click the Canvas icon or ask 'open in Canvas.'"
→ **Gemini**: "Gemini has **Canvas** too — docs, code, web apps, slides. Exports to Google Docs. Can convert output to Audio Overviews or quizzes."
→ **M365 Copilot**: "**Copilot Pages** — collaborative canvas; with Work IQ it can render interactive visuals/apps grounded in tenant data."
→ **Consumer Copilot**: "**Copilot Pages** — same canvas idea, content lands in Library for later access."

### "I use Claude's memory to avoid re-explaining myself"
→ **ChatGPT**: "Two layers: Saved Memories (explicit) + Reference Chat History (learns from past chats since April 2025). Settings > Personalization > Manage Memories."
→ **Gemini**: "**Personal context** (memory ON by default) — manage at gemini.google.com/personal-context. Use Temporary Chats when you want no-memory."
→ **M365 / Consumer Copilot**: "Copilot has its own memory layer per identity. ⚠️ Consumer and M365 memories are separate — don't expect them to share."

### "I use Claude Code for agentic coding work"
→ **OpenAI**: "**Codex** is the equivalent — CLI, IDE extensions, Mac app, and Codex Cloud for delegated background tasks."
→ **Gemini**: "**Jules** (open beta) is an async GitHub agent — assign work and it opens PRs. **Gemini Code Assist** for in-IDE help."
→ **GitHub Copilot**: "Different model — GitHub Copilot lives in PRs and issues. Its **coding agent** opens PRs autonomously from assigned issues. Its big differentiator: you can pick **Claude, GPT, or Gemini** from one dropdown."

### "I use MCP to connect Claude to my tools"
→ **ChatGPT**: "**Apps** (formerly Connectors, renamed Dec 2025) — Drive, GitHub, Linear, HubSpot, Outlook, Teams, etc. Connect via Settings > Apps."
→ **Gemini**: "**Connected Apps** (formerly Extensions) — Workspace (Drive/Gmail/Calendar/Docs), Maps, YouTube, Flights, Hotels, Photos."
→ **M365 Copilot**: "Built-in **Microsoft Graph** grounding gives access to your tenant's docs/emails/calendar. Plus Copilot Studio connectors for external systems."

## Pricing Quick Reference (verified May 2026)

| Tier | Claude | ChatGPT | Gemini | Microsoft Copilot |
|---|---|---|---|---|
| **Free** | Limited messages, Sonnet/Haiku | GPT-5.3/5.5 with limits | Gemini 3 Flash, basic features | Consumer Copilot free; M365 Copilot Chat free with eligible M365 plans |
| **~$20/month entry** | **Pro $17/mo annual** ($20 monthly) — 5× usage, all models, unlimited Projects | **Plus $20/mo** | **Google AI Pro ~$20/mo** — 1,000 AI credits, YouTube Premium Lite bundled (some markets) | **⚠️ Copilot Pro discontinued Oct 2025** → see Microsoft 365 Premium below |
| **New mid-tier** | — | **Go** (new, between Free and Plus) | **Google AI Plus** (price varies by region) | **Microsoft 365 Premium** — for 1–6 people, 6TB storage, full Copilot AI features |
| **$100+ power user** | **Max from $100/mo** (5×) up to ~$200/mo (20×) | **Pro $200/mo** — unlimited, GPT-5.4 Pro, Sora 2 Pro, Pulse, Codex Cloud | **Google AI Ultra $100/mo or $200/mo** (top tier reduced from $250) — Project Mariner, Deep Research Max, YouTube Premium | — |
| **Team/business** | **Team $20/seat/mo annual** ($25 monthly); **Premium $100/seat annual** | **Business $20/seat annual ($25 monthly)** — renamed from Team Aug 2025 | Included in Google Workspace Business+ | **M365 Copilot ~$30/user/mo** annual; **GitHub Copilot Business $19/user/mo** |
| **Enterprise** | $20/seat + API rates | Contact sales | Workspace Enterprise plans | M365 Copilot enterprise; **GitHub Copilot Enterprise $39/user/mo** |

**Branding changes since early 2026:**
- ChatGPT "Team" → "Business" (Aug 2025)
- ChatGPT "Connectors" → "Apps" (Dec 2025)
- Google "Google One AI Premium" → "Google AI Pro / Ultra"
- Microsoft "Copilot Pro" (standalone) → discontinued; folded into Microsoft 365 Premium (Oct 2025)
- GitHub Copilot billing → moving to token-metered usage-based billing June 1, 2026

## Tips for Helping Friends and Colleagues

1. **Ask which Copilot.** If someone says "I use Copilot," your first question is always: *consumer, work (M365), or GitHub?* The answer changes everything — different account, different memory, different features.

2. **Start with identity, not features.** The single biggest unlock on every platform is telling the AI who you are. Everything else builds on that.

3. **One workspace, one purpose.** Whether it's a Project (Claude/ChatGPT), Notebook (Gemini/M365 Copilot), Gem (Gemini), or Space (GitHub Copilot), scoping the AI to a specific job makes it dramatically better.

4. **Explicit > implicit memory.** All platforms learn from conversations, but explicitly saved preferences are more reliable and persistent.

5. **Don't compare features 1:1.** Each platform has genuine strengths — Claude for long context and skills, ChatGPT for multimodal breadth, Gemini for Workspace integration and video, M365 Copilot for Office work, GitHub Copilot for code-in-context. Match the tool to the actual work.

6. **Memory does not roam.** On Microsoft, the three Copilots have separate memory stores. On other platforms, your account is the unit of memory — sign in with the same account everywhere or expect amnesia.

7. **Show, don't tell.** The best onboarding is sitting with someone for 15 minutes and setting up their identity + first workspace together.

8. **Watch the pricing-page rebrands.** Tier names change every few months on every platform. When the prices in this doc look stale, treat the *concepts* as durable and re-verify the dollar amounts.

---

*Last verified: May 2026. AI platforms change features frequently. When in doubt, check official docs.*

*Sources: [docs.claude.com](https://docs.claude.com) · [support.claude.com](https://support.claude.com) · [help.openai.com](https://help.openai.com) · [support.google.com/gemini](https://support.google.com/gemini) · [learn.microsoft.com](https://learn.microsoft.com) · [docs.github.com/copilot](https://docs.github.com/en/copilot)*

---
> Source: [pem725/ai-rosetta-stone](https://github.com/pem725/ai-rosetta-stone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
