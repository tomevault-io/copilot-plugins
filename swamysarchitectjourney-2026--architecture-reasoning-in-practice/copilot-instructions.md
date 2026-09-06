## 07-file-naming-conventions

> **Last Updated**: January 1, 2026

# File Naming Conventions

**Version**: 2.0  
**Last Updated**: January 1, 2026  
**Priority**: MANDATORY - All new content must follow these conventions

---

## 🎯 Core Principle

**Files should represent concepts.  
Folders should represent structure.  
Numbers should represent ordering — sparingly.**

If you remember only one line, remember this.

---

## ❌ Anti-Patterns (What NOT to Do)

### ❌ Do NOT encode hierarchy in filenames

**Avoid**:
```
Topic-Part1-A.md
Topic-Part1-B.md
Topic-Part1-A-A.md
Topic-Part1-B-A.md
```

**Why this is wrong**:
- Filenames become brittle and ugly
- `A/B/C` carries no semantic meaning
- Refactoring becomes painful
- URLs are hard to read
- GitHub navigation is confusing

### ❌ Avoid arbitrary "Part" unless it's a published volume

**Avoid**:
```
Glossary-Part1.md
Glossary-Part2.md
```

**Why this is wrong**:
- "Part" is an editorial accident, not a concept
- No semantic meaning for readers
- Hard to maintain and reorganize

### ❌ Avoid mixing sequence, hierarchy, and versioning

**Avoid**:
```
03_Architecture-Patterns-Part1-A-A.md
03_Architecture-Patterns-Part1-B-B.md
```

**Why this is wrong**:
- Confuses sequence with structure
- Makes it impossible to understand relationships
- Breaks when content is reorganized

---

## ✅ Preferred Patterns

### Pattern 1: Semantic Names (Best for Most Cases)

**Use semantic names that describe the content**:

```
evaluation-prep/
├── glossary/
│   ├── README.md
│   ├── core-concepts.md
│   ├── system-design.md
│   ├── cloud-architecture.md
│   ├── data-platforms.md
│   └── security.md
```

**Why this works**:
- Each file has a clear semantic scope
- Easy to add, split, or merge
- GitHub renders `README.md` automatically
- URLs are clean and stable
- Recruiter-friendly and maintainer-friendly

### Pattern 2: Ordered Files (Only When Sequence Matters)

**Use numbered prefixes ONLY when there's a deliberate learning order**:

```
evaluation-prep/
├── glossary/
│   ├── 01_fundamentals.md
│   ├── 02_system-design.md
│   ├── 03_cloud.md
│   └── 04_data.md
```

**Use this ONLY if**:
- There is a deliberate reading order
- You control the learning flow
- Sequence is part of the educational design

**Otherwise, use semantic names (Pattern 1)**.

### Pattern 3: Folder-Based Structure (Best for Growth)

**When content naturally groups, use folders**:

```
evaluation-prep/
├── glossary/
│   ├── README.md        # Index + navigation
│   ├── core-concepts.md
│   ├── system-design.md
│   └── cloud/
│       ├── README.md
│       ├── azure-fundamentals.md
│       ├── aws-fundamentals.md
│       └── multi-cloud.md
```

**Why this works**:
- Structure is clear and navigable
- Easy to expand without renaming
- GitHub navigation is intuitive
- Maintains clean URLs

---

## 📋 Naming Rules

### Rule 1: Use Semantic Names

**✅ Good**:
- `core-concepts.md`
- `system-design.md`
- `azure-fundamentals.md`
- `evaluation-questions.md`

**❌ Bad**:
- `Part1-A.md`
- `Part1-B.md`
- `Glossary-Part1.md`

### Rule 2: Use Hyphens for Multi-Word Names

**✅ Good**:
- `system-design.md`
- `azure-fundamentals.md`
- `evaluation-questions.md`

**❌ Bad**:
- `system_design.md` (underscores)
- `systemDesign.md` (camelCase)
- `System Design.md` (spaces)

### Rule 3: Use Numbers Sparingly

**Use numbers ONLY when**:
- There's a deliberate learning sequence
- Order matters for comprehension
- You're certain the order won't change

**✅ Good** (when sequence matters):
- `01_fundamentals.md`
- `02_advanced-concepts.md`
- `03_practical-applications.md`

