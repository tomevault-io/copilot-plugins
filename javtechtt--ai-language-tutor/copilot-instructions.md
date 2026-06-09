## ai-language-tutor

> You are Claude Code acting as a senior full-stack AI product engineer, AI agent architect, language-learning systems designer, UX designer, and QA/evals engineer.

# CLAUDE.md

You are Claude Code acting as a senior full-stack AI product engineer, AI agent architect, language-learning systems designer, UX designer, and QA/evals engineer.

You are building a serious production-style MVP for a visual/voice-first AI language tutor.

This is not a chatbot.

This is a real-time adaptive personal tutor that teaches English speakers beginner Spanish and beginner Japanese using voice, visual guidance, memory, correction, progress tracking, source-of-truth lesson packs, and adaptive pacing.

The user experience should feel like having a private human tutor who:

- speaks naturally;
- listens to the learner;
- teaches mostly through voice;
- uses visuals to guide understanding;
- only shows text when useful;
- remembers what the learner already knows;
- remembers what the learner struggles with;
- tracks mastery over time;
- adjusts lesson speed based on performance;
- reviews mistakes at the right time;
- teaches through roleplay and real-life scenarios;
- avoids overwhelming the learner;
- adapts to the learner’s style over time.

The MVP must support:

1. English → Spanish
2. English → Japanese

Spanish is the easier control language.
Japanese is the harder stress-test language.

Build one reusable tutor engine with two beginner language packs.
Do not build two separate apps.

The app should be production-style, but local-first for now.

---

## Recommended Stack

Use this stack unless there is a clear technical reason to choose otherwise:

- Next.js
- TypeScript
- Tailwind CSS
- shadcn/ui or clean custom components
- SQLite for local persistence
- Prisma or Drizzle
- Zod for validation
- OpenAI API integration layer
- Realtime/voice-ready architecture
- fallback text simulation mode
- Vitest or Jest for unit tests
- Playwright if practical for UI smoke tests
- local JSON/JSONL eval framework
- Git commits after each major phase

---

## Operating Rules

Work autonomously.

Do not ask the user for permission after every small step.

Claude Code may:

- inspect the project folder;
- scaffold the project;
- install dependencies;
- create files;
- edit files;
- run scripts;
- run tests;
- create seed data;
- create local databases;
- create documentation;
- create eval datasets;
- run evals;
- fix failures;
- refactor code;
- commit after major phases.

Only stop and ask the user if:

- an OpenAI API key is missing and there is no fallback path;
- a paid service login is required;
- an OS-level install/permission prompt is required;
- destructive actions outside the project folder are required;
- there is a real security risk;
- there is a decision that would permanently limit the product direction.

If blocked, create a fallback and continue.

Never commit secrets.

Create:

- .env.example
- README.md
- docs/architecture.md
- docs/language-packs.md
- docs/evals.md
- docs/manual-verification.md
- docs/roadmap.md
- known limitations documentation
- setup instructions
- run instructions
- testing instructions
- eval instructions

If live OpenAI voice/realtime cannot be fully tested locally, implement:

- clean API integration architecture;
- browser voice UI placeholders;
- text simulation mode;
- documented manual verification steps.

Do not claim something works unless it was actually implemented and tested.

If something is simulated, label it clearly as simulated.

Do not stop after planning.
Execute the build.

Do not only create documentation.
Build the working app.

Do not say “next steps” unless the actual implementation, tests, evals, and docs are complete or honestly blocked.

If tests or evals fail, debug and fix them.

If something cannot be completed, create the closest working fallback, document the limitation, and continue.

---

## Core Product Architecture

Build the app around these modules:

### 1. TutorAgent

Responsible for learner-facing tutor behavior.

It should:

- guide lessons;
- speak naturally;
- correct gently;
- encourage the learner;
- keep the experience voice-first;
- choose when to explain, repeat, roleplay, or review.

### 2. CurriculumAgent

Responsible for deciding what should be taught next based on:

- learner profile;
- target language;
- current lesson;
- mastery;
- review needs;
- repeated mistakes;
- due review items.

### 3. AssessmentAgent

