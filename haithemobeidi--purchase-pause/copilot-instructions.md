## purchase-pause

> Purchase Pause is an evidence-based Chrome extension and Android app to help prevent impulse buying using CBT techniques and delay gratification strategies.

# Purchase Pause - Claude Code Project Instructions

## Project Overview
Purchase Pause is an evidence-based Chrome extension and Android app to help prevent impulse buying using CBT techniques and delay gratification strategies.

**Core Principles:**
- Evidence-based psychological techniques
- Privacy-focused (all data stored locally)
- Non-judgmental, supportive approach
- Financial wellness tool, not replacement for professional help

---

## 🚨 MANDATORY: End Session Protocol

When the user says **"end session"** or equivalent, you MUST follow this protocol:

### Step 1: Create Handoff Document

1. **Create handoff folder if it doesn't exist:**
   ```bash
   mkdir -p handoff
   ```

2. **Generate handoff filename:**
   - Format: `handoff/YYYY-MM-DD_HH-MM-SS_EST.md`
   - Use Eastern Standard Time
   - Example: `handoff/2025-10-02_14-30-45_EST.md`

3. **Handoff document template:**
   ```markdown
   # Session Handoff - [MM/DD/YYYY HH:MM:SS EST]

   ## Session Summary
   [Brief overview of session goals and outcomes]

   ## Accomplishments
   - [Detailed list of what was completed]
   - [Files modified, features added, bugs fixed]

   ## Files Modified This Session
   ### Created
   - `path/to/file.ext` - Brief description of purpose

   ### Modified
   - `path/to/file.ext` - What changed and why

   ### Deleted
   - `path/to/file.ext` - Why it was removed

   ## Current Project Status
   - [Overall project state]
   - [What's working]
   - [Known issues]

   ## Next Session Recommendations
   - [Priority tasks for next session]
   - [Specific areas needing attention]
   - [Any blocking issues to resolve]

   ## Technical Notes
   - [Build status, dependencies, etc.]
   - [Any configuration changes made]
   - [Testing status]

   ## Research & Decisions Made
   - [Any research conducted]
   - [Design decisions and rationale]
   - [Evidence-based choices]
   ```

### Step 2: Update Master Handoff Index

1. **File location:** `handoff/MASTER_HANDOFF_INDEX.md`

2. **Update with new entry at the TOP:**
   ```markdown
   ## [YYYY-MM-DD HH:MM:SS EST] - [Brief Session Title]
   **Handoff File:** `YYYY-MM-DD_HH-MM-SS_EST.md`

   **Summary:** [2-3 sentence summary of session]

   **Key Changes:**
   - [Major change 1]
   - [Major change 2]

   **Status:** [Completed/In Progress/Blocked]

   ---
   ```

### Step 3: Update Master File Index

1. **File location:** `FILE_INDEX.md`

2. **Update entries for any modified files:**
   - Add new files created
   - Update descriptions for modified files
   - Mark deleted files as deprecated

3. **Format:**
   ```markdown
   ### `/path/to/file.ext`
   **Purpose:** [What this file does]
   **Key Functions/Components:** [Main exports, functions, classes]
   **Dependencies:** [What it imports/requires]
   **Last Modified:** [Session date]
   **Status:** [Active/Deprecated/In Progress]
   ```

### Step 4: Git Commit

1. **Stage all changes:**
   ```bash
   git add .
   ```

2. **Commit with descriptive message:**
   ```bash
   git commit -m "Session [DATE]: [Brief summary]

   - [Change 1]
   - [Change 2]
   - [Change 3]

   Handoff: handoff/YYYY-MM-DD_HH-MM-SS_EST.md"
   ```

3. **Push to GitHub:**
   ```bash
   git push origin main
   ```

### Step 5: Confirm to User

Display summary to user:
```
✅ Session ended successfully

📝 Handoff created: handoff/YYYY-MM-DD_HH-MM-SS_EST.md
📋 Master index updated
📚 File index updated
✅ Changes committed and pushed to GitHub

Session Summary:
- [X] files created
- [Y] files modified
- [Z] commits made

Next session can start by reading the handoff file.
```

---

## 📂 Project File Structure

```
purchase-pause/
├── CLAUDE.md                     # This file - project instructions
├── README.md                     # Project overview
├── RESEARCH_FINDINGS.md          # All research and evidence base
├── CRITICAL_REVIEW.md            # Critical analysis and issues
├── IMPLEMENTATION_PLAN.md        # Phase-by-phase development plan
├── FILE_INDEX.md                 # Master file index (auto-updated)
├── handoff/
│   ├── MASTER_HANDOFF_INDEX.md  # Summary of all sessions
│   └── YYYY-MM-DD_HH-MM-SS_EST.md  # Individual handoff files
├── mockup-interactive.html       # Original mockup (deprecated)
├── mockup-improved.html          # Current mockup with button UX
└── [Future: src/, public/, etc.]
```

