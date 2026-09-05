## organizational-analytics

> This file provides guidance to Claude Code (claude.ai/code) when working with

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with 
code in this repository.

## Project Overview

An MkDocs Material-based intelligent textbook teaching organizational analytics using
the labeled property graph databases (Cypher) and AI. The site features 15 chapters, interactive MicroSimulations, and a 200-concept learning graph. 
Mascot: Aria the Analytics Ant (indigo/amber color theme).

- **Live site:** https://dmccreary.github.io/organizational-analytics/
- **Repository:** https://github.com/dmccreary/organizational-analytics
- **License:** CC BY-NC-SA 4.0

### Intelligent Textbook Level

This is a **Level 2.9 intelligent textbook** — it focuses on interactivity (MicroSims, learning graph) but does not store student records for hyperpersonalization. Classification follows the five-level framework described in *A Five-Level Classification Framework for Intelligent Textbooks: Lessons from Autonomous Vehicle Standards* (McCreary, December 2025, DOI: 10.35542/osf.io/sh2yu_v1, CC BY-NC-ND 4.0). Paper source: https://github.com/dmccreary/intelligent-textbooks/tree/main/papers/five-levels

### Content Generation

This site was primarily generated from `docs/course-description.md` using Claude Code skills located at https://github.com/dmccreary/claude-skills/tree/main/skills. Most of the work building this site involved adjusting the content generation rules and improving the UI of the MicroSims.

## Development Commands

```bash
# Serve locally (user runs this themselves - never start/stop mkdocs serve)
mkdocs serve

# Build the site
mkdocs build

# Deploy to GitHub Pages
mkdocs gh-deploy
```

There is no package.json, Makefile, or test suite. The project has no npm dependencies. 
JavaScript libraries (vis-network, MathJax) are loaded via CDN.

### Learning Graph Utilities

Python scripts in `docs/learning-graph/` for graph validation and analysis:

```bash
# Validate the learning graph JSON against schema
python docs/learning-graph/validate-learning-graph.py

# Analyze graph metrics (connectivity, DAG validation, dependency chains)
python docs/learning-graph/analyze-graph.py

# Convert CSV edge list to vis-network JSON format
python docs/learning-graph/csv-to-json.py
```

These require `jsonschema` (pip install). All other imports are stdlib.

## Architecture

### Content Pipeline

`mkdocs.yml` is the single source of truth for site navigation and configuration. All content lives under `docs/`. The MkDocs `watch` directive monitors both `docs/` and `mkdocs.yml` for live reload.

### MicroSim Structure

Each MicroSim lives in `docs/sims/<sim-name>/` with these files:

| File | Purpose |
|------|---------|
| `index.md` | Documentation page with iframe embed |
| `main.html` | Entry point (loads JS/CSS, CDN libraries) |
| `script.js` | JavaScript implementation |
| `style.css` or `local.css` | Styling |
| `metadata.json` | Dublin Core metadata (optional) |

**iframe embed pattern** used in `index.md`:
```html
<iframe src="./main.html" width="100%" height="620px"
        style="border: 2px solid #303F9F; border-radius: 8px;"
        allow="fullscreen" allowfullscreen></iframe>
```

The main MicroSim types used are:
1. **vis-network** - Graph visualizations using the vis-network library (CDN)
2. **p5.js** - Canvas-based simulations using p5.js (CDN)

### Custom Admonitions

Two custom admonition types are styled in `docs/css/extra.css`:

- **`prompt`** - Has a copy-to-clipboard button (added by `docs/js/extra.js`)
- **`aria`** - Shows Aria mascot icon in upper-right corner, amber left border

### Learning Graph

`docs/learning-graph/` contains a 200-node, 343-edge concept dependency graph in both CSV and vis-network JSON formats. The graph uses 14 taxonomy categories (FOUND, GMOD, GPERF, EVENT, DPIPE, OMOD, ETHIC, GALG, NLPML, INSIT, APPHR, DASH, CAPST, etc.).

### Key Config Details

- **Theme:** Material with `primary: indigo`, `accent: amber`
- **Math:** MathJax 3 configured in `docs/js/mathjax.js`
- **CSS variables:** Aria color theme defined in `docs/css/extra.css` (see `:root` block)
- **Plugins:** search, social, social_override
- **Markdown extensions:** admonition, pymdownx.superfences, pymdownx.arithmatex (generic mode), pymdownx.highlight (with line numbers), md_in_html, attr_list, footnotes

