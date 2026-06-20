## autoskill

> >-


# Adapt Skill

**One system that does everything:**
1. **Auto-detects project type** and loads matching skills
2. Instruments skills (adds learning structure if missing)
3. Scans project (finds opportunities + exemplars)
4. Proposes improvements (skill teaches project)
5. Learns patterns (project teaches skill)
6. Enables continuous learning (ongoing improvement)

---

## Three Modes

### Mode 0: Auto-Detection (On Session Start)

```
Agent starts working on project

→ Scan project for file types (*.cs, *.py, *.tsx, etc.)
→ Match file types to skill registry
→ Auto-load matching skills
→ Enable continuous learning for loaded skills

No user invocation. Happens at session start.
```

### Mode 1: Explicit Adaptation (User Invokes)

```
User: "adapt unity-game-dev.md"

→ Full scan of project
→ Propose all improvements
→ Bulk learn from project
→ One-time comprehensive pass
```

### Mode 2: Continuous Learning (Automatic)

```
Agent working on task, reads PlayerController.cs

→ Notice: "GetComponent cached in Awake" (good pattern)
→ Queue learning
→ On task complete: flush learnings to skill

No user invocation. Happens automatically.
```

**Mode 0 sets up the system. Mode 2 keeps it running. Mode 1 is for deep passes.**

---

## Mode 0: Auto-Detection Process

### When to Run

Run auto-detection at the START of a session, before any user task:

```
SESSION START
    │
    ▼
┌─────────────────────────────────────────┐
│           AUTO-DETECTION                │
│                                         │
│  1. Scan project root for file types    │
│  2. Match against skill registry        │
│  3. Load matching skills                │
│  4. Instrument if needed                │
│  5. Report what was loaded              │
│                                         │
└─────────────────────────────────────────┘
    │
    ▼
READY FOR USER TASK (skills loaded, continuous learning active)
```

### Step 1: Scan for File Types

```
Use Glob to find files in project:

file_types = {}
FOR pattern in ["*.cs", "*.py", "*.tsx", "*.ts", "*.jsx", "*.js", "*.go", "*.rs", "*.rb"]:
  count = Glob(pattern, project_root).count()
  IF count > 0:
    file_types[pattern] = count

OUTPUT: {"*.cs": 47, "*.json": 12, ...}
```

### Step 2: Skills Registry

Map file types to skills. The registry can be:

**Embedded (default):**
```yaml
skill_registry:
  "*.cs":
    - skill: unity-game-dev
      confidence: high
      requires: ["*.unity", "Assets/"]  # Optional extra checks
    - skill: dotnet-backend
      confidence: medium

  "*.py":
    - skill: python-backend
      confidence: high
    - skill: django-web
      requires: ["manage.py", "settings.py"]

  "*.tsx":
    - skill: react-frontend
      confidence: high
    - skill: nextjs-app
      requires: ["next.config.*"]

  "*.go":
    - skill: golang-backend
      confidence: high

  "*.rs":
    - skill: rust-systems
      confidence: high
```

**Or external (skill-registry.yaml in skills folder).**

### Step 3: Match and Score

```
FOR file_type, skills in skill_registry:
  IF file_type IN detected_file_types:
    FOR skill in skills:
      score = 0

      # Base score from file count
      score += min(detected_file_types[file_type] / 10, 5)

      # Confidence boost
      IF skill.confidence == "high": score += 3
      IF skill.confidence == "medium": score += 1

      # Requirements check
      IF skill.requires:
        matches = check_requirements(skill.requires)
        IF matches: score += 5
        ELSE: score -= 10  # Don't load if requirements fail

      add_candidate(skill.name, score)

# Take top skills (avoid loading too many)
selected = candidates.sort_by_score().take(3)
```

### Step 4: Load Skills

```
FOR skill_name in selected:
  skill_path = find_skill(skill_name)  # Check local .skills/, then ~/.agents/skills/

  IF skill_path:
    skill = READ(skill_path)

    # Instrument if needed (Step 0 from existing process)
    IF not_instrumented(skill):
      instrument(skill)

    # Add to loaded skills
    loaded_skills.append(skill)

    LOG: "Loaded {skill_name} for {file_types}"
```

