## 01-educational-content-rules

> enables: ["../advanced-content/"]

# Content Rules (Architecture Reasoning in Practice)

**Version**: 2.0  
**Last Updated**: January 1, 2026  
**Priority**: MANDATORY - All content creation must follow these rules without exception

---

## 📋 Rule Applicability

**IMPORTANT**: These rules apply differently based on content type:

### Practice Content (`src/01_reasoning-foundations/`, `src/02_answer-structuring/`, `src/03_tradeoff-articulation/`, `src/04_role-perspectives/`, `src/05_evaluation-scenarios/`)
- ✅ **File naming**: Descriptive names (e.g., `problem-framing.md`, `cqrs-selective-application.md`) - **NO numbering required** for content files
- ✅ **Folder naming**: Folders use numbered prefixes (`01_reasoning-foundations/`, `02_answer-structuring/`) - **ALWAYS numbered**
- ✅ **Line limits**: Recommended ≤150 lines (split, don't trim)
- ✅ **YAML frontmatter**: Recommended for content files (all 5 metadata fields when content is added)
- ✅ **Zero-copy policy**: Applies (content must be transformative)
- ✅ **File references**: Must point to existing files

**Numbering Rules Summary**:
- **Folders**: Always use numbered prefixes (`01_`, `02_`, etc.) - **NEVER use `00_`**
- **Content files**: Use descriptive names without numbering (e.g., `decision-rationale-framing.md`)
- **Split files**: When splitting content, use `-part1`, `-part2` suffixes (e.g., `topic-part1.md`, `topic-part2.md`) - no numbered prefix on content files

### Resources (`src/resources/` directory)
- ✅ **File naming**: Logical names (`frameworks.md`, `reference-materials.md`, `tools.md`)
- ✅ **Numbering**: NOT required (reference materials)
- ⚠️ **YAML frontmatter**: NOT required

---

## 🚫 Zero-Copy Policy (Non-Negotiable)

**CRITICAL**: All content (case study documentation and educational content) must be transformative, not reformative.

❌ **NEVER** copy text verbatim from books, articles, websites, videos, or third-party materials  
❌ **NEVER** mirror a source's outline, section order, headings, or example sequence  
❌ **NEVER** use "light paraphrasing" — must transform completely  
✅ **ALWAYS** create diagrams in Mermaid-first style with ASCII fallback (never embed copyrighted figures)  
✅ **ALWAYS** write fresh, minimal code from first principles  
✅ Brief quotations allowed ONLY with quotation marks and source citation

---

## 🔄 Transformative Workflow (Required Every Time)

**Step-by-step process for creating original educational content**:

1. **Source Intake**: Skim for intent and big ideas; don't copy notes verbatim
2. **Concept Map**: Create fresh outline with different sectioning tailored to Architecture Reasoning learning
3. **Teach Differently**: Use new analogies, scenarios, examples, use-cases (avoid source examples)
4. **Produce Original Artifacts**: Explanations, diagrams (Mermaid with ASCII fallback), minimal examples
5. **Cross-Link**: Add references and connections across Architecture Reasoning thinking modes
6. **Similarity Audit**: Ensure no sentences/structures resemble source
7. **Zero-Copy Verification**: **MANDATORY** - Verify no verbatim text, especially in quotes and "Key Principle" sections
8. **Optional References**: Add "References/Inspired by" links (no copied phrasing)

**Goal**: Create transformative educational content, not just reformative. Entirely new presentation, examples, and explanations that teach the same concepts through original methods.

**⚠️ CRITICAL REMINDER**: Even "Key Principle" quotes and example structures must be completely transformed. Verbatim copying of ANY text from source material violates the zero-copy policy.

---

## ⏱️ 25-Minute Learning Segments

**APPLICABILITY**: This rule applies to practice content files (`src/01_reasoning-foundations/`, `src/02_answer-structuring/`, `src/03_tradeoff-articulation/`, `src/04_role-perspectives/`, `src/05_evaluation-scenarios/`).

**For Lab Documentation**: The 150-line limit is a **recommendation**, not a strict requirement. Lab files may exceed 150 lines if needed for comprehensive instructions.

✅ **Modular content** designed for focused 25-minute sessions  
✅ **Multi-Part Structure**: Complex topics split into Part 1, Part 2, ... Part N  
✅ **One-Shot Learning**: Each segment complete and actionable within time limit  
✅ **Target Length**: 150 lines of content maximum per response (educational content)

### ⚠️ CRITICAL: Splitting vs. Trimming Policy

**APPLICABILITY**: This rule applies to practice content files (`src/01_reasoning-foundations/`, `src/02_answer-structuring/`, `src/03_tradeoff-articulation/`, `src/04_role-perspectives/`, `src/05_evaluation-scenarios/`).

**For Lab Documentation**: If lab files exceed 150 lines, consider splitting for better organization, but it's not mandatory.

**MANDATORY APPROACH** (Educational Content Only): When content exceeds 150 lines, **ALWAYS SPLIT** into multiple parts. **NEVER TRIM** or condense content.

**Why Splitting is Required:**
- ✅ **Preserves ALL educational content** - No loss of examples, explanations, or concepts
- ✅ **Maintains learning value** - Each part remains complete and actionable
- ✅ **Better learning experience** - Learners get comprehensive coverage across parts
- ✅ **Follows 25-minute principle** - Each part fits within focused learning session

**Why Trimming is Prohibited:**
- ❌ **Loses educational content** - Examples, explanations, or concepts may be removed
- ❌ **Reduces learning value** - Condensed content may miss important details
- ❌ **Violates zero-copy policy** - If source had comprehensive content, we should preserve it
- ❌ **Poor learning experience** - Learners miss important information

**Splitting Process:**
1. **Identify logical breakpoints** - Split at natural topic boundaries (e.g., "Foundation" → "Implementation" → "Advanced")
2. **Preserve all content** - Move content to appropriate part, don't delete
3. **Maintain completeness** - Each part should be self-contained and complete
4. **Use proper naming** - Follow naming convention: `Part1-A.md`, `Part1-B.md`, etc.
5. **Update references** - Update all file references after splitting

**Example**: If a file has 300 lines covering "Factory Pattern Fundamentals" and "Factory Pattern Advanced", split into:
- `01_Factory-Pattern-Part1-A.md` (Fundamentals - 150 lines)
- `01_Factory-Pattern-Part1-B.md` (Advanced - 150 lines)

**NOT**: Condense to 150 lines by removing examples or explanations.

---

## 📋 Required Content Structure

### 5 Required Metadata Fields (Educational Content Only)

**APPLICABILITY**: These metadata fields are RECOMMENDED for practice content files in `src/01_reasoning-foundations/`, `src/02_answer-structuring/`, `src/03_tradeoff-articulation/`, `src/04_role-perspectives/`, and `src/05_evaluation-scenarios/` directories (when content is added).

**NOT REQUIRED** for:
- Lab files (`src/labs/` directory)
- Notes files (`src/notes/` directory)
- Resources files (`src/resources/` directory)
- General documentation (`docs/` directory)

Every educational content file MUST include:

```yaml
---
learning_level: "Beginner" | "Intermediate" | "Advanced"
prerequisites: ["required knowledge", "prior concepts"]
estimated_time: "25 minutes"  # Standard, adjust if needed
learning_objectives:
  - "Specific, measurable outcome 1"
  - "Specific, measurable outcome 2"
related_topics:
  prerequisites: ["../prerequisite-content/"]
  builds_upon: ["../foundational-content/"]
  enables: ["../advanced-content/"]
  cross_refs: ["../related-domains/"]
---
```

### Numbering Convention

✅ **ALWAYS** use zero-padded numeric prefixes starting at `01_`  
❌ **NEVER** use `00_` prefixes - **NO EXCEPTIONS**  
✅ Keep numbering stable; add new numbers rather than renumbering widely  
✅ Use hyphens for multi-word names: `01_Software-Design-Principles/`

**CRITICAL**: This rule applies to **ALL files** in the repository:
- ✅ Practice content files (`src/01_reasoning-foundations/`, `src/02_answer-structuring/`, etc.)
- ✅ Documentation files (`docs/`)
- ✅ Any numbered files anywhere in the repository
- ❌ **NO EXCEPTIONS** - `00_` is NEVER allowed, even for meta/documentation files

**Why This Rule Exists**:
- Maintains consistent numbering across the repository
- Prevents confusion about file ordering
- Ensures predictable file organization
- `01_` clearly indicates the first item, `00_` is ambiguous

### Learning Order Requirements (CRITICAL)

**CRITICAL**: Folder numbering reflects Architecture Reasoning thinking mode progression.

**Architecture Reasoning Thinking Mode Progression** (src/01_reasoning-foundations/ → src/05_evaluation-scenarios/):

1. **01_reasoning-foundations**: Problem framing, clarification strategies, assumptions and constraints
2. **02_answer-structuring**: Top-down communication, depth control, time-boxed reasoning
3. **03_tradeoff-articulation**: Cost vs scale, simplicity vs flexibility, risk and failure framing
4. **04_role-perspectives**: How different roles think when solving the same ambiguous problem
5. **05_evaluation-scenarios**: Vague problems, conflicting requirements, legacy modernization

**Why This Order Matters**:

- Reasoning foundations (01) must come first - learners need problem framing and clarification skills
- Answer structuring (02) builds on foundations - organize communication after understanding problem framing
- Trade-off articulation (03) requires structuring skills - evaluate options after learning to organize answers
- Role perspectives (04) applies reasoning skills - understand how roles differ after mastering reasoning
- Evaluation scenarios (05) integrates all skills - practice with real scenarios after building foundation

**When Creating New Content**:

- ✅ Verify prerequisites are numbered BEFORE the new content
- ✅ Check that "enables" relationships point to content numbered AFTER
- ✅ Ensure learning dependencies match file numbering order
- ❌ NEVER place content that depends on later-numbered files

### File Naming Convention for Split Files

**CRITICAL**: When files are split into multiple parts, use `-part1`, `-part2` pattern (not `Part1-A`, `Part1-B`).

**Current Repository Pattern**:

- ✅ `02_network-abstractions-part1.md`
- ✅ `02_network-abstractions-part2.md`
- ✅ `03_consistency-models-part1.md`
- ✅ `03_consistency-models-part2.md`

**Rules**:
1. ✅ **Use `-part1`, `-part2` suffixes** - Preserves base slug for predictable links (e.g., `topic-part1.md`, `topic-part2.md`)
2. ✅ **NO numbered prefix on content files** - Content files use descriptive names, not numbered prefixes
3. ✅ **Avoid letter suffixes** - Don't use `Part1-A`, `Part1-B` patterns
4. ✅ **Maintain order** - Parts reflect sequential content flow

**When to Use This Pattern**:
- Content exceeds 150 lines
- Cannot be split into distinct semantic concepts
- Content is a single cohesive topic that must be split mechanically

**For Better Approaches**: See [File Naming Conventions](../.cursor/rules/07_file-naming-conventions.mdc) for semantic naming patterns and decision framework.

**Benefits**:
- Cleaner, more readable file names
- Easier to understand part relationships
- Maintains file reference stability
- Better than `Part1-A`, `Part1-B` patterns

---

## 🎓 Educational Excellence Standards

All content must demonstrate:

- ✅ **Clear objectives and outcomes**: Specific, measurable learning goals
- ✅ **Progressive scaffolding**: Foundations → Practice → Pitfalls → Next Steps
- ✅ **Original examples, datasets, and exercises**: Never reuse source examples
- ✅ **Mermaid-first visuals**: Primary Mermaid diagrams with ASCII fallback for compatibility
- ✅ **Cross-references across tracks**: Development, AI/ML, Data Science, DevOps

---

## ✅ Quality Gate Questions (Before Publishing)

**Self-check before finalizing any educational content**:

1. ✅ Is this explanation clearer than the source material?
2. ✅ Does this fit naturally in the learning progression?
3. ✅ Would a learner understand this without the original source?
4. ✅ Are the examples relevant and practical?
5. ✅ Does this content add educational value beyond the reference?
6. ✅ Is this content within 150 lines for effective delivery?

---

## 📁 Content Placement Policy

✅ `src/` folders are EXCLUSIVELY for learning content  
❌ Never mix planning materials, workflow docs, or meta content  
✅ Group logically by thinking mode progression (01_reasoning-foundations → 05_evaluation-scenarios)  
✅ Place content in correct numbered folder based on Architecture Reasoning thinking modes

### 🚨 CRITICAL: Code Examples Policy

**This repository is EXCLUSIVELY for educational content. Code implementations are in separate repositories.**

**Code Examples in Educational Content:**

- ✅ **Minimal, Illustrative Snippets**: Code examples should be minimal and focused on teaching concepts
- ✅ **Teaching-Focused**: Code should demonstrate principles, patterns, or concepts, not provide complete implementations
- ✅ **Reference Full Code**: Link to full implementations in separate code repositories (Python, CSharp, JavaScript, etc.)
- ❌ **NO Full Implementations**: Do not include complete, runnable code projects in this repository
- ❌ **NO Production Code**: Do not include production-ready, deployable code in educational content
- ❌ **NO Code Repositories**: All full code implementations belong in separate GitHub repositories within the organization

**When Creating Code Examples:**

1. **Keep it minimal**: Show only the essential code needed to illustrate the concept
2. **Add comments**: Explain what the code demonstrates
3. **Link to full code**: Reference the separate code repository where full implementation exists
4. **Focus on learning**: Code should teach, not provide complete solutions

**Example Structure:**

```markdown
## Factory Pattern Example

Here's a minimal example demonstrating the Factory Pattern:

```python
# Minimal illustrative example
class ProductFactory:
    def create_product(self, product_type):
        if product_type == "A":
            return ProductA()
        return ProductB()
```

> **Full Implementation**: See complete code examples in the `src/` directory
```

### Source Materials Staging Area

**Location**: `source-material/` (at repository root, git-ignored)

**Purpose**: **Staging folder for migration** - Temporary staging area where source content is placed before review and transformation into educational content.

**Critical Workflow**:

1. **Place materials**: User places source materials (transcripts, notes, documents) in `source-material/` folder
2. **Review and migrate**: AI assistant reviews content, identifies unique topics, and migrates/transforms following Educational Content Rules
3. **Verify migration**: Confirm all unique content has been migrated to appropriate `src/` folders
4. **Keep source files**: After successful migration, keep source files in `source-material/` folder - user will delete manually

**Important Notes**:

- ⚠️ **Files in `source-material/` are NOT required to be compliant** - this is a staging area for raw source content
- ❌ **NEVER MODIFY files in `source-material/`** - AI assistants must ONLY READ these files for migration purposes, never edit, format, or change them
- ✅ **Review rules apply DURING transformation** - ensure transformation process follows all Educational Content Rules
- ✅ **When user requests migration**: Review ALL files in `source-material/`, identify unique content, and migrate following Educational Content Rules
- ✅ Files here will be transformed following Educational Content Rules into compliant content
- ✅ After transformation, create compliant content in appropriate `src/` domain folders
- ✅ **After successful migration**: Keep source files in `source-material/` folder - user will delete manually
- ❌ **Never commit `source-material/` content** - it's git-ignored for a reason
- ✅ **Keep `source-material/` folder** - it's a permanent staging area for future migrations

**Transformation Workflow** (Using CoT, ReAct, and Reasoning):

**🚨 CRITICAL**: This workflow uses **THE EXACT SAME 7-CATEGORY REVIEW CHECKLIST** as the Comprehensive Content Review Process. Every file created during migration MUST pass all review checks before being considered complete.

1. **OBSERVE**: Place source materials (transcripts, notes, etc.) in `source-material/` (at repository root)
   - Scan and catalog source content
   - Identify key concepts and learning objectives
   - Understand source structure and dependencies

2. **ANALYZE**: Apply review rules DURING transformation using CoT/ReAct methodology:
   - **Reason through transformation**: Break down source content into logical learning segments
   - **Apply 7-category review checklist while transforming**:
     - Verify YAML frontmatter structure as you create it
     - Ensure content is transformative (not copied) during transformation
     - **Check line count as you build content (≤ 150 lines) - SPLIT if exceeds, NEVER TRIM**
     - Validate file naming conventions during creation
     - Verify file references as you add them
     - Ensure zero-copy policy compliance during transformation
     - Confirm learning progression logic as you structure content
     - **Preserve ALL educational content through splitting, not condensing**
   - **Use Chain-of-Thought**: Think through each transformation step explicitly
   - **Apply Reasoning**: Make logical decisions about content structure, examples, and explanations

3. **REASON**: Create new educational content in appropriate `src/01_reasoning-foundations/`, `src/02_answer-structuring/`, `src/03_tradeoff-articulation/`, `src/04_role-perspectives/`, or `src/05_evaluation-scenarios/` folders
   - Apply logical reasoning to determine correct placement
   - Ensure learning dependencies are properly structured
   - Verify content flow and progression

4. **VERIFY**: Review final content using comprehensive review checklist before committing
   - Cross-check all requirements
   - Validate compliance with all rules
   - Confirm zero-copy policy adherence
   - **Create migration verification report** in `docs/review-reports/` with date-based naming (e.g., `24Nov2025.md`)

5. **ACT**: After successful migration and verification:
   - Keep source files in `source-material/` folder - user will delete manually
   - Save migration verification report to `docs/review-reports/` with date-based filename

**Compliance Requirements**:
- ❌ `source-material/` files: **NO compliance required** (staging area - raw source content)
- ✅ **Transformation process**: **MUST follow review rules** (apply checklist during transformation)
- ✅ `src/01_reasoning-foundations/` files: **FULL compliance required** (final content - must pass all review checks)
- ✅ `src/02_answer-structuring/` files: **FULL compliance required** (final content - must pass all review checks)
- ✅ `src/03_tradeoff-articulation/` files: **FULL compliance required** (final content - must pass all review checks)
- ✅ `src/04_role-perspectives/` files: **FULL compliance required** (final content - must pass all review checks)
- ✅ `src/05_evaluation-scenarios/` files: **FULL compliance required** (final content - must pass all review checks)

**Review During Migration** (Using CoT, ReAct, and Reasoning):

- **OBSERVE**: Systematically analyze source content before transformation
- **ANALYZE**: Apply the 7-category Individual File Review Checklist **while transforming** content
  - Use Chain-of-Thought to break down transformation into logical steps
  - Apply ReAct methodology: Observe → Analyze → Reason → Verify → Act
  - Use systematic reasoning to make transformation decisions
- **REASON**: Make logical decisions about:
  - Content structure and organization
  - Example selection and creation
  - Learning progression and dependencies
  - File naming and references
- **VERIFY**: Don't wait until after migration - catch issues during the transformation process
  - Verify compliance at each step: YAML structure, line count, naming, references, zero-copy policy
  - Cross-check findings as you transform
- **ACT**: Final review after migration confirms all requirements are met

**🚨 CRITICAL RULE ALIGNMENT**: Migration and Review use **THE EXACT SAME RULES AND CHECKLIST**. The 7-category Individual File Review Checklist (YAML Frontmatter, Content Structure, File Naming, File References, Content Quality, Zero-Copy Policy, Learning Progression) MUST be applied during migration/transformation, not just during review. The same CoT, ReAct, and Reasoning methodology used for reviews MUST be applied during migration/transformation.

### Learning Paths Creation Policy

**Note**: The repository structure is already established with numbered thinking mode folders (`01_reasoning-foundations/` through `05_evaluation-scenarios/`). Content should be added progressively following Educational Content Rules aligned with Architecture Reasoning thinking modes.

---

## 🔗 File Reference Requirements (CRITICAL)

### Mandatory Practices

**CRITICAL**: All file references MUST point to existing files or be clearly marked as planned content.

### When Creating File References

1. ✅ **Verify file exists** before adding reference
2. ✅ **Use exact file names** - match actual file names exactly (including all part suffixes)
3. ✅ **Test references** - run `.\tools\psscripts\Validate-FileReferences.ps1` after adding references
4. ✅ **Update after splitting** - when splitting files, update ALL references to that file immediately

### When Splitting Files

**CRITICAL WORKFLOW**:

1. Create new part files
2. **IMMEDIATELY** run: `.\tools\psscripts\Validate-FileReferences.ps1` to find all references
3. Update ALL references to point to correct part files
4. Run validation again to verify
5. Test navigation manually

### Reference Patterns

- **Prerequisites/Builds Upon**: Use first part (`-Part1-Part1.md`) or specific part if needed
- **Enables**: Can reference any part, typically next part in sequence
- **Planned Content**: References to files that don't exist yet are acceptable IF clearly marked as planned

### Validation Checklist

Before committing:
- [ ] Run `.\tools\psscripts\Validate-FileReferences.ps1`
- [ ] All references point to existing files (or are clearly planned)
- [ ] Navigation links work in markdown preview
- [ ] No broken references remain

---

## 🎯 Content Guidelines for Architecture Reasoning

**CRITICAL**: All content in practice folders must align with Architecture Reasoning thinking modes and focus on how senior people think, reason, and communicate.

### Content Quality Policy

✅ **ALWAYS** align content with Architecture Reasoning thinking modes  
✅ **ALWAYS** include practical examples and real-world scenarios  
✅ **ALWAYS** focus on reasoning, articulation, and evaluation skills  
✅ **ALWAYS** provide hands-on examples where applicable  
❌ **NEVER** include system design depth (belongs in `system-design-in-practice`)  
❌ **NEVER** create content that's too implementation-focused  

**Rationale**: Content should be focused on architectural reasoning, decision-making, and communication skills. System design details belong in the separate `system-design-in-practice` repository.

**When Creating Practice Content**:
1. Focus on thinking modes and reasoning approaches
2. Emphasize problem framing, clarification, and articulation
3. Include examples that demonstrate reasoning under evaluation
4. Ensure content is reasoning-focused and actionable

---

## 🔍 Comprehensive Content Review Process

**APPLICABILITY**: This review process applies to practice content files (`src/01_reasoning-foundations/`, `src/02_answer-structuring/`, `src/03_tradeoff-articulation/`, `src/04_role-perspectives/`, `src/05_evaluation-scenarios/`).

**For Lab Documentation**: Review for content quality, accuracy, and consistency, but YAML frontmatter and strict line limits are not required.

**MANDATORY**: All content in `src/01_reasoning-foundations/`, `src/02_answer-structuring/`, `src/03_tradeoff-articulation/`, `src/04_role-perspectives/`, and `src/05_evaluation-scenarios/` folders must undergo comprehensive review using CoT (Chain-of-Thought), ReAct (Reasoning + Acting), and systematic reasoning.

**🚨 CRITICAL RULE ALIGNMENT**: Migration and Review use **THE EXACT SAME RULES AND CHECKLIST**. The 7-category Individual File Review Checklist MUST be applied during migration/transformation, not just during review. This ensures all content is compliant from the moment it's created.

### Review Request Protocol

**When a review is REQUESTED**:

- ✅ **MANDATORY**: Review EACH AND EVERY file individually - no file should be skipped
- ✅ **MANDATORY**: Open and examine every file in the specified scope
- ✅ **MANDATORY**: Apply the 7-category review checklist to every single file
- ❌ **NEVER**: Skip files, assume compliance, or review only a sample
- ❌ **NEVER**: Review only files that "look problematic" - review ALL files

**Scope of Review**:
- If reviewing `src/01_reasoning-foundations/` folder: Review ALL files in `src/01_reasoning-foundations/`
- If reviewing `src/02_answer-structuring/` folder: Review ALL files in `src/02_answer-structuring/`
- If reviewing `src/03_tradeoff-articulation/` folder: Review ALL files in `src/03_tradeoff-articulation/`
- If reviewing `src/04_role-perspectives/` folder: Review ALL files in `src/04_role-perspectives/`
- If reviewing `src/05_evaluation-scenarios/` folder: Review ALL files in `src/05_evaluation-scenarios/`
- If reviewing `src/labs/` folder: Review ALL files in `src/labs/`
- If reviewing a specific subfolder: Review ALL files in that subfolder
- If reviewing specific files: Review each requested file individually

### Review Methodology

**Use CoT, ReAct, and Reasoning for every review**:

1. **OBSERVE**: Systematically scan and catalog all files
   - List every file in the review scope
   - No file should be excluded or skipped
2. **ANALYZE**: Review each file individually with deep analysis
   - Open and examine every file
   - Apply review checklist to each file
3. **REASON**: Apply logical reasoning to identify issues and patterns
4. **VERIFY**: Cross-check findings and validate compliance
5. **ACT**: Document findings and update content as needed

### Individual File Review Checklist

**Review EACH AND EVERY file individually** - no file should be skipped:

#### 1. YAML Frontmatter Review
- [ ] YAML frontmatter present (starts with `---`)
- [ ] All 5 required metadata fields present:
  - [ ] `learning_level` (Beginner/Intermediate/Advanced/Reference)
  - [ ] `prerequisites`
  - [ ] `estimated_time`
  - [ ] `learning_objectives`
  - [ ] `related_topics` (with `enables:` key)
- [ ] No placeholder patterns (`$101_`, `$102_`, etc.)
- [ ] YAML syntax is valid
- [ ] `enables:` key present in `related_topics` section

#### 2. Content Structure Review
- [ ] File length ≤ 150 lines (excluding YAML frontmatter)
- [ ] **If content exceeds 150 lines**: File has been SPLIT into multiple parts (not trimmed)
- [ ] All educational content preserved across split parts
- [ ] Has clear headings (## level)
- [ ] Content is modular and focused
- [ ] Follows 25-minute learning segment principle

#### 3. File Naming Review
- [ ] Uses zero-padded numeric prefix (`01_`, `02_`, etc.)
- [ ] **CRITICAL**: Never uses `00_` prefix - **NO EXCEPTIONS** (applies to ALL files including `docs/`)
- [ ] Split files use correct naming: `Part1-A.md`, `Part1-B.md` (not `Part1A.md`)
- [ ] Hyphens used for multi-word names
- [ ] Rule applies to educational content AND documentation files

#### 4. File References Review
- [ ] All `enables:` references point to existing files
- [ ] All `prerequisites:` references point to existing files
- [ ] All `builds_upon:` references point to existing files
- [ ] File names in references match actual file names exactly
- [ ] No broken references remain

#### 5. Content Quality Review
- [ ] Has code examples (if applicable)
- [ ] Has diagrams (Mermaid or ASCII)
- [ ] Content is transformative (not copied)
- [ ] No suspicious patterns (Copyright, "All rights reserved", etc.)
- [ ] Learning objectives are clear and measurable
- [ ] Progressive scaffolding present (Foundations → Practice → Pitfalls → Next Steps)

#### 6. Zero-Copy Policy Review (CRITICAL)
- [ ] **No verbatim text from sources** - Check ALL quotes, especially "Key Principle" sections
- [ ] **All quotes transformed** - Even example quotes must use original phrasing
- [ ] **No mirrored outlines or section order** - Structure must be different from source
- [ ] **Original examples and explanations** - No source examples reused
- [ ] **Diagrams are original** - Mermaid/ASCII only, not embedded copyrighted figures
- [ ] **Content adds educational value beyond source** - Not just reformatted
- [ ] **Manual verification completed** - Searched for known source material phrases

#### 7. Learning Progression Review
- [ ] File numbering reflects logical learning dependencies
- [ ] Prerequisites come before dependent content
- [ ] `enables:` relationships point to content numbered after
- [ ] Learning order is logical and sequential

### Deep Dive Review Process

**When performing comprehensive review**:

1. **Systematic File Scanning**
   ```powershell
   # Get all files to review
   Get-ChildItem "src\01_reasoning-foundations" -Recurse -Filter "*.md" | 
       Where-Object { $_.Name -ne "README.md" }
   ```

2. **Individual File Analysis**
   - Open each file
   - Check YAML frontmatter structure
   - Verify all metadata fields
   - Count lines (must be ≤ 150)
   - Check file naming conventions
   - Validate all file references
   - Review content quality indicators

3. **Reference Validation**
   ```powershell
   # Run reference validation
   .\tools\psscripts\Validate-FileReferences.ps1
   ```

4. **Content Quality Analysis**
   - Check for code examples
   - Check for diagrams (Mermaid/ASCII)
   - Check for headings
   - Check for suspicious patterns
   - Verify zero-copy compliance

5. **Documentation**
   - Document all findings
   - Create issue list with file paths
   - Prioritize issues (critical vs. warnings)
   - Track fixes and verification

### Review Tools

**Use these tools for comprehensive review**:

- `.\tools\psscripts\Comprehensive-ReferenceReview.ps1` - Deep dive review with CoT/ReAct methodology
- `.\tools\psscripts\Validate-FileReferences.ps1` - File reference validation
- `.\tools\psscripts\Review-EducationalContent.ps1` - General compliance review

### Review Frequency

- **Before committing**: Review all modified files
- **After splitting files**: Review all affected files and references
- **Periodic audits**: Comprehensive review of educational content folders
- **After major changes**: Full review of affected sections

### Review Documentation

**Always document review findings**:

- Create review reports with findings
- List all issues found (with file paths)
- Track compliance statistics
- Document fixes applied
- Verify fixes after remediation

---
> Source: [SwamysArchitectJourney-2026/architecture-reasoning-in-practice](https://github.com/SwamysArchitectJourney-2026/architecture-reasoning-in-practice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
