## inneros-zettelkasten

> > **Purpose**: Modular AI assistant behavior guidelines


# Windsurf Rules - InnerOS Zettelkasten

> **Purpose**: Modular AI assistant behavior guidelines  
> **Updated**: 2025-10-23 (Added consolidated guides reference)  
> **Structure**: 10 focused rule files, each under 12KB limit

---

## 📋 Core Rule Files

### Development & Architecture
- **`updated-development-workflow.md`** - TDD methodology, architectural safeguards, Git standards
- **`architectural-constraints.md`** - God class prevention, size limits, refactoring triggers
- **`automation-monitoring-requirements.md`** ⭐ NEW - Phase 3 & 4 mandatory requirements
  - Event-driven and scheduled automation
  - Monitoring, metrics, health checks, alerting
  - Daemon integration patterns

### Project Organization & Content
- **`updated-file-organization.md`** - Directory structure, metadata schema, templates
- **`content-standards.md`** - Note quality standards, literature notes, permanent notes

### AI Integration & Context
- **`updated-ai-integration.md`** - AI feature guidelines, workflow patterns
- **`updated-session-context.md`** - Project context, current priorities
- **`updated-current-issues.md`** - Active bugs, system integrity issues

### Security & Ethics
- **`privacy-security.md`** - Data preservation, ethics, user decision-making

---

## 🎯 How to Use

**AI Assistant (Cascade)**:
- Automatically loads ALL files in this directory
- Follows guidelines across all files
- Modular structure prevents 12KB limit issues

**Developers**:
- Update individual files as needed
- Keep each file focused and under 12KB
- Archive deprecated files to `.windsurf/archive/`

---

## 📊 File Size Guidelines

- **Target**: <8KB per file (comfortable margin)
- **Warning**: 8-10KB (consider splitting)
- **Limit**: 12KB (hard limit, must split)

**Current Sizes** (2025-10-23):
- updated-development-workflow.md: 11,985 bytes (11.7KB) ⚠️ (near limit, stable)
- updated-session-context.md: 11,223 bytes (11.0KB) ⚠️ (near limit, includes guide refs)
- updated-ai-integration.md: 8,066 bytes (7.9KB) ✅
- automation-monitoring-requirements.md: 11,322 bytes (11.1KB) ✅
- updated-current-issues.md: 6,938 bytes (6.8KB) ✅
- updated-file-organization.md: 6,768 bytes (6.6KB) ✅
- architectural-constraints.md: 5,226 bytes (5.1KB) ✅
- README.md: 2,956 bytes (2.9KB) ✅
- content-standards.md: 1,659 bytes (1.6KB) ✅
- privacy-security.md: 1,449 bytes (1.4KB) ✅

**Total**: 63,336 bytes (61.9KB) - well within limits

---

## 📚 Consolidated Development Guides

**Location**: `.windsurf/guides/` (Created 2025-10-23)

For universal development patterns extracted from 34+ lessons-learned iterations:

- **`tdd-methodology-patterns.md`** (12.4KB) - TDD wisdom from 34+ iterations
  - RED → GREEN → REFACTOR patterns
  - Test coverage strategies (10-25 tests)
  - Minimal implementation, utility extraction
  - Time management, success metrics

- **`ai-integration-patterns.md`** (16.9KB) - AI integration from 15+ iterations
  - Mock-first development, graceful degradation
  - File hash caching (85-95% hit rate)
  - Quality gates (82% bad output reduction)
  - Batch processing, error handling

- **`SESSION-STARTUP-GUIDE.md`** (7.7KB) - Quick reference
  - Pattern lookup by task type
  - Common scenarios & anti-patterns
  - Fast navigation to relevant wisdom

- **`README.md`** (7.9KB) - Guide index & maintenance

**Benefits**: 85% content reduction, 50-70% faster context loading vs reading 34 documents

**Usage**: See guides at session start for proven patterns instead of rediscovering

---

## 🔄 Maintenance

**Monthly Review** (First Monday):
1. Check file sizes (`wc -c .windsurf/rules/*.md`)
2. Identify files approaching 10KB
3. Split or refactor if needed
4. Archive deprecated content

**When Adding New Rules**:
1. Check total character count
2. Keep focused (single responsibility)
3. Update this README
4. Link related files

---

## 🗂️ Archived/Deprecated

Files moved to `.windsurf/archive/rules-backup-YYYY-MM-DD/`:
- `file-organization.md` (2025-10-07) - Superseded by `updated-file-organization.md`
- `updated-windsurfrules-v4-concise.md` (2025-10-07) - Monolithic approach superseded by modular structure

---

**Last Updated**: 2025-10-23  
**Active Rules Files**: 10  
**Rules Total**: 63KB (well within limits)  
**Guides Total**: 46KB (reference documentation)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thaddiusatme) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
