## anti-ai-exam-skill

> |


# Anti-AI Exam Designer

Design exams that make AI answers **detectably wrong** — rewarding students who attend class, read carefully, and understand the material.

Based on adversarial testing against Claude, ChatGPT, and Codex.

---

## Orchestrator

On invocation, determine the entry state and route accordingly.

### Step 1: Check for Existing Profile

Look for an `anti-ai-profiles/` directory in the current working directory.

**If no profile exists** (new professor):
- Announce: "No course profile found. Let's set one up — I'll ask a few questions about your course, then generate tailored anti-AI strategies."
- Proceed to **Phase 1: Consulting**

**If profile exists** (returning professor):
- Read `anti-ai-profiles/` subdirectories to list available courses
- Present menu using `AskUserQuestion`:

```
Welcome back! I found profiles for: [list courses]

What would you like to do?
1. Design a new exam (using existing strategies)
2. Audit exam questions against AI
3. Update your course profile / strategies
4. Add a new course
```

- Option 1 → Phase 3 (Output Generation), loading existing strategies
- Option 2 → Phase 2 (Adversarial Audit)
- Option 3 → Phase 1 (Consulting), updating existing profile
- Option 4 → Phase 1 (Consulting), new course

**If professor pastes exam questions directly** (no menu needed):
- Detect question content in the message
- Ask which course profile to use (if multiple exist) or create quick profile
- Proceed to **Phase 2: Adversarial Audit**

### Step 2: Profile Directory Structure

All course data is stored in `anti-ai-profiles/<course-code>/`:

```
anti-ai-profiles/<course-code>/
├── profile.json          # Course metadata, subject, level, logistics
├── strategies.md         # Active trap strategies for this course
├── trap-history.json     # Which traps used per exam (avoid reuse)
├── notation.md           # Custom notation definitions (if applicable)
└── exams/
    └── <exam-name>/
        ├── exam.docx         # (optional) Generated exam document
        ├── solution.docx     # (optional) Solution key
        ├── rubric.md         # (optional) Grading rubric with AI-flag scoring
        ├── ai-baseline.md    # (optional) What AI answered per question
        ├── audit-report.md   # Adversarial audit results
        └── detection-checklist.md  # Per-answer + cross-student red flags
```

---

## Phase 1: Consulting (Adaptive Interview)

**Goal:** Build the course profile and generate tailored anti-AI strategies.

Ask questions **one at a time** using `AskUserQuestion`. Prefer multiple choice. Be conversational, not bureaucratic.

### Core Questions (always ask)

**Q1: Subject Area**
```
What CS course is this for?
- Databases (SQL, relational algebra, schema design)
- Data Structures (arrays, trees, graphs, linked lists)
- Algorithms (sorting, searching, complexity, proofs)
- Computer Architecture (assembly, CPU design, memory)
- Networking (protocols, TCP/IP, packet analysis)
- Operating Systems (scheduling, memory management, synchronization)
- Programming Languages (OOP, functional, systems programming)
- Other (describe your course)
```

**Q2: Course Level**
```
What level?
- Introductory (first exposure to the topic)
- Intermediate (builds on prerequisites)
- Advanced (senior/graduate level)
```

**Q3: Exam Format**
```
How do students take the exam?
- Written on paper (in-person, handwritten answers)
- Typed on computer (in-person, digital submission)
- Take-home exam
- Mixed (some in-person, some take-home)
```

**Q4: Class Size**
```
Approximately how many students?
- Small (under 25)
- Medium (25-60)
- Large (60-150)
- Very large (150+)
```

### Adaptive Branch (subject-specific, ask only when there's a payoff)

Based on the subject selected in Q1, ask follow-up questions that unlock subject-specific traps:

**Databases:**
- "Do students write SQL, Relational Algebra, or both on exams?"
- "Do you use standard RA symbols (sigma, pi, bowtie) or custom notation?"
- "Do you have a custom schema you use across the semester?"

**Data Structures:**
- "Do students write pseudocode or real code? Which language?"
- "Do you test tracing/dry-run (step through an algorithm by hand)?"
- "Do you define custom method names or use textbook standard ones?"

**Algorithms:**
- "Are exam questions proof-based, implementation-based, or analysis-based (Big-O)?"
- "Do you use a specific format for recurrence relations or proof structure?"