### Step 5: Report

```
At session start, briefly report:

"Auto-loaded skills for this project:
  - unity-game-dev (47 .cs files, Assets/ folder detected)
  - netcode-multiplayer (NetworkBehaviour found)

Continuous learning is active."
```

---

## Default Skill Registry

Built-in mappings (can be overridden):

```yaml
# Game Development
"*.cs" + "*.unity":     unity-game-dev
"*.cs" + "*.uproject":  unreal-cpp  # Unreal uses .cs for build scripts
"*.gd":                 godot-gdscript

# Web Frontend
"*.tsx" + "*.jsx":      react-frontend
"*.vue":                vue-frontend
"*.svelte":             svelte-frontend
"next.config.*":        nextjs-app
"nuxt.config.*":        nuxt-app

# Web Backend
"*.py" + "manage.py":   django-web
"*.py" + "app.py":      flask-api
"*.py" + "main.py":     fastapi-backend
"*.rb" + "Gemfile":     rails-backend
"*.go" + "go.mod":      golang-backend
"*.rs" + "Cargo.toml":  rust-backend

# Mobile
"*.swift":              ios-swift
"*.kt" + "*.gradle":    android-kotlin

# Infrastructure
"*.tf":                 terraform-infra
"*.yaml" + "k8s/":      kubernetes-ops
"docker-compose.*":     docker-compose

# Data/ML
"*.ipynb":              jupyter-datascience
"*.py" + "requirements.txt" + ("torch"|"tensorflow"): ml-pytorch
```

### Custom Registry

Projects can override by creating `.skills/skill-registry.yaml`:

```yaml
# .skills/skill-registry.yaml
overrides:
  "*.cs":
    - skill: my-custom-unity-skill
      confidence: high

disable:
  - django-web  # Don't auto-load this even if manage.py exists

always_load:
  - team-conventions  # Always load this for our team
```

---

## Process Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    ADAPT-SKILL PROCESS                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────┐                                            │
│  │ STEP 0      │  Check if skill has learning structure     │
│  │ INSTRUMENT  │  If not → add it                           │
│  └──────┬──────┘                                            │
│         ▼                                                    │
│  ┌─────────────┐                                            │
│  │ STEP 1      │  Extract principles, techniques,           │
│  │ PARSE       │  detection patterns from skill             │
│  └──────┬──────┘                                            │
│         ▼                                                    │
│  ┌─────────────┐                                            │
│  │ STEP 2      │  Search project for anti-patterns          │
│  │ SCAN        │  (opportunities) and good patterns         │
│  └──────┬──────┘  (exemplars)                               │
│         ▼                                                    │
│  ┌─────────────┐                                            │
│  │ STEP 3      │  Score by impact, link to exemplars        │
│  │ RANK        │                                            │
│  └──────┬──────┘                                            │
│         ▼                                                    │
│  ┌─────────────┐                                            │
│  │ STEP 4      │  Generate before/after for each            │
│  │ PROPOSE     │  (Mode 1 only)                             │
│  └──────┬──────┘                                            │
│         ▼                                                    │
│  ┌─────────────┐                                            │
│  │ STEP 5      │  User approves changes                     │
│  │ REVIEW      │  (Mode 1 only)                             │
│  └──────┬──────┘                                            │
│         ▼                                                    │
│  ┌─────────────┐                                            │
│  │ STEP 6      │  Apply approved edits to project           │
│  │ APPLY       │  (Mode 1 only)                             │
│  └──────┬──────┘                                            │
│         ▼                                                    │
│  ┌─────────────┐                                            │
│  │ STEP 7      │  Validate exemplars against principles     │
│  │ LEARN       │  Extract project-specific patterns         │
│  └──────┬──────┘                                            │
│         ▼                                                    │
│  ┌─────────────┐                                            │
│  │ STEP 8      │  Write learnings to skill file             │
│  │ UPDATE      │                                            │
│  └──────┬──────┘                                            │
│         ▼                                                    │
│  ┌─────────────┐                                            │
│  │ STEP 9      │  Summary of changes and learnings          │
│  │ REPORT      │                                            │
│  └─────────────┘                                            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Step 0: Instrument Skill (If Needed)