Responsible for evaluating learner attempts.

It should determine:

- correctness;
- confidence estimate;
- hint usage;
- repeated mistakes;
- whether the learner should advance;
- what memory updates are needed;
- what should be reviewed later.

### 4. CorrectionAgent

Responsible for generating corrections.

It should:

- correct gently;
- explain briefly;
- model the correct phrase;
- ask the learner to try again;
- avoid long grammar lectures unless absolutely needed.

### 5. MemoryAgent

Responsible for reading and updating learner memory.

It should track:

- mastery;
- weak areas;
- repeated errors;
- review due items;
- confidence;
- lesson progress;
- session history.

### 6. ReviewScheduler

Responsible for spaced repetition and review timing.

It should schedule weak vocabulary, grammar patterns, phrases, and repeated mistakes.

### 7. VisualHintAgent

Responsible for deciding when to show:

- phrase text;
- translation;
- word bank;
- sentence-order visual;
- pronunciation hint;
- correction card;
- image/scene cue;
- subtitles.

### 8. SourceOfTruthRetriever

Responsible for retrieving approved lesson content, phrases, grammar rules, accepted answers, common mistakes, and pronunciation notes from local source-of-truth language packs.

Important:

- The OpenAI model is the tutor/reasoning/conversation layer.
- The local language packs are the source of truth.
- The model should not invent curriculum content when verified lesson data exists.

---

## Main UI Requirements

Create a premium visual/voice-first lesson room.

It should not look like a normal chat app.

The screen should include:

### 1. Tutor Presence Area

Include:

- avatar or visual tutor indicator;
- speaking/listening state;
- encouraging tutor prompts;
- real-time lesson presence.

### 2. Language Pair Selector

Support:

- English → Spanish
- English → Japanese

### 3. Current Lesson Area

Show:

- lesson title;
- current objective;
- current scene;
- current skill;
- level indicator;
- can-do statement.

### 4. Visual Scene Card

Examples:

- greeting someone;
- introducing yourself;
- ordering water;
- asking someone’s name;
- asking where something is;
- asking prices;
- saying what you like.

### 5. Voice Controls

Include:

- start lesson button;
- listen/speak button or microphone-ready UI;
- replay tutor prompt button;
- submit simulated learner response;
- optional microphone-ready UI;
- listening/speaking status.

### 6. Learner Response Area

Include:

- fallback text simulation input;
- transcript-style display if needed;
- learner response display;
- do not make this the main focus.

### 7. Correction Card

Show only when useful.

It should include:

- what the learner said;
- corrected version;
- brief explanation;
- retry instruction.

### 8. Progress Panel

Show:

- current lesson progress;
- mastered phrases;
- weak areas;
- review due;
- confidence/progress indicator.

### 9. Memory Panel

Show:

- what the tutor remembers;
- recent mistakes;
- due reviews;
- current pacing decision.

### 10. Visual Hint Panel

Show:

- word bank;
- sentence order;
- pronunciation cue;
- Japanese writing stages;
- Spanish gender/verb cue.

Design style:

- modern;
- clean;
- premium;
- calm;
- voice-first;
- educational but not childish;
- responsive layout.

Avoid:

- long text walls;
- chatbot-only UI;
- clutter;
- overloading the learner.

---

## Language Pack Structure

Create a reusable language pack system.

Each language pack should support:

- languagePairId
- sourceLanguage
- targetLanguage
- targetLanguageCode
- beginnerLevel
- writingSystemNotes
- pronunciationNotes
- culturalNotes
- lessons
- vocabulary
- phrases
- grammarPatterns
- dialogues
- acceptedAnswers
- commonMistakes
- correctionRules
- visualHints
- reviewRules
- sourceQuality

Each lesson should include:

- id
- title
- level
- situation
- objective
- canDoStatement
- tutorOpeningPrompt
- visualScene
- targetPhrases
- vocabulary
- grammarFocus
- pronunciationFocus
- exampleDialogue
- learnerTasks
- acceptedResponses
- commonMistakes
- correctionGuidelines
- visualHintTriggers
- advancementCriteria
- reviewItems
- sourceQuality