**Architecture:**
- "Which assembly dialect? (MARIE, MIPS, x86, ARM, other)"
- "Do students hand-assemble (convert to binary) on exams?"
- "Do you use custom opcode mnemonics or standard ones?"

**Networking:**
- "Do students analyze protocol headers, trace packets, or write configs?"
- "Do you use a specific diagram notation for network topologies?"

**OS:**
- "Which topics appear on exams? (scheduling, memory management, synchronization, file systems)"
- "Do students fill in trace tables (Gantt charts, page tables, etc.)?"
- "Do you use custom state labels or standard ones?"

**Programming:**
- "Which language(s)?"
- "Do you require specific coding style (naming conventions, comment format, structure)?"
- "Do students explain their approach in English before/alongside code?"

### Context Questions (ask only if useful)

Only ask these if they unlock additional strategies:

- "Have you tried any anti-cheating approaches before? What worked or didn't?" (avoid recommending what already failed)
- "Can you do oral follow-ups with flagged students?" (depends on class size — skip if >150)
- "Any constraints? (e.g., department requires single version, exam is take-home, no in-person component)"

### After Interview: Generate Profile and Strategies

1. Create `anti-ai-profiles/<course-code>/profile.json` (see Profile Schema section)
2. Select applicable strategies from **Universal Strategies** + the relevant **Subject Module**
3. Filter by prerequisites:
   - Skip "verbal correction trap" if exam is take-home
   - Skip "oral follow-up" suggestions if class is very large (150+)
   - Skip "version watermarking" if department prohibits multiple versions
4. Rank strategies by: catch rate (highest first), then implementation difficulty (lowest first)
5. Generate `strategies.md` with:
   - Each strategy: name, catch rate, how it works, implementation example using the professor's course content, detection flags, prerequisites
   - Recommended combination (layer 3-4 independent traps for near-zero AI success rate)
6. If custom notation strategies are selected, generate `notation.md` with suggested custom symbols

Present the strategies to the professor for review. Ask:
```
Here are the recommended strategies for your course. Would you like to:
1. Proceed with all of these
2. Remove some strategies
3. Hear more details about a specific strategy
4. Adjust and regenerate
```

After approval, save files and announce: "Profile saved. You can now design an exam (Phase 3) or audit existing questions (Phase 2)."

Proceed to Phase 2 or Phase 3 based on professor's choice.

---

## Universal Strategies

These work across ALL CS subjects. Always include in strategy recommendations.

### Version Watermarking (~100% catch rate)

**How it works:** Print 2-3 versions of the exam with different constants (years, names, thresholds, values). Distribute alternating versions.

**Implementation:**
- Choose 3-4 values that appear in problems (years, prices, names, array sizes)
- Create Version A and Version B with different values
- Same structure, same difficulty, different constants

**Detection:**
- Student with Version A paper submits Version B constants → copied
- Multiple students submit identical AI-generated structure with different constants → structure is the flag

**Prerequisites:** In-person exam, ability to print multiple versions

---

### Verbal Correction Trap (~100% catch rate)

**How it works:** Print a deliberate error in the exam (misspelled column name, swapped attribute, wrong constant). Announce the correction verbally at exam start.

**Implementation:**
- Choose one element to misspell or swap (e.g., column "aname" should be "name")
- At exam start: "On question 3, 'aname' should be 'name'"
- All students present hear it; AI never sees it

**Detection:** Every AI-generated answer will use the wrong name. Instant flag.

**Prerequisites:** In-person exam only. Not for take-home.

---

### Step-by-Step Format Requirement (~90% catch rate)

**How it works:** Require intermediate steps with labeled assignments, not single nested expressions.

**Implementation:**
- Print on exam: "For each answer, show intermediate steps using assignment notation, then the final expression."
- Example format: "R1 <- ..., R2 <- ..., Result <- ..."

**Detection:**
- AI produces single clean nested expressions
- Students who follow instructions write labeled steps
- Missing intermediate steps = likely AI-generated
- Also enables partial credit for correct intermediate steps

**Prerequisites:** Applicable to any subject with multi-step solutions

---

### Cross-Student Pattern Detection

**How it works:** Not a trap — a detection framework applied during grading.

**Red flags per answer:**
- No scratch work or erasures (suspiciously clean)
- Single expression instead of required step-by-step
- Standard notation when custom was required
- Symbols not taught in class (e.g., gamma for aggregation)
- Inconsistent quality (perfect easy questions, wrong hard ones — or vice versa)