Before anything else, check if skill has learning structure:

```
READ skill_path

CHECK for required sections:
  □ domain.file_patterns in frontmatter
  □ domain.watch_for in frontmatter
  □ ## Principles section
  □ ## Project Learnings section

IF any missing:
  RUN auto-instrumentation
```

### Auto-Instrumentation Process

```
1. DETECT domain from skill content:
   - Mentions Unity, C#, MonoBehaviour → file_patterns: ["*.cs"]
   - Mentions React, JSX, hooks → file_patterns: ["*.tsx", "*.jsx"]
   - Mentions Python, Django → file_patterns: ["*.py"]

2. EXTRACT principles from skill:
   - Look for "Anti-Patterns" section → convert to principles
   - Look for "Rules", "Guidelines", "Best Practices" → extract principles
   - Look for "Avoid", "Never", "Don't" statements → convert to principles

3. GENERATE watch_for patterns:
   - From code examples: extract the patterns shown
   - From anti-patterns: create detection regex
   - From "prefer X over Y" statements: X = good pattern, Y = anti-pattern

4. ADD missing sections to skill:
   - Add domain to frontmatter
   - Add ## Principles if missing (extracted from content)
   - Add ## Project Learnings at end

5. WRITE updated skill
```

### Instrumentation Template

Add this to frontmatter if missing:

```yaml
domain:
  file_patterns:
    - "*.{ext}"           # Detected from skill content
  keywords:
    - {keyword}           # Extracted from skill mentions
  watch_for:
    - pattern: "{regex}"  # From techniques/examples
      category: "{name}"
      type: exemplar|anti-pattern

principles_locked: true
```

Add this section if missing:

```markdown
## Principles
<!-- 🔒 LOCKED: Agent cannot modify -->

1. **{Principle}**: {Extracted from anti-patterns/rules}
```

Add this section at end if missing:

```markdown
## Project Learnings
<!-- 🤖 AUTO-UPDATED: Agent writes learnings here -->

_Updated automatically as agent works on projects._
```

---

## Step 1: Parse Skill

Read the skill file and extract structured knowledge:

```
READ skill_path

EXTRACT:
  principles[] = lines under "## Principles" until next ##
    Each principle has: id, statement, keywords (inferred)

  techniques[] = sections under "## Techniques"
    Each technique has:
      - id (T1, T2, ...)
      - category (parent ### heading)
      - name
      - detect_anti_pattern (from watch_for where type=anti-pattern)
      - detect_good_pattern (from watch_for where type=exemplar)
      - fix_template (code blocks showing correct approach)

  watch_for[] = from domain.watch_for in frontmatter

IF skill lacks explicit watch_for:
  INFER from prose:
    - "Avoid X in Y" → anti_pattern: X in Y
    - "Use X instead of Y" → anti_pattern: Y, good_pattern: X
    - "Cache X" → anti_pattern: X not cached, good_pattern: X in Awake/init
```

**Output:**
```
Parsed {N} principles, {M} techniques, {P} watch patterns
```

---

## Step 2: Scan Project

For each technique and watch pattern, search the codebase:

```
# Get relevant files first
files = Glob(domain.file_patterns, project_root)

# Skip noise
EXCLUDE:
  - **/test/**
  - **/tests/**
  - **/*.test.*
  - **/*.spec.*
  - **/node_modules/**
  - **/vendor/**
  - **/*.generated.*

FOR each watch_for pattern:

  IF pattern.type == "anti-pattern":
    matches = Grep(pattern.pattern, files)
    FOR match in matches:
      record_opportunity(pattern.category, match)

  IF pattern.type == "exemplar":
    matches = Grep(pattern.pattern, files)
    FOR match in matches:
      record_exemplar(pattern.category, match)
```

**Output:**
```
Scanned {N} files
Found {X} opportunities (code to improve)
Found {Y} exemplars (good patterns to learn)
```

---

## Step 3: Rank Opportunities