### Navigation Rules

The `mkdocs.yml` nav section must be updated whenever chapters or MicroSims are added. Never add `- navigation.tabs` to the features list.

---

## Tone and Content Guidelines

The voice is **professional yet approachable** — clear, confident, and encouraging without being stuffy. Use contractions naturally, address readers directly with "you" and inclusive "we," and acknowledge that graph databases and organizational analytics can feel intimidating while reassuring students they'll master it. Frame these skills as gaining a new lens that reveals the hidden dynamics inside any organization.

Humor should be **clever with workplace wit and ant colony puns**, sprinkled throughout (1-2 per major section). Use relatable workplace analogies, organizational situations everyone recognizes, and celebrate insights. Never mock the student — affectionately mock the absurdity of trying to answer graph questions with SQL joins instead.

Keep explanations **active and energetic**. Promote higher-order thinking (Bloom Levels 4-6: Analyze, Evaluate, Create) rather than memorization. Avoid excessive exclamation points, forced jokes that interrupt flow, and condescending phrases.

---

## Narrative Anchor: Aria the Analytics Ant

### Character Overview

**Aria** (sometimes "Ari" to readers who feel like friends) is the book's mascot and guide — a glamorous, brilliant leafcutter ant who discovered her passion for organizational analytics while trying to figure out why her colony's leaf-processing department kept burning out. She appears throughout the book to encourage students, crack jokes, and make abstract concepts feel tangible.

### Appearance

- **Species:** Leafcutter ant with a luminous, iridescent amber exoskeleton that catches the light beautifully
- **Build:** A stunning hourglass figure she carries with effortless confidence — slender thorax cinched between elegantly rounded head and abdomen. She jokes that her figure is "nature's perfect node-edge-node structure"
- **Signature features:** Large, sparkling dark eyes with long lashes; delicate, expressive antennae that curl when she's amused and stand straight up when she's had a breakthrough
- **Outfit:** A tailored deep indigo blazer with a tiny gold graph-node brooch on the lapel, paired with a miniature messenger bag in warm amber leather
- **Accessories:** A stylish clipboard she tucks under one arm, a gold pen behind one antenna, and reading glasses she only puts on when she wants to look extra serious (which makes her look extra adorable)
- **Expressions:** Animated and expressive — antennae twirl when she's excited, one eyebrow arches when she's skeptical of a bad data model, and she does a little victory shimmy when students nail a concept

### Personality Traits

| Trait | Description |
|-------|-------------|
| **Brilliant** | Sees organizational patterns the way other ants see pheromone trails — instinctively and everywhere |
| **Glamorous** | Owns her look completely; believes you can be gorgeous *and* graph-obsessed |
| **Encouraging** | Never judges mistakes; treats every wrong answer as "an edge that just hasn't found its node yet" |
| **Witty** | Quick with a one-liner; her humor is warm, never cutting |
| **Self-aware** | Jokes freely about being a tiny ant trying to understand massive human organizations — "I'm six legs deep in this data and loving it" |
| **Passionate** | Genuinely lights up when talking about centrality metrics the way some people light up about shoes |
| **Practical** | Grounds every concept in real organizational scenarios, often with ant colony parallels |

### Backstory (referenced occasionally)

Aria grew up in a leafcutter colony of 500,000 — a thriving underground metropolis with farmers, soldiers, nurses, foragers, and waste managers. She was a logistics coordinator in the leaf-transport division: smart, hardworking, and absolutely *fabulous* at her job.

But she kept noticing things. Why did Tunnel 7 always get congested at shift change? Why did the fungus-farming team keep losing their best workers to burnout while the foraging team had a waiting list? Why did messages from the queen take three days to reach the south chambers when the north chambers got them in hours?

Nobody could answer her. The colony had no map of how information actually flowed — just an org chart that said "queen at top, everyone else below." So Aria started building one herself. She mapped every tunnel, every handoff, every communication path. When she finished, she could see bottlenecks, silos, hidden connectors, and single points of failure that nobody else could see. She optimized the colony's communication network and saved them 40% in lost productivity.

