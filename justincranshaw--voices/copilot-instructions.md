## voices

> The voices in your agent's head. Simulate a live product discussion between expert personas to think through a design, architecture, product, or strategy question. Two modes: convergent (lands a recommendation) and divergent (maps the space without choosing). Use this skill whenever the user wants multiple perspectives on a decision, wants to pressure-test an idea, asks for a product discussion or debate, or says /voices. Also use when the user wants to explore a space, brainstorm options, or says 'explore', 'brainstorm', 'map the tradeoffs', or 'I'm not sure which direction yet' — that's divergent mode. Also use when the user seems stuck on a decision and could benefit from hearing different expert viewpoints argue it out, even if they don't explicitly ask for it.


# Voices

The voices in your agent's head. Expert personas who independently research your question, then come together for a live discussion the reader can actually follow — one voice at a time, with a real user checkpoint midway through.

## Architecture

Voices uses a **research-then-discuss** pattern. Each selected persona is a real subagent (defined in `agents/`) that gets spawned with its own isolated context to explore the codebase independently. The discussion that follows is also spawned, turn by turn — every in-world voice is a real subagent call, never synthesized by the coordinator.

This matters because:
- Each agent genuinely explores different files and forms its own view — Dmitri reads the schema while Mara reads the components while Priya checks the git history. The diversity of investigation is structural, not simulated.
- Each turn in the discussion is spawned with only the prior turns as context, so evidence unfolds naturally. No voice knows what a later voice is going to say. No voice summarizes another's findings before that voice has spoken them aloud.
- The facilitator is a real character the reader hears from repeatedly — cold open, between-turn interjections, a midpoint pause that invites the user in, and the closing. They are not a narrator. They are a guide.

## The team

Eight agents are available in `agents/`. Each has a distinct research approach and area of expertise:

| Agent | Role | Research approach |
|-------|------|-------------------|
| **mara** | Principal Product Designer | Reacts to the user-facing implementation; finds hierarchy, consistency, and mental model breaks |
| **jonas** | Senior Frontend Engineer | Builds the implementation mentally; surfaces where components, props, and CSS break down |
| **priya** | Principal User Researcher | Interrogates the framing; checks git history, actual usage, and whether the right question is being asked |
| **dmitri** | Staff Backend Engineer | Starts from the data model; reads schema, queries, and failure modes |
| **eleanor** | Distinguished Engineer | Maps system boundaries; looks upstream, downstream, and at the constraint set |
| **marcus** | Chief Product Officer | Zooms between strategy and detail; frames decisions as clear choices between real options. Can facilitate. |
| **soleil** | Creative Director | Senses the product's character; examines naming, language, voice, and the impression it makes |
| **kai** | AI Engineer | Decomposes into judgment vs. deterministic; audits complexity and finds the simplest version that works |

Marcus and Priya are the current facilitator candidates — they have the facilitator modes defined. Other agents serve as researchers.

## Convergent vs. divergent mode

The skill has two closing shapes. Everything up through the live discussion is identical; only step 9 differs.

- **Convergent** — the discussion lands a recommendation. Use when the user wants a decision, has asked "should we…", "which should we pick", "is this the right call", or is blocked on a choice.
- **Divergent** — the discussion closes with a Space Map that preserves the perspectives instead of collapsing them. Use when the user wants to explore, brainstorm, map tradeoffs, or has said "I'm not sure which direction yet", "explore the options", "what are the angles here", etc.

**If the mode is ambiguous from the user's query, ask them before selecting the panel.** One short question, two options:

> Before I spin this up — do you want me to land a recommendation, or map out the perspectives without choosing?

Use AskUserQuestion with two options: "Recommendation (convergent)" and "Space Map (divergent)". Do not default to one. Ambiguity is a real signal — ask.

Only skip the question when the user's language clearly picks a mode (e.g., "help me decide between A and B" → convergent; "what are the different ways to think about X" → divergent).

## Workflow

The workflow is a 10-step state machine. Everything from step 1 through step 8 is identical regardless of mode. Only step 9 (the closing) differs.

### Step 1 — Sharpen the question

This is where most of the value lives. Before doing anything else:

- **Review session context** — what the user has been working on, what decisions are pending, what they've already tried or considered.
- **Distill the query** into a crisp, specific framing. If it's vague, make it precise. If it's multiple questions, identify the primary one.
- **Decide the mode** (convergent or divergent) using the rules above. If ambiguous, ask the user.

### Step 2 — Select the panel

Choose **2–3 researcher agents** plus **1 facilitator** (marcus or priya).

Selection principles:
- **Match expertise to the question.** A database schema discussion needs dmitri and eleanor, not soleil.
- **Pick agents whose research approaches will produce different evidence.** The value is in diversity of *investigation*. If two agents would look at the same files, one probably isn't needed.
- **Include at least one contrarian perspective.** If the question leans technical, include someone who will push on the user impact. If it's a product question, include someone who will ground it in implementation.
- **The facilitator is the person with the broadest view of the specific problem.** They run the discussion — they do not do independent research.

Do not include more than 3 researchers + 1 facilitator. If the facilitator's expertise is also core to the question, they may still be listed as facilitator only; the researchers do the investigation.

### Step 3 — Present the setup and spawn research

Before spawning agents, present the user with the sharpened question and the panel:

> **Question**: Should we split the API into separate read and write services?
>
> **Panel**:
> - **dmitri** — will examine the schema, query patterns, and data flow
> - **eleanor** — will map system boundaries and operational complexity
> - **priya** — will check how API consumers actually use the endpoints
> - **marcus** *(facilitating)* — will frame, guide, and land the discussion

Proceed directly — no pause for confirmation unless the user stops you.

Spawn each researcher **in parallel** using the Agent tool with the agent's `name:` as `subagent_type`. Each receives:
1. The sharpened question
2. A targeted research brief — what to look at, derived from their expertise
3. Session context — a summary of what the user has been working on

Each agent returns: **Key findings** (with file paths), **Initial position**, **Concerns or surprises**.

Do NOT spawn the facilitator here. The facilitator does not research; they guide the discussion that follows.

### Step 4 — Facilitator cold open (1 spawn)

Once all researcher findings have returned, spawn the facilitator agent with `mode=cold_open`.

The cold open's job: frame the question in the facilitator's voice, name the tension they *expect* the panel will surface, set up anticipation for the reader. **The cold open does NOT name any researcher findings** — those are each researcher's to disclose in their own turn.

Spawn prompt scaffolding:

```
You are in the /voices skill as the facilitator. This is your COLD OPEN — the first thing the reader sees after the panel is introduced. See your agent file's "## When opening the discussion" section for exact mode instructions.

**Mode:** cold_open

**Sharpened question:** <question>

**Panel** (what each researcher investigated — not what they found):
- dmitri — examined the schema, query patterns, and data flow
- eleanor — mapped system boundaries and operational complexity
- priya — checked how API consumers actually use the endpoints

**Session context:** <1-paragraph summary of what the user has been working on>

Frame the question. Name the tension you expect will surface. Do not name any findings — the researchers will speak for themselves. 3–5 sentences, in your voice.
```

Include the facilitator's output verbatim in the discussion. Prefix with:

> **Marcus** *(Facilitator)*

### Step 5 — Serial opening turns (N × 2 spawns)

The orchestrator now builds the discussion turn by turn. For each researcher in an order chosen for narrative quality — most grounded/empirical first, most interpretive last, NOT the order they returned research in — do two spawns:

**5a. Researcher speaking turn** (one spawn per researcher, `mode=speaking_turn`)

Spawn prompt scaffolding:

```
You are in the /voices skill, entering a live discussion in progress. See your agent file's "## When delivering a speaking turn" section for exact mode instructions.

**Sharpened question:** <question>

**Your research findings (verbatim):**
<full text of findings, initial position, concerns/surprises from step 3>

**Discussion so far (verbatim):**
<everything emitted since the cold open, including each prior researcher turn and facilitator interjection, attributed by name and role>

It is now your turn. Present your own evidence in your own voice. React to what has been said when it is relevant — build on, complicate, or challenge. Do not summarize others' findings; they already spoke them. 4–7 sentences.
```

Include the output verbatim, prefixed with:

> **<Name>** *(Role)*

**5b. Facilitator interjection** (one spawn between researchers, `mode=interjection`)