```
FOR each opportunity:

  READ file context (surrounding 20 lines)

  ASSESS:
    - Is this actually a problem? (not false positive)
    - Is this in a hot path? (performance critical)
    - How many times does this pattern appear?

  SCORE (0-100):
    severity = hot_path ? 40 : moderate ? 25 : 10
    scope = occurrences > 5 ? 30 : occurrences > 1 ? 20 : 10
    ease = simple_fix ? 30 : moderate ? 20 : complex ? 10
    impact = severity + scope + ease

  LINK to exemplars:
    - Find exemplar in same project that shows correct pattern
    - Reference it in proposal: "See {file}:{line}"

SORT by impact DESC
TAKE top 10 for Mode 1 (avoid overwhelming)
```

---

## Step 4: Generate Proposals (Mode 1 Only)

```
FOR each top opportunity:

  GENERATE proposal:
    - Technique name and why it matters
    - Current code (the problem)
    - Proposed code (the fix)
    - Exemplar reference if available
    - Principle it relates to

  FORMAT:
    ### {Technique}: {Category}
    **File:** {path}:{line}
    **Principle:** {which principle this applies}

    **Current:**
    ```{lang}
    {bad code}
    ```

    **Proposed:**
    ```{lang}
    {good code}
    ```

    **Why:** {rationale from principles}
    **Example in project:** {exemplar reference if exists}
```

---

## Step 5: User Review (Mode 1 Only)

```
PRESENT all proposals grouped by category

ASK:
  "Found {N} improvements based on {skill_name}.

   Summary:
   - {category}: {count} instances
   - {category}: {count} instances

   Options:"

OPTIONS:
  - "All" → apply everything
  - "Select" → interactive selection
  - "Learn only" → skip fixes, just learn patterns
  - "Review each" → show one by one
```

---

## Step 6: Apply Changes (Mode 1 Only)

```
FOR each approved proposal:

  READ current file
  APPLY edit (Edit tool)
  VERIFY change was made correctly

  LOG:
    - File: {path}
    - Change: {description}
    - Technique: {name}
```

---

## Step 7: Learn from Project

**This step runs in BOTH modes.**

```
FOR each exemplar found:

  1. EXTRACT the pattern:
     - What specific API/utility does project use?
     - What naming convention?
     - What file organization?

  2. CHECK frequency:
     - Pattern appears 2+ times? → worth learning
     - Only once? → skip (might be one-off)

  3. VALIDATE against principles:
     FOR each principle:
       Does this pattern VIOLATE the principle?
       IF yes: SKIP (it's actually an anti-pattern)

  4. CHECK novelty:
     - Is this already in the skill? → skip
     - Is this project-specific? → learn it

  5. PREPARE learning:
     "[Learned: {date}]: {description}"
     Include: file reference, specific API, usage example
```

---

## Step 8: Update Skill

```
READ current skill file

FIND "## Project Learnings" section
  IF not exists: create it at end

FOR each validated learning:

  CHECK for existing similar learning:
    IF exists: update with new info
    IF new: add as new line

  FORMAT:
    "- [Learned: {YYYY-MM-DD}]: {description} (see {file})"

WRITE updated skill file
```

### Example Update

```markdown
## Project Learnings

### Horse & Knight Arena (2026-03-21)
- [Learned: 2026-03-21]: Components cached in Awake using expression body (see EnemyController.cs)
- [Learned: 2026-03-21]: ObjectPool<T> utility at /Scripts/Utils/ObjectPool.cs
- [Learned: 2026-03-21]: Events use ScriptableObject channels at /Events/
- [Learned: 2026-03-21]: [RequireComponent] used on all dependent components
```

---

## Step 9: Report

```
# Skill Adaptation Report

## Skill: {skill_name}
## Project: {project_name}
## Date: {date}

### Instrumentation
- {Added domain configuration / Already instrumented}
- {Added principles section / Already had principles}

### Scan Results
- Files scanned: {N}
- Opportunities found: {X}
- Exemplars found: {Y}

### Changes Applied (Mode 1)
- {file}: {technique}
- {file}: {technique}

### Learnings Added
- {learning 1}
- {learning 2}

### Skill Status
- Principles: {N} (locked)
- Techniques: {M}
- Project learnings: {L}

### Next Time
- Continuous learning is now active
- Skill will improve as you work
```