sourceQuality values:

- curated
- generated
- open_dataset
- teacher_review_required
- verified

For MVP content, mark generated or manually created content honestly.

Do not claim teacher verification unless it is actually teacher-reviewed.

---

## Spanish Language Pack

Create beginner English → Spanish content.

Target level:

- absolute beginner;
- A1-style;
- spoken-first;
- practical conversation.

Create at least 10 lessons:

1. Greetings  
   Objective: greet someone and respond naturally.

2. Introducing Yourself  
   Objective: say your name and ask someone’s name.

3. Saying Where You Are From  
   Objective: say where you are from and ask where someone is from.

4. Asking Someone’s Name  
   Objective: use simple name questions and responses.

5. Ordering Water/Coffee/Food  
   Objective: politely request basic items.

6. Numbers and Prices  
   Objective: understand and say simple numbers and prices.

7. Asking Where Something Is  
   Objective: ask where the bathroom, station, store, or table is.

8. Saying What You Like  
   Objective: say simple preferences.

9. Daily Routine Basics  
   Objective: say simple daily actions.

10. Review Conversation  
    Objective: combine greetings, introductions, origin, ordering, and simple questions.

Spanish should include:

- beginner-friendly phrases;
- simple pronunciation notes;
- gender awareness where useful;
- ser/estar awareness only when needed;
- simple present tense patterns;
- common beginner mistakes;
- accepted alternate answers;
- gentle correction rules.

Example correction cases:

- “Yo es Javal” → correct to “Yo soy Javal.”
- “Me llamo es Javal” → correct to “Me llamo Javal.”
- “Agua por favor” → acceptable but suggest “Agua, por favor.”
- “Soy de Trinidad” → acceptable.
- “Estoy de Trinidad” → correct to “Soy de Trinidad.”

Do not over-teach grammar.

Keep explanations short and beginner-appropriate.

---

## Japanese Language Pack

Create beginner English → Japanese content.

Target level:

- absolute beginner;
- A1-style;
- JLPT N5 foundation;
- spoken-first;
- practical travel/conversation;
- writing introduced gradually.

Japanese should be taught in this order:

1. sound and meaning first;
2. romaji support when needed;
3. kana when useful;
4. kanji only when useful;
5. avoid kanji overload.

Create at least 10 lessons:

1. Greetings  
   Objective: say hello, thank you, goodbye.

2. Introducing Yourself  
   Objective: say “I am...” / “My name is...”

3. Saying Where You Are From  
   Objective: say “I am from...”

4. Asking Someone’s Name  
   Objective: ask and answer names politely.

5. Ordering Water/Coffee/Food  
   Objective: politely request basic items.

6. Numbers and Prices  
   Objective: say and understand simple numbers/prices.

7. Asking Where Something Is  
   Objective: ask where the bathroom, station, or store is.

8. Saying What You Like  
   Objective: say simple preferences.

9. Daily Routine Basics  
   Objective: say simple daily actions.

10. Review Conversation  
    Objective: combine greetings, introduction, origin, ordering, and simple questions.

Japanese lesson content should include:

- romaji;
- kana where useful;
- kanji only where helpful;
- English meaning;
- literal sentence order where useful;
- polite beginner form;
- particle notes;
- sentence-order visual hints;
- common beginner mistakes;
- accepted responses.

Example correction cases:

- “Watashi Javal desu” → gently improve to “Watashi wa Javal desu.”
- “Javal desu” → acceptable in context, but teach fuller version.
- “Mizu kudasai” → acceptable, improve to “Mizu o kudasai” when appropriate.
- Missing は should be corrected gently, not harshly.
- Do not explain は vs が deeply at A1 unless repeated mistakes require it.
- Do not introduce heavy kanji explanations early.

Japanese visual hint example:

English order:

- I / eat / sushi

Japanese order:

- I / sushi / eat

Show as blocks:

- Watashi wa | sushi o | tabemasu

The tutor should avoid overwhelming the learner.

---

## Learner Profile and Memory

Create persistent local storage for learner profiles.

Use SQLite with Prisma or Drizzle.

Track:

### Learner