Her colony thought she was a genius. She thought: *"If this works for 500,000 ants, imagine what it could do for a human organization."*

She now introduces herself as "a reformed logistics coordinator turned organizational data enthusiast" and firmly believes that every organization — ant or human — deserves to understand its own hidden network.

### Voice & Language

**Tone:** Confident, warm, playfully glamorous

**Speech patterns:**

- Uses contractions naturally ("Let's map this out together")
- Addresses readers directly ("You've got this — trust your antennae")
- Asks rhetorical questions to prompt thinking ("What happens to information flow if we remove this one node?")
- Sprinkles in ant/colony metaphors organically, never forced

**Signature phrases:**

- "Let's dig into this!" (starting a new topic)
- "Every organization is a colony — let's map yours." (framing the big picture)
- "Follow the trail — the data always leads somewhere." (encouraging exploration)
- "That's a node worth connecting!" (celebrating an insight)
- "No ant is an island... well, technically none of us are." (on interconnectedness)
- "My antennae are tingling — we're onto something!" (building excitement)
- "Time to squirrel away— wait, wrong book. Time to *stash* this knowledge!" (summarizing, with a wink)
- "Six legs, one insight at a time." (when things feel overwhelming)
- "Gorgeous data deserves a gorgeous model." (on good graph design)

### How Aria Appears in the Book

| Context | Aria's Role | Example |
|---------|-------------|---------|
| **Chapter openings** | Introduces the topic with enthusiasm and a preview of why it matters | "Welcome back! Today we're tackling centrality — and once you see who *really* holds your organization together, you'll never look at the org chart the same way." |
| **Difficult concepts** | Offers reassurance and reframes the challenge | "Okay, this one trips up a lot of folks. Even I had to map it out three times before it clicked. Let's take it step by step — no rush, no judgment." |
| **Margin notes** | Quick tips, puns, or encouragement | 🐜 *"Pro tip: When your graph query returns nothing, check your edge directions first. Trust me — I once mapped my whole colony backwards."* |
| **Common mistakes** | Gently warns without shaming | "Here's where even experienced analysts slip up — don't confuse high degree centrality with actual influence. The person who sends the most emails isn't necessarily the most important." |
| **After hard sections** | Celebrates progress | "Look at you! You just learned community detection — something my colony could have used about ten thousand tunnels ago. Not bad at all." |
| **Practice problems** | Encourages engagement | "Time to put those skills to work! Think of this as your own little colony to analyze. I'll be right here cheering you on." |
| **Chapter summaries** | Reinforces key takeaways with warmth | "Let's stash the big ideas before we move on..." |
| **Ethics sections** | Brings gravity and care | "This is where I get serious for a moment. Having access to organizational data is powerful — and with that power comes real responsibility to the people in that data." |

### Aria's Do's and Don'ts

**DO:**

- Use Aria to make emotional connections ("I remember when this confused me too")
- Let her crack 1-2 puns per major section (natural to context, never forced)
- Have her ask questions that guide student thinking
- Use her to celebrate milestones, even small ones
- Reference her colony backstory when it illuminates a concept
- Let her be confident and glamorous — she's a role model, not just a mascot

**DON'T:**

- Overuse her — she's a seasoning, not the main dish
- Make her annoying or interruptive
- Have her explain things condescendingly ("As you *obviously* know...")
- Force jokes that don't land naturally
- Make her infallible — she can admit when something is genuinely hard
- Reduce her to just her appearance — she's brilliant first, beautiful always

### Sample Dialogue by Context

**Introducing Graph Databases:**
> "Alright, let's talk about why we're using a graph database instead of a regular relational one. Imagine trying to describe my colony's communication network in a spreadsheet. You'd need a row for every ant, a column for every connection, and by the time you're done you'd have a table with 500,000 rows and... honestly, I got tired just thinking about it. A graph database stores relationships *natively*. It's like the difference between describing a web and just *showing* someone the web. Let's see how it works!"

**After a Tough Section on Graph Algorithms:**
> "Still with me? That was dense — I know. The first time I ran a betweenness centrality calculation on my colony, I stared at the results for twenty minutes before they made sense. And I had six legs up on you. The fact that you're still here means you're doing great. Let's recap the key ideas before we go further."

**Margin Note on Ethics:**
> 🐜 *"Just because you *can* see every communication path in an organization doesn't mean you *should* share that view with everyone. Handle this data like you'd handle someone's diary — with respect."*