**❌ Bad** (arbitrary numbering):
- `01_glossary.md` (if order doesn't matter)
- `02_questions.md` (if order doesn't matter)

### Rule 4: Use Folders for Structure

**When you need hierarchy, use folders, not filenames**:

**✅ Good**:
```
evaluation-prep/
├── questions/
│   ├── fundamentals.md
│   ├── system-design.md
│   └── cloud.md
```

**❌ Bad**:
```
evaluation-prep/
├── questions-fundamentals.md
├── questions-system-design.md
└── questions-cloud.md
```

---

## 🔄 When to Split Files

### Decision Framework

**Question 1**: Does the content exceed 150 lines?
- **Yes** → Continue to Question 2
- **No** → Keep as single file

**Question 2**: Can the content be split into distinct semantic concepts?
- **Yes** → Split into separate files with semantic names
- **No** → Continue to Question 3

**Question 3**: Is there a natural learning progression?
- **Yes** → Use numbered prefixes: `01_fundamentals.md`, `02_advanced.md`
- **No** → Use semantic names: `fundamentals.md`, `advanced.md`

**Question 4**: Will these concepts be expanded further?
- **Yes** → Create a folder structure
- **No** → Keep as separate files

### Example: Splitting a Large Glossary

**Before** (problematic):
```
01_Glossary-Part1-A.md  (150 lines)
01_Glossary-Part1-B.md  (150 lines)
01_Glossary-Part1-C.md  (150 lines)
```

**After** (semantic):
```
glossary/
├── README.md
├── core-concepts.md
├── system-design.md
└── cloud-architecture.md
```

**Or** (if sequence matters):
```
glossary/
├── README.md
├── 01_fundamentals.md
├── 02_system-design.md
└── 03_cloud-architecture.md
```

---

## 📁 Folder Structure Best Practices

### Use Folders When:

1. **Content naturally groups**:
   ```
   evaluation-prep/
   ├── questions/
   │   ├── fundamentals.md
   │   ├── system-design.md
   │   └── cloud.md
   ```

2. **You expect expansion**:
   ```
   evaluation-prep/
   ├── scenarios/
   │   ├── README.md
   │   ├── e-commerce.md
   │   ├── social-media.md
   │   └── video-streaming.md
   ```

3. **You need navigation**:
   ```
   evaluation-prep/
   ├── glossary/
   │   ├── README.md  # Index with links
   │   ├── core-concepts.md
   │   └── advanced.md
   ```

### Folder Naming

- Use semantic names: `questions/`, `scenarios/`, `glossary/`
- Use hyphens for multi-word: `system-design/`, `cloud-architecture/`
- Use numbers ONLY when sequence matters: `01_fundamentals/`, `02_advanced/`

---

## 🔍 Examples: Current vs. Recommended

### Example 1: Glossary

**Current** (problematic):
```
01_Glossary-Part1-A.md
01_Glossary-Part1-B.md
```

**Recommended**:
```
glossary/
├── README.md
├── core-concepts.md
├── system-design.md
```

### Example 2: Architecture Patterns

**Current** (problematic):
```
03_Architecture-Patterns-Part1-A.md
03_Architecture-Patterns-Part1-B.md
03_Architecture-Patterns-Part1-C.md
```

**Recommended**:
```
architecture-patterns/
├── README.md
├── fundamentals.md
├── microservices.md
└── event-driven.md
```

**Or** (if sequence matters):
```
architecture-patterns/
├── README.md
├── 01_fundamentals.md
├── 02_microservices.md
└── 03_event-driven.md
```

### Example 3: Question Banks

**Current** (problematic):
```
03_Question-Bank-Part1-A-A.md
03_Question-Bank-Part1-A-B.md
03_Question-Bank-Part1-B-A.md
03_Question-Bank-Part1-B-B.md
```

**Recommended**:
```
question-bank/
├── README.md
├── fundamentals.md
├── intermediate.md
├── advanced.md
└── expert.md
```

**Or** (if organized by topic):
```
question-bank/
├── README.md
├── azure.md
├── kubernetes.md
├── system-design.md
└── devops.md
```

---

## 🚨 Migration Strategy

### For New Content

**ALWAYS** use semantic names. Never use `Part1-A` patterns.

### For Existing Content

**Option A: Gradual Migration** (Recommended)
- New content uses semantic names
- Existing content remains until refactored
- Migrate when files are updated or expanded

**Option B: Systematic Refactoring**
- Create migration plan
- Refactor folder by folder
- Update all references
- Test thoroughly

---

## 📝 Applying Naming Conventions

### When Creating New Content

**Before writing**:

1. **Choose the right folder** based on learning progression or content type
   - For learning content: Use numbered folders (`01_Reference/`, `02_Learning/`, `src/01_introduction/`, etc.)
   - For reference-style content: Create a folder with `README.md` index (e.g., `glossary/`)

2. **Choose a filename that matches the concept boundary**:
   - Prefer `NN_concept-name.md` for sequence-based modules (e.g., `01_fundamentals.md`)
   - Prefer `concept-name.md` inside reference-style folders (e.g., `core-concepts.md`)

3. **Keep names URL-friendly**: lowercase, hyphen-separated words, avoid spaces

### When Splitting a File

**Decision process**:

1. **First try to split by concept** (new file) instead of by "Part"
   - Example: `fundamentals.md` and `advanced.md` instead of `topic-part1.md` and `topic-part2.md`

2. **If you still need parts** (mechanical splitting for 150-line limit):
   - Use `-part1`, `-part2` (not `A/B/C`)
   - Preserve the base slug so links remain predictable
   - Example: `03_consistency-models-part1.md`, `03_consistency-models-part2.md`

3. **If hierarchy is needed**: Move structure into folders + `README.md` indexes instead of growing filename suffixes

### When Reviewing/Editing Content

**During review, treat naming as part of "maintainability"**:

- **Does the filename match the scope?** If the file title/scope drifted, rename the file or split it
- **Is hierarchy in the right place?** Move structure into folders + `README.md` indexes instead of growing filename suffixes
- **Is ordering intentional?** If numbers exist, they should reflect a real learning sequence. Don't renumber casually (it creates churn and breaks links)
- **Are split files clean?** Prefer `-partN` over letter suffixes; ensure parts are similarly sized and self-contained
- **Are links updated?** If any rename happened, update all inbound links and re-run validation

### Recommended Patterns by Folder Type

**In main learning folders** (e.g., `src/01_Reference/`, `src/02_Learning/`, `src/01_introduction/`):
- Use ordered files: `NN_topic-slug.md`
- If you must split: `NN_topic-slug-part1.md`, `NN_topic-slug-part2.md` (keep the same `NN_` prefix)

**In reference-style subfolders** (e.g., `glossary/`, `evaluation-prep/`):
- Prefer `README.md` as the index + semantic topic files like `core-concepts.md`, `security.md`
- Add ordering numbers only if there is a deliberate reading sequence

**In technical evaluation prep folders** (e.g., `src/03_Technical-Evaluation-Prep/`, `src/02_evaluation-prep/`):
- Use semantic names: `evaluation-overview.md`, `study-roadmap.md`, `skills-checklist.md`
- Use folders for grouping: `azure-architect/`, `devops-architect/`
- Use `README.md` for navigation within each folder

---

## 📝 Quick Reference

### Decision Tree

```
Is content > 150 lines?
├─ No → Single file with semantic name
└─ Yes → Can it be split into distinct concepts?
    ├─ Yes → Separate files with semantic names
    └─ No → Is there a learning sequence?
        ├─ Yes → Numbered files: 01_, 02_, etc.
        └─ No → Use -part1, -part2 for mechanical splitting
```

### Rules of Thumb

- **If you feel the need to add `A/B/C` → you need folders**
- **If you feel the need to add `Part1/Part2` → you need better conceptual boundaries**
- **If the filename is doing structural work → it's already wrong**
- **Use `-part1`, `-part2` only for mechanical splitting** (to respect ~150 line limit), not as a long-term hierarchy

---

## ✅ Checklist for New Files

Before creating a new file, ask:

- [ ] Does the filename describe the content semantically?
- [ ] Is the filename readable in a URL?
- [ ] Will the filename survive reordering?
- [ ] Does the filename work with GitHub navigation?
- [ ] Am I encoding hierarchy in the filename? (If yes, use folders instead)

---

**Related Rules**:
- [Educational Content Rules](./01_educational-content-rules.mdc) - Content structure and splitting policy
- [Repository Structure](./02_repository-structure.mdc) - Folder organization

---
> Source: [SwamysArchitectJourney-2026/architecture-reasoning-in-practice](https://github.com/SwamysArchitectJourney-2026/architecture-reasoning-in-practice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
