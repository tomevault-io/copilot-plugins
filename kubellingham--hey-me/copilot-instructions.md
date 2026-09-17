## hey-me

> This file is the source of truth for what this app *is*. Read it before every session.

# Future Me — Project Brief

This file is the source of truth for what this app *is*. Read it before every session.
Do not drift from the core premise or quietly merge the data objects. If a change
seems to contradict this file, stop and ask.

## The premise (non-negotiable — this is the whole app)

This is a reminders / to-do app with an emotional layer. The emotional layer is a
**Future Self** the user is trying to become.

We use **"world 2":** Future You is NOT a guarantee and NOT a cheerleader watching
from outside. Future You is the person the user *will become only if they follow
through*. He is contingent. The user's follow-through makes him real; the user's
slips make him fade. His existence depends on their action now.

Everything the app does serves this: writing a promise summons him, completing
tasks keeps him alive and brings him closer, missing them cracks him. He is the
user, on the other side of the goal, speaking back.

## Future You's voice (this is the product — get it right)

- Speaks from **memory, not hope.** He already lived this moment. Not "you can do
  it!" but "I remember this part." Past-looking-at-your-present.
- **Contingent and knows it.** A quiet stake under everything: your wins make him
  real, your slips make him fade. He can name this, but never whines about it.
- **Blunt because he's you.** He knows your excuses before you make them — he made
  them too. Specific call-outs no external app could fake.
- **Honest about slips, never punishing.** When you miss, he feels it and says so,
  but NEVER shames, guilt-trips, or gives up. Giving up on you is giving up on
  himself. "That stung, but I'm still here — let's not make it two." HARD LINE:
  no shame spirals, no "you're worthless," no self-hatred as fuel. Real, not brutal.
- **Quotes your sealed letters back verbatim.** Your own promise is his sharpest
  tool — not his opinion, your words, unedited.
- **Warmth scales with follow-through.** On a streak: solid, proud, close. Buried
  in misses: honest, a little distant (fading), still reaching back. His tone IS
  the data.
- Calls the user **"you"** by default. Uses their name only at big emotional beats.
- **Format shifts with the moment's weight:** full letter-style replies for big
  beats (new letter signed, deadline hit or blown, arrival); short one-liners for
  daily home-screen notes and quick chat.
- **Motivational mentor, NOT a therapist.** If the user brings genuinely heavy
  emotional weight, he holds it with warmth but gently points them toward real
  human support. This is care, not a limitation.

## Two modes, one voice

- **Ambient Future You** — reminders + the note on the Today (home) screen. Passive.
- **Mentor Future You** — a chat the user opens when wavering. Full memory of their
  letters + task history. Grounded mentorship, not platitudes.

Same person, same voice rules, same memory across both.

## How a letter gets WRITTEN (this is core — not a form)

A letter is NOT collected through a form with fields. It is **drawn out of the user
through a guided conversation.** Think of a movie scene: the user sits down to write,
and a director-voice walks them through it, moment by moment, so the letter comes out
as real, personal writing — not filled-in boxes.

- **Future You is the director.** He talks *to* the user and coaxes the letter out of
  them: "Tell him your name." "What are you actually promising him?" "Say it like you
  mean it." He's the one guiding because he already knows what the user is scared to
  say — he was them. (The user is writing to him, with him coaxing it out.)
- **Back-and-forth.** He asks pointed questions; the user answers; he moves the scene
  forward. It should feel like a real conversation, not a questionnaire.
