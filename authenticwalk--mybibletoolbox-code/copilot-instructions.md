## mybibletoolbox-code

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the myBibleToolbox project - an initiative to create the largest AI-readable commentary on the entire Bible. The goal is to provide extensive context data for AI systems to ground their responses in truth when working with Biblical texts, especially for Bible translators, pastors, and students.

### Theological Foundation: Conservative Protestant Christian

**CRITICAL**: This is a CHRISTIAN project, grounded in conservative Protestant orthodoxy. All theological content follows this framework:

**1. Primary Perspective: Historic Christian Orthodoxy**
- Conservative Protestant Christian is the PRIMARY theological perspective
- Grounded in Scripture as God's inerrant Word
- Affirms historic Christian creeds (Nicene, Apostles', Athanasian)
- Core doctrines: Trinity, deity of Christ, salvation by grace through faith, biblical authority

**2. Christian Denominational Variations**
- Show views of ALL Christian traditions for consideration:
  - Protestant (various denominations)
  - Catholic (Roman Catholic)
  - Orthodox (Eastern Orthodox, Oriental Orthodox)
  - Coptic
  - Other historic Christian traditions
- Present these as VALID CHRISTIAN perspectives worth considering
- Note where they differ on non-essential matters while affirming shared orthodoxy

**3. Non-Orthodox Views (Cults and Other Religions)**
- May note alternate views from:
  - Cults claiming to be Christian (Jehovah's Witnesses, Mormons, etc.)
  - Non-Christian religions (Judaism, Islam, etc.)
- MUST explain WHY these are rejected by Christian orthodoxy
- Label clearly as "not recognized by Christian orthodoxy"
- Purpose: Help translators avoid theological errors, understand why certain interpretations are problematic

**Language to Use**:
- ✅ "Christian orthodox position"
- ✅ "Denominational considerations within Christianity"
- ✅ "Rejected by Christian orthodoxy because..."
- ✅ "Non-orthodox view from [group]"
- ❌ "All interpretations equally valid"
- ❌ Treating cults/other religions as valid Christian alternatives

**Scope**: Applies to all theological content - TBTA features, commentary, translation guidance

## AGENT behaviour
 - NEVER write notes or summaries to the home directory, instead create a plan in ./plans/{your-plan}/README.md then update it with results.  Keep the home directory clean.
 - When moving files always do a git commit before editing it so the git history is preserved
 - **Use subagents proactively from the start** - Don't wait until context is full. Delegate web research, file searches, experiments, and large tasks to subagents immediately to stay under 80% context. Focus on oversight, synthesis, and planning.
 - Prefer calling subagents in parallel, when possible 8 in parallel.
 - **Push after EVERY commit** - Run `git push -u origin <branch>` immediately after EVERY `git commit`. Never batch commits before pushing. Pattern: make change → commit → push → repeat.

### The Problem Being Solved

AI text prediction models tend to be more confident than accurate when dealing with Biblical texts. While they perform well on commonly cited passages (like John 3:16) in popular translations (NIV, KJV), accuracy degrades significantly with:
- Lesser-quoted texts (minor prophets, Numbers, etc.)
- Rare or minority language translations
- Verses that lack extensive Internet training data

### The Solution

By providing extensive contextual data (essentially "a book's worth of information") for every verse broken into smaller task focused files, AI systems can be grounded in truth rather than relying solely on compressed training data.

### Really important files

The following are key files you can load.  Don't load them automatically as it will fill your context, I've summarized the key points, read it for edge cases

 - STANDARDIZATION.md
   - 3 char ISO-639 language codes
   - USFM 3.0 for Bible book codes
   - pad numbers so they sort nicely
   - /commentary/{BOOK}/{chapter:03d}/{BOOK}.{chapter:03d}.{verse:03d}-{tool}.yaml for commentary files
   - /words/strongs/(G|H){strongs-number:04d}/{strongs-number}.strongs-{tool}.yaml for source language data
   - /topics/{lcc-code}/{slug}/{slug}-{tool}.yaml for topics
 - SCHEMA.md
   - cite sources inline with {source-id}: `{lang}-{version}` → `{lang}-{version}-{year}` for specificity
   - NEVER hallucinate: if from your memory use {llm-cs45}, if uncertain omit it
   - standard sections: source, translation, words, grammar, context, themes, cross_refs
   - loosely structured for easy merging/filtering across tools, use standard names so combines nicely on common structures, unstructured for unique values of the tool
 - REVIEW-GUIDELINES.md
   - 3 validation levels: critical (must pass), high priority (80%+), medium (60%+)
   - no fabrication, inline citations, no number predictions, data-file grounding only
   - new sources must be in ATTRIBUTION.md
 - ATTRIBUTION.md
   - all sources with copyright notices and citation codes
   - required for new sources: copyright, citation code, license type, purchase link
   - also has websites with useful content, preference given to sites allowing for url templating (ex. site.com/gen/1 or site.com?book=gen&chapter=1)
 - bible-study-tools/TEMPLATE.md
   - template for creating new Bible study tools
   - 3-phase: data extraction first, then analysis, then validation
   - output schema with required fields (verse, inline citations, metadata)
   - define tool-specific Level 2 validation requirements
 - **Working with TBTA features**: Read `/plan/tbta-rebuild-with-llm/README.md`

## Repository Structure

### Data Organization

All generated commentary data follows this strict directory structure: (See STANDARIZATION.md for examples, edge cases if unsure)

```
$DATA_DIR/commentary/{BOOK}/{chapter:03d}/{verse:03d}/{BOOK}-{chapter:03d}-{verse:03d}-{tool}.yaml
$DATA_DIR/strongs/(H|G){strongs-number:04d}/(H|G){strongs-number:04d}-{tool}.strongs.yaml
$DATA_DIR/topics/{lcc-code}/{slug}/{slug}[-{subsection}]-{tool}.yaml
$DATA_DIR/languages/{ISO-639-3}/{ISO-639-3}-{tool}.yaml
$DATA_DIR/languages/{ISO-639-3}/words/{word}/{ISO-639-3}-{word}-{tool}.yaml

```

Default $DATA_DIR should be .data
If unset/not exists run `setup-minimal-data.sh`



## Development Notes

- The `.claude/` directory contains Claude Code configuration and slash commands
- Book codes follow standard 3-letter abbreviations (MAT, JHN, GEN, etc.) (USFM 3.0)
- Language codes follow (ISO-639-3)
- This is an open-source project under MIT License
- When creating new names, taxonomies, etc prioritize following known standards to remove the need for lookups

## Working Preferences

### File Organization
- **NO summary files in root directory** - Do not create CHANGES-SUMMARY.md, COMPLETION-SUMMARY.md, PR-DESCRIPTION.md, or similar files in the project root
- **Use `/plan` directory** - For planning and tracking work, create files in `/plan/{task-name}.md` and update them as you learn and progress
- **Keep root clean** - Root should only contain permanent project documentation
- **Organize your task directory** 
  - When there will be more than one file or version put in subdirectories (ex. ./experiments/v1; ./data; ./research)
  - Clean up all your thinking, analysis, temporary and other files.  Be concise, I brag about how few lines something took not how massive a project grew.

### Documentation Philosophy
- **Prefer concise over comprehensive** - Simple instructions let AI figure things out; verbose explanations create confusion
- **Don't over-explain mechanics** - Avoid detailed explanations of "how to use tokens" or basic processes unless solving a specific issue
- **Inline key info, reference details** - Summarize essential standards in tool READMEs; link to full docs for edge cases with "see {doc} for details"
- **Avoid redundancy** - Don't create "-quick" versions of docs; extract relevant parts directly into tool READMEs
- **Use progressive disclosure for ALL .md files** - When creating/editing ANY markdown file, use `/progressive-disclosure` skill: README ≤200 lines (self-contained overview), topic files ≤400 lines, plan ahead to create directories if content will exceed limits, append to existing files before creating new ones

### Fixing Bugs and Problems

 - When something does not work (ex. can't import a file or call Quote Verse), fix it.  Never just add a placeholder and pretend your results worked.
 - Debug what went wrong in your instructions; update that in a generic and very concise way so other sessions will not have the same problem.  Commit that change independently with git flow of "FIX: AI SYSTEM: {short summary}\n{diagnosis and solution}

### Tool Development Process
- **Experiments optimize, researchers execute** - Tool experimentation phase should test sources and optimize lookups; researchers should use the optimized approach directly
- **Document sources, not tools** - In tool READMEs, list helpful webpages, not the obvious fact that WebSearch/WebFetch exist
- **Tailored standards** - Tools dealing with words need word standards; tools without words don't need them. Include only relevant standards.


### Working with Sparse Checkout

The data directory uses Git sparse-checkout to limit which files are downloaded. This is important to know:

- **Adding files**: If you try to create/add files in directories not in the sparse-checkout scope, Git will filter them out on commit
- **Solutions**:
  - Add the directory to sparse-checkout: `cd $DATA_DIR && git sparse-checkout add commentary/ROM`
  - Or disable sparse-checkout temporarily: `cd $DATA_DIR && git sparse-checkout disable`
- **Check current scope**: `cd $DATA_DIR && git sparse-checkout list`
- **Re-enable**: `cd $DATA_DIR && git sparse-checkout init --cone` then set patterns again

---
> Source: [authenticwalk/mybibletoolbox-code](https://github.com/authenticwalk/mybibletoolbox-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