---

## 🔄 Session Start Protocol

At the start of each session:

1. **Read the latest handoff:**
   ```bash
   # Find most recent handoff
   ls -t handoff/*.md | head -1
   ```

2. **Review MASTER_HANDOFF_INDEX.md** for context

3. **Check FILE_INDEX.md** to understand current codebase

4. **Review any "Next Session Recommendations"**

---

## 📋 Development Guidelines

### Evidence-Based Approach
- All features must have research backing (see RESEARCH_FINDINGS.md)
- Cite sources for psychological techniques
- Follow user retention best practices (see CRITICAL_REVIEW.md)

### Code Standards
- TypeScript for type safety
- React functional components with hooks
- Zustand for state management
- Comment complex logic thoroughly
- Follow mobile-first design principles

### UX Principles (CRITICAL)
- **NO text input in interventions** (use buttons/sliders only)
- 60px minimum button height for mobile
- Progress indicators on multi-step flows
- Target completion time: <30 seconds
- Non-judgmental language throughout

### Testing Requirements
- Test on 20+ major e-commerce sites
- Verify Schema.org detection accuracy (>90%)
- Mobile responsive testing required
- User testing before major releases

### Privacy & Security
- All data stored locally (chrome.storage.local)
- No external API calls for user data
- Clear privacy policy
- GDPR compliance considerations

---

## 🎯 Current Project Phase

**Status:** Pre-Development / Planning Complete

**Completed:**
- ✅ Research and evidence gathering
- ✅ Critical review and optimization
- ✅ Detailed implementation plan
- ✅ Improved mockup (button-based UX)

**Next Phase:** Phase 1 - MVP Foundation
- Set up project structure
- Build detection engine
- Create side panel UI (passive tracking only)

**Timeline:** 6-8 weeks for Phase 1

---

## 🔧 Technical Stack

**Chrome Extension:**
- React 18 + TypeScript
- Zustand (state management)
- Vite (build tool)
- Lucide React (icons)
- Manifest V3

**Future (Android):**
- React Native + Expo

---

## 📊 Success Metrics to Track

### User Retention
- Week 1: >40%
- Week 4: >20%
- Month 3: >15%

### Intervention Completion
- Target: 40-50%
- Time: <30 seconds

### Savings
- Average: $150-250/month
- Target reduction: 50-70% of impulse spending

---

## 🚫 What NOT to Do

❌ **Never** create text input fields in interventions
❌ **Never** intervene on every purchase (use smart triggers)
❌ **Never** use judgmental language ("bad decision", "stop buying")
❌ **Never** commit without user approval
❌ **Never** skip the end session protocol

---

## 📚 Key Documents Reference

### For Research Questions:
- Read `RESEARCH_FINDINGS.md` - All psychological research and data

### For Design Decisions:
- Read `CRITICAL_REVIEW.md` - Known issues and solutions

### For Implementation:
- Read `IMPLEMENTATION_PLAN.md` - Step-by-step build guide

### For Current Status:
- Read `handoff/MASTER_HANDOFF_INDEX.md` - Recent session history
- Read latest handoff file - Detailed current state

---

## 🔐 Git & Version Control

### Versioning Strategy
- **Semantic versioning:** MAJOR.MINOR.PATCH
- **Current version:** 0.1.0 (pre-release)
- **Version increments:**
  - MAJOR: Breaking changes or major releases
  - MINOR: New features, phases completed
  - PATCH: Bug fixes, minor updates

### Commit Message Format
```
Type: Brief description (max 50 chars)

- Detailed change 1
- Detailed change 2
- Detailed change 3

[Optional: References to issues, handoffs]
```

**Types:**
- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation changes
- `refactor:` Code refactoring
- `test:` Adding tests
- `chore:` Maintenance tasks

### Branch Strategy
- `main` - Stable, deployable code
- `develop` - Active development (future)
- `feature/[name]` - Feature branches (future)

---

## 💡 Session Best Practices

1. **Always start by reading the latest handoff**
2. **Use TodoWrite tool** for task tracking during session
3. **Update FILE_INDEX.md** as you modify files
4. **Ask for user confirmation** before major changes
5. **Follow evidence-based decision making** (see CLAUDE_AGENT_COORDINATION.md in user's global config)
6. **End every session with the protocol** (non-negotiable)

---

## 🎓 Learning from Crisis Helper

This project mirrors the successful Crisis Helper architecture:
- React + TypeScript + Zustand
- Chrome Manifest V3
- Side panel API
- Evidence-based interventions
- Privacy-first design

**Key Differences:**
- Financial focus (not mental health crisis)
- Smart intervention triggers (not constant monitoring)
- Tiered opt-in system (passive → active)
- Cumulative spending tracking

---

## Last Updated
**Date:** 2025-10-02
**Session:** Initial project setup
**Version:** 0.1.0

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/haithemobeidi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/haithemobeidi)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
