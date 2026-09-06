## 03-quality-assurance

> **Last Updated**: January 1, 2026

# Quality Assurance Checklist

**Version**: 1.1  
**Last Updated**: January 1, 2026

---

## ✅ Quality Assurance Checklist

**Before ANY content creation or modification**:

### Content Quality (Educational Standards)

- [ ] **Zero-Copy Policy Verified**: No verbatim text from sources; completely transformed
- [ ] **Transformative Workflow Applied**: Fresh outline, new examples, original explanations
- [ ] **25-Minute Segment Compliance**: Content within 150 lines, modular structure
- [ ] **5 Required Metadata Fields**: learning_level, prerequisites, estimated_time, learning_objectives, related_topics
- [ ] **Original Examples**: All code, diagrams, datasets, and exercises are original
- [ ] **ASCII-First Diagrams**: Diagrams redrawn in ASCII, never embedded copyrighted figures
- [ ] **Learning Objectives**: Clear, specific, and measurable outcomes
- [ ] **Progressive Scaffolding**: Foundations → Practice → Pitfalls → Next Steps
- [ ] **Proper Learning Level**: Assigned (Stage 1-9 or Beginner/Intermediate/Advanced)
- [ ] **Cross-References**: Prerequisites, builds-upon, enables, cross-refs included
- [ ] **Quality Gate Passed**: All 6 quality gate questions answered affirmatively

### Technical Quality

- [ ] Character encoding verified (UTF-8, no symbols)
- [ ] **YAML frontmatter validated**: No placeholder patterns (`$101_`, `$102_`, `$103_`), `enables:` key present, proper structure
- [ ] **File references validated**: All file references point to existing files (run `.\tools\psscripts\Validate-FileReferences.ps1`)
- [ ] **No broken references**: All YAML and markdown links resolve correctly
- [ ] **File naming validated**: **CRITICAL** - Check for `00_` prefix - **NEVER ALLOWED** (applies to ALL files including `docs/`)
  - Run: `Get-ChildItem -Recurse -Filter "*00_*"` to find violations
  - All numbered files must use `01_`, `02_`, etc. - **NO EXCEPTIONS**
- [ ] Markdownlint passes: `npx markdownlint-cli2 "file.md"`
- [ ] Lychee link check passes: `docker run --rm -v "${PWD}:/workspace" lycheeverse/lychee "file.md"`
- [ ] **Numbering consistent**: Uses `01_`, `02_`, etc. - **NEVER `00_`** (applies to ALL files including `docs/`)
- [ ] Code fences have language specified
- [ ] Blank lines proper (MD022/031/032)
- [ ] 2-space list indentation (MD007)

### Documentation Updates (CRITICAL)

- [ ] ✅ `copilot-instructions.md` updated if structure changed
- [ ] ✅ `README.md` updated if structure changed
- [ ] ✅ Related documentation updated
- [ ] ✅ Verification script run: `.\Quick-HealthCheck.ps1`

### File Reference Validation (CRITICAL)

- [ ] ✅ **All file references validated**: Run `.\tools\psscripts\Validate-FileReferences.ps1`
- [ ] ✅ **No broken references**: All YAML `prerequisites`, `builds_upon`, `enables` references exist
- [ ] ✅ **All markdown links work**: Test navigation links in preview
- [ ] ✅ **Part file references correct**: If files were split, references use exact part file names
- [ ] ✅ **Updated after splitting**: If you split a file, updated ALL references to that file

---
> Source: [SwamysArchitectJourney-2026/architecture-reasoning-in-practice](https://github.com/SwamysArchitectJourney-2026/architecture-reasoning-in-practice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