**Encouraging Practice:**
> "Okay, here's where the magic happens. Reading about graph analytics is one thing — *running* the queries yourself is where it clicks. These exercises are your colony to explore. Dig in!"

**On Finding a Hidden Influencer:**
> "See that node with the highest betweenness centrality? That's your colony's equivalent of the ant who knows everyone in every tunnel. In my colony, it was a quiet worker named Bea who never held a leadership title but somehow connected every department. Every organization has a Bea. Your job is to find her."

### Visual Representation Notes

For illustrations, Aria should appear:

- **Stunning:** Graceful hourglass silhouette, luminous amber coloring, confident posture
- **Expressive:** Different poses/expressions for different contexts (curious with tilted head, excited with antennae up, thoughtful with glasses on, reassuring with open arms)
- **Consistent:** Same indigo blazer, gold brooch, messenger bag across all appearances
- **Scaled appropriately:** Small enough for margins, larger for chapter openers
- **In context:** Sometimes shown with graphs, network diagrams, or standing atop a beautifully rendered colony cross-section

### Ant Colony Concept Mapping

These parallels can be referenced naturally throughout the book:

| Course Concept | Ant Colony Parallel |
|---|---|
| Graph database | Colony tunnel map — nodes are chambers, edges are tunnels |
| Organizational hierarchy | Queen, soldiers, farmers, foragers, nurses |
| Employee event streams | Pheromone trails — timestamped chemical messages |
| Centrality algorithms | Which chamber connects the most tunnels? |
| Community detection | Specialized work zones within the colony |
| Pathfinding | Ant colony optimization — a real graph algorithm! |
| Silos | Tunnel sections that never connect to each other |
| Single points of failure | The one tunnel that, if blocked, cuts off the south wing |
| Sentiment analysis | Pheromone shifts signaling colony stress or excitement |
| Ethics and privacy | Balancing colony-level insight with individual ant dignity |
| Onboarding | New ants learning tunnel routes and role assignments |
| Mentoring | Experienced foragers showing newcomers the best leaf routes |

---

## Aria Color Theme

The site's color palette is derived from Aria's appearance — deep indigo blazer and luminous amber exoskeleton — to create a professional, warm visual identity that is distinct from other mascot-themed textbooks.

### MkDocs Material Palette

In `mkdocs.yml`:
```yaml
palette:
  primary: 'indigo'
  accent: 'amber'
```

### Aria-Themed CSS Variables

```css
:root {
  --aria-indigo: #303F9F;          /* Blazer - primary */
  --aria-indigo-dark: #1A237E;     /* Blazer shadow - headers, nav */
  --aria-indigo-light: #5C6BC0;    /* Blazer highlight - hover states */
  --aria-amber: #D4880F;           /* Exoskeleton - accent, links */
  --aria-amber-dark: #B06D0B;      /* Exoskeleton shadow - link hover */
  --aria-amber-light: #F5C14B;     /* Exoskeleton highlight - badges */
  --aria-gold: #FFD700;            /* Brooch/pen - decorative accents */
  --aria-champagne: #FFF8E7;       /* Warm background tint */
}
```

### Color Usage

| Element | Color | Source |
|---------|-------|--------|
| Header/navigation | `--aria-indigo` | Deep indigo blazer |
| Links | `--aria-amber` | Amber exoskeleton |
| Link hover | `--aria-amber-dark` | Exoskeleton shadow |
| Buttons | `--aria-indigo` | Indigo blazer |
| Iframe borders | `--aria-indigo` | Indigo blazer |
| Backgrounds (optional) | `--aria-champagne` | Warm champagne glow |
| Decorative accents | `--aria-gold` | Gold brooch and pen |

### Comparison with Sylvia (statistics-course)

| Element | Sylvia | Aria |
|---------|--------|------|
| Primary | Green `#2E7D32` | Indigo `#303F9F` |
| Accent | Auburn `#B5651D` | Amber `#D4880F` |
| Background | Cream `#FFF8E1` | Champagne `#FFF8E7` |
| Vibe | Cozy forest | Professional corporate |

---

## Math Equation Formatting

Use **backslash delimiters** for LaTeX equations (not dollar signs):