---

## Continuous Learning Mode (Mode 2)

When NOT explicitly invoked, the adaptation logic runs **automatically during normal work**:

### Trigger: File Read

```
WHEN agent reads a file:

  1. CHECK if file matches any loaded skill's domain.file_patterns
     IF no match: skip

  2. FOR each matching skill:
     SCAN file content against skill.domain.watch_for

  3. FOR each match:
     IF type == "exemplar":
       QUEUE learning: {skill, category, pattern, file, snippet}
     IF type == "anti-pattern":
       NOTE but don't propose (not explicit mode)
```

### Trigger: Task Complete

```
WHEN agent completes a task:

  1. CHECK learning queue
     IF empty: done

  2. FOR each queued learning:
     - Validate against principles
     - Check frequency (2+ occurrences?)
     - Check novelty

  3. FLUSH valid learnings to skill files
     - Silent update, no report needed

  4. CLEAR queue
```

### What User Sees

**Nothing.** Continuous learning is invisible.

User might notice over time:
- Agent suggestions become more project-specific
- Agent references "your project's pattern" without being told
- Checking skill file shows accumulated learnings

---

## Quality Gates

### Instrumentation
- NEVER modify ## Principles content (only ADD section if missing)
- ALWAYS preserve existing skill content
- ONLY add structure, don't change techniques

### Scanning
- SKIP test files, generated files, vendor code
- REQUIRE 2+ occurrences to consider a pattern learnable
- VALIDATE against principles before learning anything

### Learning
- NEVER learn patterns that violate principles
- ALWAYS include date and file reference
- KEEP original techniques intact (add, don't replace)
- DEDUPLICATE before adding

### Applying Changes (Mode 1)
- ALWAYS show before/after
- NEVER auto-apply without user approval
- VERIFY syntax after edit

---

## Example: Full Adaptation Run

```
> adapt unity-game-dev.md

Checking skill structure...
  ✓ domain.file_patterns: present
  ✓ domain.watch_for: 9 patterns
  ✓ ## Principles: 7 principles
  ✓ ## Project Learnings: present
  → Skill already instrumented

Parsing skill...
  ✓ 7 principles (locked)
  ✓ 12 techniques
  ✓ 9 watch patterns

Scanning project (148 .cs files)...
  ✓ Searching for anti-patterns...
  ✓ Searching for good patterns...

Results:
  Opportunities: 8
    - component-caching: 5
    - pooling: 2
    - object-references: 1

  Exemplars: 12
    - component-caching: 6
    - events: 4
    - networking: 2

Top proposals:
  1. PlayerController.cs:45 → Cache GetComponent (HIGH: 92)
  2. ProjectileManager.cs:112 → Use pooling (HIGH: 85)
  3. NPCBehavior.cs:78 → Cache GetComponent (MEDIUM: 67)
  [...]

Apply changes? [All / Select / Learn only / Review each]
> Select

[User selects 1, 2, 5]

Applying changes...
  ✓ PlayerController.cs:45 - cached Rigidbody in Awake
  ✓ ProjectileManager.cs:112 - using ObjectPool.Get()
  ✓ UIManager.cs:34 - cached CanvasGroup

Learning from project...
  ✓ [Learned]: Expression body caching: _rb = GetComponent<Rigidbody>();
  ✓ [Learned]: ObjectPool<T> at Scripts/Utils/ObjectPool.cs
  ✓ [Learned]: [RequireComponent] on all MonoBehaviours
  ✗ Skipped: FindObjectOfType in Awake (violates principle #2)

Updating skill...
  + 3 learnings added to ## Project Learnings

Done. Continuous learning now active.
```

---

## Summary

| Capability | How It Works |
|------------|--------------|
| **Instrument** | Adds learning structure if missing |
| **Scan** | Finds opportunities and exemplars |
| **Propose** | Shows before/after (Mode 1) |
| **Apply** | Edits project code (Mode 1, with approval) |
| **Learn** | Extracts patterns, validates, updates skill |
| **Continuous** | Runs automatically during normal work |

**One skill. Everything you need.**

---
> Source: [Samurai412/autoskill](https://github.com/Samurai412/autoskill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