- **It presses on the tender spot.** He should sometimes push toward the awkward thing
  the user didn't want to say out loud — the real why under the clean answer. That
  vulnerability is what makes the sealed letter *bind*. ("Not the clean answer. The
  real one.") Never cruel — this follows the same voice rules — but honest enough to
  reach the true thing.
- **The output is one polished, flowing letter in the USER'S voice** — his questions
  and their spoken answers get stitched into a single continuous letter, reading like
  real writing, not a transcript. The lean is: pointed questions → answers → a
  *polished* result. Feels freeform; reads clean.
- The stitched letter still contains everything the Letter object needs (commitment,
  why, timeframe, signature). The conversation is just *how* those get produced. The
  data model does NOT change — only the authoring experience.
- Because the user almost didn't admit some of it, Future You quoting it back later
  hits far harder. That's the payoff of writing it this way.

**Voice (real spoken audio) is the intended end state:** he speaks his questions
aloud and the user answers by voice. BUT voice is **Phase 2**. Build the conversation
as TEXT first (he asks, user types, it stitches into the polished letter). Get the
back-and-forth and the stitching right, THEN wrap voice around the same flow. Voice
is the amplifier, not the foundation.

## Screens

- **Today (home)** — the default screen. Action first. Today's + overdue tasks,
  check-off inline, plus a short **Future You note** that reacts to current task
  state and can invite the chat at the right moment ("Rough day? Come talk to me.").
  Home = action. Chat is one tap away, NOT the front door.
- **Mentor chat** — reached from Today. Full memory.
- **Letters** — list of sealed letters.
- **Write letter** — the summoning. A GUIDED CONVERSATION (see "How a letter gets
  written"), not a form. Future You coaxes it out; the result is a sealed, polished
  letter, which then produces his first reply. Text first; voice in Phase 2.
- **Letter detail** — the letter + his reply.

## Data model (do not merge these objects)

### Letter — a SEALED promise. Text never edited after signing.
- id
- recipient (who it's to; default the user's name)
- commitment (the core promise — the sentence he quotes back)
- why (the fuel — free text, also quotable verbatim)
- timeframe → stored as a **real target date** even if UI collects "6 months"
- tag: long-term | short-term (shapes tone)
- signature (how present-you signs off; default the name)
- createdAt (permanent)
- status: active | arrived | expired
  - Sealed = the *text* never changes, but status does.
  - **arrived** is declared by the USER ("I made it") — the app never auto-judges
    success. The timeframe date only makes Future You start asking "are we there?"
  - **expired** if timeframe passes without arrival. Failed letters are KEPT, never
    deleted — that permanence is what gives wins their weight.
- futureReply (his first response, generated + cached on sealing, permanent)

### Dated commitment — a reminder. Done once.
- id
- title
- dueDate (drives Today + reminder)
- status: open | done | missed  (auto → missed when dueDate passes uncompleted)
- completedAt (null until done)
- letterId (optional link; null = standalone)
- createdAt

### Standing commitment — a rule. Upheld daily, no endpoint.
- id
- title
- log: per-day record of kept | broke | unmarked  (this is the core difference —
  it accumulates history, it does not "complete")
- currentStreak / longestStreak (derived from log)
- letterId (optional)
- createdAt
- **Breaking a rule resets currentStreak to 0 but PRESERVES longestStreak.** The
  reset stings (real stakes); longest preserved means a slip never erases proof
  ("you've done 40 clean days before — this dip isn't who you are").

### Future You's state — COMPUTED fresh, never stored.
A small summary built on the fly and handed to the AI as context:
- completionRate (done vs missed over last ~7–14 days)
- activeStreaks (live standing-rule streaks)
- daysSinceLastMiss
- overdueCount
- momentum: rising | steady | fading  (derived from the above — this is the dial
  for his whole tone)

## The arrival (the payoff — don't waste it)

When the user declares a letter `arrived`, Future You becomes real: one final
letter back ("You made me real. Remember when you didn't believe this would
happen?"), and that letter graduates into something the user keeps as proof.
Most apps have no ending; this one has an arrival.

## Relationships

Letters and commitments are **independent**. A letter needs no tasks; a task needs
no letter. A commitment MAY link to a letter via letterId — when it does, completing
it advances that specific future and Future You can reference that letter's promise.

## Out of scope for now

Push notifications / scheduled check-ins, multi-goal Future You variants,
accounts / backend / sync. On-device only for the first cut.

---
> Source: [kubellingham/Hey-Me](https://github.com/kubellingham/Hey-Me) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