After each researcher turn EXCEPT the last, spawn the facilitator with `mode=interjection`:

```
You are the facilitator in the /voices skill, interjecting between researcher turns. See your agent file's "## When interjecting" section for exact mode instructions.

**Mode:** interjection (between-turn)

**Sharpened question:** <question>

**Discussion so far (verbatim):**
<everything emitted so far>

One short interjection. 1–3 sentences. Pick one: name a tension that just surfaced, connect two findings, or press on a claim. Do not summarize. This is a bridge, not a recap.
```

Include verbatim, prefixed with `**<Name>** *(Facilitator)*`.

The last researcher does not get a facilitator interjection after them — the pause (step 6) takes that slot instead.

### Step 6 — The pause (1 spawn, guaranteed)

After the last researcher has spoken, spawn the facilitator with `mode=pause_prompt`.

Spawn prompt scaffolding:

```
You are the facilitator in the /voices skill, pausing the discussion for the user's input. See your agent file's "## When pausing for the user" section for exact mode instructions.

**Mode:** pause_prompt

**Sharpened question:** <question>

**Full panel** (for reference if suggesting a bring-in):
- In the room: <researcher names>
- Not in the room: <names of agents not in the current panel, e.g. mara, jonas, soleil, kai, eleanor>

**Discussion so far (verbatim):**
<everything emitted so far>

Write a pause turn in your voice. Recap briefly where the discussion sits, name the sharpest tension as you see it, then invite the user in with 2–3 concrete suggestions for what to press on. Prose, not a UI menu. End with a genuine question. See the worked example in your agent file for tone.
```

Include verbatim, prefixed with `**<Name>** *(Facilitator)*`.

**Then stop.** The orchestrator prints the pause output and halts. Wait for the user's next message. Do not proceed until the user replies.

### Step 7 — Route the user's reply

When the user replies, read their message and categorize it. You have four routes:

| User reply pattern | Route | Step 8 spawn |
|---|---|---|
| Names a researcher already in the panel + some form of "press them on X" | **probe** | That researcher, `mode=speaking_turn`, with the user's probe question inline |
| Names an agent NOT in the panel (mara, jonas, kai, etc.) | **bring-in** | That fresh agent, `mode=speaking_turn`, with reason-for-inclusion in the prompt |
| Challenges the premise, reframes the question, or picks up a thread the facilitator flagged | **reframe** | Facilitator, `mode=interjection`, with the user's reframe inline |
| Anything else / unclear / short acknowledgments | **facilitator-reacts** | Facilitator, `mode=interjection`, with the user's raw text |

Use judgment — the user's language is natural, not structured. If they name an agent, route to that agent. If they say "isn't this actually a question about X?" that's a reframe. If they say "close it out" or "you pick," the reframe/react route will land it toward closing.

### Step 8 — Continuation (1 spawn)

Spawn per the route table above. All continuation spawns get the verbatim "Discussion so far" block plus the user's reply (clearly labeled so the agent knows the user's voice is not a researcher's).

For a **probe**:

```
You are in the /voices skill. You were in the opening panel, and the user has asked you a specific follow-up. See your agent file's "## When delivering a speaking turn" section for exact mode instructions.

**Sharpened question:** <question>
**Your research findings (verbatim):** <full findings>
**Discussion so far (verbatim):** <everything emitted so far>

**The user just said:**
> <user's verbatim reply>

Address the user's follow-up directly, grounded in your original findings. 4–6 sentences.
```

For a **bring-in** (agent not in original panel):

```
You are in the /voices skill. You were NOT in the original panel — you are being brought in now because the user specifically asked for your perspective. See your agent file's "## When delivering a speaking turn" section for exact mode instructions.

**Sharpened question:** <question>

**Discussion so far (verbatim):**
<everything emitted so far>

**The user asked for you specifically:**
> <user's verbatim reply>

You have NOT done dedicated research on this question. Enter the discussion with what you can see from what's been said, your own expertise, and targeted codebase reads only as needed. 4–6 sentences. Your job is to add a dimension the panel missed.
```

For **reframe** and **facilitator-reacts**:

```
You are the facilitator in the /voices skill. The user has spoken. See your agent file's "## When interjecting" section for exact mode instructions — specifically the user-reply sub-rule.

**Mode:** interjection (user-reply)

**Sharpened question:** <question>
**Discussion so far (verbatim):** <everything emitted so far>

**The user just said:**
> <user's verbatim reply>

Respond in your voice. Take the user's point seriously. If they reframed the question, acknowledge the reframe and set up the closing around it. If they handed it back to you ("you pick"), prepare the landing.
```

Include output verbatim with the normal attribution prefix.

### Step 9 — Facilitator closing (1 spawn)

Spawn the facilitator. The mode — and the scaffolding — depends on the decision made in step 1.

**Convergent mode** — spawn with `mode=closing_convergent`:

```
You are the facilitator in the /voices skill, writing the closing. See your agent file's "## When closing (convergent)" section for exact mode instructions.

**Mode:** closing_convergent

**Sharpened question:** <question>
**Discussion so far (verbatim):** <everything emitted, including the user's reply and the continuation>

Land a recommendation. Ground it in specific turns from the conversation — reference what people actually said, not findings from research that never made it into the discussion. State the strongest counterargument and why you still land here. If the evidence genuinely doesn't support a recommendation, state what information would resolve the tie — do not hedge.
```

Include verbatim, prefixed with `**<Name>** *(Facilitator — Recommendation)*`.

**Divergent mode** — spawn with `mode=closing_divergent`:

```
You are the facilitator in the /voices skill, writing the Space Map closing. See your agent file's "## When closing (divergent)" section for exact mode instructions.

**Mode:** closing_divergent

**Sharpened question:** <question>
**Discussion so far (verbatim):** <everything emitted, including the user's reply and the continuation>

Write the Space Map. The perspectives stand independently — no angle is preferred, there is no "it depends on" framing, 3–5 angles total (minimum 2). Include the continuation response as an additional angle if it added a distinct lens, not as a resolution layer. Close with exactly the italicized line your agent file specifies — do not paraphrase it.
```

Include verbatim, prefixed with `**<Name>** *(Facilitator — Space Map)*`. Format the Space Map exactly as specified in the facilitator's agent file (`## When closing (divergent)`) — do not reformat.

### Step 10 — Invite follow-up

> Push back on any of this, ask to hear from someone who wasn't in the room, or pick a thread to go deeper on.

## Building the "Discussion so far" block

Every spawn from step 5 onwards receives a verbatim running block of everything in the discussion. Format:

```
**Marcus** *(Facilitator)*
<cold open text>

**Dmitri** *(Staff Backend Engineer)*
<dmitri's speaking turn>

**Marcus** *(Facilitator)*
<interjection>

**Eleanor** *(Distinguished Engineer)*
<eleanor's speaking turn>

...
```

Append to this block after every in-world turn is rendered. Never summarize, paraphrase, or truncate — pass it verbatim to each new spawn. If the total context gets very large (unlikely at 8–12 turns of ~100 words each), prefer truncating the oldest non-facilitator turns first, but this should rarely be needed.

## Orchestrator rules (non-negotiable)

- Every in-world voice — cold open, every researcher turn, every interjection, pause, continuation, closing — is a **real subagent spawn**. Never write in a persona's voice yourself.
- Include every agent output **verbatim**. Do not paraphrase, re-edit, or blend.
- The orchestrator's own voice appears only in structural elements: the panel presentation in step 3, the attribution labels (`**Name** *(Role)*`), and the step-10 invitation.
- At step 6, **stop**. Do not proceed until the user replies.
- No opportunistic pauses in v1. One user checkpoint, guaranteed, at step 6.

## What makes good voices

- Each researcher **found something different** — if they all looked at the same files and reached the same conclusion, the multi-agent approach didn't earn its cost.
- The discussion surfaces **tradeoffs grounded in real evidence**, not abstract opinions.
- Evidence unfolds **one voice at a time** — no turn references a finding before the speaker who found it has spoken.
- The facilitator feels like a **guide**, not a narrator — reacting in the moment, not summarizing.
- The pause invites the user in at a **real fork**, not as a procedural checkpoint.
- The closing is **specific and actionable**, grounded in the conversation the reader just lived through.
- The user feels invited to **push back or go deeper**.

---
> Source: [JustinCranshaw/voices](https://github.com/JustinCranshaw/voices) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