**Cross-student red flags:**
- Version constant mismatch
- Identical variable names across students
- Same unusual error in multiple submissions (same AI session)

**Follow-up protocol (3+ flags):**
1. Oral follow-up: ask student to explain their hardest answer step by step
2. Compare submission against saved AI baseline (from Phase 2)
3. Document pattern for academic integrity proceedings

---

## Subject Strategy Modules

### MODULE: Databases

#### Undefined Semantics Trap (~75% catch rate)

**How it works:** Design the schema so a column name *suggests* one meaning, but teach a *different* meaning in class. Do NOT print the real meaning on the exam.

**Example:**
```
Schema: artist(aid, name, affiliation), draw(aid, pid, year), painting(pid, title, year, price, aid)

- draw.year comment: "year = completion year"
- painting.aid and painting.year: NO comment
- In class: teach painting.aid = buyer/owner (not creator), painting.year = exhibition year
```

**Why it works:** AI sees `painting.aid` and `artist.aid`, assumes "creator," joins directly — skipping the `draw` table. Wrong answer, but looks plausible.

**Tested result:** Without clarifying notes, AI scored **10/30 points** (6/8 questions wrong).

**Detection flags:** Direct join between tables that should go through an intermediate table; skipping the correct join path taught in class.

**Adapting:** Any course can create ambiguous naming:
- Data Structures: method `getNext()` that actually returns the previous in your implementation
- OS: variable `quantum` that means something different from default in your scheduling variant
- Networking: header field with non-obvious meaning taught in lecture

---

#### Custom RA Notation Trap (~100% catch rate)

**How it works:** Define custom abbreviations for relational algebra operators in class. Require them on the exam.

**Example:**
```
Print on exam: "Use ONLY the notation from Lecture 12."
Class-defined notation:
  SEL = selection (instead of sigma)
  PROJ = projection (instead of pi)
  NJOIN = natural join (instead of bowtie)
  AGGR = aggregation (instead of gamma)
```

**Why it works:** AI always produces standard notation. It cannot guess custom abbreviations.

**Detection:** Any answer using standard symbols instead of custom notation is flagged immediately.

---

#### Division / Universal Quantifier Trap (~90% catch rate)

**How it works:** Include at least one question with universal quantification ("find X who did EVERY Y"). This requires RA division or SQL double-NOT-EXISTS.

**Example:** "List artists who drew every painting priced above $3000"

**Why it works:** AI almost always returns the *existential* answer (ANY) instead of the *universal* (EVERY). The answer looks correct but uses the wrong operator/logic.

**Detection:** Check if the answer returns "artists who drew at least one" vs "artists who drew all." AI gets this wrong consistently.

---

#### Step-by-Step Assignment Format (~90% catch rate)

**How it works:** Require labeled intermediate assignments for every RA/SQL query.

**Example required format:**
```
R1 <- SEL(condition)(table)
R2 <- NJOIN(R1, other_table)
Result <- PROJ(columns)(R2)
```

**AI behavior:** Produces single nested expression: `PROJ_name(SEL_{year=2024}(painting) NJOIN artist)`

**Detection:** No intermediate steps = likely AI. Also allows partial credit for correct intermediate logic.

---

### MODULE: Data Structures

#### Method Name Misdirection (~75% catch rate)

**How it works:** Define a method name that sounds like it does X but actually does Y in your class implementation. Do NOT document the real behavior on the exam.

**Example:**
- Class defines `BST.successor(node)` that returns the **in-order predecessor** (not successor) in your specific implementation
- Exam asks: "What does `tree.successor(node_X)` return?"
- AI assumes standard textbook definition → wrong answer

**Detection:** Student uses the class-specific behavior → correct. Student uses textbook definition → likely AI or didn't attend class.

---

#### Trace Format Requirements (~85% catch rate)

**How it works:** Require a specific trace table format taught in class for dry-run exercises.

**Example:**
```
Required format (from Lecture 8):
| Step | Operation | Stack State | Output | Notes |
|------|-----------|-------------|--------|-------|
```

**AI behavior:** Produces a different trace format — usually narrative text or a non-matching table structure.

**Detection:** Wrong table format = didn't learn the class convention.

---

#### Custom Pseudocode Conventions (~80% catch rate)

**How it works:** Establish class-specific pseudocode keywords and structure.