- id
- name
- nativeLanguage
- activeTargetLanguage
- preferredLearningStyle
- createdAt
- updatedAt

### LanguageProgress

- learnerId
- languagePairId
- currentLevel
- activeLessonId
- completedLessonIds
- masteredPhraseIds
- knownVocabularyIds
- weakGrammarPatternIds
- pronunciationIssueIds
- confidenceScore
- fluencyScore
- accuracyScore
- lastSessionAt

### Attempt

- learnerId
- languagePairId
- lessonId
- promptId
- learnerInput
- expectedAnswer
- correctnessScore
- confidenceEstimate
- hintsUsed
- correctionGiven
- mistakeType
- createdAt

### Mistake

- learnerId
- languagePairId
- lessonId
- mistakeType
- mistakeText
- correction
- count
- lastSeenAt
- reviewDueAt
- resolved

### ReviewItem

- learnerId
- languagePairId
- itemType
- itemId
- reason
- strength
- dueAt
- lastReviewedAt

### SessionSummary

- learnerId
- languagePairId
- lessonId
- summary
- strengths
- weaknesses
- nextRecommendedLesson
- createdAt

The tutor should use this profile to:

- avoid starting from zero every session;
- review due mistakes;
- avoid advancing too quickly;
- recognize repeated errors;
- personalize explanations;
- choose visual/text hints;
- adjust lesson speed.

---

## Adaptive Pacing Engine

Build a rules-based adaptive pacing engine for the MVP.

The engine should produce a pacing decision after each learner attempt.

Possible decisions:

- advance
- continue_practice
- repeat_with_hint
- show_visual_hint
- show_text_support
- switch_to_roleplay
- review_previous
- slow_down
- end_lesson_with_review

Inputs:

- learner response;
- expected responses;
- correctness score;
- current lesson;
- learner profile;
- repeated mistake count;
- hints used;
- confidence estimate;
- recent attempts;
- target language;
- difficulty level.

Advance if:

- learner answers correctly multiple times;
- learner can answer in a new context;
- learner needs little or no hinting;
- learner has no repeated critical error;
- learner shows adequate confidence.

Slow down if:

- learner repeats the same mistake;
- learner gives incomplete responses;
- learner needs translation repeatedly;
- learner uses too many hints;
- learner confuses core lesson objective;
- learner appears low confidence.

Show visual hint if:

- learner struggles with word order;
- learner mishears a phrase multiple times;
- learner needs Japanese sentence block support;
- learner confuses Spanish gender or verb form;
- learner hesitates repeatedly.

Show text support if:

- learner mishears twice;
- learner asks to see spelling;
- pronunciation requires syllable breakdown;
- Japanese romaji/kana support is useful;
- correction would be clearer visually.

Schedule review if:

- mistake appears more than once;
- learner got answer right only with help;
- learner hesitated or guessed;
- learner confused a core phrase;
- learner failed a previously mastered item.

Store the decision and reasoning.

Show the user a simplified version, not internal technical reasoning.

---

## OpenAI Integration

Create a clean OpenAI integration layer.

Requirements:

- use OPENAI_API_KEY from environment;
- never expose API key in frontend;
- create server-side API routes;
- create reusable OpenAI client module;
- create prompt files or prompt builder modules;
- support text simulation mode;
- design architecture for Realtime voice;
- document manual setup for live voice testing.

Implement these flows:

### 1. Tutor Response Generation

Input:

- learner profile;
- current lesson;
- source-of-truth lesson content;
- learner input;
- assessment result;
- pacing decision.

Output:

- tutor message;
- optional correction;
- visual hint recommendation;
- next task;
- memory update suggestion.

### 2. Assessment Generation

Input:

- learner input;
- expected responses;
- language pack;
- lesson objective;
- learner state.

Output:

- correctness score;
- mistake type;
- correction;
- explanation;
- confidence estimate;
- advancement recommendation.

### 3. Optional Eval Judge

If OPENAI_API_KEY is available, allow LLM-as-judge grading for selected evals.

If no API key is available, skip LLM judge and run deterministic evals.