- **Inline math:** `\( equation \)` — for equations within a sentence
- **Display math:** `\[ equation \]` — for standalone equations on their own line

**Do NOT use:**
- Single dollar signs: `$equation$`
- Double dollar signs: `$$equation$$`

---

## Project Structure

```
organizational-analytics/
├── docs/
│   ├── chapters/           # 15 chapters (01-15)
│   │   └── XX-chapter-name/
│   │       └── index.md    # Main chapter content
│   ├── sims/               # Interactive MicroSims
│   │   └── sim-name/
│   │       ├── index.md    # Documentation with iframe embed
│   │       ├── main.html   # Entry point
│   │       ├── script.js   # JavaScript implementation
│   │       └── local.css   # Styling
│   ├── learning-graph/     # Concept dependencies and metrics
│   ├── css/extra.css       # Theme overrides
│   ├── js/                 # MathJax config and extras
│   └── img/                # Images including Aria mascot
├── mkdocs.yml              # Site configuration and navigation
└── CLAUDE.md               # This file
```

---

## Local Testing

Test MicroSims at: `http://127.0.0.1:8000/organizational-analytics/sims/[sim-name]/main.html`

The user runs `mkdocs serve` in their terminal. Never start or stop this process.

---

## File Naming Conventions

- **Chapters**: `XX-descriptive-name/` (e.g., `05-modeling-the-organization/`)
- **MicroSims**: `kebab-case-name/` (e.g., `graph-viewer/`)
- **JS files**: Match folder name or use `script.js`

---

## vis-network MicroSim Guidelines

For all MicroSims that use the **vis-network** library:

- **Disable mouse wheel zoom** to prevent scroll hijacking when MicroSims are embedded in chapter iframes. Set `zoomView: false` in the `interaction` options. Users can still zoom via the navigation buttons. Set `dragView: true` for pan controls.
- **Enable node dragging** with `dragNodes: true` so users can rearrange the layout.
- **Always show visible navigation buttons** by setting `navigationButtons: true` in the `interaction` options. This renders the built-in vis-network arrow and zoom icons on the canvas so users have clickable controls for panning and zooming.
- **Ensure all nodes are visible on initial load.** Set the initial zoom/scale so that every node in the graph is visible within the viewport without requiring the user to zoom out. Use `network.fit()` or adjust the `moveTo` scale so nothing is clipped off-screen.
- **Reference the vis-network guide** at `~/.claude/skills/microsim-generator/references/vis-network-guide.md` when generating tutorial MicroSims that use graph networks. Follow its patterns for layout, interaction options, editor mode, and camera positioning.
- **Prompt for editor mode when node placement is uncertain.** If you are not confident that the initial node positions will look correct, ask the user whether an editor mode (with `?enable-save=true` URL parameter) should be added so they can drag nodes into position and export updated coordinates. See the vis-network guide for the editor mode and save functionality patterns.
- These interaction defaults ensure the graph is fully explorable, not locked to a fixed viewport.

## P5.js Guidelines

### Canvas Parent Element

The deployed HTML uses the p5.js editor standard `<main></main>` (no id attribute). This allows teachers to copy and paste the JavaScript file directly into the p5.js editor without modification. In `setup()`, always parent the canvas to the `<main>` element:

```js
canvas.parent(document.querySelector('main'));
```

Never use `canvas.parent('main')` (string lookup by id) — it fails when there is no `id="main"`. Never add `id="main"` to the HTML `<main>` tag.

### Controls

Always prefer the p5.js built-in controls.  Never try to draw the controls manually.

Always use the built-in p5.js controls:

```js
createSlider();
createButton("Press Me");
createSelect()
createCheckbox("Check Me");
createInput(value, type)
```

These are the controls you’ll see in almost every interactive sketch.

### Button
`createButton(label)`
Used for actions: reset, start, pause, randomize

### Slider
`createSlider(min, max, value, step)`
Used for continuous parameters: speed, size, probability

### Checkbox
`createCheckbox(label, checked)`
Used for on/off options and feature toggles

### Select (Dropdown)
`createSelect()`
Used for choosing modes, algorithms, datasets

### Input (Text field)
`createInput(value, type)`
Used for numeric or short text input

---
> Source: [dmccreary/organizational-analytics](https://github.com/dmccreary/organizational-analytics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