**Example:**
```
Class convention:
- Use "REPEAT-UNTIL" instead of "do-while"
- Use "TRAVERSE(node)" instead of "visit" or "process"
- Indent with 3 spaces, not 4
- All variables declared at top of block with TYPE prefix (INT_count, STR_name)
```

**Detection:** AI uses standard pseudocode conventions. Class-specific conventions are an instant differentiator.

---

#### Implementation Detail Trap (~70% catch rate)

**How it works:** Ask about internal behavior of a data structure that was only discussed in class — not in the textbook or standard definition.

**Example:** "In our linked list implementation, what happens when you call `remove()` on the last element?" — where your class implementation has a specific edge case behavior (e.g., sets head to a sentinel node) different from textbook.

**Detection:** AI gives the textbook answer. Students who attended give the class-specific answer.

---

### MODULE: Algorithms

#### Proof Format Trap (~85% catch rate)

**How it works:** Require a specific proof structure taught in class (e.g., your 4-step format for induction proofs).

**Example required format:**
```
Step 1: STATE the claim formally
Step 2: BASE case (show n=1 or n=0)
Step 3: INDUCTIVE HYPOTHESIS (state IH explicitly, labeled "IH:")
Step 4: INDUCTIVE STEP (reference IH by label, show n+1)
```

**AI behavior:** Produces a correct but differently structured proof — missing your labels, combining steps, or using a different format.

**Detection:** Missing "IH:" label, missing step numbers, or non-matching structure.

---

#### Non-Standard Recurrence Notation (~80% catch rate)

**How it works:** Use class-defined symbols for recurrence relations.

**Example:**
```
Class notation:
- T(n) written as C_n (cost at level n)
- Use ">>" for "dominates" instead of Big-O comparison
- Floor/ceiling written as [n/2]_d and [n/2]_u (down/up) instead of standard symbols
```

**Detection:** AI uses standard T(n), Big-O, floor/ceiling notation.

---

#### Algorithm Variant Trap (~75% catch rate)

**How it works:** Teach a specific variant of an algorithm in class that differs from the textbook default.

**Example:** Your quicksort always picks the **median of first, middle, last** as pivot (not random, not first, not last). Ask: "Trace quicksort on this array. Show pivot selection at each step."

**Detection:** AI uses random or first-element pivot (textbook default). Students use median-of-three.

---

#### "Explain Your Approach" Requirement (~90% catch rate)

**How it works:** Before the formal solution, require a 2-3 sentence English explanation referencing class discussion.

**Example:** "Before writing your recurrence, explain in 2-3 sentences WHY this problem has optimal substructure. Reference the proof technique from Lecture 15."

**Detection:** AI cannot reference specific lecture content. Generic explanations = flag.

---

### MODULE: Computer Architecture

#### Assembly Dialect Trap (~85% catch rate)

**How it works:** Use dialect-specific mnemonics and behavior. MARIE, MIPS, x86, and ARM all have different instruction sets, register conventions, and addressing modes.

**Example (MARIE):**
```
MARIE uses: Load, Store, Add, Subt, Input, Output, Halt, Skipcond, Jump, Clear, AddI, JumpI, LoadI, StoreI
NOT: mov, add, sub, lw, sw (MIPS) or push, pop, mov (x86)
```

**Detection:** AI may produce MIPS-style or generic assembly when MARIE was specified. Any non-MARIE instruction is an instant flag.

---

#### Hand-Assembly Step Format (~90% catch rate)

**How it works:** Require a specific 3-step hand-assembly process taught in class.

**Example required format:**
```
Step 1: ASSIGN ADDRESSES (list each instruction with its hex address)
Step 2: BUILD SYMBOL TABLE (list each label with its address)
Step 3: CONVERT TO BINARY (show opcode + address for each instruction)
```

**AI behavior:** AI produces the final binary directly or uses a different conversion process.

**Detection:** Missing intermediate steps (address table, symbol table) = likely AI.

---

#### Custom Opcode Mnemonics (~100% catch rate)

**How it works:** Define abbreviated or modified mnemonics in class.

**Example:**
```
Class mnemonics (from Lecture 6):
  LD = Load (instead of LOAD)
  ST = Store (instead of STORE)
  SKP = Skipcond (instead of SKIPCOND)
  JP = Jump (instead of JUMP)
  HLT = Halt (instead of HALT)
```

**Detection:** AI uses full standard mnemonics. Students who attended use abbreviations.