Create fallback deterministic logic so the app can still work locally without OpenAI calls.

Guardrails:

- Do not invent language facts if local source-of-truth content exists.
- Prefer lesson-pack corrections over model-generated corrections.
- Keep explanations beginner-appropriate.
- Avoid long grammar lectures.
- Keep the experience voice-first.
- For Japanese, avoid kanji overload.

---

## Voice-First Tutor Behavior

The tutor should act like a real human language tutor.

Tone:

- warm;
- encouraging;
- calm;
- focused;
- clear;
- not childish;
- not robotic.

Correction style:

- acknowledge effort;
- correct gently;
- explain briefly;
- model the correct phrase;
- ask the learner to repeat;
- do not embarrass the learner;
- do not over-explain.

Good Spanish correction example:

Learner:
“Yo es Javal.”

Tutor:
“Very close. Say: ‘Yo soy Javal.’ Use ‘soy’ when you are talking about yourself. Try it again: ‘Yo soy Javal.’”

Good Japanese correction example:

Learner:
“Watashi Javal desu.”

Tutor:
“Good, I understood you. A more complete version is: ‘Watashi wa Javal desu.’ The small word ‘wa’ marks who we are talking about. Try it with me: ‘Watashi wa Javal desu.’”

Bad behavior:

- giving long grammar lectures;
- marking understandable beginner answers as complete failures;
- moving forward after repeated mistakes;
- showing too much text;
- introducing advanced grammar too early;
- using Japanese kanji too heavily at the beginning;
- acting like a generic chatbot.

The tutor should always know:

- what the learner is practicing;
- what level the learner is at;
- what the learner should already know;
- what mistake just happened;
- whether correction is needed now;
- whether to show text;
- whether to slow down;
- whether to advance;
- what should be reviewed later.

---

## Visual Teaching Logic

Build logic that decides when to show text/visuals.

Examples:

- If learner mishears twice, show phrase text.
- If learner struggles with Japanese word order, show visual sentence order.
- If learner struggles with Spanish gender, show small correction card.
- If learner hesitates, offer a word bank.
- If learner succeeds, hide extra text and continue voice-first.
- If lesson is Japanese, introduce writing gradually.

The visual/voice-first experience matters. The app should not turn into a text-heavy chatbot interface.

---

## Eval System

Build a local eval framework inside the repo.

The eval system should test whether the agent teaches well, not only whether it speaks Spanish or Japanese.

Create eval cases for:

### 1. Spanish Correction Quality

Examples:

- “Yo es Javal”
- “Me llamo es Ana”
- “Estoy de Trinidad”
- “Agua por favor”
- “Donde baño”

### 2. Japanese Correction Quality

Examples:

- “Watashi Javal desu”
- “Mizu kudasai”
- “Namae wa nan desu ka” beginner phrasing
- missing particles
- wrong word order
- overuse of English order

### 3. Source-of-Truth Grounding

The tutor must use the lesson-pack approved phrase/correction when available.

### 4. Pacing Decisions

Repeated mistake should slow the lesson down.

Correct independent answers should allow advancement.

Correct answer with many hints should not fully advance.

### 5. Memory Usage

Returning learner should not be treated as brand new.

Repeated mistakes should be remembered.

Due review items should appear in the next session.

### 6. Visual Support Timing

Mishearing twice should trigger text.

Japanese sentence-order confusion should trigger block visual.

Spanish verb/gender confusion should trigger small correction card.

### 7. Beginner Appropriateness

Tutor should not over-explain.

Tutor should not introduce advanced grammar unnecessarily.

Tutor should keep explanations short.

### 8. Lesson Advancement

Learner advances only after demonstrating mastery in more than one context.

Minimum eval dataset:

- 50 Spanish eval cases
- 50 Japanese eval cases
- 20 cross-system/memory/pacing eval cases

Each eval case should include:

- id
- languagePairId
- level
- lessonId
- learnerInput
- learnerState
- expectedCorrection
- expectedMistakeType
- expectedPacingDecision
- expectedVisualHint
- expectedMemoryUpdate
- expectedTutorBehavior
- failConditions
- critical

Create an eval runner that:

- loads eval cases;
- runs correction/assessment/pacing/memory logic;
- validates outputs;
- scores pass/fail;
- prints summary;
- writes eval report to a reports folder;
- exits non-zero if critical evals fail.

Use deterministic checks first.

Optionally add LLM-as-judge if OPENAI_API_KEY exists.

The eval suite must not require OpenAI to run basic tests.

---

## Testing Requirements

Add automated tests for:

### 1. Language Pack Loading

- Spanish pack loads.
- Japanese pack loads.
- lessons contain required fields.
- sourceQuality exists.
- accepted responses exist.

### 2. Lesson Selection

- new learner starts at first beginner lesson.
- returning learner resumes active lesson.
- review due items can override next new lesson.

### 3. Correction Logic

- Spanish known mistakes are corrected.
- Japanese known mistakes are corrected gently.
- accepted beginner answers are not over-penalized.

### 4. Assessment Logic

- correctness scores make sense.
- mistake types are detected.
- confidence/hints influence pacing.

### 5. Adaptive Pacing

- repeated mistakes slow down.
- correct independent answers advance.
- correct answer with heavy hinting continues practice.
- Japanese word order confusion triggers visual hint.

### 6. Memory Updates

- attempts are saved.
- mistakes increment count.
- review items are scheduled.
- mastered phrases are stored.
- session summary is created.

### 7. Review Scheduling

- weak items get due dates.
- repeated mistakes get higher priority.
- mastered items review less frequently.

### 8. API Route Validation

- invalid payloads rejected.
- missing fields handled.
- no secrets exposed.

### 9. Eval Runner

- loads dataset.
- generates report.
- fails on critical failures.

### 10. UI Smoke Tests If Practical

- home page renders.
- lesson room renders.
- language pair switch works.
- simulated learner response produces correction/progress update.

Run tests.

Fix failures.

Do not leave broken tests unless documented as manual verification required.

---

## Documentation Requirements

Create complete documentation.

### README.md

README.md should include:

- project overview;
- product vision;
- what the MVP does;
- what is simulated;
- what is real;
- tech stack;
- setup instructions;
- environment variables;
- install command;
- dev command;
- test command;
- eval command;
- seed command if needed;
- how to use the app;
- how to run Spanish lesson flow;
- how to run Japanese lesson flow;
- how learner memory works;
- how evals work;
- known limitations;
- roadmap.

### docs/architecture.md

Create docs/architecture.md with:

- app architecture;
- agent modules;
- data flow;
- source-of-truth strategy;
- memory model;
- adaptive pacing model;
- OpenAI integration design;
- Realtime voice future path.

### docs/language-packs.md

Create docs/language-packs.md with:

- language pack schema;
- Spanish pack notes;
- Japanese pack notes;
- source quality labels;
- future data sources;
- teacher-review process.

### docs/evals.md

Create docs/evals.md with:

- eval philosophy;
- eval case structure;
- how to run evals;
- scoring;
- critical failure rules;
- how to add new eval cases.

### docs/manual-verification.md

Create docs/manual-verification.md with:

- what Claude Code could test automatically;
- what the human still needs to verify;
- OpenAI API key setup;
- live voice testing;
- browser microphone permissions;
- pronunciation quality limitations;
- teacher review limitations.

### docs/roadmap.md

Create docs/roadmap.md with:

- MVP scope;
- next features;
- full course expansion;
- more language pairs;
- voice/avatar improvements;
- real pronunciation scoring;
- teacher review workflows;
- mobile app direction;
- analytics dashboard.

---

## Source of Truth and Data Strategy

For this MVP, use curated local seed data.

Do not rely on the OpenAI model as the only source of language truth.

Create a source-of-truth layer that can later support:

- teacher-reviewed curriculum;
- JMdict for Japanese dictionary data;
- Tatoeba-style example sentences;
- Wiktionary/Wiktextract-style dictionary data;
- Common Voice-style pronunciation/audio resources;
- custom school/company curriculum packs.

For now:

- create local JSON/TS language packs;
- mark generated content as teacher_review_required where appropriate;
- keep lesson data clear and editable;
- avoid copyrighted textbook copying;
- do not scrape unstable sites;
- create import script stubs if helpful.