---

#### Execution Trace Format (~85% catch rate)

**How it works:** Require a specific register/AC trace table format.

**Example required format:**
```
| PC | IR | AC | MAR | MBR | Output | Notes |
|----|----|----|-----|-----|--------|-------|
```

With specific conventions: PC shown in hex, AC in decimal, IR as "opcode addr".

**Detection:** AI produces a different trace format or uses different conventions for displaying values.

---

### MODULE: Networking

#### Protocol Field Meaning Trap (~75% catch rate)

**How it works:** Use a protocol header field whose meaning in your class differs from what AI assumes.

**Example:** In your class, the "window size" field in a custom protocol means "number of bytes the sender is willing to buffer" (not the receiver's window). Ask questions that depend on this interpretation.

**Detection:** AI uses the standard TCP interpretation → wrong answer on questions requiring the class-specific meaning.

---

#### Non-Standard Diagram Notation (~80% catch rate)

**How it works:** Use class-specific symbols in network diagrams.

**Example:**
```
Class convention:
- Routers drawn as hexagons (not circles)
- Switches drawn as diamonds (not rectangles)
- Wireless links shown as zigzag lines (not dashed)
- Label format: "DEVICE_ID:INTERFACE" (not just device name)
```

**Detection:** AI uses standard Cisco-style or textbook notation.

---

#### Packet Trace Format (~85% catch rate)

**How it works:** Require specific header field ordering and notation in packet traces.

**Example required format:**
```
Layer 4: [SRC_PORT | DST_PORT | SEQ | ACK | FLAGS | WIN]
Layer 3: [VER | TTL | PROTO | SRC_IP | DST_IP]
Layer 2: [DST_MAC | SRC_MAC | TYPE]
```

**Detection:** AI uses a different field ordering or includes/excludes different fields.

---

#### Layer Model Trap (~70% catch rate)

**How it works:** Your class uses a specific layering model that differs from the textbook default (e.g., 5-layer TCP/IP model vs 7-layer OSI, or a custom simplified model).

**Example:** Ask "At which layer does X happen?" where the answer depends on whether you use OSI or TCP/IP or your custom model.

**Detection:** AI defaults to one model; students use the class-specific one.

---

### MODULE: Operating Systems

#### Scheduling Variable Naming Trap (~75% catch rate)

**How it works:** Use variable names that suggest one scheduling algorithm but implement another.

**Example:** Variable `quantum` in your round-robin implementation actually represents the **priority level** (you renamed it in class for a specific pedagogical reason). Ask: "What happens when Process A has quantum=3?"

**Detection:** AI assumes quantum = time slice. Students know quantum = priority in your class.

---

#### Custom State Diagram Notation (~80% catch rate)

**How it works:** Use class-specific process state labels.

**Example:**
```
Class state labels:
  N = New (instead of "created" or "new")
  X = Running (instead of "running" — because "eXecuting")
  W = Waiting (instead of "blocked" or "waiting")
  R = Ready
  T = Terminated (instead of "exit" or "terminated")
```

**Detection:** AI uses standard textbook state names. Students use class abbreviations.

---

#### Trace Table Format (~85% catch rate)

**How it works:** Require a specific Gantt chart or trace table format for scheduling problems.

**Example required format:**
```
Time:    0  1  2  3  4  5  6  7  8
CPU:    [P1|P1|P2|P2|P2|P3|P1|P1|--]
Ready:  [P2,P3|P3|P3|--|P3|--|--|--|--]
Wait:   [--|--|--|--|--|--|--|--|--]

Metrics (use CLASS FORMULA):
  Turnaround = Completion - Arrival (not including I/O wait, per Lecture 9 definition)
```

**Detection:** AI uses a different Gantt format or standard turnaround formula.

---

#### Synchronization Notation (~80% catch rate)

**How it works:** Use class-defined notation for semaphores, mutexes, or monitors.

**Example:**
```
Class notation:
  GRAB(S) = wait/P operation on semaphore S
  FREE(S) = signal/V operation on semaphore S
  ZONE_START = enter critical section
  ZONE_END = exit critical section
```

**Detection:** AI uses standard wait()/signal() or P()/V() notation.

---

### MODULE: Programming Languages

#### Custom Style Requirements (~80% catch rate)

**How it works:** Mandate class-specific coding style that AI won't reproduce.

**Example:**
```
Class style (from syllabus):
- All functions must start with a verb: get_, set_, calc_, check_
- Constants use SCREAMING_SNAKE with module prefix: MOD_MAX_SIZE
- Every function has a one-line "PURPOSE:" comment before it
- Error returns use -999 (not -1, null, or exceptions)
```

**Detection:** AI uses standard conventions (camelCase, PEP8, etc.). Class-specific style is an instant differentiator.

---

#### "Explain Before Code" Format (~90% catch rate)

**How it works:** Require English explanation before every function/block.

**Example:**
```
Required format:
// PURPOSE: [one sentence]
// APPROACH: [which technique from class]
// EDGE CASES: [list]
[code here]
```

**Detection:** AI writes code first (or code only). Missing PURPOSE/APPROACH/EDGE CASES block = flag.

---

#### Deliberate API Naming Trap (~85% catch rate)

**How it works:** Define class-specific method names that differ from standard library equivalents.

**Example:** In your class, the linked list method to add at the end is called `append_tail()` (not `append()`, `push_back()`, or `add()`). Exam says: "Using the LinkedList API from class, write code to..."

**Detection:** AI uses standard library method names. Students use class-defined names.

---

#### Output Format Trap (~75% catch rate)

**How it works:** Require specific output formatting only discussed in class.

**Example:** "All output must use the format from Lab 3: `[RESULT] value = X (type: Y)`"

**Detection:** AI produces standard `print(X)` or generic formatting. Class-specific format is a differentiator.

---

## Phase 2: Adversarial Audit

**Goal:** Test exam questions against AI and report vulnerabilities.

### Input Handling

Accept exam questions via:
1. **Pasted in chat** — parse into numbered question list
2. **File upload** (docx/pdf/md) — use `markitdown` skill to convert, then parse
3. **Reference to existing exam** in profile — read from `anti-ai-profiles/<course>/exams/<name>/`

After parsing, confirm: "I found X questions. Ready to audit?"

### Pass 1: Simulated Analysis (fast, always runs)

For each question, evaluate against this checklist:

| Check | Score Impact |
|-------|-------------|
| Relies on in-class-only knowledge? | -3 (good) |
| Requires custom notation? | -3 (good) |
| Requires step-by-step format? | -2 (good) |
| Involves universal quantification / division? | -2 (good) |
| Has verbal correction planted? | -3 (good) |
| Has version-watermarked constants? | -1 (good) |
| Standard textbook problem with no traps? | +4 (bad) |
| Explicit instructions that help AI? | +3 (bad) |

**Vulnerability Score:** Start at 5 (neutral). Apply modifiers. Clamp to 0-10.
- **0-3:** STRONG — AI likely fails detectably
- **4-6:** MODERATE — AI might get it right, limited detection
- **7-10:** WEAK — AI likely solves correctly, hard to detect

**Output per question:**
```
Q[N]: "[question text summary]"
  Vulnerability: X/10 [STRONG/MODERATE/WEAK]
  Traps active: [list]
  Missing traps: [suggestions to harden]
  Hardening recommendation: [specific edit]
```

**Summary after Pass 1:**
```
AUDIT SUMMARY (Simulated)
  Questions: X total
  Strong (0-3): Y questions
  Moderate (4-6): Y questions
  Weak (7-10): Y questions
  Overall resistance: X% of questions are AI-resistant

  Weakest questions: Q3, Q7 — recommend hardening before exam
```

Then ask:
```
Would you like me to live-test any questions against AI? (Pass 2)
I'll attempt to answer them as a student submitting to AI — no class context, no notes.
Enter question numbers to test, or "all", or "weak only", or "skip".
```

### Pass 2: Live Adversarial Test (optional, professor selects)

For each selected question:

1. **Role-switch:** Attempt to answer the question as if a student photographed it and submitted to AI. Use ONLY the information visible on the exam — no class context, no schema notes, no custom notation knowledge.

2. **Self-analyze:** After generating the answer, break character and evaluate:
   - Did I get it right or fall into the trap?
   - What notation/format/join path did I use?
   - Would my answer trigger detection flags?

3. **Report per question:**
```
Q[N]: "[question text summary]"
├── AI Answer: [the actual generated answer]
├── Correct: [checkmark or X] ([explanation])
├── Traps triggered: [which traps caught AI]
├── Detection flags: [what a grader would notice]
├── Verdict: STRONG / MODERATE / WEAK
└── Notes: [any observations]
```

### Audit Report Generation

After both passes, generate `audit-report.md`:

```markdown
# Adversarial Audit Report
**Course:** [code] | **Exam:** [name] | **Date:** [date]

## Overall Score
- **X/Y questions AI-resistant** (AI fails detectably)
- **Risk level:** LOW / MEDIUM / HIGH

## Per-Question Breakdown
| Q# | Topic | Vulnerability | Traps Active | Pass 1 | Pass 2 | Verdict |
|----|-------|--------------|--------------|--------|--------|---------|
| 1  | ...   | 2/10         | notation, format | STRONG | STRONG | STRONG |

## Weakest Questions
[List questions with vulnerability > 5, with specific hardening suggestions]

## Strongest Traps
[List questions with vulnerability < 3, explaining what makes them effective]

## Recommendations
[Prioritized list of edits to improve weak questions]
```

Save to `anti-ai-profiles/<course>/exams/<exam-name>/audit-report.md`

---

## Phase 3: Output Generation

### Core Output (always generated)

After consulting (Phase 1) and/or auditing (Phase 2), generate these files:

1. **`strategies.md`** — Already generated in Phase 1. Update if audit revealed new insights.

2. **`detection-checklist.md`** — Customized to the active traps:
```markdown
# Detection Checklist for [Exam Name]

## Per-Answer Red Flags
- [ ] No scratch work or erasures (suspiciously clean)
- [ ] Single expression instead of step-by-step (format trap active)
- [ ] Standard notation used (custom notation "[list custom symbols]" was required)
- [ ] Uses the uncorrected typo "[typo]" (verbal correction was announced)
- [ ] Inconsistent quality across questions
- [ ] Symbols not taught in class: [list common AI symbols for this subject]

## Cross-Student Red Flags
- [ ] Version constant mismatch (compare [list watermarked values])
- [ ] Identical variable names across 2+ students
- [ ] Same unusual error in multiple submissions

## Investigation Protocol
When 3+ flags are triggered:
1. Schedule oral follow-up within 48 hours
2. Ask student to explain Q[hardest question] step by step
3. Compare submission against AI baseline (see ai-baseline.md)
4. Document: which flags triggered, comparison results, oral follow-up notes
```

3. **`trap-history.json`** — Record which traps were used (see Profile Schema section).

### Optional Exam Package

After generating core output, ask:
```
Would you like me to generate any of these?
1. Exam document (.docx) — with traps baked in and format requirements in the header
2. Solution key (.docx) — step-by-step solutions in the required notation/format
3. AI baseline — what AI would answer per question (for comparison during grading)
4. Grading rubric (.md) — point allocation, partial credit rules, AI-detection flag scoring
5. All of the above
```

#### Exam Document Generation

When selected, use the `docx` skill to create the exam:

**Header section must include:**
```
[Course Code] — [Exam Name]
[Date] | Version [A/B/C]

INSTRUCTIONS:
- Show ALL intermediate steps using assignment notation (label: R1, R2, etc.)
- Use ONLY the notation from [Lecture N] (see notation reference below)
- [Any additional format requirements]

[If custom notation: include a PARTIAL reference — enough to remind students
who attended, but not enough for AI to decode the full system]
```

**For each question:**
- Embed active traps into question wording
- Use undefined terms (from semantics trap) without explanation
- Include version-specific constants
- Include the deliberate typo (if verbal correction trap is active)

**If version watermarking is active:**
- Generate 2-3 versions with different constants
- Same structure, same difficulty
- Name files: `exam-vA.docx`, `exam-vB.docx`, etc.

#### Solution Key Generation

- Write solutions using the required notation/format
- Show all intermediate steps (model the expected student format)
- Mark trap-related elements with [TRAP] annotations for grader reference
- Include the verbal correction (corrected version)

#### AI Baseline Generation

For each question:
```markdown
## Q[N]: [question summary]

### AI Would Answer:
[Generate the answer AI would produce — no class context]

### Key Differences from Correct Answer:
- [What AI gets wrong and why]
- [Which trap catches it]

### Grader Notes:
- If student answer matches this pattern → investigate
- Specific phrases/structure to watch for: [list]
```

#### Grading Rubric Generation

```markdown
# Grading Rubric — [Exam Name]

## Point Allocation
| Q# | Topic | Points | Breakdown |
|----|-------|--------|-----------|
| 1  | ...   | 10     | Steps: 4, Answer: 4, Notation: 2 |

## Partial Credit Rules
- Correct intermediate steps with wrong final answer: award step points
- Correct logic with wrong notation: deduct notation points only
- Correct answer with no intermediate steps: award max [50%] — format was required

## AI-Detection Flag Scoring
| Flag | Weight |
|------|--------|
| Wrong notation (used standard instead of custom) | 1 |
| No intermediate steps | 1 |
| Used uncorrected typo | 2 (strong signal) |
| Version constant mismatch | 2 (strong signal) |
| Identical structure to AI baseline | 1 |
| Uses symbols not taught in class | 1 |

**Investigation threshold:** 3+ flag points → schedule oral follow-up

## Oral Follow-Up Questions (per problem)
| Q# | Follow-Up Question |
|----|--------------------|
| 1  | "Walk me through why you chose [specific step]. What alternatives did you consider?" |
| 2  | "Explain what [custom term] means and why it matters here." |

## Version Cross-Reference
| Element | Version A | Version B |
|---------|-----------|-----------|
| [value1] | X | Y |
| [value2] | X | Y |
```

### Semester Evolution

When the professor returns for the next exam:

1. **Load trap history:** Read `trap-history.json`
2. **Check for reuse:** If the same traps were used in the last exam, warn:
   ```
   Warning: You used [trap names] on the [last exam]. Students may have shared
   strategies. Consider rotating to: [suggest alternatives]
   ```
3. **Suggest rotation:** Recommend different trap combinations
4. **After new exam:** Update `trap-history.json` with new entries

---

## Profile Schemas

### profile.json

```json
{
  "course_code": "string — e.g., cs286",
  "course_name": "string — e.g., Computer Architecture",
  "subject": "string — one of: databases, data_structures, algorithms, architecture, networking, os, programming",
  "level": "string — one of: intro, intermediate, advanced",
  "exam_format": "string — one of: written_paper, typed_computer, take_home, mixed",
  "class_size": "number",
  "class_size_category": "string — one of: small, medium, large, very_large",
  "can_oral_followup": "boolean",
  "subject_details": {
    "_comment": "Subject-specific fields from adaptive interview",
    "assembly_dialect": "string (architecture only)",
    "notation_type": "string (databases only — sql, ra, both)",
    "language": "string (programming only)",
    "trace_format": "boolean — whether students do dry-run traces"
  },
  "constraints": ["array of strings — e.g., 'no multiple versions', 'take-home only'"],
  "active_strategies": ["array of strategy IDs currently in use"],
  "created": "ISO date string",
  "last_updated": "ISO date string"
}
```

### trap-history.json

```json
{
  "history": [
    {
      "exam_name": "string — e.g., midterm-2026-spring",
      "date": "ISO date string",
      "traps_used": ["array of strategy IDs"],
      "questions_count": "number",
      "ai_resistant_count": "number",
      "ai_resistant_percentage": "number",
      "notes": "string — optional observations"
    }
  ]
}
```

---

## Philosophical Note

These strategies are not "gotcha" traps. They reward students who **attend class**, **read carefully**, and **understand the material**. Every trap has a straightforward correct answer for the prepared student.

The goal is to make AI answers *detectably wrong* — shifting the cost-benefit calculation away from cheating.

The most effective approach is **layering multiple independent traps**. Any single trap might fail; but combining 3-4 independent traps drops the probability of AI navigating all of them to near zero.

---

## Extensibility

### Adding a New Subject Module

To add support for a new discipline (e.g., Mathematics, Physics), add a new MODULE section following this template:

```markdown
### MODULE: [Subject Name]

#### [Strategy Name] (~XX% catch rate)

**How it works:** [1-2 sentence explanation]

**Example:**
[Concrete example from the discipline]

**Why it works:** [Why AI fails at this]

**Detection flags:** [What to look for in student answers]
```

Requirements for a new module:
1. Subject name and prerequisite context
2. 4-6 strategies with estimated catch rates
3. Concrete examples (ideally adversarially tested)
4. Detection flags for each strategy
5. Adaptive interview questions for Phase 1

The orchestrator automatically picks up new modules — no changes to interview or audit logic needed. Simply add the module section and corresponding adaptive interview branch.

---
> Source: [weihaoqu/anti-ai-exam-skill](https://github.com/weihaoqu/anti-ai-exam-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