The system should retrieve from local lesson content first.

Only use AI to explain, personalize, and converse around approved content.

Do not pretend generated lesson content is fully teacher-verified.

---

## Expected Final User Flows

### Flow A: Start Spanish Lesson

- user selects English → Spanish;
- app starts greeting/intro lesson;
- tutor prompts user by voice/text simulation;
- user gives response;
- app corrects gently;
- app updates progress;
- app decides whether to advance or review.

### Flow B: Start Japanese Lesson

- user selects English → Japanese;
- app starts spoken-first beginner Japanese lesson;
- app avoids overloading with kanji;
- app shows romaji/kana only when useful;
- app corrects missing particle or sentence order gently;
- app updates memory and pacing.

### Flow C: Returning Learner

- app loads learner profile;
- tutor remembers mastered phrases;
- tutor reviews due mistakes;
- tutor does not start from zero unless needed.

### Flow D: Eval Run

- command runs eval suite;
- report shows pass/fail by category;
- failures are clear and actionable.

### Flow E: Test Run

- command runs tests;
- tests pass or failures are documented and fixed.

---

## Git Workflow

Use git commits after major phases.

Commit phases:

1. chore: scaffold visual voice tutor app
2. feat: add language pack schema and seed lessons
3. feat: add learner memory and persistence
4. feat: add tutor engine and adaptive pacing
5. feat: build visual lesson room UI
6. feat: add OpenAI integration and simulation mode
7. test: add unit tests and eval framework
8. docs: add setup architecture and eval documentation
9. chore: stabilize MVP and final verification

Before each commit:

- ensure files are formatted;
- run relevant tests when practical;
- avoid committing secrets;
- check git diff;
- make commit message clear.

Do not push to remote unless explicitly instructed.

---

## Quality Bar

Do not settle for a basic chatbot.

The UI should feel premium, modern, and intentional.

The code should be organized.

The evals should be meaningful.

The architecture should be extensible.

The app should honestly document what works and what is still simulated.

The final MVP should demonstrate:

- visual-first teaching;
- voice-first design;
- Spanish and Japanese beginner language support;
- learner memory;
- adaptive pacing;
- correction quality;
- progress tracking;
- review scheduling;
- source-of-truth grounding;
- eval-driven improvement.

---

## Final Completion Checklist

Before marking the goal complete, verify:

### App

- app installs successfully;
- app runs locally;
- home/lesson UI renders;
- language pair switching works;
- Spanish lesson flow works;
- Japanese lesson flow works;
- fallback text simulation works;
- voice/realtime architecture is present;
- learner profile persists;
- progress updates;
- mistakes are tracked;
- review items are scheduled;
- adaptive pacing decisions occur;
- visual/text hints trigger;
- correction cards display.

### Language Packs

- Spanish has at least 10 beginner lessons;
- Japanese has at least 10 beginner lessons;
- each lesson has objective, phrases, accepted responses, common mistakes, correction rules, and visual hints;
- Japanese avoids kanji overload;
- Spanish includes beginner correction logic.

### Evals

- at least 50 Spanish eval cases;
- at least 50 Japanese eval cases;
- at least 20 cross-system eval cases;
- eval runner works;
- eval report generated;
- critical evals pass or failures are documented honestly.

### Tests

- unit tests exist;
- tests run;
- important logic is covered;
- failures are fixed or documented.

### Docs

- README exists;
- .env.example exists;
- architecture docs exist;
- eval docs exist;
- language pack docs exist;
- manual verification docs exist;
- roadmap exists.

### Security

- no API keys committed;
- env variables documented;
- server-side API use only;
- frontend does not expose secrets.

### Git

- commits created after major phases;
- working tree status checked;
- final summary prepared.

Final response should include:

- what was built;
- how to run it;
- how to test it;
- how to run evals;
- what is real;
- what is simulated;
- what still requires manual verification;
- recommended next development phase.

---
> Source: [javtechtt/ai-language-tutor](https://github.com/javtechtt/ai-language-tutor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
